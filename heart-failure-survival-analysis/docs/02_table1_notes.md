# Phase 2: Baseline Characteristics (Table 1)

## Objective

Produce a "Table 1" — the standard first table in essentially every
clinical research paper — summarizing patient characteristics at baseline,
stratified by the outcome (`DEATH_EVENT`), with appropriate statistical
tests comparing the two groups. The purpose of a Table 1 is not just
descriptive: it lets a reader judge, at a glance, which baseline factors
already differ between patients who died and patients who survived,
before any modeling is applied.

## Method

Built using the Python `tableone` package (Pollard et al., *Journal of
Open Source Software*), the closest Python equivalent to R's `gtsummary`
for this specific purpose.

```python
table1 = TableOne(
    data=heart_failure_tbl1,
    columns=continuous_columns + categorical_columns,
    categorical=categorical_columns,
    groupby="DEATH_EVENT",
    nonnormal=nonnormal_override,
    pval=True,
    pval_test_name=True,
    overall=True,
    rename=display_labels,
    label_suffix=True,
)
```

### Variable-by-variable test selection, and why

Every variable in a Table 1 needs a deliberate choice of summary
statistic and statistical test — using the same defaults for every
column regardless of its distribution is a common mistake that makes a
table look complete while actually being statistically inappropriate for
some of its rows.

