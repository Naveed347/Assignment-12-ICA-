# 1_attempt.md — Diagnosis, planned decisions, hash

> **Provenance note:** This diagnosis was produced with AI assistance (Claude) per the instructor's instruction to use AI for the whole assignment. It is **not** an independent pre-AI attempt, and no pre-AI snapshot exists. The hash below identifies the **raw input file**, not a pre-AI commit.

**Raw file:** `teaching_prices_messy.csv`
**SHA-256:** `5818dad3807487d4e37f764e5d19cbaa152edd78fe4db182eb28b087c8709d35`
**Git commit hash of this attempt:** `[add if you commit this folder]`

## Expected structure
12 weekly dates (2026-01-01 to 2026-03-19) × 4 items (Staple A, Oil B, Pulse C, Sweetener D) × 2 cities (City 1, City 2) = **96 rows**. File has **97**.

## Diagnosis and planned decisions
| # | Problem | Code that revealed it | Where | Planned decision | Reason |
|---|---|---|---|---|---|
| 1 | Exact duplicate row | `raw.duplicated().sum()` → 1 | Row 96 repeats row 8 (2026-01-08, Staple A, City 1, 99.5) | **Correct (drop the repeat)** | Identical in every column and appended at the end; the grid expects one observation per date-item-city. Keep first occurrence. |
| 2 | Inconsistent label `oil b` | `raw.item.value_counts()` → 5 items, one lowercase | Row 40: 2026-02-05, City 1, 108.5 | **Correct + flag** (inferred) | Oil B City 1 already exists that day (318.0); Staple A City 1 is missing that day; 108.5 fits the Staple A series (~105–107). Evidence is strong but inferred, so flag it for source confirmation. |
| 3 | Price stored as text, contains `1,250` | `raw.dtypes` (price is str); `str.contains(r'[^0-9.]')` | Row 17: 2026-01-15, Staple A, City 2 | **Fix format, flag, exclude value** | Comma is a format issue, but 1250 is ~10× its neighbours (105–122). Neither 1250 nor 125 matches the trend (~109), so any "correction" would be a guess. Keep row, exclude price from analysis. |
| 4 | Sentinel `9999` | `describe()` → Sweetener D max 9999, std ≈ 2007 | Row 30: 2026-01-22, Sweetener D, City 1 | **Flag + set to missing** | 9999 is a placeholder, not a price; neighbours are 153–162. Do not impute. |
| 5 | Genuinely missing price | `raw.isna().sum()` → price 1 | Row 5: 2026-01-01, Pulse C, City 2 | **Retain + flag** | The observation slot is real; only the value is absent. Dropping the row would break the grid; imputing would invent data. |
| 6 | Constant columns `unit`, `data_label` | `nunique()` → 1 each | All rows | **Leave alone** | Valid metadata. `data_label` states the data is illustrative, not actual PBS observations; it must stay attached. |

## Things checked and found fine
- `city`: only City 1 / City 2 (no spelling variants). `date`: 12 clean ISO dates. No leading/trailing whitespace in `item` or `city`.
- No other outliers: after the fixes, every item's range is tight (Oil B 298–350, Pulse C 200–213.5, Staple A 99.5–122, Sweetener D 149–184.5).
