# Heart Failure Survival Analysis: A Clinical Biostatistics Portfolio Project

## Overview

This project performs a complete survival analysis on the **UCI Heart Failure
Clinical Records dataset** (Chicco & Jurman, 2020, *BMC Medical Informatics
and Decision Making*), a cohort of 299 patients followed after a heart
failure diagnosis. The analysis follows a standard clinical-research
workflow: descriptive baseline characteristics, univariate survival
comparison, multivariable adjustment, and formal model diagnostics.

- **Dataset**: [UCI ML Repository, Dataset #519](https://archive.ics.uci.edu/dataset/519/heart+failure+clinical+records)
- **N = 299** patients, **96 deaths (32.1%)** observed during follow-up
- **Tools**: Python (`pandas`, `tableone`, `lifelines`)
- **Structure**: each analysis phase has its own notebook (`notebooks/`) and
  a companion notes file (`docs/`) documenting the statistical reasoning,
  clinical interpretation, and any issues encountered along the way.

---

## Key Findings

### 1. Baseline Characteristics (Table 1)

Patients who died during follow-up were, on average, **older** (65.2 vs.
58.8 years, p < 0.001), had **lower ejection fraction** (33.5% vs. 40.3%,
p < 0.001), **higher serum creatinine** (median 1.3 vs. 1.0 mg/dL,
p < 0.001), and **lower serum sodium** (135.4 vs. 137.2 mEq/L, p = 0.002)
than those who survived. Comorbidities commonly assumed to be prognostic —
anaemia, diabetes, hypertension, sex, and smoking status — did **not**
differ significantly between groups at baseline in this cohort.

### 2. Kaplan-Meier Survival Analysis (by Hypertension Status)

Unadjusted survival curves showed a statistically significant difference
between hypertensive and non-hypertensive patients (**log-rank p = 0.036**),
with hypertensive patients showing consistently lower survival probability
throughout follow-up. Median survival was **not reached** in either group,
meaning more than half of patients in both strata survived the full
285-day observation window.

### 3. Multivariable Cox Proportional Hazards Model

Adjusting for age, hypertension, smoking, and serum creatinine
simultaneously:

| Variable | HR | 95% CI | p-value |
|---|---|---|---|
| Age (per 1-year increase) | 1.04 | 1.03 – 1.06 | <0.001 |
| Hypertension (Yes vs. No) | 1.47 | 0.97 – 2.23 | 0.067 |
| Current Smoker (Yes vs. No) | 1.15 | 0.74 – 1.77 | 0.540 |
| Serum Creatinine (per 1 mg/dL increase) | 1.33 | 1.19 – 1.49 | <0.001 |

- **Model discrimination**: C-index = 0.675 (moderate)
- **Overall model significance**: Log-likelihood ratio test, p < 0.0001

**The central finding of this project**: hypertension's association with
mortality, significant in the univariate Kaplan-Meier comparison (p =
0.036), **weakens to non-significance (p = 0.067) once adjusted for age and
renal function**. This is a textbook example of confounding — hypertensive
patients in this cohort tend to be older, and age is doing much of the
explanatory work that appeared to belong to hypertension in the unadjusted
comparison. Age and serum creatinine emerge as the strongest independent
predictors of mortality, consistent with the clinical concept of
**cardiorenal syndrome**, in which declining kidney function and heart
failure progression are mechanistically linked.

### 4. Proportional Hazards Assumption Check (Schoenfeld Residuals)

All four covariates satisfied the proportional hazards assumption
(all p > 0.05 via the Schoenfeld residuals test), confirming that each
variable's hazard ratio can be validly interpreted as constant across the
full follow-up period.

| Variable | Test statistic | p-value |
|---|---|---|
| Age | 0.29 | 0.589 |
| Hypertension | 0.53 | 0.465 |
| Serum Creatinine | 0.25 | 0.618 |
| Smoking | 0.30 | 0.586 |

---

## Repository Structure

```
heart-failure-portfolio/
├── README.md                              ← you're here
├── data/
│   └── heart_failure_clinical_records_dataset.csv
├── analysis/
│   └── heart_failure_survival_analysis.ipynb
│       
├── docs/
│   ├── 01_data_loading_notes.md
│   ├── 02_table1_notes.md
│   ├── 03_km_notes.md
│   ├── 04_cox_model_notes.md
│   └── 05_assumption_check_notes.md
└── outputs/
    ├── table1_baseline_characteristics.csv
    ├── km_curve_hypertension.png
    └── cox_ph_hazard_ratios.csv
```

---

## Methodological Notes

- **Statistical tests** in Table 1 were chosen per-variable based on
  distributional shape: Welch's t-test for approximately Normal continuous
  variables (age, ejection fraction, serum sodium), Kruskal-Wallis for
  right-skewed labs (creatinine phosphokinase, platelets, serum creatinine,
  follow-up time), and Chi-squared for categorical comparisons.
- **Kaplan-Meier and log-rank results are unadjusted** — they describe a
  single variable's raw association with survival, without controlling for
  confounders. The Cox model in Phase 4 addresses this directly.
- **Cox model variable selection** (age, hypertension, smoking, serum
  creatinine) was specified a priori on clinical grounds rather than
  data-driven selection, consistent with standard practice in confirmatory
  clinical analyses.
- **Event-per-variable ratio**: with 96 events and 4 covariates (24
  events/variable), the model is comfortably powered above the common
  rule-of-thumb minimum of ~10 events per covariate.

## Limitations

- Single-center, retrospective cohort (per the original data source); findings
  are descriptive of this dataset and not intended for clinical
  generalization.
- The Cox model includes only four pre-specified covariates; it does not
  represent an exhaustive multivariable risk model (ejection fraction and
  serum sodium, both significant in Table 1, were intentionally excluded
  to keep this phase focused on the four specified variables).
- No internal validation (e.g., bootstrapping, train/test split) was
  performed on the Cox model's discrimination estimate; the reported
  C-index reflects in-sample performance.

## References

Chicco, D., & Jurman, G. (2020). Machine learning can predict survival of
patients with heart failure from serum creatinine and ejection fraction
alone. *BMC Medical Informatics and Decision Making, 20*(16).
