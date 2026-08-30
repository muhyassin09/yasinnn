# 02: Kaplan-Meier Survival Analysis - Clinical Summary

## Objective
To estimate and compare the cumulative survival probabilities of heart failure patients over time, stratified by their baseline hypertension status (`Hypertension: Yes` vs. `Hypertension: No`).

## Key Clinical Findings

### 1. Curve Separation and Patient Progression
* The red curve (`Hypertension: Yes`) drops below the blue curve (`Hypertension: No`) early in the study timeline (around day 25).
* This negative separation remains consistent throughout the 250+ day follow-up window.
* **Clinical Insight:** Patients entering the study with high blood pressure face a consistently higher velocity of clinical decline and mortality risk compared to those with normal blood pressure.

### 2. Median Survival Time Status: "Not Reached" (NR)
* **Hypertension: Yes Strata:** Not Reached (The curve terminates at ~0.53 survival probability)
* **Hypertension: No Strata:** Not Reached (The curve terminates at ~0.61 survival probability)
* **Clinical Meaning:** "Not Reached" is standard medical nomenclature indicating that more than half of the patients in both groups survived past the final recorded follow-up day (285 days). The cohort's overall short-term survival rate is high enough that the curves never dip down to the 50% marker.

### 3. Study Limitations and Confounding Risk
Because this curve evaluates hypertension in isolation, it is an **unadjusted analysis**. We cannot confirm if hypertension itself is causing the faster death rate, or if the hypertensive patients happen to be older or have weaker kidneys. This descriptive phase serves as the clinical justification to advance to multivariate modeling.
