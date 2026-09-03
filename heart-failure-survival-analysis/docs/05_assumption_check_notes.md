# Phase E: Proportional Hazards Assumption Check (Schoenfeld Residuals)

## Objective

Formally test whether the Cox Proportional Hazards model fit in Phase 4
satisfies its core statistical assumption — that each covariate's hazard
ratio remains constant across the entire follow-up period — using the
Schoenfeld residuals test.

## Why This Check Cannot Be Skipped

Every hazard ratio reported in Phase 4 carries an implicit assumption:
that the effect of that variable on risk does not change at different
points in time. For example, the model assumes that hypertension's effect
on hazard is the same on day 10 of follow-up as it is on day 200. This is
called the **proportional hazards assumption**, and it is not automatically
true just because a Cox model can technically be fit to any dataset — it
is a testable claim, and if it is violated for a given variable, that
variable's single reported HR would be a misleading average that doesn't
actually describe its true, time-varying effect on risk at any specific
point in time.

## Method

The Schoenfeld residuals test was run using `lifelines`' formal
statistical test, matching R's `survival::cox.zph()` function.

```python
from lifelines.statistics import proportional_hazard_test

schoenfeld_test_results = proportional_hazard_test(
    cox_ph_model, cox_model_data, time_transform="rank"
)
print(schoenfeld_test_results.summary)
```

### How the Test Actually Works, Conceptually

For each covariate in the model, the test examines the residuals (the
difference between a patient's observed and model-predicted contribution
to risk) at every event time, and checks whether those residuals show any
systematic trend across time. If a covariate's true effect on hazard were
genuinely constant, its residuals should show no such trend — they should
look like unstructured noise scattered around zero regardless of whether
you look early or late in the follow-up period. If a trend is present, it
indicates the covariate's effect on hazard is actually changing over
time, which violates the assumption the Cox model depends on.

### Reading the Result: Why a Large p-value Is the Good Outcome Here

This is worth stating explicitly, because it runs opposite to the
intuition built up in every other test in this project. In Table 1 and
the Cox model's own covariate table, a **small** p-value was the
interesting, "something is going on" result. Here, it is the reverse: a
**small** p-value (p < 0.05) would indicate a significant violation of
the assumption — evidence that the covariate's effect is changing over
time, which is the undesired outcome. A **large** p-value (p > 0.05)
means no such trend was detected, supporting the validity of that
variable's reported hazard ratio as a genuinely constant effect
throughout follow-up.

## Result

| Variable | Test statistic | p-value |
|---|---|---|
| Age | 0.29 | 0.589 |
| Hypertension | 0.53 | 0.465 |
| Serum Creatinine | 0.25 | 0.618 |
| Smoking | 0.30 | 0.586 |

All four p-values are comfortably above the 0.05 threshold. The
software's built-in plain-language verdict independently confirmed this
same conclusion:

```
Proportional hazard assumption looks okay.
[]
```

(The empty list here indicates zero covariates were flagged as
violations — an empty result is the correct and expected output when
every variable passes.)

## Clinical and Statistical Interpretation

None of the four covariates in the Phase 4 model showed evidence of a
time-varying effect on the hazard of death. This means:

- The single hazard ratio reported for each variable in Phase 4 (for
  example, HR = 1.33 for serum creatinine) can be validly interpreted as
  a genuinely constant effect across the entire 285-day follow-up window,
  rather than an average masking meaningfully different effects at
  different times.
- No further modeling adjustments (such as adding a time-interaction term
  for a specific covariate, or stratifying the model by that covariate)
  are statistically necessary based on this diagnostic.
- This result strengthens, rather than merely supplements, the
  confidence placed in the Phase 4 findings — particularly the central
  finding that hypertension's association with mortality did not survive
  adjustment for age and kidney function. Since hypertension itself
  passed this check (p = 0.465), that non-significant result is not an
  artifact of a broken model assumption; it reflects a genuinely weak
  association even under a model whose underlying assumptions are valid.

## Issues Encountered

None. All four covariates passed on the first assumption check, requiring
no remedial modeling steps (such as stratification or the addition of
time-varying covariate terms). This clean result is recorded in full
detail here, including the exact per-covariate test statistics, rather
than reported only as a pass/fail verdict.
