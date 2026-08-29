# Procedure 02: Statistical Methodology & Selection Notes

This document details the statistical rationale behind the tests applied to generate the baseline characteristics table.

## Methodological Framework

### 1. Continuous Variables (Normally Distributed)
* **Variables:** Age (years), Ejection Fraction (%), Serum Sodium (mEq/L).
* **Metric Displayed:** Mean (Standard Deviation).
* **Statistical Test:** **Welch's T-test**.
* **Rationale:** Used to compare the means of two independent groups when the data follows a continuous normal distribution. Welch's variant is preferred over Student's T-test as it handles unequal variances safely.

### 2. Continuous/Ordinal Variables (Skewed Distributions)
* **Variables:** Creatine Phosphokinase (mcg/L), Platelets (kiloplatelets/mL), Serum Creatinine (mg/dL), Follow-up Time (days).
* **Metric Displayed:** Median [Quarter 1 (25th percentile), Quarter 3 (75th percentile)].
* **Statistical Test:** **Kruskal-Wallis Test** (Non-parametric).
* **Rationale:** Medical metrics like Creatine Phosphokinase and Creatinine often contain extreme outliers or highly skewed distributions. The Kruskal-Wallis test evaluates differences in medians without assuming normality.

### 3. Categorical Variables
* **Variables:** Anaemia, Diabetes Mellitus, Hypertension, Sex, Current Smoker, DEATH_EVENT.
* **Metric Displayed:** Count, $n$ (Percentage, %).
* **Statistical Test:** **Pearson's Chi-squared Test**.
* **Rationale:** Evaluates whether there is a significant association between two categorical variables (e.g., whether smoking distribution differs significantly between survival outcomes).
