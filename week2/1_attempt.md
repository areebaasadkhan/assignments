# 1_attempt.md: Diagnosis and planned decisions (before any cleaning code)

Nothing was changed while writing this. Every problem below is tied to the code that revealed it. Only methods from class are used: `shape`, `dtypes`, `isna().sum()`, `duplicated()`, `describe()`, `value_counts()`, `groupby`.

**Note:** the brief says `teaching_prices_messy.csv`. The supplied file is `teaching_prices_clean.csv`. I audited the supplied file.

## What the file is
96 rows and 6 columns: `date`, `item`, `city`, `price`, `unit`, `data_label`. It is a weekly price ledger: 12 Thursdays (1 Jan to 19 Mar 2026), 4 items (Staple A, Oil B, Pulse C, Sweetener D), 2 cities (City 1, City 2). Prices are PKR per kg. The `data_label` column says it is illustrative teaching data, not real PBS observations.

## Diagnosis table

| # | Check | Code that revealed it | What I found | Problem? | Planned decision | Reason |
|---|---|---|---|---|---|---|
| 1 | Missing values | `df.isna().sum()` | 0 missing in every column | No | **Retain** | Nothing to fix. Filling or dropping would only invent or lose data. |
| 2 | Duplicate rows | `df.duplicated().sum()` and `df.duplicated(subset=['date','item','city']).sum()` | 0 and 0 | No | **Retain** | A real duplicate means same date, item and city. None exist. |
| 3 | Same date and item appears twice | `df.duplicated(subset=['date','item']).sum()` | 48 | **Not a problem.** It looks like a duplicate but is City 1 and City 2 for the same item and date. | **Retain** | This is the trap. Dropping on date and item alone would delete a whole city. |
| 4 | Date stored as text | `df.dtypes` | `date` is a string, not a date | Yes (small) | **Correct** (convert to datetime) | Dates must be dates to sort, group and plot by time. No value changes. |
| 5 | Date gaps | `pd.to_datetime(df['date']).drop_duplicates().sort_values().diff().value_counts()` | all 11 gaps are exactly 7 days | No | **Retain** | No week is missing. |
| 6 | Rows per date, item and city | `df['date'].value_counts()`, `df['item'].value_counts()`, `df['city'].value_counts()` | 8 per date, 24 per item, 48 per city | No | **Retain** | Matches the expected 12 dates x 4 items x 2 cities = 96. The panel is complete. |
| 7 | Label spelling, case, spaces | `value_counts()` on item, city, unit; `(col != col.str.strip()).sum()` | only 4 item names, 2 city names, 1 unit. No stray spaces. | No | **Retain** | Labels are consistent. Changing case would rename valid labels for no gain. |
| 8 | Price range, impossible values | `df['price'].describe()`; `(df['price'] <= 0).sum()` | min 99.5, max 350, none at or below 0 | No | **Retain** | All prices are positive and plausible for PKR per kg. |
| 9 | Outliers by item | `df.groupby('item')['price'].agg(['min','max','mean'])`; largest weekly change with `pct_change()` | Each item stays in a narrow band. Largest weekly move is 3.5%. | No | **Retain** | The overall range (99.5 to 350) is wide only because the items are different goods, not because of errors. Oil B at ~300 is normal for Oil B. |
| 10 | City gap | pivot of City 2 minus City 1, then `value_counts()` | City 2 is exactly 6.0 above City 1 in all 48 pairs | Flag, not fix | **Flag** | A constant gap every week is very regular for real price data. It fits the "illustrative" label. I would not change it, but anyone using this for conclusions about city differences should know. |
| 11 | Decimal prices | `(df['price'] % 1 != 0).sum()` | 36 prices end in .5 | No | **Retain** | .5 prices are valid and real. Rounding would change 36 numbers. |
| 12 | Constant columns | `value_counts()` on `unit` and `data_label` | both have one value in all 96 rows | No | **Retain (flag)** | Useless for analysis but `data_label` is the provenance warning. It must stay with the data. |
| 13 | Provenance | `df['data_label'].unique()` | "ILLUSTRATIVE TEACHING DATA - NOT ACTUAL PBS OBSERVATIONS" | Flag | **Flag** | Results from this file must not be presented as real price findings. |

## Summary of planned decisions
- **Correct (1):** convert `date` from text to datetime.
- **Flag (2):** constant City 2 = City 1 + 6 gap; illustrative-data label.
- **Retain (everything else):** no missing values, no duplicates, no outliers, no label problems.
- **Investigate / Exclude:** nothing to investigate or exclude. No row is removed.

## Expected result after cleaning
96 rows, 0 missing, 0 duplicates, same price mean (202.17), min (99.5) and max (350). Each item 24 times, each city 48 times, each date 8 times.

---

## Hash
SHA-256 of the first 35 lines of this file (everything above the line):

`449928dc7b4e00ad5523e7ef5aedbba203c9f0540910cb97147961b286717b54`

To check it, run: `head -n 35 1_attempt.md | sha256sum`

Git snapshot:
```
git add 1_attempt.md
git commit -m "attempt-before-AI"
git tag attempt-before-AI
```
