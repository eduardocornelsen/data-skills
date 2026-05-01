<p align="center">
  <img src="assets/logo_final.png" alt="data-skills logo" width="180"/>
</p>

<p align="center">
  <strong>AI can write notebooks. data-skills makes them production-grade.</strong>
</p>

<p align="center">
  Turn Claude Code into a senior data science team — with playbooks, personas,<br/>
  and guardrails for every phase of a DS project.
</p>

<p align="center">
  <a href="#quick-start--run-a-full-pipeline-in-one-command">Quick Start</a> ·
  <a href="#skill-reference">Skills</a> ·
  <a href="#personas-your-virtual-ds-team">Personas</a> ·
  <a href="#reference-implementation">Examples</a>
</p>

---

## The Problem

You ask Claude to analyze a dataset. It writes a notebook. That notebook:

- Skips assumption checks before running statistical tests
- Leaks target information into features
- Trains one model with no comparison baseline
- Has no experiment tracking
- Can't be reproduced by anyone else

Not because Claude is bad. Because Claude has no structure to follow.

---

## The Solution

`data-skills` gives Claude that structure.

It installs a library of **playbooks**, **personas**, and **guardrails** directly into Claude Code. Claude reads them. Claude follows them. You get senior-quality output.

| Without `data-skills` | With `data-skills` |
|---|---|
| One notebook, no structure | Phase-by-phase notebooks with clear outputs |
| No assumption checks | Full statistical diagnostics before every test |
| No experiment tracking | Every run logged to MLflow automatically |
| Data leakage goes undetected | Reviewer persona audits the full pipeline |
| Inconsistent quality | Same playbook, every time |

No coding experience required. You talk to Claude in plain English. It follows the playbooks and adopts the right persona for each task.

---

## See It in Action

