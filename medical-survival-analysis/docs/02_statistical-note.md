# 02: Kaplan-Meier & Log-Rank Test - Statistical Methodology

This document details the mathematical rationale and sample dynamics behind the Phase 3 survival visualization.

## Statistical Framework

### 1. The Log-Rank Test Mechanics
* **Test Formula Metric:** $\text{Log-rank test } p = 0.036$
* **Decision Rule:** Since $p < 0.05$, we reject the null hypothesis ($H_0$). 
* **Statistical Rationale:** The Log-rank test is a non-parametric test that compares the entire survival experience of two independent groups. It calculates the expected number of events (deaths) at every unique time point under the assumption that survival is identical, comparing it mathematically to the observed events.

### 2. Sample Size Degradation and Variance (The Risk Table)
The "Number at Risk" data directly explains why the shaded 95% Confidence Interval (CI) ribbons grow wider as time passes:
* **Initial State (Day 0):** Both groups have high sample sizes ($n = 105$ and $n = 194$), resulting in narrow, tight confidence bands.
* **Late State (Day 200+):** The number of patients actively being observed drops dramatically ($n = 14$ for Hypertension, $n = 63$ for Non-Hypertension) due to patients either passing away or reaching the end of their tracking period (censoring).
* **Statistical Warning:** When sample sizes drop this low, variance skyrockets. The far-right tail of the curve is driven by very few individuals, making the statistical estimates noisy and less reliable than the early segments.
