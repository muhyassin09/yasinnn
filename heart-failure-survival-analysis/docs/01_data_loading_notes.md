# Phase A: Data Loading & Validation

## Objective

Load the UCI Heart Failure Clinical Records dataset into the analysis
environment and confirm — rather than assume — that the file is complete,
correctly typed, and free of data-quality problems before any statistical
work is built on top of it. In clinical research, this step is not a
formality: every downstream table, curve, and model in this project
inherits whatever errors slip through here undetected.

## Data Source

- **Repository**: UCI Machine Learning Repository, Dataset #519
- **Citation**: Chicco, D., & Jurman, G. (2020). Machine learning can
  predict survival of patients with heart failure from serum creatinine
  and ejection fraction alone. *BMC Medical Informatics and Decision
  Making, 20*(16).
- **Expected size**: 299 patient records, 13 columns (12 clinical
  variables + 1 outcome)
- **Access method used**: local CSV, loaded directly with
  `pandas.read_csv()`

## Method

```python
heart_failure_data = pd.read_csv("heart_failure_clinical_records_dataset.csv")
heart_failure_data = heart_failure_data[expected_columns].apply(pd.to_numeric)
```

### Step-by-step reasoning

1. **`read_csv()`** — loads the raw file directly, since it is manually
   downloaded from UCI and uploaded into the Colab session beforehand.
   An earlier draft of this script included a three-tier fallback (local
   file → `ucimlrepo` package → direct zip download) to handle the case
   where no file was present locally. That logic was deliberately removed:
   it added real complexity for a scenario that does not occur in this
   project's actual workflow, since the file is always provided manually.
   Simpler, direct code that matches how the script is actually used is
   preferable to defensive code written for a hypothetical case.

2. **`.apply(pd.to_numeric)`** — explicitly coerces every loaded column to
   numeric dtype. This matters because `pandas` sometimes infers a column
   as `object` (essentially: "text") if even one value in it looks
   unusual — and an `object`-typed numeric column will silently break
   later statistical functions (`tableone`, `lifelines`) with a confusing
   error, rather than an obvious one. Coercing explicitly, right after
   loading, converts a possible late-stage mystery bug into an early and
   clear one.

**Note on schema validation:** an earlier draft of this script also
included an explicit column-schema assertion (checking that all 13
expected columns were present before proceeding). That check was
deliberately removed as well, for the same reason as the ingestion
fallback logic above: it exists to protect against a file with the wrong
or missing columns, which is not a realistic scenario here — the source
file is fixed, downloaded once from a known location, and already
visually verified. Keeping the check would have been defensive
programming for a risk that does not actually apply to this project's
actual, one-person, one-file workflow.

### Additional checks performed (recommended and applied)

Beyond the two steps above, three inexpensive checks were added because
they catch the specific failure modes most likely to occur with any
externally sourced clinical dataset:

```python
# 1. Missing values — confirm zero, don't assume zero
print(heart_failure_data.isnull().sum())

# 2. Row count and duplicate check
print(f"Rows: {len(heart_failure_data)} | Duplicates: {heart_failure_data.duplicated().sum()}")

# 3. Range / plausibility check on every numeric column
heart_failure_data.describe()
```

**Why each of these matters, in plain terms:**

- *Missing values*: a dataset can have zero missing values by luck of
  the specific file you downloaded, or it can have zero missing values
  because the variable was actually collected completely for every
  patient. Only the second is something you should claim in a report —
  and you can only claim it if you checked.
- *Duplicate check*: a duplicated patient row is one of the most common
  and dangerous silent errors in clinical datasets — it doesn't just add
  noise, it can artificially inflate statistical significance by
  effectively double-counting one patient's outcome.
- *Range / plausibility check*: `.describe()` surfaces implausible values
  at a glance — for example, a negative age, an `ejection_fraction`
  outside the physiologically possible 0–100% range, or a `time` value
  of zero would all be visible immediately in the min/max rows, before
  they have a chance to distort a mean, a p-value, or a hazard ratio
  several phases downstream.

## Result

Confirmed directly from the executed notebook cells:

- **N = 299** patient records loaded, matching the published cohort size
  exactly.
- **All columns numeric** after `.apply(pd.to_numeric)` coercion; no
  silent text-typed columns remained.
- **96 deaths (32.1%)** recorded in `DEATH_EVENT`, consistent with the
  proportion reported in the original Chicco & Jurman publication.
- `.info()` output showed 299 non-null entries for every one of the 13
  columns, with no column falling short of the full row count.

## Clinical Interpretation

The `.info()` output already confirms no column has a null count below
299, which is consistent with this specific public dataset's known,
previously reported characteristic of having been pre-cleaned by the
original authors before release. 

## Issues Encountered

None so far. The file matched the expected size and type requirements on
the first load, and no null values were visible in the `.info()`
non-null counts. This is documented anyway, rather than omitted, because
a clean validation pass is itself a result worth recording. 
