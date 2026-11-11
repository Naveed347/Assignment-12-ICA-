# Cover — Assignment 2: The Dirty Ledger

**Course:** ICA-F26 · The Prices Desk  **Week:** 2 · Loading, cleaning and data quality
**Student:** [your name / ID]  **Due:** Mon 9 Nov, 23:59

## What is in this folder
| File | Contents |
|---|---|
| `0_cover.md` | This cover and AI-use disclosure |
| `1_attempt.md` | Diagnosis, planned decisions, data hash |
| `2_work.ipynb` | Cleaning on a copy; before/after table; expected-count check |
| `3_log_reflection.md` | Court verdicts, cleaning log rows, reflection |

## Summary
`teaching_prices_messy.csv` (97 rows) has 6 problems: 1 exact duplicate, 1 mislabeled item (`oil b`), 1 comma-formatted price (`1,250`), 1 sentinel (`9999`), 1 genuinely missing price, and a text-typed price column. After cleaning: **96 rows**, 4 items, every date × item × city appears exactly once (12 × 4 × 2), 3 missing `price_clean` values, all flagged. The raw file is untouched (SHA-256 unchanged).

## AI-use disclosure (honest statement)
This submission was **produced with AI assistance (Claude)** after the instructor instructed students to use AI for the whole assignment and discuss it afterwards in class. Because of that:
- There is **no independent human "attempt-before-AI" diagnosis**. `1_attempt.md` is labelled accordingly and does not claim otherwise.
- The cleaning code was written by the AI and then **run and checked** against before/after evidence (see `2_work.ipynb`). Verdicts in `3_log_reflection.md` are based on that real output.
- Entry for `AI_USE_LOG.md` is included at the end of `3_log_reflection.md`.
- No personal or confidential data was used. The dataset is labelled illustrative teaching data, not actual PBS observations.
