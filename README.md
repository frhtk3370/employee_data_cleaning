# Employee Data Cleaning Project

## Overview
A synthetic HR dataset (1,020 rows, 12 columns) was audited and cleaned end-to-end using pandas. The dataset is publicly available on Kaggle: [Messy Employee Dataset](https://www.kaggle.com/datasets/desolution01/messy-employee-dataset).

Unlike a dataset full of obvious, surface-level mess (missing values everywhere, inconsistent spelling), this one looks deceptively clean at first glance — most columns pass a basic check. The real risk here isn't visible dirt, it's **silent corruption**: issues that don't throw an error and don't show up unless you go looking for them. This project is a demonstration of that kind of diagnostic work.

## Key diagnostic findings
These are the three issues that wouldn't show up from a casual `.info()` glance — each required actively questioning a result that looked fine on the surface:

1. **A phone-number column that was actually a hash artifact.** `phone` loaded as `int64` with no errors. Running `.describe()` on it produced a mean of roughly -4.9 billion — statistically meaningless for a phone number. Investigating further showed **100% of the 1,020 values were negative**, with inconsistent digit lengths (7–10 digits). A single anomaly could be random noise; a 100% occurrence rate is a signal that the field was likely generated as a hash-based ID rather than sampled as a real phone number. Reloading with `dtype=str` confirmed the raw values were already corrupted at the source — not a formatting issue introduced by pandas.
2. **A validation check that silently miscounted missing values as decimals.** `(df['age'] % 1 != 0).sum()` returned exactly 211 — a suspiciously exact match to the known missing-value count. `NaN % 1 != 0` evaluates to `True` in pandas, so this check was quietly counting missing ages as "decimal" ages. Caught by comparing the result against a number already known from an earlier step, not by any error message.
3. **A date conversion that reported success without guaranteeing correctness.** `pd.to_datetime()` on the join-date column returned zero conversion failures — but zero failures only means no *format conflict* was detected, not that every date was interpreted in the correct day/month order. Documented as an open risk rather than treated as a solved problem.

## Full list of issues addressed
| Issue | Column(s) | Description |
|---|---|---|
| Identifier misread as numeric | `phone` | Purely numeric-looking values were auto-cast to `int64` on load, corrupting the field |
| Missing values | `age`, `salary` | 211 and 24 missing values respectively |
| Compound column | `department_region` | Two attributes (department, region) combined into one field with a `-` separator |
| Inconsistent column names | all | Mixed casing and spaces |
| Text-typed date | `join_date` | Dates stored as strings instead of a proper date type |
| Invalid values | `phone` | Every value carried an unexplained negative sign, with inconsistent digit lengths (7-10 digits) |

## Approach
1. Loaded the file with an explicit `dtype` for the `phone` column to prevent pandas from silently converting it to a number.
2. Ran `.shape`, `.dtypes`, and `.info()` for an initial health check, followed by `.head()`, `.sample()`, and `.describe()` for a closer look.
3. Confirmed there were no duplicate rows.
4. Standardized all column names to `snake_case`.
5. Converted `join_date` to `datetime64[ns]` and verified the conversion introduced no failures.
6. Split `department_region` into two clean columns: `department` and `region`.
7. Checked all categorical columns (`status`, `performance_score`, `department`, `region`) for spelling/casing inconsistencies — none were found.
8. Investigated `age` and `salary` for outliers or invalid values — none found beyond the pre-existing missing values.
9. Diagnosed the `phone` anomaly: 100% of values were negative, which ruled out random corruption and pointed to a systematic artifact from how the synthetic dataset was generated. Stripped the sign and flagged entries with fewer than 10 digits as invalid.

## Result
- 1,020 rows retained (no rows dropped)
- 2 new columns (`department`, `region`) replacing 1 compound column
- `phone` reduced to 928 valid, standardized values; 92 flagged as invalid
- All column names, data types, and categorical values standardized
- Missing `age`/`salary` values left as `NaN` by design, since they cannot be reliably imputed from other fields

See `employee_data_cleaning.ipynb` for the full, documented walkthrough and `employee_data_cleaned.csv` for the output file.

## Tools
Python, pandas, numpy, Jupyter Notebook

1. Loaded the file with an explicit `dtype` for the `phone` column to prevent pandas from silently converting it to a number.
2. Ran `.shape`, `.dtypes`, and `.info()` for an initial health check, followed by `.head()`, `.sample()`, and `.describe()` for a closer look.
3. Confirmed there were no duplicate rows.
4. Standardized all column names to `snake_case`.
5. Converted `join_date` to `datetime64[ns]` and verified the conversion introduced no failures.
6. Split `department_region` into two clean columns: `department` and `region`.
7. Checked all categorical columns (`status`, `performance_score`, `department`, `region`) for spelling/casing inconsistencies — none were found.
8. Investigated `age` and `salary` for outliers or invalid values — none found beyond the pre-existing missing values.
9. Diagnosed the `phone` anomaly: 100% of values were negative, which ruled out random corruption and pointed to a systematic artifact from how the synthetic dataset was generated. Stripped the sign and flagged entries with fewer than 10 digits as invalid.

## Result
- 1,020 rows retained (no rows dropped)
- 2 new columns (`department`, `region`) replacing 1 compound column
- `phone` reduced to 928 valid, standardized values; 92 flagged as invalid
- All column names, data types, and categorical values standardized
- Missing `age`/`salary` values left as `NaN` by design, since they cannot be reliably imputed from other fields

See `employee_data_cleaning.ipynb` for the full, documented walkthrough and `employee_data_cleaned.csv` for the output file.

## Tools
Python, pandas, numpy, Jupyter Notebook
