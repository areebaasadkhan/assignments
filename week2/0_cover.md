# Assignment 2: The Dirty Ledger

**Course:** ICA-F26, The Prices Desk
**Due:** Mon 9 Nov, 23:59
**Folder:** assignments/week2/

## What I did
I audited the price ledger, wrote a cleaning log, and put ten lines of AI-suggested cleaning code on trial, one line at a time.

## Headline finding
The file I was given is named `teaching_prices_clean.csv`, not `teaching_prices_messy.csv` as the brief says. When I audited it I found no real mess: 96 rows, no missing values, no duplicates, no stray spaces, no impossible prices. The one thing I fixed is that dates were stored as text. Everything else was left alone, and I say why in each log row.

The bigger result is from the code trial. Three of the AI's lines would have quietly damaged this clean file:
- `drop_duplicates(subset=['date','item'])` deleted all 48 City 2 rows.
- `round(0)` changed 36 prices.
- `.str.lower()` changed all 96 item labels.

## Files
| File | What it is |
|---|---|
| 0_cover.md | this page |
| 1_attempt.md | diagnosis, planned decisions, hash |
| 2_work.ipynb | cleaning on a copy, before/after table, court trial (all cells run) |
| 3_log_reflection.md | court verdicts, cleaning log rows, reflection |
| AI_USE_LOG.md | how AI was used |
| teaching_prices_clean.csv | original input, untouched |
| teaching_prices_cleaned_output.csv | output of the notebook |

## AI use
Claude was used to help draft the diagnosis, the code and the written logs. Details and what was checked are in AI_USE_LOG.md. The data is labelled illustrative teaching data, not real PBS observations, and no personal or confidential data was used.
