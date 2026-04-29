# Exploratory Data Analysis Prompt

## Role

You are a senior Data Scientist responsible for delivering a rigorous EDA on an **ad-click dataset**. Your output is a well-structured Databricks notebook (`eda.ipynb`) that a business stakeholder can read top-to-bottom and a ML engineer can use as the foundation for feature engineering.

---

## Environment

**Platform:** Databricks Serverless (cloud extension)  
**Compute:** Databricks Serverless compute — no cluster management required; notebooks attach automatically  
**Storage:** Delta Lake / Unity Catalog  
**Primary API:** PySpark (use pandas only for small aggregates after `.toPandas()`)  
**Visualization:** Use `display()` for native Databricks rendering; fall back to matplotlib/seaborn for custom charts  
**MLflow:** Available by default — log key statistics and plots via `mlflow.log_metric` / `mlflow.log_figure` for traceability

---

## Dataset Context

**Domain:** Digital advertising  
**Target variable:** `Click on Ad` (binary: 1 = clicked, 0 = did not click)  
**Key feature groups:**

| Group | Features |
|---|---|
| User behavior | clicks per session, time spent on site, daily internet usage |
| Demographics | age, gender, income |
| Contextual | city, country, device type, ad topic, timestamp |
| Performance | `Click on Ad` (target) |

---

## Deliverables

1. **`eda.ipynb`** — fully executed Databricks notebook with narrative, code, and visualizations
2. **`utils/`** — any reusable helper functions extracted to `.py` scripts and importable via `%run` or as a Databricks Workspace library
3. **`requirements.txt`** — additional pip dependencies (beyond what Databricks Serverless pre-installs)

---

## Notebook Structure

### 0. Setup
- Install any extra packages with `%pip install` (run in its own cell so the kernel restarts cleanly)
- Import libraries (pyspark.sql.functions, pandas, numpy, matplotlib, seaborn, plotly, scipy, etc.)
- Confirm Spark session is active: `spark.version`
- Set global plot style and random seed
- Define reusable helper functions (or `%run ./utils/eda_helpers` to import from the utils notebook)

### 1. Data Loading & Initial Inspection
- Load the dataset from Unity Catalog / Delta table or DBFS path using `spark.read`
  - Prefer Delta format: `spark.read.format("delta").load("<path>")` or `spark.table("<catalog>.<schema>.<table>")`
  - If loading CSV: use `inferSchema=True` and then validate inferred types
- Use `display(df)` for an interactive preview; also call `df.printSchema()` and `df.describe().display()`
- Report row count (`df.count()`), column count, and a sample of distinct values per column
- Check for duplicate rows with `df.groupBy(df.columns).count().filter("count > 1")`

### 2. Data Quality Assessment
- **Missing values:** use `df.select([F.count(F.when(F.col(c).isNull(), c)).alias(c) for c in df.columns])` — report count and percentage per column; visualize with a bar chart after `.toPandas()`
- **Duplicates:** identify and document handling strategy
- **Type validation:** confirm numeric vs. categorical assignments are correct; cast if needed with `df.withColumn(..., F.col(...).cast(...))`
- **Outlier scan:** compute per-column mean/std via `df.summary()`; flag columns with values beyond 3σ or IQR fences
- **Date/time parsing:** cast timestamp strings to `TimestampType` and extract `hour`, `day_of_week`, `month` using `F.hour()`, `F.dayofweek()`, `F.month()`
- **Delta table metadata** (if source is Delta): run `DESCRIBE HISTORY` to confirm data freshness and last write timestamp

### 3. Univariate Analysis
For every feature, report:
- **Numeric:** compute mean, median, std, min, max, skewness, kurtosis with `df.select(F.mean(), F.stddev(), F.skewness(), F.kurtosis())` — convert to pandas for plotting histograms and boxplots
- **Categorical:** `df.groupBy(col).count().orderBy("count", ascending=False).display()` for value counts and top-N bar chart
- Use `display()` for quick interactive exploration; use matplotlib/seaborn for publication-quality charts
- Highlight any suspicious distributions (near-zero variance, extreme skew)

### 4. Target Variable Analysis
- Class distribution: absolute counts and percentages
- Class imbalance ratio — flag if > 3:1
- Click rate by each categorical feature (bar charts sorted by rate)
- Click rate bucketed by key numeric features (quartile bins)

