# Phase 3: Kaplan-Meier Survival Analysis

## Objective

Estimate and compare survival probability over time between hypertensive
and non-hypertensive heart failure patients, using the Kaplan-Meier
estimator and a log-rank significance test, and visualize the result in a
publication-standard survival curve.

## Why Kaplan-Meier, and Not a Simple Proportion Comparison

Table 1 already showed that hypertension status was **not** significantly
associated with the binary `DEATH_EVENT` outcome alone (p = 0.214). But a
simple yes/no comparison like that throws away one of the most important
pieces of information in this dataset: **when** each event happened, and
**how long** each patient was actually observed for.

Kaplan-Meier analysis is specifically designed for exactly this kind of
"time-to-event" data, and it correctly handles a problem that a simple
proportion comparison cannot: **right-censoring**. A patient who was still
alive when the study ended is not the same as a patient known to have
survived indefinitely — they are a patient we simply stopped observing.
Kaplan-Meier uses each such patient's data up until the point they were
last observed, without incorrectly treating them as either "dead" or
"guaranteed to live forever."

## Method

```python
kmf_hypertension = KaplanMeierFitter(label="Hypertension: Yes")
kmf_no_hypertension = KaplanMeierFitter(label="Hypertension: No")

kmf_hypertension.fit(
    durations=hypertension_group["time"],
    event_observed=hypertension_group["DEATH_EVENT"]
)
kmf_no_hypertension.fit(
    durations=no_hypertension_group["time"],
    event_observed=no_hypertension_group["DEATH_EVENT"]
)

logrank_result = logrank_test(
    durations_A=hypertension_group["time"],
    durations_B=no_hypertension_group["time"],
    event_observed_A=hypertension_group["DEATH_EVENT"],
    event_observed_B=no_hypertension_group["DEATH_EVENT"]
)
```

### How the Kaplan-Meier estimator actually works, conceptually

At every timepoint where at least one death occurs, the estimator
recalculates the probability of having survived up to that point, using
only patients who were still under observation immediately beforehand.
Each of these "conditional survival probabilities" is multiplied together
as time progresses, producing the step-shaped curve seen in the plot —
each step down corresponds to one or more deaths at that exact follow-up
day, and the height of the curve at any point represents the estimated
probability of a patient from that group surviving at least that long.

### What the log-rank test is actually testing

The log-rank test compares the two curves across their **entire**
observed time range, not just at one cutoff point. Its null hypothesis is
that the two groups (hypertensive vs. non-hypertensive) are being drawn
from survival distributions that are, underneath the noise of a finite
sample, identical. A small p-value means the observed separation between
the two curves is unlikely to have occurred by chance alone if that null
hypothesis were true.

## Result

- **Log-rank test: p = 0.036** — statistically significant at the
  conventional α = 0.05 threshold.
- Median survival time: **not reached** in either group (neither curve's
  survival probability dropped to 50% within the 285-day follow-up
  window).
- At the end of follow-up, the Kaplan-Meier estimated survival
  probability was approximately **0.53** for hypertensive patients versus
  approximately **0.61** for non-hypertensive patients.
- Risk table: the hypertensive group shrank from 105 patients at risk at
  day 0 to 1 patient still at risk by the final follow-up interval; the
  non-hypertensive group shrank from 194 to 10 over the same period.

## Clinical Interpretation

Patients with hypertension in this cohort showed a consistently lower
survival probability throughout the follow-up period compared to patients
without hypertension, and this separation was large enough, given the
sample size, to be statistically significant (p = 0.036). This is
mechanistically plausible: chronic hypertension places sustained pressure
overload on the left ventricle, which over time drives structural
remodeling of heart muscle (hypertrophy and fibrosis) that can worsen the
prognosis of a heart that is already failing.

**"Median survival not reached" is a result, not a missing value.** It
means that more than half of the patients in both groups were still
alive at the point the study's observation period ended — the curve
never fell far enough for a median to be mathematically defined. This
same phrase, "not reached," is standard terminology in real clinical
trial reporting (used constantly in oncology and cardiology survival
tables) and should never be interpreted as an error or a gap in the data.

**A crucial caveat, carried forward into Phase 4:** this result is
**unadjusted**. The log-rank test and Kaplan-Meier curve compare
hypertensive and non-hypertensive patients as two raw groups, without
accounting for the possibility that these two groups also differ in
other ways that independently affect survival — for example, if
hypertensive patients in this cohort also tend to be older on average,
some or all of this apparent hypertension effect could actually be an age
effect appearing indirectly through the hypertension variable. This
exact question — does hypertension's association with survival hold up
once age and kidney function are statistically controlled for — is what
the multivariable Cox Proportional Hazards model in Phase 4 was built to
answer.

## Issues Encountered and Resolved

**Bug — `inf` not caught by the missing-value check.** The initial script
checked whether median survival time was undefined using `pd.isna()`,
which only detects `NaN` values. In `lifelines`, a curve that never
reaches 50% survival returns `np.inf` (infinity) instead of `NaN` for its
median survival time — so the original check let this value through
unflagged, and it printed as a nonsensical raw number ("inf days") instead
of the correct, human-readable "Not reached." This was corrected by
switching the check to `np.isinf()`. This is a useful lesson for future
survival analyses: always confirm which specific missing/undefined value
a library uses in a given edge case, rather than assuming it follows the
same convention (`NaN`) used elsewhere.
