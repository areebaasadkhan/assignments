# 3_log_reflection.md

Evidence for every verdict is in `2_work.ipynb` (Part C). Before = 96 rows, 0 missing, 0 duplicates, price mean 202.17, min 99.5, max 350.

## 1. Court verdicts (AI cleaning code, line by line)

| # | AI line | What I ran | Evidence | Verdict | Reason |
|---|---|---|---|---|---|
| 1 | `df.columns = df.columns.str.strip().str.lower()` | applied to a copy | 96 rows. Column names unchanged (already clean lowercase). | **Keep** | Harmless and I can explain it: it only tidies column names. |
| 2 | `df['item'] = df['item'].str.strip().str.lower()` | applied to a copy | 96 rows, but **96 labels changed** ("Staple A" became "staple a"). | **Modify** | Keep `.str.strip()` (0 labels changed). Remove `.lower()`: it renames every valid label for no gain and the diagnosis found no case problem. |
| 3 | `df['date'] = pd.to_datetime(df['date'])` | applied to a copy | dtype became datetime64, 0 missing dates, range 1 Jan to 19 Mar 2026. | **Keep** | This is the one real fix in the file. |
| 4 | `df['price'] = pd.to_numeric(df['price'], errors='coerce')` | applied to a copy | 0 new missing, 0 prices changed. | **Modify** | Change to `errors='raise'`. With `coerce`, a bad value would silently become NaN and be hidden. With `raise`, the code stops and I see it. |
| 5 | `df = df.dropna()` | applied to a copy | 0 missing before, 96 rows before and after, 0 removed. | **Modify** | Print the missing count first and drop on `price` only, so the number of rows removed is never a surprise. Here it removes nothing. |
| 6 | `df = df.drop_duplicates(subset=['date', 'item'])` | applied to a copy | **96 rows down to 48. All 48 removed rows are City 2.** The correct check (date + item + city) finds 0 duplicates. | **Reject** | Wrong key. It deletes valid observations: half the data. No real duplicates exist. |
| 7 | `df['price'] = df['price'].fillna(df['price'].median())` | applied to a copy | 0 missing prices, 0 prices changed. | **Reject** | Nothing to fill. If a price were missing, the overall median (192.25) mixes Oil B (~325) and Staple A (~111), so it would invent a wrong number. |
| 8 | IQR outlier removal on the whole `price` column | applied to a copy, then again per item | Fences 3.69 to 373.19. 0 removed both ways. | **Reject** | Removes nothing here, so it is pointless. The method is also risky: prices of different goods share one rule, so it could delete valid values. The diagnosis found max weekly move 3.5%, so no outliers. |
| 9 | `df['price'] = df['price'].round(0)` | applied to a copy | **36 prices changed** (e.g. 99.5 to 100.0, 202.5 to 202.0). | **Reject** | Silently alters real numbers. Python rounds .5 to the nearest even number, so some go up and some down. |
| 10 | `df = df.drop(columns=['data_label'])` | applied to a copy | Column removed. Its only value is "ILLUSTRATIVE TEACHING DATA - NOT ACTUAL PBS OBSERVATIONS". | **Reject** | It is the provenance warning. Deleting it lets the data pass as real. |
| 11 | `df = df.sort_values(['date','item','city']).reset_index(drop=True)` | applied to a copy | 96 rows, same content as the original when both sorted. | **Keep** | Only changes row order. |

Result: 3 keep, 3 modify, 5 reject. Three of the five rejects (6, 9, 10) and one modify (2) would have changed or deleted real data in a file that did not need it.

## 2. Before and after table

| | rows | missing | duplicate rows | price mean | price min | price max |
|---|---|---|---|---|---|---|
| BEFORE | 96 | 0 | 0 | 202.17 | 99.5 | 350.0 |
| AFTER | 96 | 0 | 0 | 202.17 | 99.5 | 350.0 |

