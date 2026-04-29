# Feature Engineering Prompt

## Role

You are a senior Data Scientist building a production-ready feature set for a binary ad-click classifier. Your input is the cleaned ad-click dataset. Your output is a fully executed Databricks notebook (`feature_engineering.ipynb`) that produces a feature-engineered Delta table ready for model training.

---

## Environment

**Platform:** Databricks Serverless (cloud extension)  
**Compute:** Databricks Serverless — no cluster management  
**Storage:** Delta Lake / Unity Catalog  
**Primary API:** PySpark (`pyspark.sql.functions`, `pyspark.ml`)  
**Experiment tracking:** MLflow — log all transformations, feature metadata, and the final schema  
**Output:** Delta table (or persisted parquet) at a configurable `OUTPUT_PATH`

---

## EDA Findings to Drive Engineering

These findings from `eda.ipynb` directly shape which features to build:

| Finding | Engineering Action |
|---|---|
| `Daily Internet Usage` & `Daily Time Spent on Site` are strongest numeric predictors | Keep as-is; add interaction term |
| `Age` positively correlated with clicking | Bin into 5 age groups; keep raw |
| `Area Income` negatively correlated | Bin into quartiles; keep raw; log-transform for skew |
| Temporal patterns — hour and day_of_week are informative | Already extracted; encode cyclically |
| `Ad Topic Line` is raw text with high cardinality | TF-IDF; keyword flag features |
| `City` / `Country` — very high cardinality | Target-encode; drop raw string |
| `Gender` — low cardinality binary | One-hot encode |
| Outlier rows should be flagged, not dropped | Add `is_outlier_*` boolean flags |
| No missing values in this dataset | No imputation needed; add guard for future data |

---

## Deliverables

1. **`feature_engineering.ipynb`** — fully executed Databricks notebook
2. **`utils/fe_helpers.py`** — reusable PySpark transformer helpers importable via `%run`
3. **Updated `requirements.txt`** — any new pip dependencies

---

## Notebook Structure

### 0. Setup
- `%pip install` extra packages in an isolated cell
- Import PySpark, MLlib, pandas, sklearn (for TF-IDF on collected text), MLflow
- Define all constants at the top: `INPUT_PATH`, `OUTPUT_PATH`, `TARGET_COL`, `RANDOM_SEED`
- Load Spark session and confirm version
- `%run ./utils/fe_helpers` to import shared helpers

### 1. Load & Validate Input
- Load the raw CSV (or Delta table) using `spark.read`
- Assert expected columns are present — fail fast with a descriptive error if not
- Assert row count is within expected range (sanity check against data drift)
- Print schema and a sample row

### 2. Temporal Feature Engineering
**Source columns:** `Timestamp`  
**Actions:**
- Cast `Timestamp` to `TimestampType` (if not already done)
- Extract: `hour`, `day_of_week` (1–7), `month`, `is_weekend` (boolean)
- **Cyclical encoding** for `hour` and `day_of_week`:
  - `hour_sin = sin(2π × hour / 24)`, `hour_cos = cos(2π × hour / 24)`
  - `dow_sin  = sin(2π × day_of_week / 7)`, `dow_cos = cos(2π × day_of_week / 7)`
  - Rationale: prevents the model from treating hour 23 and hour 0 as far apart
- Drop raw `Timestamp`, `hour`, `day_of_week` after encoding (keep `month` as ordinal)

### 3. Numeric Feature Engineering
**Source columns:** `Daily Time Spent on Site`, `Age`, `Area Income`, `Daily Internet Usage`  
**Actions:**
- **Interaction term:** `usage_time_interaction = Daily Internet Usage × Daily Time Spent on Site`
  - Captures the combined signal found strongest in EDA
- **Log transforms** for right-skewed features: `log1p(Area Income)`
- **Age bucketing:** `age_group` → ordinal int (0=<25, 1=25-34, 2=35-44, 3=45-54, 4=55+)
- **Income quartile:** `income_quartile` (1–4) via `ntile(4)`
- **Outlier flags:** for each numeric column, add `is_outlier_<col>` boolean using IQR bounds computed on training data only (pass bounds as parameters — never recompute on test set)
- **Min-max scaling:** apply `pyspark.ml.feature.MinMaxScaler` after vectorisation (record scaler params for inference)

### 4. Categorical Feature Engineering
**Source columns:** `Gender`, `Country`, `City`  
**Actions:**
- **`Gender`:** `StringIndexer` → `OneHotEncoder` (drop one category to avoid dummy trap)
- **`Country`:** target-encode using the training-set click rate per country
  - Formula: `(click_count + α) / (total_count + 2α)` with smoothing `α = 10`
  - Store the encoding map as a Delta table / dict for inference reuse
  - Drop raw `Country` string column after encoding
- **`City`:** same target-encoding approach; given very high cardinality, also add `city_impression_count` (log-scaled) as a frequency feature
  - Drop raw `City` string column after encoding
