# Phase D: Multivariable Cox Proportional Hazards Model

## Objective

Fit a multivariable Cox Proportional Hazards model using age, hypertension
status, smoking status, and serum creatinine to predict the hazard of
death over follow-up time, and determine whether each variable's
association with survival holds up once the others are statistically
accounted for.

## Why a Multivariable Model, After Already Having Kaplan-Meier

Phase 3 answered a narrower question: does hypertension, considered on
its own, appear to affect survival? Phase 4 answers a more clinically
useful question: does hypertension's apparent effect on survival remain
once we control for other variables that might explain it instead —
specifically, age and kidney function (serum creatinine), plus smoking
status as an additional modifiable risk factor.

This distinction matters because two variables can be correlated with
each other in a dataset purely by chance of sampling, or because one
genuinely causes changes in the other's typical values (for instance,
hypertension becomes more prevalent with age). If hypertensive patients
in this cohort simply tend to be older, then some of what looked like a
"hypertension effect" in Phase 3 could actually be an age effect showing
through indirectly. A multivariable model is the standard statistical
tool for disentangling this.

## Method

```python
cox_ph_model = CoxPHFitter()
cox_ph_model.fit(
    df=cox_model_data,
    duration_col="time",
    event_col="DEATH_EVENT",
)
```

### Variable Selection Rationale

Variables were specified in advance, on clinical grounds, rather than
chosen by a data-driven selection procedure (such as stepwise selection)
— this is standard practice for a confirmatory analysis, where the goal
is to test pre-specified clinical hypotheses rather than search the data
for whatever combination of variables happens to produce the smallest
p-values.

- **Age**: the strongest, most consistently reported non-modifiable
  mortality risk factor in heart failure and cardiovascular disease more
  broadly.
- **Hypertension**: the variable under direct investigation in this
  analysis, carried forward from Phase 3.
- **Smoking**: a modifiable cardiovascular risk factor, included to see
  whether it carries independent prognostic weight once age and
  hypertension are already in the model.
- **Serum creatinine**: a kidney function marker, included because of its
  strong, well-documented link to heart failure mortality via
  cardiorenal syndrome (the interdependence between heart and kidney
  function described in Phase 2's notes).

### What a Hazard Ratio Actually Means

The Cox model does not estimate a fixed probability of death; it
estimates each covariate's effect on the **instantaneous risk (hazard) of
death at any given moment during follow-up**, relative to how that risk
would look for a patient with a lower (or reference) value of that
covariate, holding every other covariate in the model constant.

- **HR = 1** means no association with risk.
- **HR > 1** means an increased hazard — the risk of death at any given
  moment is higher for a one-unit increase in that variable (or for
  the "yes" category compared to "no," for binary variables), all else
  held equal.
- **HR < 1** means a decreased (protective) hazard.

The **95% confidence interval** expresses the range of HR values
consistent with the data; if this interval includes 1.0, the association
is not considered statistically significant at the conventional
threshold, since a true HR of exactly 1 (no effect at all) cannot be
ruled out.

## Result

| Variable | HR | 95% CI | p-value |
|---|---|---|---|
| Age (per 1-year increase) | 1.04 | 1.03 – 1.06 | **<0.001** |
| Hypertension (Yes vs. No) | 1.47 | 0.97 – 2.23 | 0.067 |
| Current Smoker (Yes vs. No) | 1.15 | 0.74 – 1.77 | 0.540 |
| Serum Creatinine (per 1 mg/dL increase) | 1.33 | 1.19 – 1.49 | **<0.001** |

- **Concordance Index (C-index): 0.675**
- **Log-Likelihood Ratio Test: p < 0.0001**

## Clinical Interpretation

**Age** — each additional year of age is associated with a 4% increase
in the instantaneous hazard of death (HR = 1.04), independent of
hypertension, smoking, and kidney function. Over a clinically meaningful
age gap (for example, 10 years), this compounds substantially, consistent
with age's well-established role as a primary driver of heart failure
mortality.

**Serum creatinine** — each 1 mg/dL increase in serum creatinine is
associated with a 33% increase in hazard (HR = 1.33), independent of age,
hypertension, and smoking. This is the single strongest modifiable
laboratory predictor in this model, and reinforces the cardiorenal
syndrome interpretation raised in Phase 2: declining kidney function
appears to carry substantial independent prognostic weight in heart
failure, beyond what age alone would predict.

**Hypertension — the central finding of this analysis.** In the
unadjusted Kaplan-Meier comparison (Phase 3), hypertension was
statistically significant (log-rank p = 0.036). Here, in the
multivariable model, its association weakens to **non-significance**
(HR = 1.47, 95% CI 0.97–2.23, p = 0.067). The confidence interval crosses
1.0, meaning a true "no effect" cannot be ruled out at the conventional
threshold.

This is not a contradiction between the two analyses — it is exactly
the kind of result multivariable adjustment is designed to reveal, and it
is arguably the single most instructive finding in this entire project.
The most likely explanation is **confounding**: hypertensive patients in
this cohort likely skew older on average than non-hypertensive patients,
and once age is explicitly accounted for in the model, much of what
appeared to be "hypertension's effect" in the univariate comparison is
better explained as age's effect operating indirectly through the
hypertension variable. (This can be directly checked with a simple
`heart_failure_data.groupby("high_blood_pressure")["age"].mean()`
comparison on the underlying data.)

**Smoking** showed no meaningful independent association with mortality
in this cohort (HR = 1.15, p = 0.540) once the other three variables were
accounted for.

## Model Fit and Overall Validity

**Concordance Index (C-index) = 0.675.** This measures the model's
ability to correctly rank which of two randomly chosen patients died
first, based on their predicted risk scores. A value of 0.5 indicates no
better performance than random chance; 1.0 would indicate perfect
discrimination. A value of 0.675 represents moderate, genuine
discriminative ability — comparable to many published clinical
prognostic scores in cardiology, though clearly imperfect, meaning
individual-level predictions from this model should be interpreted with
appropriate caution.

**Log-Likelihood Ratio Test, p < 0.0001.** This tests whether the four
covariates, considered jointly, explain meaningfully more of the observed
survival variation than a model with no covariates at all. This is a
different question from each covariate's individual p-value: the
individual p-values above test each variable's own marginal contribution
after the other three are already included in the model, while this test
evaluates the combined explanatory contribution of all four together. The
model passes this joint test overwhelmingly, even though two of its four
individual variables (hypertension, smoking) did not reach individual
significance — this is possible, and not contradictory, because age and
serum creatinine alone are contributing enough signal to make the whole
model significant.

## Event-Per-Variable Consideration

With 96 observed deaths and 4 covariates in the model, this analysis has
approximately 24 events per covariate — comfortably above the commonly
cited rule-of-thumb minimum of roughly 10 events per covariate for a
reasonably stable Cox model. This was checked deliberately rather than
assumed, since an underpowered model (too few events relative to the
number of covariates) can produce unstable or overfit hazard ratio
estimates.

## Formatting Note

The Log-Likelihood Ratio Test p-value was returned by the software as
`0.0000`, which is a display artifact of floating-point rounding, not a
literal p-value of exactly zero. This is reported as **p < 0.0001** in
this document, following standard clinical reporting convention (the same
convention used for the `<0.001` values in the individual covariate
table above).
