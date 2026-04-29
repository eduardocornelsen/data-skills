# Inferencing Prompt

## Role

You are a senior Data Scientist operationalising the champion ad-click classifier. Your input is raw ad-impression data in the same schema as the original training CSV. Your output is a fully executed Databricks notebook (`inferencing.ipynb`) that loads the registered production model, applies the full feature engineering pipeline, scores a batch of impressions, writes predictions to Delta, and performs lightweight data-drift monitoring.

---

## Environment

**Platform:** Databricks Serverless (cloud extension)  
**Compute:** Databricks Serverless — no cluster management; no `sparkContext` access  
**Storage:** Delta Lake — read new data, write predictions  
**Model source:** MLflow Model Registry — champion registered as `ad_click_classifier/Production`  
**Artifact source:** `fe_output/artifacts/` — fitted pipelines and encoding maps  
**Output table:** `inference_output/predictions` (Delta)

---

## Inference Flow

```
Raw impressions (CSV / Delta)
        │
        ▼
 [1] Schema validation & null guard
        │
        ▼
 [2] Temporal features (same as FE notebook)
        │
        ▼
 [3] Numeric features (same transforms + outlier flags)
        │
        ▼
 [4] Categorical encoding (load saved gender/country/city pipelines + maps)
        │
        ▼
 [5] Text features (load saved TF-IDF pipeline)
        │
        ▼
 [6] Final VectorAssembler (load saved scale pipeline)
        │
        ▼
 [7] Load champion model → score → attach probability + binary prediction
        │
        ▼
 [8] Write predictions to Delta + drift report
```

---

## Deliverables

1. **`inferencing.ipynb`** — fully executed Databricks notebook
2. **`inference_output/predictions`** — Delta table with scored impressions
3. **`inference_output/drift_report`** — Delta table with per-feature drift stats

---

## Notebook Structure

### 0. Setup
- `%pip install` in its own cell
- Imports: PySpark, MLflow, pandas, scipy (KS test for drift)
- Constants: `INFERENCE_INPUT_PATH`, `FE_ARTIFACTS_PATH`, `MODEL_NAME`, `MODEL_STAGE`, `OUTPUT_PATH`, `PREDICTION_THRESHOLD`, `DRIFT_KS_ALPHA`

### 1. Load & Validate Inference Data
- Load raw impressions from `INFERENCE_INPUT_PATH` via `spark.read`
- Assert the same `REQUIRED_COLS` as training (reuse `assert_columns` from `utils/fe_helpers`)
- Check for unexpected nulls — log a warning (do **not** silently drop rows)
- Report row count and a sample

### 2. Apply Feature Engineering Pipeline
Apply **every transformation in the exact same order** as `feature_engineering.ipynb`:
- **Temporal:** cast `Timestamp`, extract `hour`, `day_of_week`, `month`, `is_weekend`, cyclical encoding, drop raw columns
- **Numeric:** interaction term, `log1p(Area Income)`, age bins, income quartile; load `outlier_bounds.json` and apply outlier flags (use saved bounds — do **not** recompute)
- **Categorical — Gender:** load `fe_output/artifacts/gender_pipeline` with `PipelineModel.load()`; transform; drop raw column
- **Categorical — Country/City:** load `country_te_map.json` and `city_te_map.json`; apply `apply_target_encoding`; add `log_city_imp_count` using **training-time** city counts from a saved reference (stored as `city_counts.json` — add saving step to FE notebook if missing); drop raw columns
- **Text:** load `fe_output/artifacts/tfidf_pipeline` with `PipelineModel.load()`; apply keyword flags first, then TF-IDF; drop raw text column

### 3. Final Feature Assembly
- Load `fe_output/artifacts/scale_pipeline` with `PipelineModel.load()`
- Transform data through the fitted `VectorAssembler` + `MinMaxScaler` → `features_scaled`
- Assemble final feature vector: `VectorAssembler([features_scaled, gender_ohe, tfidf_selected]) → features`
- Note: for unseen categories (OHE `handleInvalid="keep"`) and unseen country/city values (target-encoding fallback), the pipeline handles gracefully

### 4. Load & Apply Champion Model
- Load model: `mlflow.spark.load_model(f"models:/{MODEL_NAME}/{MODEL_STAGE}")`
- Score: `predictions = model.transform(df_features)`
  - Output columns: `prediction` (0.0 / 1.0), `probability` (vector), `rawPrediction`
- Extract click probability: `prob_col = F.udf(lambda v: float(v[1]))(F.col("probability"))`
- Apply custom threshold: `predicted_click = (prob_col >= PREDICTION_THRESHOLD).cast("int")`
- Attach `inference_timestamp` column: `F.current_timestamp()`
- Display: prediction distribution, mean predicted probability, top-probability impressions

### 5. Post-Process & Enrich Output
- Select output columns:
  - All original raw input columns (for business context)
  - `click_probability`, `predicted_click`, `inference_timestamp`
- Add `model_version` tag (MLflow run ID or registry version)
- Compute and display: predicted CTR, confidence distribution histogram

### 6. Write Predictions to Delta
- `predictions_out.write.format("delta").mode("append").save(f"{OUTPUT_PATH}/predictions")`
- Use `.mode("append")` — predictions accumulate over time
- Print row count written and sample output rows

### 7. Data Drift Monitoring
Lightweight statistical drift check comparing inference data distributions against training reference stats:
- Load training reference stats from `fe_output/artifacts/train_stats.json` (mean, std, min, max per numeric feature — add saving step to FE notebook if missing)
- For each numeric feature: run **Kolmogorov-Smirnov test** (scipy `ks_2samp`) between training sample and inference batch
- Report: KS statistic, p-value, drift flag (`p < DRIFT_KS_ALPHA`)
- Produce a drift summary table and bar chart of KS statistics
- Write drift report to `inference_output/drift_report` as Delta

### 8. MLflow Logging
- Log to a new run under experiment `ad_click_inferencing`:
  - **Params:** `model_name`, `model_stage`, `n_input_rows`, `prediction_threshold`, `drift_alpha`
  - **Metrics:** `n_predictions`, `predicted_ctr`, `n_drift_alerts`, mean and std of `click_probability`
  - **Artifacts:** drift report CSV, prediction distribution chart

---

## Code Standards

- Every section has a Markdown cell explaining what and why
- No `spark.sparkContext` — Serverless incompatible
- Encoders loaded from disk — never refit on inference data
- `display()` for interactive previews; matplotlib for saved charts
- Constants defined at top; no magic numbers
- `%pip install` in its own first cell

---

## Constraints & Guardrails

- **Never** refit or retrain any encoder on inference data — load from `fe_output/artifacts/` only
- **Never** drop rows silently — flag or log unexpected nulls and unseen categories
- **Never** use `sparkContext` — Serverless incompatible
- **Threshold** for `predicted_click` must be a constant (`PREDICTION_THRESHOLD`), not hardcoded `0.5`
- Write mode for predictions must be **`append`** — do not overwrite historical predictions

---

## Acceptance Criteria

| Criterion | Definition of Done |
|---|---|
| Pipeline integrity | All FE steps applied in exact training order using saved artifacts |
| Null safety | Null guard runs before any transformation |
| Predictions written | Delta table at `OUTPUT_PATH/predictions` with probability + binary prediction |
| Drift report | KS test run on all numeric features; report written to Delta |
| No leakage | Zero encoder refitting on inference data |
| MLflow logged | Run with params, metrics, and ≥ 2 artifacts logged |
| Reproducibility | Notebook runs clean top-to-bottom on Databricks Serverless |