> Demo GIF coming soon — see the [reference implementation](#reference-implementation) for a full walkthrough.

*One command. Full pipeline. Structured outputs. Reproducible artifacts.*

---

## Quick Start — Run a Full Pipeline in One Command

Already have Claude Code installed and a dataset ready? Copy this into Claude Code and go:

```
/batch-analysis

dataset: data/my_dataset.csv
target: churned
problem_type: binary_classification
business_goal: Predict which customers will churn in the next 30 days
```

That's it. Claude will run the entire data science pipeline automatically:

1. **EDA** — profile the data, generate visualizations, build a hypothesis register
2. **Hypothesis Testing** — rigorous statistical tests with effect sizes and power analysis
3. **Feature Engineering** — encode, scale, and transform features; create train/val/test splits
4. **Model Training** — train 3+ candidate models, tune hyperparameters, log to MLflow, pick a champion
5. **Model Evaluation** — deep evaluation with calibration, fairness, error analysis, and Go/No-Go promotion gate
6. **Inferencing** — score data with the champion model, validate schema
7. **Monitoring** — set up drift detection, performance decay tracking, retraining triggers, and rollback
8. **Stakeholder Communication** — translate results into executive summaries, reports, slide decks, and one-pagers
9. **Review** — audit the full pipeline for leakage, statistical validity, and reproducibility

Each phase produces a self-contained notebook and reusable utility code. All experiments are tracked in MLflow.

> Don't have Claude Code yet? See [Prerequisites](#prerequisites) and [Setup Guide](#setup-guide-step-by-step) below.

---

## Table of Contents

- [Quick Start — Run a Full Pipeline in One Command](#quick-start--run-a-full-pipeline-in-one-command)
- [What This Is](#what-this-is)
- [Who This Is For](#who-this-is-for)
- [What You Get](#what-you-get)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Setup Guide (Step by Step)](#setup-guide-step-by-step)
- [How to Use: Full Cycle Walkthrough](#how-to-use-full-cycle-walkthrough)
- [Skill Reference](#skill-reference)
- [Batch Analysis: Run Everything at Once](#batch-analysis-run-everything-at-once)
- [Personas: Your Virtual DS Team](#personas-your-virtual-ds-team)
- [Reference Implementation](#reference-implementation)
- [Roadmap](#roadmap)
- [License](#license)

---

## What This Is

`data-skills` is a **prompt engineering framework for data science**. It packages the knowledge, workflows, and quality gates of a senior data science team into structured Markdown files that Claude Code can follow step by step.

It is **not** a Python library or a CLI tool. It is a set of instructions that turn Claude into a domain expert that:

- Writes production-quality Jupyter notebooks
- Follows statistical best practices (assumption checks, effect sizes, multiple-testing correction)
- Tracks experiments with MLflow
- Produces reusable utility code
- Catches data leakage, train/test contamination, and other common pitfalls
- Explains every decision in plain language

Think of it as hiring a senior data scientist who never forgets a best practice.

---

## Who This Is For

| You are... | How this helps |
|---|---|
| **A non-coder / business analyst** | Tell Claude what you want to analyze in plain English. It writes the notebooks, runs the code, and explains the results. |
| **A junior data scientist** | Learn best practices by watching Claude follow professional playbooks. Every step is documented and explained. |
| **A senior data scientist** | Skip boilerplate. Let Claude handle the mechanical parts while you focus on the interesting decisions. |
| **A team lead** | Standardize how your team approaches DS projects. Everyone follows the same playbooks and quality gates. |

---

## What You Get

### Playbooks — *How* to do each DS phase

Step-by-step instructions that Claude follows to produce notebooks, code, and artifacts. Each playbook is dataset-agnostic — it works with any tabular dataset.

| Playbook | What it does |
|---|---|
| `00_PROBLEM_FRAMING.md` | Problem framing — business question, KPI, baseline, cost asymmetry, stakeholder map, scope |
| `01_DATA_CONTRACT.md` | Data contracts — schema validation, freshness SLA, PII flags, value ranges, drift tolerance |
| `01b_DATA_MODELING.md` | Dimensional modeling — star/snowflake schemas, SCD types, marts, data quality tests, semantic layer |
| `02_EDA.md` | Exploratory data analysis — profiling, distributions, correlations, hypothesis register |
| `03_HYPOTHESIS_TESTING.md` | Rigorous statistical tests — assumption checks, effect sizes, power analysis, multiple-testing correction |
| `04_FEATURE_ENGINEERING.md` | Transform raw data into model-ready features — encoding, scaling, temporal features, text features |
| `05_MODEL_TRAINING.md` | Train and compare candidate models — hyperparameter tuning, MLflow tracking, champion selection |
| `06_MODEL_EVALUATION.md` | Deep model evaluation — calibration, fairness, error analysis, SHAP stability, Go/No-Go promotion gate |
| `07_INFERENCING.md` | Score new data in production — schema validation, feature pipelines, drift monitoring |
| `08_MONITORING.md` | Operational monitoring — feature drift, prediction drift, performance decay, retraining triggers, rollback |
| `09_EXPERIMENTATION.md` | Experiment design — power analysis, MDE sensitivity, randomization, sequential testing, CUPED (standalone) |
| `10_STAKEHOLDER_COMMUNICATION.md` | Business translation — audience mapping, SCR narrative, impact quantification, executive summaries |

### Personas — *Who* does the work

Role-specific system prompts that set Claude's tone, focus areas, and quality gates. Each persona behaves like a real team member with years of experience.

| Persona | Role |
|---|---|
| `data_analyst` | EDA and business intelligence — thinks in SQL and story |
| `analytics_engineer` | Dimensional modeling, semantic layer, marts, data quality testing |
| `data_engineer` | Pipeline architecture, data quality, infrastructure |
| `ml_engineer` | MLOps, model deployment, monitoring, CI/CD for ML |
| `data_scientist_reviewer` | Statistical rigor gatekeeper — the last quality gate |
| `product_manager` | Feature prioritization, roadmaps, stakeholder alignment |
| `ux_researcher` | User behavior analysis, usability testing |
| `frontend_developer` | Data visualization UIs, dashboard implementation |
| `qa_tester` | Test automation, edge cases, data quality validation |

### Reference Implementation — *What* the output looks like

A fully executed end-to-end pipeline on an ad-click prediction dataset (10,000 records), showing exactly what Claude produces when following the playbooks.

---

## Project Structure

```
data-skills/
│
├── README.md                          # You are here
├── PLAN.md                            # Development roadmap and architectural decisions
├── CLAUDE.md                          # Agent instructions (Claude reads this automatically)
├── LICENSE                            # MIT License
│
├── .claude/
│   └── skills/                        # Claude Code skill definitions (invocable via /slash commands)
│       ├── playbooks/
│       │   ├── problem-framing/SKILL.md        # /problem-framing — scope the problem
│       │   ├── data-contract/SKILL.md         # /data-contract — validate data quality
│       │   ├── data-modeling/SKILL.md         # /data-modeling — dimensional modeling
│       │   ├── eda/SKILL.md                    # /eda — run exploratory data analysis
│       │   ├── hypothesis-testing/SKILL.md    # /hypothesis-testing — statistical tests
│       │   ├── feature-engineering/SKILL.md    # /feature-engineering — build features
│       │   ├── model-training/SKILL.md         # /model-training — train models
│       │   ├── model-evaluation/SKILL.md       # /model-evaluation — deep evaluation & promotion gate
│       │   ├── inferencing/SKILL.md            # /inferencing — score new data
│       │   ├── monitoring/SKILL.md             # /monitoring — drift detection & retraining
│       │   ├── experimentation/SKILL.md        # /experimentation — A/B test design (standalone)
│       │   ├── stakeholder-communication/SKILL.md # /stakeholder-communication — business translation
│       │   └── batch-analysis/SKILL.md         # /batch-analysis — run full 9-phase pipeline
│       └── personas/
│           ├── persona-analytics-engineer/SKILL.md
│           ├── persona-data-analyst/SKILL.md
│           ├── persona-data-engineer/SKILL.md
│           ├── persona-data-scientist-reviewer/SKILL.md
│           ├── persona-ml-engineer/SKILL.md
│           ├── persona-product-manager/SKILL.md
│           ├── persona-ux-researcher/SKILL.md
│           ├── persona-frontend-developer/SKILL.md
│           └── persona-qa-tester/SKILL.md
│
├── playbooks/                         # Canonical playbook content (skills reference these)
│   ├── 00_PROBLEM_FRAMING.md          #   Business problem framing
│   ├── 01_DATA_CONTRACT.md            #   Data contract validation
│   ├── 01b_DATA_MODELING.md           #   Dimensional modeling & marts
│   ├── 02_EDA.md                      #   Exploratory data analysis
│   ├── 03_HYPOTHESIS_TESTING.md       #   Statistical hypothesis testing
│   ├── 04_FEATURE_ENGINEERING.md      #   Feature engineering
│   ├── 05_MODEL_TRAINING.md           #   Model training & selection
│   ├── 06_MODEL_EVALUATION.md         #   Deep evaluation & Go/No-Go gate
│   ├── 07_INFERENCING.md              #   Batch inference & drift detection
│   ├── 08_MONITORING.md               #   Operational monitoring & rollback
│   ├── 09_EXPERIMENTATION.md          #   A/B test design (standalone)
│   └── 10_STAKEHOLDER_COMMUNICATION.md #  Business translation & deliverables
│
├── personas/                          # Canonical persona content (skills reference these)
│   ├── _template.md                   #   Meta-template for creating new personas
│   ├── analytics_engineer.md
│   ├── data_analyst.md
│   ├── data_engineer.md
│   ├── data_scientist_reviewer.md
│   ├── ml_engineer.md
│   ├── product_manager.md
│   ├── ux_researcher.md
│   ├── frontend_developer.md
│   └── qa_tester.md
│
├── templates/                         # Copy-paste scaffolds (playbooks fill these in)
│   └── problem_statement.md           #   Problem framing output template
│
├── examples/
│   └── reference_implementations/
│       └── ad_click/                  # Full executed pipeline (10k ad impressions)
│           ├── README.md
│           ├── ad_10000records.csv    #   Source dataset
│           ├── eda.ipynb              #   EDA notebook (executed)
│           ├── feature_engineering.ipynb
│           ├── model_training.ipynb
│           ├── inferencing.ipynb
│           ├── databricks.yml         #   Databricks Asset Bundle config
│           ├── requirements.txt       #   Extra pip dependencies
│           └── utils/                 #   Reusable Python helpers
│               ├── eda_helpers.py
│               ├── fe_helpers.py
│               └── model_helpers.py
│
└── assets/
    └── logo_final.png                 # Project logo
```

### What each folder does

| Folder | Purpose | Who uses it |
|---|---|---|
| `playbooks/` | Source-of-truth instructions for each DS phase. Claude reads these before writing any code. | Claude (automatically) |
| `personas/` | Role-specific system prompts. Each one makes Claude behave like a specific team member. | You (choose which persona to activate) |
| `.claude/skills/` | Thin wrappers that make playbooks and personas invocable as `/slash` commands in Claude Code. | You (type `/eda` or `/persona-data-analyst`) |
| `examples/` | Pre-executed notebooks showing what the output looks like. Use as reference, not as templates. | You (for reference) |

---

## Prerequisites

You need exactly **two things**:

### 1. Claude Code

Claude Code is Anthropic's AI coding agent. Install it using one of these methods:

**Option A: Terminal (CLI)**
```bash
# Install via npm (requires Node.js 18+)
npm install -g @anthropic-ai/claude-code
```

**Option B: Desktop App**
- Download from [claude.ai/code](https://claude.ai/code) (available for Mac and Windows)

**Option C: VS Code Extension**
- Open VS Code → Extensions → Search "Claude Code" → Install

> You will need an [Anthropic API key](https://console.anthropic.com/) or a Claude Pro/Max subscription to use Claude Code.

### 2. A Dataset

Any tabular dataset in CSV, Parquet, or Delta format. The playbooks are dataset-agnostic — they work with any structured data.

If you don't have a dataset, you can use the included `ad_10000records.csv` (10,000 ad impressions) to follow along.

### Optional: Databricks

The reference implementation runs on Databricks Serverless with PySpark and Delta Lake. But the playbooks work with plain pandas too — Claude adapts automatically based on your environment.

---

## Setup Guide (Step by Step)

### Step 1: Get the repository

```bash
# Clone the repository
git clone https://github.com/eduardocornelsen/data-skills.git

# Navigate into it
cd claude-code-data-science
```

Or, to add as a submodule to an existing project:

```bash
git submodule add https://github.com/eduardocornelsen/data-skills.git data-skills
git submodule update --init --recursive
```

### Step 2: Open in Claude Code

**If using the terminal CLI:**
```bash
cd claude-code-data-science
claude
```

**If using VS Code:**
1. Open the `claude-code-data-science` folder in VS Code
2. Open the Claude Code panel (sidebar icon or `Ctrl+Shift+P` → "Claude Code")

**If using the Desktop App:**
1. Open the app
2. Click "Open Folder" and select `claude-code-data-science`

### Step 3: Verify it works

Type this in Claude Code:

```
What skills do you have available? List the playbooks and personas.
```

Claude should list all the available `/eda`, `/feature-engineering`, `/model-training`, `/inferencing`, and persona skills.

### Step 4: Place your dataset

Copy your CSV or Parquet file into the project root (or any subfolder). Then tell Claude where it is:

```
My dataset is in data/my_dataset.csv. It contains customer purchase records
with columns for customer_id, purchase_date, amount, category, and churned.
```

That's it. You're ready to go.

---

## How to Use: Full Cycle Walkthrough

Below is a complete end-to-end workflow. Each step shows the exact prompt to type into Claude Code. You can run them one at a time, or use `/batch-analysis` to run everything at once (see [Batch Analysis](#batch-analysis-run-everything-at-once)).

### Phase 1: Exploratory Data Analysis

**What it does:** Profiles your data, generates visualizations, finds patterns, checks distributions, and builds a hypothesis register.

**Prompt:**
```
/eda

My dataset is in data/my_dataset.csv.
The business goal is to predict customer churn.
The target variable is "churned".
```

**What Claude produces:**
- `eda.ipynb` — a complete EDA notebook with 10+ sections
- `utils/eda_helpers.py` — reusable visualization and profiling functions
- A hypothesis register summarizing what to test next

---

### Phase 2: Hypothesis Testing

**What it does:** Takes the hypotheses from EDA and tests them rigorously — assumption checks, effect sizes, power analysis, confidence intervals, and multiple-testing correction.

**Prompt:**
```
/hypothesis-testing

Use the hypothesis register from the EDA notebook.
The target variable is "churned".
Focus on the high-priority hypotheses first.
```

**What Claude produces:**
- `hypothesis_testing.ipynb` — complete testing notebook with diagnostics
- `utils/hypothesis_helpers.py` — reusable functions for assumption checks and effect sizes
- Updated hypothesis register with verdicts, effect sizes, and practical significance

---

### Phase 3: Feature Engineering

**What it does:** Transforms raw columns into model-ready features — encoding, scaling, temporal features, text features, interaction terms.

**Prompt:**
```
/feature-engineering

Use the findings from the EDA and hypothesis testing notebooks.
The target variable is "churned".
Create train/validation/test splits (70/15/15).
```

**What Claude produces:**
- `feature_engineering.ipynb` — full feature pipeline
- `utils/fe_helpers.py` — reusable transformers
- Train/val/test splits saved to disk

---

### Phase 4: Model Training

**What it does:** Trains multiple candidate models, tunes hyperparameters, logs everything to MLflow, and selects a champion.

**Prompt:**
```
/model-training

Use the feature-engineered splits.
This is a binary classification problem (churn prediction).
Compare at least 3 model families.
```

**What Claude produces:**
- `model_training.ipynb` — training notebook with MLflow experiment tracking
- `utils/model_helpers.py` — MLflow logging and evaluation utilities
- A champion model selected and registered

---

### Phase 5: Model Evaluation

**What it does:** Deep evaluation of the champion model — calibration analysis, fairness assessment, error analysis, SHAP stability, robustness checks — culminating in a formal Go/No-Go promotion decision.

**Prompt:**
```
/model-evaluation

Evaluate the champion model from model_training.ipynb.
Check calibration, fairness across gender and age groups,
and identify worst-performing segments.
```

**What Claude produces:**
- `model_evaluation.ipynb` — complete evaluation with all analyses
- `utils/eval_helpers.py` — evaluation and fairness helper functions
- Promotion decision (PROMOTE / CONDITIONAL / REJECT) logged to MLflow

---

### Phase 6: Inferencing

**What it does:** Scores new data using the promoted champion model, validates schema, and applies the feature engineering pipeline.

**Prompt:**
```
/inferencing

Use the champion model from training.
Score the data in data/new_customers.csv.
Include drift monitoring against the training distribution.
```

**What Claude produces:**
- `inferencing.ipynb` — batch scoring pipeline
- Predictions saved to disk (CSV or Delta)
- Drift monitoring report comparing new data to training distributions

---

### Phase 7: Monitoring Setup

**What it does:** Configures operational monitoring — feature drift detection, prediction drift, performance decay tracking, retraining triggers, alerting, and a rollback runbook.

**Prompt:**
```
/monitoring

Set up monitoring for the deployed churn model.
Track drift on all features using KS, PSI, and JS divergence.
Configure alerts for performance decay below AUC 0.80.
```

**What Claude produces:**
- `monitoring_setup.ipynb` — monitoring configuration notebook
- `utils/monitoring_helpers.py` — drift computation and alerting helpers
- `monitoring_runbook.md` — standalone operational runbook for on-call response

---

### Phase 8: Stakeholder Communication

**What it does:** Transforms technical results into audience-tailored business communication — executive summaries, detailed reports, slide decks, and one-pagers.

**Prompt:**
```
/stakeholder-communication

Create deliverables for the churn prediction project.
Audiences: VP Marketing (executive), Analytics Lead (domain expert),
and the ML team (technical peers).
```

**What Claude produces:**
- `stakeholder_communication.ipynb` — chart generation and translation scripts
- `executive_summary.md` — answer-first summary with impact quantification
- `detailed_technical_report.md` — full methodology and results
- `slide_deck_outline.md` — 10-15 slide structure
- `one_pager.md` — problem, approach, finding, impact, recommendation

---

### Phase 9: Review (Optional but Recommended)

**What it does:** Activates the Data Scientist Reviewer persona — a methodological gatekeeper who audits your entire pipeline.

**Prompt:**
```
/persona-data-scientist-reviewer

Review the full pipeline: eda.ipynb, hypothesis_testing.ipynb,
feature_engineering.ipynb, model_training.ipynb, model_evaluation.ipynb,
inferencing.ipynb, monitoring_setup.ipynb, and all stakeholder deliverables.
Check for data leakage, statistical validity, and reproducibility.
```

**What Claude produces:**
- A structured review report with APPROVED / REVISE / REJECTED verdict per notebook/deliverable
- Specific findings with line references and recommendations

---

### Standalone: Experiment Design (Use Anytime)

**What it does:** Designs rigorous A/B tests and experiments — power analysis, MDE sensitivity, randomization, SRM detection, sequential testing, CUPED variance reduction. **Not part of the linear pipeline** — invoke whenever you need causal validation.

**Prompt:**
```
/experimentation

Design an A/B test for our new onboarding flow.
Primary metric: 7-day retention rate (baseline: 35%).
We need to detect a 3 percentage point lift.
Daily eligible traffic: 5,000 users.
```

**What Claude produces:**
- `experiment_design.ipynb` — power analysis, MDE curves, and design specification
- `utils/experiment_helpers.py` — power computation and SRM detection helpers
- Experiment design document ready for stakeholder sign-off

---

## Skill Reference

### Playbook Skills

| Skill | Command | Description |
|---|---|---|
| Problem Framing | `/problem-framing` | Frame a business question into a scoped problem statement |
| Data Contract | `/data-contract` | Validate schema, freshness, PII, and quality guarantees |
| Data Modeling | `/data-modeling` | Design dimensional models with staging, intermediate, and marts |
| EDA | `/eda` | Exploratory data analysis — profiling, distributions, correlations, hypothesis register |
| Hypothesis Testing | `/hypothesis-testing` | Statistical tests with assumption checks, effect sizes, power analysis |
| Feature Engineering | `/feature-engineering` | Build model-ready features from raw data |
| Model Training | `/model-training` | Train, compare, and select champion model |
| Model Evaluation | `/model-evaluation` | Deep evaluation — calibration, fairness, error analysis, Go/No-Go gate |
| Inferencing | `/inferencing` | Score new data and monitor for drift |
| Monitoring | `/monitoring` | Set up drift detection, performance decay, retraining triggers |
| Experimentation | `/experimentation` | Design A/B tests — power analysis, MDE, randomization (standalone) |
| Stakeholder Communication | `/stakeholder-communication` | Translate results into business-ready deliverables |
| Batch Analysis | `/batch-analysis` | Run the full 9-phase pipeline end-to-end (see below) |

### Persona Skills

| Skill | Command | Description |
|---|---|---|
| Analytics Engineer | `/persona-analytics-engineer` | Dimensional modeling, semantic layer, data quality tests |
| Data Analyst | `/persona-data-analyst` | EDA and business intelligence — thinks in SQL and story |
| Data Engineer | `/persona-data-engineer` | Pipeline architecture, data quality, infrastructure |
| ML Engineer | `/persona-ml-engineer` | MLOps, model deployment, monitoring |
| DS Reviewer | `/persona-data-scientist-reviewer` | Statistical rigor gatekeeper — quality audits |
| Product Manager | `/persona-product-manager` | Feature prioritization, roadmaps |
| UX Researcher | `/persona-ux-researcher` | User behavior analysis, usability testing |
| Frontend Dev | `/persona-frontend-developer` | Data visualization UIs, dashboards |
| QA Tester | `/persona-qa-tester` | Test automation, edge cases, validation |

---

## Batch Analysis: Run Everything at Once

The `/batch-analysis` skill runs the complete data science pipeline in a single command. It executes all 9 phases sequentially (EDA -> Hypothesis Testing -> Feature Engineering -> Model Training -> Model Evaluation -> Inferencing -> Monitoring -> Stakeholder Communication -> Review), with each phase building on the outputs of the previous one.

> **Note:** The Experimentation playbook (`/experimentation`) is standalone — it is not part of the batch pipeline. Invoke it separately when experiment design is needed.

### Usage

```
/batch-analysis

dataset: data/my_dataset.csv
target: churned
problem_type: binary_classification
business_goal: Predict which customers will churn in the next 30 days
```

### What happens

1. **EDA** — Claude profiles the dataset, generates visualizations, and creates a hypothesis register
2. **Hypothesis Testing** — Statistical tests validate EDA findings with effect sizes and power analysis
3. **Feature Engineering** — Transforms are applied based on EDA and hypothesis findings, splits are created
4. **Model Training** — Multiple models are trained, compared, and a champion is selected
5. **Model Evaluation** — Deep evaluation with Go/No-Go promotion gate (blocks pipeline if REJECT)
6. **Inferencing** — The promoted champion model scores data with schema validation
7. **Monitoring** — Drift detection, performance decay tracking, retraining triggers, and rollback runbook
8. **Stakeholder Communication** — Executive summary, detailed report, slide deck, and one-pager
9. **Review** — A final quality gate checks the entire pipeline

### Parameters

| Parameter | Required | Description |
|---|---|---|
| `dataset` | Yes | Path to your CSV, Parquet, or Delta table |
| `target` | Yes | Name of the target/label column |
| `problem_type` | Yes | `binary_classification`, `multiclass_classification`, or `regression` |
| `business_goal` | No | Plain-English description of what you're trying to achieve |
| `output_dir` | No | Where to save notebooks and artifacts (default: project root) |
| `skip_review` | No | Set to `true` to skip the final review phase |

### Example with all options

```
/batch-analysis

dataset: data/customer_data.parquet
target: lifetime_value
problem_type: regression
business_goal: Predict customer lifetime value to optimize acquisition spend
output_dir: analysis/ltv_model
```

---

## Personas: Your Virtual DS Team

Each persona makes Claude behave like a specific team member. They have defined expertise, communication styles, focus areas, and quality gates.

### When to use personas

- **You don't need to pick a persona** for normal playbook work — the playbooks already embed the right expertise.
- **Use personas when** you want Claude to focus on a specific role's perspective, or when you're doing work outside the standard playbook flow.

### Examples

**Ask the Data Analyst to explain findings to stakeholders:**
```
/persona-data-analyst

Summarize the EDA findings from eda.ipynb for a non-technical audience.
Focus on the top 3 actionable insights for the marketing team.
```

**Ask the ML Engineer about deployment:**
```
/persona-ml-engineer

How should we deploy the champion model from model_training.ipynb?
We need real-time scoring with <100ms latency.
```

**Ask the Reviewer to audit a notebook:**
```
/persona-data-scientist-reviewer

Review feature_engineering.ipynb for data leakage.
Pay special attention to the temporal features.
```

---

## Reference Implementation

The `examples/reference_implementations/ad_click/` directory contains a fully executed pipeline on an ad-click prediction dataset. This shows exactly what Claude produces when following the playbooks.

### Dataset

`ad_10000records.csv` — 10,000 ad impressions with:
- User behavior metrics (daily time spent on site, daily internet usage)
- Demographics (age, area income, gender)
- Contextual features (ad topic, city, country)
- Target: `Click on Ad` (binary)

### Notebooks

| Notebook | Description |
|---|---|
| `eda.ipynb` | Complete EDA with 70+ visualizations, correlation analysis, hypothesis register |
| `feature_engineering.ipynb` | Feature pipelines including TF-IDF, temporal features, interaction terms |
| `model_training.ipynb` | 3 candidate models (Logistic Regression, Random Forest, GBT) with MLflow tracking |
| `inferencing.ipynb` | Batch scoring pipeline with schema validation and drift monitoring |

### How to explore it

```
Open examples/reference_implementations/ad_click/eda.ipynb and walk me through
what this EDA found. What were the key insights?
```

---

## Roadmap

This project is in active development. Key planned additions:

| Priority | What | Status |
|---|---|---|
| High | Problem framing playbook (`00_PROBLEM_FRAMING.md`) | **Done** |
| High | Hypothesis testing playbook (`03_HYPOTHESIS_TESTING.md`) | **Done** |
| High | Data contracts playbook (`01_DATA_CONTRACT.md`) | **Done** |
| High | Data modeling playbook (`01b_DATA_MODELING.md`) | **Done** |
| High | Analytics engineer persona | **Done** |
| High | Model evaluation playbook (`06_MODEL_EVALUATION.md`) | **Done** |
| High | Monitoring playbook (`08_MONITORING.md`) | **Done** |
| High | Experimentation playbook (`09_EXPERIMENTATION.md`) | **Done** |
| High | Stakeholder communication playbook (`10_STAKEHOLDER_COMMUNICATION.md`) | **Done** |
| High | Full 9-phase batch pipeline | **Done** |
| Medium | Premises, checklists, and templates | Planned |
| Low | Domain accelerators (churn, forecasting, fraud, recommenders) | Planned |

See [PLAN.md](./PLAN.md) for the full roadmap, architectural decisions, and sequencing.

---

## FAQ

**Q: Do I need to know Python?**
No. Claude writes all the code. You describe what you want in plain English.

**Q: Do I need Databricks?**
No. The playbooks work with any Python environment. Claude adapts to pandas if PySpark/Databricks is not available.

**Q: Can I use my own dataset?**
Yes. The playbooks are dataset-agnostic. Just tell Claude where your data is and what the target variable is.

**Q: What if a playbook doesn't exist yet?**
Tell Claude what you need. It will use its training knowledge plus the existing playbooks as style guides. The planned playbooks in the roadmap will be added over time.

**Q: Can I customize the playbooks?**
Yes. They're plain Markdown files. Edit them to match your team's conventions, preferred libraries, or domain-specific requirements.

**Q: How do I add this to my own project?**
Add it as a git submodule (see [Setup Guide](#step-1-get-the-repository)). Claude Code auto-discovers the skills in `.claude/skills/`.

---

## License

MIT — see [LICENSE](./LICENSE).

---
<p align="center">
  <sub>
    Built with ☕ by <strong><a href="https://eduardocornelsen.com">Eduardo Cornelsen</a></strong>
  </sub>
  <br/>
  <sub>
    <a href="https://eduardocornelsen.com">Portfolio</a> · <a href="https://www.linkedin.com/in/eduardo-cornelsen/">LinkedIn</a>
  </sub>
  <br/>
  <sub>
    Creator of <strong>data-skills</strong> — an AI-native data science framework for Claude Code.
  </sub>
  <br/>
  <sub>
    Revenue Operations · Data Engineering · AI/ML · Full-Stack Development
  </sub>
</p>
