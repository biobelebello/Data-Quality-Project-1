# Bank Marketing: Data Quality, Cleaning and Decision Brief

**Author:** Joy Bello  
**Dataset:** [UCI Bank Marketing](https://archive.ics.uci.edu/dataset/222/bank+marketing): `bank-additional-full.csv`, 41,188 telemarketing call records from a Portuguese bank (May 2008 to Nov 2010)  
**Business question:** *Who should the bank call, and how many times, in its next term-deposit campaign?*

## Deliverables

| Deliverable | File |
|---|---|
| Data-quality notebook (executed, with outputs) | [`notebooks/01_data_quality.ipynb`](notebooks/01_data_quality.ipynb) |
| Cleaned dataset (41,176 rows × 26 columns) | [`data/processed/bank_marketing_clean.csv`](data/processed/bank_marketing_clean.csv) |
| Data dictionary | [`docs/data_dictionary.md`](docs/data_dictionary.md) (also as [`.csv`](docs/data_dictionary.csv)) |
| One-page decision brief | [`docs/decision_brief.pdf`](docs/decision_brief.pdf) |
| Quality reports, validation results, figures | [`outputs/`](outputs/) |

## Repository structure

```
├── data/
│   ├── raw/bank-additional-full.csv        # original UCI file, never modified
│   └── processed/bank_marketing_clean.csv  # output of the notebook
├── notebooks/01_data_quality.ipynb         # profile -> issues log -> clean -> validate -> analyse -> export
├── docs/
│   ├── data_dictionary.md / .csv
│   └── decision_brief.pdf
├── outputs/                                # quality reports, validation checks, segment tables
│   └── figures/
└── requirements.txt
```

## How to reproduce

```bash
pip install -r requirements.txt
jupyter notebook notebooks/01_data_quality.ipynb   # Run All
```

The notebook uses `data/raw/` if the file is there, otherwise it downloads the official UCI archive. Re-running it regenerates the cleaned CSV and every file in `outputs/`.

## Data-quality issues found and decisions taken

| Issue | Rows | Decision |
|---|---|---|
| Exact duplicate rows | 12 | Removed |
| Hidden missing values coded `"unknown"` (6 columns; `default` 20.9%) | 12,716 cells | Kept as an explicit category, not imputed or dropped |
| Week 3 quality check reported 0 unknowns (dtype check skipped pandas `str` columns) | n/a | Profiling function fixed |
| `pdays = 999` placeholder for "not contacted" | 39,673 | Converted to null in `days_since_prev_contact` |
| `pdays = 999` but `previous > 0` (contradiction) | 4,110 | `previously_contacted` derived from `previous` |
| `duration` only known after the call (target leakage) | all | Kept for description, excluded from decision analysis |
| Very high call counts (up to 56) | 869 > 10 calls | Kept, flagged with `high_contact_flag` |
| Dotted column names, `default` is an SQL keyword | n/a | Renamed to snake_case / `credit_default` |

All **12 automated validation checks** pass (see notebook section 6 and `outputs/validation_checks.csv`).

## Headline findings

- Baseline subscription rate: **11.3%**.
- Previous-campaign successes subscribe at **65.1%**. Age 60+ at **39.6%**, students **31.4%**, retired **25.3%**.
- Mobile **14.7%** vs landline **5.2%**.
- Calls after the 3rd attempt use **26.5%** of call volume for **12.0%** of subscriptions.
- A priority segment that is **7.7%** of customers produces **29.1%** of subscriptions (42.7% conversion).

**Recommendation:** call the priority segment first, cap at 3 attempts, prefer mobile numbers, and validate with a controlled pilot.

## Assumptions

1. The 12 duplicates are data-entry repeats (there is no customer ID to confirm this).
2. `"unknown"` means the value wasn't recorded. It is treated as a category, not a value to guess.
3. Where `pdays` and `previous` disagree, `previous` is correct (it agrees with `poutcome`).
4. Very high call counts are genuine operational records, not errors.

## Limitations

1. One bank, one country, 2008–2010. Interest rates, channels and customer behaviour have changed since then.
2. There is no year column, so financial-crisis effects can't be separated from month effects.
3. The data is observational. Findings are associations, not causes, which is why the brief recommends a randomised pilot.
4. There is no cost or deposit-value data, so the analysis optimises conversion, not profit.
5. The classes are imbalanced (11.3% positive), so small segments have wide confidence intervals (shown throughout).
6. Any future predictive model must exclude `duration`.

## Source

Moro, S., Cortez, P. and Rita, P. (2014) 'A data-driven approach to predict the success of bank telemarketing', *Decision Support Systems*, 62, pp. 22–31. Dataset: UCI Machine Learning Repository, Bank Marketing (ID 222).