### 5. Bivariate & Multivariate Analysis
- **Numeric vs. target:** box plots and violin plots for each numeric feature split by `Click on Ad`
- **Categorical vs. target:** grouped bar charts, heatmaps (e.g., device × day_of_week)
- **Numeric vs. numeric:** scatter matrix / pairplot colored by target
- **Correlation matrix:** Pearson for numeric features; annotate cells with |r| > 0.5
- **Cramér's V matrix:** for categorical feature pairs

### 6. Temporal Analysis
- Hourly click rate line chart (average across all days)
- Day-of-week click rate bar chart
- Heatmap: hour × day_of_week vs. click rate
- Identify peak and off-peak engagement windows

### 7. Segment & Cohort Analysis
- Click rate by age group (binned: <25, 25–34, 35–44, 45–54, 55+)
- Click rate by income quartile
- Click rate by gender × age group (stacked bar)
- Top 10 and bottom 10 countries/cities by click rate (horizontal bar)
- Device type breakdown and its interaction with time-of-day

### 8. Ad Performance Analysis
- Click rate by ad topic / category (sorted bar chart)
- Click rate by ad placement / position (if available)
- Funnel visualization: impressions → clicks → conversions (if conversion data exists)

### 9. Outlier & Anomaly Deep-Dive
- Investigate flagged outliers from Section 2 in context of the target
- Determine whether outliers are data errors or genuine extreme behavior
- Document the recommended treatment (cap, remove, keep, or flag as a feature)

### 10. Key Findings & Hypotheses
- Bullet-point summary of the top 5–10 findings
- For each finding: observation → business interpretation → suggested follow-up
- List features ranked by apparent predictive signal for `Click on Ad`
- Identify features that are likely redundant or low-value for modeling

---

## Code Standards

- Every section starts with a Markdown cell explaining *what* is being done and *why*
- No magic numbers — define constants (catalog name, table path, target column name) at the top of the notebook
- Each visualization has: title, labeled axes, and a one-sentence insight caption below it
- Functions longer than ~15 lines go in `utils/eda_helpers.py` and are loaded with `%run ./utils/eda_helpers`
- The notebook must run end-to-end with **Run All** without errors on Databricks Serverless
- **Spark best practices:**
  - Avoid `.collect()` on large DataFrames — aggregate first, then collect small results
  - Cache (`df.cache()`) only DataFrames reused more than twice; unpersist when done
  - Prefer Spark SQL aggregations over converting to pandas early
  - Use `F` alias for `pyspark.sql.functions` consistently throughout
- **MLflow logging:** at the end of the notebook, log key summary stats and all figures to an MLflow run for reproducibility

---

## Constraints & Guardrails

- Do **not** perform model training or feature engineering in this notebook — that is a separate step
- Do **not** drop columns or rows silently — document every removal decision
- Flag but do **not** automatically fix data quality issues; document the recommended fix
- Keep individual cells focused — one analysis per cell
- Total notebook length: aim for **30–60 cells**
- **Databricks-specific:**
  - Do **not** use `pd.read_csv()` directly for large files — load via Spark then call `.toPandas()` on aggregates
  - Do **not** use `%sh pip install` — use `%pip install` (cell-magic form) so the driver and workers stay in sync
  - Do **not** hardcode DBFS paths as `/dbfs/...` strings — use the `dbutils.fs` API or Unity Catalog table references for portability

---

## Acceptance Criteria

| Criterion | Definition of Done |
|---|---|
| Coverage | All features analyzed in at least one section |
| Target analysis | Class imbalance documented; click rate computed per segment |
| Visualizations | Minimum 15 distinct charts, each with caption |
| Quality report | Missing values, duplicates, and outliers explicitly reported |
| Findings | At least 5 actionable hypotheses stated in Section 10 |
| Reproducibility | Notebook runs clean from top to bottom on Databricks Serverless |
| Spark correctness | No `.collect()` calls on raw/unfiltered large DataFrames |
| MLflow | Key metrics and at least 3 figures logged to an MLflow run |
| Dependencies | `requirements.txt` lists only packages not pre-installed on Databricks Serverless |
