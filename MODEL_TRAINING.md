# Model Training Prompt

## Role

You are a senior Data Scientist building and selecting the best binary classifier for the ad-click prediction task. Your input is the feature-engineered Delta splits produced by `feature_engineering.ipynb`. Your output is a fully executed Databricks notebook (`model_training.ipynb`) that trains multiple candidate models, tunes the best one, evaluates all on a held-out test set, and registers the champion to the MLflow Model Registry.

---

## Environment

**Platform:** Databricks Serverless (cloud extension)  
**Compute:** Databricks Serverless — no cluster management; no `sparkContext` access  
**Storage:** Delta Lake — read splits from `fe_output/{train,val,test}`  
**ML Framework:** PySpark MLlib (primary); scikit-learn only for metric plotting utilities  
**Experiment Tracking:** MLflow — `mlflow.spark.autolog()` + manual metric logging  
**Model Registry:** MLflow Model Registry — register champion as `ad_click_classifier`

---

## Inputs from Feature Engineering

| Artifact | Path | Description |
|---|---|---|
| Train split | `fe_output/train` | Delta table, `features_scaled` + `gender_ohe` + `tfidf_selected` + `label` |
| Val split | `fe_output/val` | Same schema |
| Test split | `fe_output/test` | Same schema — **only touched in final evaluation** |
| Scale pipeline | `fe_output/artifacts/scale_pipeline` | Fitted `MinMaxScaler` for inference |
| TF-IDF pipeline | `fe_output/artifacts/tfidf_pipeline` | Fitted TF-IDF + ChiSqSelector |
| Gender pipeline | `fe_output/artifacts/gender_pipeline` | Fitted `StringIndexer` + `OneHotEncoder` |

---

## Deliverables

1. **`model_training.ipynb`** — fully executed Databricks notebook
2. **`utils/model_helpers.py`** — evaluation, plotting, and MLflow helpers
3. **`fe_output/artifacts/champion_model/`** — saved PipelineModel of the best model

---

## Candidate Models

| Model | MLlib Class | Role |
|---|---|---|
| Logistic Regression | `LogisticRegression` | Interpretable baseline |
| Random Forest | `RandomForestClassifier` | Strong ensemble baseline |
| Gradient Boosted Trees | `GBTClassifier` | Expected champion for tabular data |

---

## Notebook Structure

### 0. Setup
- `%pip install` in its own cell (seaborn, scikit-learn)
- Import PySpark MLlib, MLflow, pandas, sklearn metrics
- Define constants: `FE_OUTPUT_PATH`, `MODEL_NAME`, `RANDOM_SEED`, `METRIC_FOR_SELECTION`
- Start MLflow experiment: `mlflow.set_experiment("ad_click_model_training")`

### 1. Load Feature-Engineered Splits
- Load `train`, `val`, `test` from Delta using `spark.read.format("delta").load(...)`
- Assert expected columns present: `features_scaled`, `gender_ohe`, `tfidf_selected`, `label`
- Print row counts and class balance per split
- **Do not touch `test` split until Section 8**

### 2. Final Feature Assembly
- Use `VectorAssembler` to combine `features_scaled`, `gender_ohe`, `tfidf_selected` → `features`
- This is the single feature vector passed to all models
- Print total feature dimensionality
- Cache `df_train_ready` and `df_val_ready` — reused across all model fits

### 3. Train Logistic Regression (Baseline)
- `LogisticRegression(featuresCol="features", labelCol="label", maxIter=100)`
- Wrap in a `Pipeline` with the final assembler
- Fit on `df_train_ready`; evaluate on `df_val_ready`
- Log: `val_auc_roc`, `val_auc_pr`, `val_f1` to MLflow

### 4. Train Random Forest
- `RandomForestClassifier(numTrees=100, maxDepth=8, seed=RANDOM_SEED)`
- Fit and evaluate same way; log metrics
- Extract and display top-20 feature importances

### 5. Train Gradient Boosted Trees
- `GBTClassifier(maxIter=50, maxDepth=6, stepSize=0.1, seed=RANDOM_SEED)`
- Fit and evaluate; log metrics
- Extract and display feature importances

### 6. Hyperparameter Tuning — Best Candidate
- Identify best model from Sections 3–5 by `val_auc_roc`
- Run `TrainValidationSplit` (faster than `CrossValidator` on Serverless) on the winner:
  - `trainRatio=0.8`, `evaluator=BinaryClassificationEvaluator(metricName="areaUnderROC")`
  - Grid: 2–3 key hyperparameters, 3–4 values each (≤ 12 total fits)
- Log best params and tuned val AUC to MLflow

### 7. Model Comparison on Validation Set
- Collect all models' val metrics into a comparison table (pandas DataFrame)
- Bar chart: AUC-ROC, AUC-PR, F1 side-by-side for all candidates
- ROC curves overlaid on a single plot (use sklearn `roc_curve` on `.toPandas()` predictions)
- Precision-Recall curves overlaid
- Identify and print the champion model

### 8. Final Evaluation on Test Set
- Apply **only the champion model** to `df_test_ready` (first and only time)
- Report: AUC-ROC, AUC-PR, F1, Precision, Recall, Accuracy
- Confusion matrix heatmap
- Calibration plot: predicted probability vs. actual click rate (10 bins)
- Log all test metrics to MLflow with `test_` prefix

### 9. Register Champion to MLflow Model Registry
- `mlflow.spark.log_model(champion_model, "champion_model", registered_model_name=MODEL_NAME)`
- Transition the new version to `"Production"` stage
- Log feature schema, training data path, and model type as MLflow tags
- Print the registered model URI

### 10. MLflow Summary
- Log final comparison table as a CSV artifact
- Log all charts as PNG artifacts
- Set tags: `champion_model_type`, `feature_version`, `test_auc_roc`

---

## Code Standards

- Every section has a Markdown cell explaining what and why
- All model hyperparameters defined as constants at the top — no magic numbers inside cells
- `%pip install` in its own first cell
- No `spark.sparkContext` — Serverless incompatible
- `display()` for interactive DataFrames; matplotlib/seaborn for saved charts
- Cache DataFrames reused > twice; unpersist after use
- Use `F` alias for `pyspark.sql.functions`
- Functions > 15 lines go in `utils/model_helpers.py`

---

## Constraints & Guardrails

- **Do not** use the test set before Section 8 — report test metrics once only
- **Do not** tune hyperparameters based on test set performance
- **Do not** use `CrossValidator` with > 5 folds on Serverless — prefer `TrainValidationSplit`
- **Do not** hardcode file paths — use constants
- The registered model must include the full `Pipeline` (assembler + classifier), not just the classifier

---

## Acceptance Criteria

| Criterion | Definition of Done |
|---|---|
| Three models trained | LR, RF, GBT all fitted and evaluated on val set |
| Hyperparameter tuning | `TrainValidationSplit` run on the best candidate |
| Comparison charts | ROC, PR, and metric bar charts produced |
| Test evaluation | Champion evaluated on test set exactly once |
| Model registered | Champion `Pipeline` in MLflow Model Registry as `"Production"` |
| MLflow run | All params, metrics (val + test), and ≥ 3 charts logged |
| Reproducibility | Notebook runs clean top-to-bottom on Databricks Serverless |
