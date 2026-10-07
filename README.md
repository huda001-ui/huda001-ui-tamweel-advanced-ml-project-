# tamweel-advanced-ml-project-
Huda Makki Mohammed
[SDAIA Academy] (https://github.com/SDAIAAcademy)

## Project Scenario

**Tamweel Lite** is a synthetic educational machine-learning project that simulates a financing organization receiving customer applications and using machine learning to estimate the probability of **default within 90 days of application**.

The objective is not to automatically approve or reject financing applications. Instead, the model produces risk estimates that are converted into **review flags** to support an educational review workflow.

The decision problem reflects two important operational considerations.

First, missing a true default case (**False Negative**) is treated as more costly than sending a non-default case for review (**False Positive**). The project therefore uses the following simulated educational decision cost:

**Decision Cost = 10 × False Negatives + 1 × False Positives**

Second, operational review capacity is limited to:

**12% of the application batch**

Therefore, the final workflow does not rely on model probability alone. It combines predictive evidence, probability calibration, threshold selection, simulated decision cost, and review capacity.

> **Important:** This project uses synthetic educational data and is not a production lending system or an automated financing approval/rejection system.

---

## Project Objectives

The objective is to build a complete, reproducible, and leakage-aware machine-learning workflow for default-risk classification.

The project aims to:

- Validate data quality and environment readiness before model development.
- Prevent **data leakage** by respecting temporal ordering, customer separation, and label maturity.
- Compare **Logistic Regression, XGBoost, and LightGBM** using consistent evaluation evidence.
- Apply **Forward Out-of-Fold (OOF) Validation** across future validation periods.
- Perform controlled hyperparameter tuning using **Optuna**.
- Evaluate ranking performance using **ROC-AUC** and **Average Precision**, particularly in the presence of class imbalance.
- Select a decision policy using the simulated cost function **10 × FN + FP** and the **12% review-capacity limit**.
- Interpret model behavior using **Permutation Importance and SHAP**.
- Evaluate and improve **probability calibration**.
- Compare the best single models with **Equal Averaging, Weighted Averaging, and Stacking**.
- Determine whether the additional complexity of an ensemble is justified by forward OOF evidence.
- Apply the frozen final procedure to the challenge dataset without using challenge labels for training or tuning.
- Produce reproducible project evidence, reports, final model artifacts, and `submission.csv`.

---

## Dataset Overview

The project uses the **Tamweel Lite synthetic educational dataset**.

| Dataset | Rows | Predictors | Missing Feature Values |
|---|---:|---:|---:|
| Training | 10,000 | 22 | 766 |
| Dirty | 10,000 | 22 | 766 |
| Challenge | 2,500 | 22 | 216 |

The target variable is:

`default_within_90d`

The training dataset has a positive-class rate of approximately:

**7.89%**

Because the positive class is relatively uncommon, the project does not rely on accuracy alone. **ROC-AUC** and **Average Precision (AP)** are used as important model-comparison metrics.

Challenge labels are unavailable and were **not used for model training, tuning, calibration, model selection, or final performance claims**.

---

# Project Journey

The project was developed through a structured sequence of notebooks.

| Notebook | Stage | Purpose |
|---|---|---|
| `00_readiness_check.ipynb` | Readiness Check | Verify environment, data, dependencies, and runtime compatibility |
| `01_baseline_boosting.ipynb` | Baseline & Boosting | Compare Logistic Regression, XGBoost, and LightGBM |
| `02_validation_tuning.ipynb` | Validation & Tuning | Apply leakage-aware forward validation and bounded Optuna tuning |
| `03_cost_sensitive_decision.ipynb` | Decision Policy | Evaluate thresholds, simulated decision cost, and capacity constraints |
| `04_explain_calibrate.ipynb` | Interpretation & Calibration | Evaluate model interpretation and probability calibration |
| `05_final_model.ipynb` | Final Model | Compare single models and ensembles, freeze the policy, and score challenge applications |

---

# 00 — Readiness Check

Before model development, the project environment and dataset were validated.

The readiness workflow verified:

- Required Python packages and versions
- Dataset structure
- Expected feature availability
- Logistic Regression compatibility
- XGBoost compatibility
- LightGBM compatibility
- SHAP margin additivity
- Sigmoid calibration
- Optuna compatibility

The readiness process completed successfully with:

**Status: READY**

This established a verified environment before beginning model development.

---

# 01 — Baseline & Boosting

Three classification models were initially compared:

- Logistic Regression
- XGBoost
- LightGBM

All three models were evaluated on the same **2,000 comparison rows**, with a positive-class rate of approximately **7.9%**.

## Initial Model Comparison

| Model | ROC-AUC | Average Precision |
|---|---:|---:|
| **Logistic Regression** | **0.8213** | 0.3258 |
| XGBoost | 0.8124 | **0.3338** |
| LightGBM | 0.8138 | 0.3248 |

Logistic Regression achieved the highest ROC-AUC, while XGBoost achieved the highest Average Precision in this initial comparison.

The results demonstrated that the boosting models did not uniformly outperform the simpler Logistic Regression baseline.

> These results represent an initial comparison and were not used alone to determine the final shipped model.

## Learning Curves

The learning-curve analysis was used to examine model behavior as the amount of training data increased.

![Day 1 Learning Curves](artifacts/day1_learning_curves.png)

---

# 02 — Leakage-Aware Validation & Tuning

A major focus of the project was preventing optimistic performance estimates caused by **data leakage**.

The validation strategy preserved:

- Temporal ordering
- Customer separation
- Label maturity
- Separation between training and validation periods
- Separation between model tuning and outer validation

## Forward Validation Strategy

Three forward validation periods were used.

The following visualization shows the growth of mature training data across the forward validation periods and confirms **zero shared customers within each fold**.

![Leakage-Aware Forward Validation](artifacts/day2_fold_sizes.png)

A bounded **Optuna** search was used for controlled LightGBM hyperparameter tuning.

The tuning process completed:

- **8 attempted trials**
- **8 completed trials**
- Best tuning Average Precision: **0.333269**
- Stopping reason: `TRIAL_LIMIT`

The project also included an intentionally leaky comparison to demonstrate how leakage can produce misleadingly strong performance.

This reinforced the importance of clean forward validation rather than relying only on random validation.

---

# 03 — Cost-Sensitive Decision Policy

Predictive scores alone do not define an operational decision.

The project therefore evaluated model probabilities using a cost-sensitive decision framework.

The simulated educational decision cost was:

**10 × False Negatives + 1 × False Positives**

A False Negative is therefore treated as ten times as costly as a False Positive in this educational scenario.

The decision policy also considers the operational review-capacity constraint:

**Maximum Review Capacity = 12%**

Threshold selection therefore considers both:

- Simulated decision cost
- Operational review capacity

## ROC & Precision-Recall Evidence

ROC and Precision-Recall analysis was used alongside the decision-policy evaluation.

![Day 3 ROC and Precision-Recall Analysis](artifacts/day3_roc_pr.png)

The project also evaluated threshold behavior across time periods and performed descriptive regional diagnostics.

> Regional diagnostics are descriptive only and do not constitute a fairness certification.

---

# 04 — Interpretation & Probability Calibration

Model behavior was examined using **Permutation Importance** and **SHAP-based explanations**.

## Example High-Risk Application

One high-score synthetic application was examined in detail:

**Application ID: `TR-009585`**

For this example:

- Raw model probability: **0.9031**
- Calibrated probability: **0.4795**

Important local SHAP contributions included:

| Feature | Value | SHAP Contribution |
|---|---:|---:|
| `bureau_score` | 497 | +2.27 |
| `dti` | 1.281 | +1.20 |
| `loan_amount_sar` | 93,437.29 | +0.20 |
| `savings_balance_sar` | 6,636.08 | +0.17 |
| `age` | 21 | +0.16 |
| `months_employed` | 17 | −0.09 |

## Local SHAP Explanation

![Local SHAP Explanation for TR-009585](artifacts/shap_waterfall.png)

SHAP values in this visualization are expressed in **raw log-odds**, not probability points.

> **Interpretation is associative, not causal.**

Day 4 explanations are model-specific and should not automatically be treated as explanations of a differently refitted final model.

---

## Probability Calibration

A high raw model score should not automatically be interpreted as a reliable probability.

Probability calibration was therefore evaluated separately.

For the example application:

**Raw Probability: 0.9031**

↓

**Calibrated Probability: 0.4795**

Calibration evidence was evaluated on **1,733 evaluation rows**.

| Metric | Raw | Sigmoid | Change |
|---|---:|---:|---:|
| Brier Score | 0.113027 | 0.067112 | **−0.04592** |
| ECE | 0.146871 | 0.022486 | **−0.12438** |

Both diagnostics decreased after sigmoid calibration.

The negative changes indicate improved probability reliability according to these calibration diagnostics.

---

# 05 — Final Model & Ensemble Decision

Final model selection was based on **nested forward out-of-fold evidence**, rather than on a single random split.

The final comparison used:

**2,155 Forward OOF Predictions**

across:

**3 Forward Validation Periods**

The candidate models included:

### Single Models
- LightGBM
- XGBoost
- Logistic Regression

### Ensemble Candidates
- Equal Ensemble
- Weighted Ensemble
- Stack

---

## Final Model Comparison

| Candidate | Mean AP | Fold SD | Mean Brier | Mean ECE |
|---|---:|---:|---:|---:|
| LightGBM | 0.34549 | 0.04348 | 0.06608 | 0.02311 |
| XGBoost | 0.35263 | 0.02904 | 0.06566 | 0.02276 |
| **Logistic Regression** | **0.39166** | 0.02981 | **0.06327** | 0.01882 |
| Equal Ensemble | 0.37170 | 0.03258 | 0.06435 | 0.02038 |
| Weighted Ensemble | 0.38942 | **0.02906** | 0.06332 | **0.01772** |
| Stack | 0.38314 | 0.02949 | 0.06603 | 0.03106 |

## Final Model & Ensemble Evidence

![Final Model and Ensemble Comparison](artifacts/day5_ensemble_comparison.png)

---

# Ensemble Decision

## KEEP SINGLE — Logistic Regression

The final project retained **Logistic Regression** rather than shipping an ensemble.

Logistic Regression achieved the highest Mean OOF Average Precision:

**Mean AP = 0.39166**

with:

**Fold SD = 0.02981**

The Weighted Ensemble achieved:

**Mean AP = 0.38942**

Its lift relative to the best single model was:

**−0.00224**

None of the ensemble candidates passed the predefined worth-it gate.

Therefore, the additional complexity of an ensemble was not supported by the observed forward OOF evidence.

---

# Final OOF Decision Policy

After model selection, the OOF decision policy was frozen.

The selected raw OOF probability threshold was:

**0.16892**

Across the **2,155 OOF rows**, the selected operating point produced:

| Metric | Result |
|---|---:|
| True Positives | 84 |
| False Positives | 161 |
| False Negatives | 95 |
| True Negatives | 1,815 |
| Flagged | 245 |
| Flag Fraction | 11.37% |
| Recall | 0.4693 |
| Precision | 0.3429 |
| False Positive Rate | 0.0815 |
| Accuracy | 0.8812 |
| Educational Loss | 1,111 units |

The maximum period-level flag fraction was approximately:

**11.75%**

This remained within the configured **12% review-capacity constraint**.

---

# Final Challenge Decision

The frozen final procedure was applied to all:

**2,500 Challenge Applications**

After final model fitting and probability calibration, the calibrated threshold used for the challenge batch was approximately:

**0.12226**

The final decision flow was:

**2,500 Applications → 330 Above Calibrated Threshold → 12% Capacity → 300 Final Flags**

| Final Decision Metric | Result |
|---|---:|
| Challenge applications | 2,500 |
| Above calibrated threshold | 330 |
| Review capacity | 12% |
| Maximum review cases | 300 |
| Removed by capacity | 30 |
| **Final flags** | **300** |

The resulting batch audit confirmed:

**`within_capacity = True`**

The capacity constraint therefore reduced the number of threshold-eligible applications from **330 to 300**.

> Because challenge labels are unavailable, no final challenge ROC-AUC, Average Precision, False Positive Rate, or simulated-loss performance is claimed.

---

# Key Results

| Project Result | Value |
|---|---:|
| Training applications | 10,000 |
| Challenge applications | 2,500 |
| Training positive rate | 7.89% |
| Forward OOF predictions | 2,155 |
| Forward validation periods | 3 |
| **Selected model** | **Logistic Regression** |
| **Final Mean OOF AP** | **0.39166** |
| Fold SD | 0.02981 |
| Raw OOF threshold | 0.16892 |
| Calibrated challenge threshold | ≈ 0.12226 |
| Review capacity | 12% |
| Threshold-eligible challenge cases | 330 |
| **Final flagged applications** | **300** |
| Challenge labels used for training/tuning | **No** |

---

# Why Logistic Regression?

The final model was not selected simply because it was the simplest candidate.

It was retained because the forward OOF evidence showed that it achieved the strongest Mean Average Precision among the evaluated candidates.

The final evidence showed:

- **Highest Mean OOF AP: 0.39166**
- Competitive calibration-quality metrics
- No ensemble demonstrated sufficient incremental value
- Weighted Ensemble did not outperform the best single model
- The simpler final model avoided unsupported additional complexity
- The final decision policy remained within the **12% capacity constraint**

Therefore:

> **KEEP SINGLE — Logistic Regression**

---

# Leakage Prevention

Leakage prevention was treated as a central part of the modeling workflow.

The project preserved:

- **Temporal ordering** — future observations were not used to train models evaluated on earlier periods.
- **Customer separation** — repeated customers were separated across validation boundaries.
- **Label maturity** — only labels available at the relevant decision time were used.
- **Forward validation** — model performance was evaluated on later periods.
- **Separate calibration** — calibration data was kept separate from model-selection evidence.
- **Challenge isolation** — challenge labels were not used for training or tuning.

This design helps reduce optimistic validation results caused by information leakage.

---

# Key Limitations

The project has several important limitations:

- Challenge labels are unavailable, so final predictive performance on the challenge dataset cannot yet be measured.
- The project uses **synthetic educational data** and should not be interpreted as a production credit-risk system.
- Fold-to-fold variability is descriptive and is not a formal statistical significance test.
- Regional diagnostics are descriptive and do not constitute a fairness certification.
- Day 4 explanations are model-specific and should not automatically be interpreted as explanations of a differently refitted final model.
- Calibration results should be interpreted within the evaluation evidence on which they were measured.

---

# Monitoring Plan

If this procedure were evaluated in a controlled future setting, monitoring should include:

### Calibration
Monitor probability reliability using metrics such as Brier Score and ECE when labels become available.

### Drift
Monitor changes in feature distributions, model scores, and predictive behavior.

### Capacity
Monitor threshold-eligible and final review rates against the **12% capacity limit**.

### Regional Diagnostics
Monitor descriptive differences in false-positive rates and recall across regions.

### Model Performance
Reassess model performance when newly labeled data becomes available.

Material drift, capacity violations, or deterioration in calibration should trigger new development evidence rather than silent threshold changes.

---

# Repository Structure

```text
tamweel/
│
├── artifacts/
│   ├── final_model/
│   ├── day1_learning_curves.png
│   ├── day2_fold_sizes.png
│   ├── day3_roc_pr.png
│   ├── shap_waterfall.png
│   └── day5_ensemble_comparison.png
│
├── data/
│
├── notebooks/
│   ├── 00_readiness_check.ipynb
│   ├── 01_baseline_boosting.ipynb
│   ├── 02_validation_tuning.ipynb
│   ├── 03_cost_sensitive_decision.ipynb
│   ├── 04_explain_calibrate.ipynb
│   └── 05_final_model.ipynb
│
├── presentation/
│   └── final_presentation.pdf
│
├── reports/
│   ├── MODEL_CARD.md
│   ├── DECISION_CARD.md
│   ├── INTERPRETABILITY_REPORT.md
│   └── ENSEMBLE_DECISION.md
│
├── scripts/
│
├── submission/
│   └── submission.csv
│
├── tamweel/
│   ├── __init__.py
│   └── inference.py
│
├── README.md
├── PROJECT_README.md
├── constraints.txt
└── requirements-colab.txt
```

---

# Supporting Reports

The project includes four supporting reports.

### Model Card
`reports/MODEL_CARD.md`

Documents model purpose, validation evidence, model selection, calibration, limitations, and intended use.

### Decision Card
`reports/DECISION_CARD.md`

Documents the cost-sensitive and capacity-aware decision policy.

### Interpretability Report
`reports/INTERPRETABILITY_REPORT.md`

Documents model interpretation evidence, including global and local explanations and their limitations.

### Ensemble Decision
`reports/ENSEMBLE_DECISION.md`

Documents the comparison between single models and ensemble candidates and explains why the final workflow retained Logistic Regression.

---

# Project Outputs

The final project produces:

- Verified readiness evidence
- Data-quality evidence
- Model comparison results
- Forward OOF predictions
- Hyperparameter tuning evidence
- Cost-sensitive threshold evidence
- Capacity-aware decision evidence
- SHAP interpretation evidence
- Probability-calibration evidence
- Ensemble comparison evidence
- Final model artifacts
- Supporting reports
- Final presentation
- `submission.csv`
- Reproducible project bundle

---

# Reproducibility

The notebooks should be executed in sequence:

```text
00_readiness_check.ipynb
        ↓
01_baseline_boosting.ipynb
        ↓
02_validation_tuning.ipynb
        ↓
03_cost_sensitive_decision.ipynb
        ↓
04_explain_calibrate.ipynb
        ↓
05_final_model.ipynb
```

---