| Variable | Summary | Test | Why |
|---|---|---|---|
| Age | Mean (SD) | Welch's t-test | Approximately normally distributed in the literature for this cohort. Welch's version (not Student's) was used because it does not assume the two groups have equal variance — a safer default when sample sizes differ (96 vs. 203) and there is no prior reason to assume equal spread. |
| Ejection Fraction | Mean (SD) | Welch's t-test | Same reasoning as age — approximately normal. |
| Serum Sodium | Mean (SD) | Welch's t-test | Same reasoning. |
| Creatinine Phosphokinase | Median [Q1, Q3] | Kruskal-Wallis | This enzyme is released during muscle damage and its distribution is strongly right-skewed (a small number of patients have very high values). A mean would be pulled upward by these outliers and would misrepresent a "typical" patient; the median is robust to this. Kruskal-Wallis is the appropriate rank-based test for comparing two skewed distributions (with two groups, it is mathematically equivalent to the Mann-Whitney U test). |
| Platelets | Median [Q1, Q3] | Kruskal-Wallis | Same skewness reasoning. |
| Serum Creatinine | Median [Q1, Q3] | Kruskal-Wallis | Same skewness reasoning — kidney function markers are typically right-skewed. |
| Follow-up Time | Median [Q1, Q3] | Kruskal-Wallis | Follow-up time is mechanically skewed by censoring (see "Clinical Interpretation" below). |
| Anaemia, Diabetes, Hypertension, Sex, Smoking | n (%) | Chi-squared | Standard test for comparing proportions between two groups on a categorical variable. `tableone` automatically substitutes Fisher's exact test instead, for any 2×2 comparison where an expected cell count falls below 5 — this did not trigger for any variable here, meaning every categorical comparison had large enough group sizes for the chi-squared approximation to be considered reliable. |

## Result

| Characteristic | Overall (N=299) | Deceased (n=96) | Survived (n=203) | p-value | Test |
|---|---|---|---|---|---|
| Age (years), mean (SD) | 60.8 (11.9) | 65.2 (13.2) | 58.8 (10.6) | **<0.001** | Welch's t-test |
| Creatine Phosphokinase, median [IQR] | 250.0 [116.5, 582.0] | 259.0 [128.8, 582.0] | 245.0 [109.0, 582.0] | 0.684 | Kruskal-Wallis |
| Ejection Fraction (%), mean (SD) | 38.1 (11.8) | 33.5 (12.5) | 40.3 (10.9) | **<0.001** | Welch's t-test |
| Platelets, median [IQR] | 262000 [212500, 303500] | 258500 [197500, 311000] | 263000 [219500, 302000] | 0.425 | Kruskal-Wallis |
| Serum Creatinine (mg/dL), median [IQR] | 1.1 [0.9, 1.4] | 1.3 [1.1, 1.9] | 1.0 [0.9, 1.2] | **<0.001** | Kruskal-Wallis |
| Serum Sodium (mEq/L), mean (SD) | 136.6 (4.4) | 135.4 (5.0) | 137.2 (4.0) | **0.002** | Welch's t-test |
| Follow-up Time (days), median [IQR] | 115.0 [73.0, 203.0] | 44.5 [25.5, 102.2] | 172.0 [95.0, 213.0] | **<0.001** | Kruskal-Wallis |
| Anaemia, n (%) Yes | 129 (43.1) | 46 (47.9) | 83 (40.9) | 0.307 | Chi-squared |
| Diabetes Mellitus, n (%) Yes | 125 (41.8) | 40 (41.7) | 85 (41.9) | 1.000 | Chi-squared |
| Hypertension, n (%) Yes | 105 (35.1) | 39 (40.6) | 66 (32.5) | 0.214 | Chi-squared |
| Sex, n (%) Male | 194 (64.9) | 62 (64.6) | 132 (65.0) | 1.000 | Chi-squared |
| Current Smoker, n (%) Yes | 96 (32.1) | 30 (31.2) | 66 (32.5) | 0.932 | Chi-squared |

## Clinical Interpretation

**Significant at baseline (p < 0.05):** age, ejection fraction, serum
creatinine, serum sodium, and follow-up time.

- **Age** — older patients died at a higher rate, consistent with age
  being the single most reliable non-modifiable prognostic factor across
  essentially all cardiovascular disease.
- **Ejection fraction** — the percentage of blood the left ventricle
  pumps out per heartbeat. The Deceased group's ventricles were pumping
  out substantially less blood per beat (33.5% vs. 40.3%), meaning their
  hearts were mechanically weaker at baseline — this is one of the two
  strongest known predictors of heart failure mortality in the published
  literature on this exact dataset.
- **Serum creatinine** — a marker of kidney filtration function. Higher
  values mean the kidneys are clearing waste less efficiently. Its
  association with mortality reflects **cardiorenal syndrome**: the heart
  and kidneys are physiologically interdependent, and a failing heart
  reduces blood flow to the kidneys, which in turn worsens fluid overload
  and further strains the heart — a feedback loop that shows up
  statistically as this variable's strong association with death.
- **Serum sodium** — lower sodium (hyponatremia) in the Deceased group
  reflects overactivation of the body's fluid-regulating hormone systems
  (the renin-angiotensin-aldosterone and antidiuretic hormone pathways),
  which is a recognized marker of more advanced, decompensated heart
  failure.
- **Follow-up time** — this difference requires a different kind of
  interpretation than the others. It is **not** a baseline patient
  characteristic in the same sense as age or creatinine; it is
  mechanically shorter in the Deceased group because, by definition, a
  patient who dies during the study stops being followed at that point,
  while a patient who survives continues to be observed until the study
  ends. This is called **informative censoring**, and its presence here
  is expected and correct, not a data quality issue — it will be handled
  properly by the survival analysis methods in Phases 3 and 4, which are
  specifically designed to account for exactly this kind of
  time-to-event structure.

**Not significant at baseline:** anaemia, diabetes, hypertension, sex,
and smoking status. This means that, considered on their own, these five
characteristics were similarly distributed between patients who lived and
died — none of them, by themselves, distinguished the two outcome groups
in this cohort. This does not necessarily mean these variables are
clinically irrelevant to heart failure in general; it means that within
this specific 299-patient sample, their raw (unadjusted) association with
the outcome did not reach statistical significance. Notably, hypertension
re-appears in Phase 3 as significant once follow-up **time** (not just
the binary DEATH_EVENT) is taken into account via survival analysis — a
useful illustration of how Table 1 and survival analysis can surface
different signals from the same variable, because they are answering
subtly different statistical questions.

## Issues Encountered and Resolved

**Bug — `DEATH_EVENT` stratified against itself.** In an early draft of
this table, `DEATH_EVENT` was mistakenly included both as the `groupby`
variable and as a row inside the table itself. This produced a nonsensical
result (100% vs. 0% in each group, p < 0.001) because the outcome was
being statistically compared to itself. This was caught during review and
fixed by explicitly excluding `DEATH_EVENT` from the `columns` list passed
to `TableOne`, since it belongs only in the `groupby` argument. This is
recorded here deliberately: catching and correcting a logical error before
publishing a result is itself part of a rigorous analysis, and is more
credible in a portfolio than a repo that shows no evidence of any mistake
ever having been made.
