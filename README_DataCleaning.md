# 🧹 Student Data Cleaning Pipeline (Python · pandas)

A column-by-column data cleaning project that takes a deliberately "super dirty" student
dataset — 1,000 records riddled with nulls, inconsistent formats, numbers written as words,
malformed emails, broken addresses, and nested JSON — and produces a single analysis-ready
CSV.

---

## 📌 Overview

Real-world data almost never arrives clean. This project simulates that reality: every one
of the 18 source columns contains a different kind of mess, and the notebook works through
them one at a time — profiling the problem, fixing it, and documenting the reasoning inline.

**Input:** `super_dirty_students.csv` — 1,000 rows × 18 columns
**Output:** `students_cleaned.csv` — 1,000 rows × 24 columns (nested fields expanded)

**Stack:** Python · pandas · NumPy · regex (`re`) · `json` · `email_validator`

---

## 🔍 What was wrong with the data

The source dataset intentionally combines many real-world data-quality problems:

| Problem | Example |
|---------|---------|
| Numbers written as words | `age = "twenty"`, `score = "ninety"`, `gpa = "four point five"` |
| Inconsistent categories | `gender`: `fmale`, `Femlae`, `FEMALE`, blank |
| Label drift | `course`: `DATA SCIENCE`, `data-sciens`, `data_sciense`, `ds` |
| Mixed date formats | ISO dates, `YYYY/MM/DD`, and raw Unix timestamps in the same column |
| Malformed emails | `someonegmail.com`, `psmith@@chen.com`, `special` |
| Messy phone numbers | `+1-619-379-4152x102`, `001-182-659-5631x02803` |
| Free-text addresses | `"Apartment 37, South Kevin district, Tashkent, UZ, 100539"`, plus broken rows |
| Nested JSON | `profile_json` with hobbies, nested skills, family, and device arrays |
| Currency noise | `$135`, `175 USD`, `85,00` |
| Whitespace & casing | `" Claudia Short "`, `"  excellent  "` |

---

## 🛠️ Cleaning pipeline

The notebook (`cleaning.ipynb`) processes each column in sequence:

1. **Profiling** — for each column: unique values, dtype, duplicate check, and missing-value count.
2. **String columns** (`name`, `city`, `status`, `remarks`) — trim whitespace, standardize casing with `str.title()`.
3. **Numeric columns** (`age`, `score`, `attendance`, `gpa`, `money_spent`) — convert word-numbers to digits, strip units/symbols (`$`, `%`, `USD`), coerce to numeric types, impute missing values.
4. **Categorical normalization** — `gender` collapsed to `Male` / `Female` / `Unknown`; `course` mapped to canonical labels via a lookup table.
5. **Email validation** — lowercase and repair common defects (`@@`, missing `@`, leading junk), then validate syntax with a regex **and** the `email_validator` library; invalid addresses set to null.
6. **Phone standardization** — strip extensions, separators, and country-code noise into a consistent format.
7. **Date parsing** — `date_of_join` and `event_time` parsed to datetime with `errors='coerce'`.
8. **Address parsing** — regex extraction of `adr_city`, `adr_district`, and `adr_postal` from free-text; broken rows set to null.
9. **JSON flattening** — `profile_json` parsed (with a repair step for unquoted keys / single quotes) and expanded into `hobbies`, `skills`, `family`, and `devices` columns.
10. **Export** — the cleaned frame is written to `students_cleaned.csv`.

Highlights worth calling out: the **email repair + validation** stack, the **regex address parser**
(handles Tashkent, districts, `-kv` notation, and postal codes), and the **nested-JSON flattener**.

---

## 📂 Repository structure

```
Data_Cleaning_Project/
├── data/
│   ├── super_dirty_students.csv     # Raw input (1,000 × 18)
│   └── students_cleaned.csv         # Cleaned output (1,000 × 24)
├── cleaning.ipynb                   # The cleaning notebook
├── DataCleaning_Project_Presentation.pptx   # Project presentation
└── README.md
```

---

## ▶️ How to run

**Prerequisites:** Python 3.9+ and the packages below.

```bash
pip install pandas numpy email_validator
jupyter notebook cleaning.ipynb
```

Then run the notebook top to bottom. Before running:

- **Update the input path.** The first cell reads the CSV from an absolute Windows path.
  Change it to point at your local `data/super_dirty_students.csv` (a relative path is recommended
  for portability).

Running all cells produces `students_cleaned.csv` in the working directory.

---

## 📊 Results

- **1,000** records cleaned end to end.
- **18 → 24** columns (nested `profile_json` and `address_raw` expanded into structured fields).
- Word-numbers converted, categories standardized, emails validated, addresses parsed, JSON flattened.

See `DataCleaning_Project_Presentation.pptx` for a visual before/after walkthrough.

---

## 🚧 Known limitations & roadmap

Honest notes on the current state and planned improvements:

- **Mixed date formats.** The date columns contain a mix of human-readable dates and raw
  Unix timestamps. The current `to_datetime(errors='coerce')` step parses the human dates
  but coerces the numeric timestamps to `NaT`. *Planned:* detect all-numeric values and
  convert them with `pd.to_datetime(..., unit='s')` so no dates are lost.
- **Numeric range validation.** Some numeric columns (notably `gpa`) contain out-of-range
  values. `.abs()` handles negatives, but upper-bound validation isn't applied yet, so a
  small number of invalid values can pass through and skew mean-based imputation. *Planned:*
  clip values to a valid domain (e.g. GPA ∈ [0, 4.0]) before imputing.
- **Imputation strategy.** Missing numeric values are filled with the column mean. *Planned:*
  evaluate median imputation and/or a `was_missing` flag to preserve the signal that a value
  was originally absent.
- **Refactor into reusable functions.** The pipeline is currently procedural, cell by cell.
  *Planned:* extract tested, reusable cleaning functions so any new batch of records can be
  cleaned automatically.
- **Portable paths.** The input path is currently absolute; moving to relative paths (or a
  config value) makes the notebook runnable anywhere.

---

## 📝 Notes

This is a personal learning/portfolio project built around a synthetic "dirty data" exercise.
The goal was a clean, logically consistent, analysis-ready dataset — and, just as importantly,
practice with the messy edge cases that real data always brings.