- Log the full encoding map to MLflow as a JSON artifact

### 5. Text Feature Engineering — Ad Topic Line
**Source column:** `Ad Topic Line`  
**Actions:**
- **Keyword flags:** create binary columns for high-CTR topic keywords identified in EDA
  - `topic_has_tech`, `topic_has_finance`, `topic_has_health` (regex-based)
- **TF-IDF:**
  - Tokenize via `pyspark.ml.feature.Tokenizer` + `StopWordsRemover`
  - `HashingTF` (numFeatures=512) → `IDF` → dense top-N feature extraction
  - Reduce to top 20 TF-IDF features using `ChiSqSelector` against the target
  - Name selected features `tfidf_0` … `tfidf_19`
- Drop raw `Ad Topic Line` after encoding

### 6. Feature Assembly & Scaling
- Collect all engineered feature columns into a single vector using `VectorAssembler`
  - Exclude: `Timestamp`, raw string columns, `TARGET_COL`
- Apply `MinMaxScaler` to the assembled vector → output column `features_scaled`
- Keep `TARGET_COL` (renamed to `label` for MLlib compatibility) alongside `features_scaled`
- Print the full feature list with types and value ranges

### 7. Train / Validation / Test Split
- Split `(train=0.70, val=0.15, test=0.15)` using `df.randomSplit([0.70, 0.15, 0.15], seed=RANDOM_SEED)`
- **Important:** fit all encoders (target encoder, TF-IDF IDF, scaler) on **training set only**; transform val and test using fitted params
- Report class balance in each split — flag if any split deviates > 5 pp from overall

### 8. Feature Importance Proxy
Before modelling, get a quick signal ranking without training a full model:
- **Correlation with target:** absolute Pearson r for each numeric feature (post-encoding)
- **Chi-squared scores:** for binary/ordinal features via `pyspark.ml.feature.ChiSqSelector`
- Produce a ranked feature importance bar chart
- Identify and flag any near-zero-variance features (`std < 0.01`) for removal

### 9. Save Engineered Dataset
- Write train/val/test splits to Delta format at `OUTPUT_PATH/{train,val,test}`
  - `df_train.write.format("delta").mode("overwrite").save(f"{OUTPUT_PATH}/train")`
- Save transformer artifacts (scaler model, TF-IDF pipeline, encoding maps) to `OUTPUT_PATH/artifacts/`
- Print final schema and row counts per split

### 10. MLflow Logging & Feature Registry
- Log to an MLflow run named `ad_click_feature_engineering`:
  - **Params:** all constants, feature counts, split sizes, TF-IDF `numFeatures`, smoothing `α`
  - **Metrics:** train/val/test row counts, class balance per split, near-zero-variance feature count
  - **Artifacts:** encoding maps (JSON), feature list (CSV), at least 3 charts (correlation bar, feature distribution overlays, outlier flag rates)
- Log the final feature schema as an MLflow tag `feature_schema`

---

## Code Standards

- Every section starts with a Markdown cell explaining *what* and *why*
- All transformer parameters defined as constants at the top — no magic numbers inside cells
- Encoders fitted on training data only; transformations applied consistently to all splits
- Functions > 15 lines go in `utils/fe_helpers.py`, loaded via `%run ./utils/fe_helpers`
- Notebook runs end-to-end with **Run All** on Databricks Serverless without errors
- **Spark best practices:**
  - Cache `df_train` after fitting steps (reused multiple times); unpersist before saving
  - Use `F` alias for `pyspark.sql.functions` throughout
  - No `.collect()` on unfiltered DataFrames
  - `%pip install` in its own first cell

---

## Constraints & Guardrails

- Do **not** train any ML model in this notebook — that is the next step
- Do **not** recompute encoding statistics on validation or test data — data leakage
- Do **not** drop the `TARGET_COL` / `label` column from saved splits
- Do **not** silently coerce nulls — raise or log a warning if unexpected nulls appear post-join
- Do **not** hardcode file paths — use the `OUTPUT_PATH` constant throughout
- Target-encoding smoothing **must** use training data statistics only

---

## Acceptance Criteria

| Criterion | Definition of Done |
|---|---|
| All EDA-driven features built | Interaction term, cyclical encoding, target encoding, TF-IDF present |
| No data leakage | All encoders fitted on train only |
| Splits | 70/15/15 split with reproducible seed; balance reported |
| Output persisted | Delta files written to `OUTPUT_PATH/{train,val,test}` |
| Artifacts saved | Scaler, TF-IDF pipeline, encoding maps saved for inference reuse |
| Feature ranking | Correlation + chi-squared proxy importance chart produced |
| MLflow logged | Params, metrics, and ≥ 3 artifacts in one run |
| Reproducibility | Notebook runs clean top-to-bottom on Databricks Serverless |