Checks on the final data:
- **Was any valid observation removed?** No. Compared row by row against the original, same rows and same values. Prices changed: 0. Labels changed: 0.
- **Did any label or number change without me knowing?** No. The only change is `date` going from text to datetime.
- **Does every item appear the expected number of times?** Yes. 12 dates x 2 cities = 24 rows for each of Staple A, Oil B, Pulse C, Sweetener D. 48 rows per city. 8 rows per date.

## 3. Cleaning log rows

| Row | Problem | Code that revealed it | Decision | Action taken | Rows affected | Reason |
|---|---|---|---|---|---|---|
| 1 | `date` stored as text | `df.dtypes` | Correct | `pd.to_datetime` | 96 values, type only | Needed for time sorting and plotting. No value changed. |
| 2 | Missing values | `df.isna().sum()` | Retain | none | 0 | Nothing missing. |
| 3 | Duplicate rows | `df.duplicated()` | Retain | none | 0 | None exist on date + item + city. |
| 4 | Date + item repeats | `df.duplicated(subset=['date','item'])` = 48 | Retain | none | 0 (48 look like duplicates) | These are City 1 and City 2. Not duplicates. |
| 5 | Label spacing and case | `value_counts()`, `.str.strip()` comparison | Retain | strip only (changes nothing) | 0 | Labels already consistent. |
| 6 | Outliers | `describe()`, `groupby('item')`, `pct_change()` | Retain | none | 0 | Max weekly move 3.5%. Wide overall range is because items differ. |
| 7 | Constant City 2 = City 1 + 6 | pivot of City 2 minus City 1 | Flag | none | 48 pairs | Too regular for real prices. Fits the illustrative label. Do not draw conclusions about city gaps. |
| 8 | Illustrative data label | `df['data_label'].unique()` | Flag | column kept | 96 | Results must not be presented as real PBS prices. |
| 9 | Decimal .5 prices | `df['price'] % 1` | Retain | none | 36 | Valid values. Rounding rejected (court line 9). |
| 10 | File name mismatch | brief vs upload | Flag | audited the supplied file | whole file | Brief says `teaching_prices_messy.csv`. File supplied is `teaching_prices_clean.csv`. Worth confirming with Ms. Noor that this is the right file. |

## 4. Reflection

**Which of my decisions differed from the AI's, and which was better supported?**
My plan was to change almost nothing: only convert the dates, and retain, flag or leave alone everything else. The AI's code was much more aggressive. It removed rows, rounded prices, lowercased labels, dropped a column and filled missing values that did not exist. In every case my decision was better supported, because each "no change" was backed by a check showing the thing was not broken: 0 missing, 0 duplicates, 24 rows per item. The AI's lines were written for a generic messy file and not for this one. The worst was line 6. Its key left out `city`, so it deleted 48 valid rows, and without the before and after row counts I could not have seen that from the code alone.

**Where the AI was useful.** Lines 1, 3 and 11 are sensible and I kept them. It also suggested `to_numeric`, which led me to the better `errors='raise'` version.

**What would I have missed alone?**
- That `errors='coerce'` can turn a bad value into a hidden NaN. I would likely have used it without thinking.
- Running the outlier rule per item as a fairer second test.
- Comparing the whole cleaned table to the original with `.equals`. This is a stronger check than counting rows, because it catches changed values as well as removed rows.

**What the AI would have missed without me.** It did not look at the data first. It had no reason to know that City 2 rows share date and item with City 1 rows, that the `.5` prices are real, or that `data_label` carries the warning that the data is illustrative.

**Main lesson.** Cleaning code should be judged by what it changes, not by whether it runs. A file can be correct from the start, and the honest result of an audit is sometimes "almost nothing needed fixing". An undocumented change to a clean file is worse than no change.

**Limits.** This is a 96-row illustrative file. Real PBS data would likely have problems this file does not, and the same method (diagnose first, one line at a time, before and after counts, and a full comparison to the original) would still apply.
