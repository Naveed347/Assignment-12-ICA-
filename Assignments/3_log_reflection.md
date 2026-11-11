# 3_log_reflection.md — Court verdicts, log rows, reflection

> Written with AI assistance (Claude) per the instructor's instruction. All evidence below comes from running `2_work.ipynb`.

## 1. The Cleaning Code Court

### A. Naive one-liners tried on copies
| Line | Evidence (before → after) | Verdict |
|---|---|---|
| `df.dropna()` | 97 → 96 rows; removes the real Pulse C / City 2 / 2026-01-01 observation (only its price is missing) | **Reject** — removes a valid observation; breaks the 96-row grid |
| `df['item'].str.title()` | `oil b` → `Oil B`; Oil B / City 1 / 2026-02-05 now appears **2 times** (expected 1) | **Reject** — silently relabels a Staple A row and creates a duplicate key |
| `pd.to_numeric(df.price, errors='coerce')` | missing prices 1 → 2; `1,250` silently erased | **Modify** — strip the comma first, and flag the value instead of erasing it silently |
| `df.drop_duplicates(subset=['date','item','city'])` | 97 → 96 | **Modify** — gives the right count here, but on keys alone it could hide two different prices for one key. Use full-row matching, as the log shows. |
| Quantile trim (`p < p.quantile(.99)`) | 97 → 95; drops a valid Oil B price (350) as well as the error | **Reject** — removes valid high prices; outliers should be judged against the item's own series |

### B. Line-by-line trial of the final cleaning code (notebook section 3)
| Code line | Evidence | Verdict |
|---|---|---|
| `df=raw.copy()` | raw sha256 unchanged at end | **Keep** |
| `df['row_id']=raw.index`, `df['flag']=''` | every changed row traceable to its original position | **Keep** |
| `dup=df.duplicated(subset=[all columns], keep='first')`; `df=df[~dup]` | drops only row_id 96; rows 97 → 96 | **Keep** |
| `df['price_raw']=df.price` | original text preserved (`1,250`, `9999`) | **Keep** |
| `to_numeric(price_raw.str.replace(',','')...)` | price dtype → float; no new NaNs created by the conversion itself | **Keep** |
| Relabel `oil b` → `Staple A` (mask on item, date, city) | mask hits exactly 1 row; Staple A / City 1 count becomes 12; Oil B / City 1 stays 12 | **Keep** — inferred, flagged for source confirmation |
| `price==9999 → NaN` | hits exactly 1 row (Sweetener D / City 1 / 2026-01-22); Sweetener D max 9999 → 184.5 | **Keep** |
| `price_clean = price.where(~(price_raw=='1,250'))` | hits exactly 1 row; Staple A max 1250 → 122; mean 158.3 → 111.3 | **Keep** — chose exclusion over a guessed correction |
| Missing price flag, no imputation | row retained; missing count is explicit | **Keep** |
| `reset_index(drop=True)` | `row_id` preserves the original positions | **Keep** |

### Before / after table
| Metric | Before | After |
|---|---|---|
| Rows | 97 | 96 |
| Missing price | 1 | 3 (1 source-missing, 1 sentinel, 1 excluded `1,250`) |
| Distinct item labels | 5 | 4 |
| Max price | 9999 | 350 |
| Mean price | 315.46 | 203.61 |

**Do all items appear the expected number of times?** Yes. Each of the 4 items appears 12 times per city (24 total), and every date × item × city combination appears exactly once (`True` in the notebook check).

## 2. Cleaning log rows
| ID | row_id | Date / item / city | Problem | Action | Reason | Risk if wrong |
|---|---|---|---|---|---|---|
| L1 | 96 | 2026-01-08 / Staple A / City 1 | Exact duplicate | Dropped | Identical to row 8; grid needs one per key | Low |
| L2 | 40 | 2026-02-05 / `oil b` / City 1 | Wrong label | Relabeled to Staple A, flagged | Oil B already present that day; Staple A missing; 108.5 fits Staple A trend | **Medium** — inferred; confirm with source |
| L3 | 17 | 2026-01-15 / Staple A / City 2 | `1,250` text, ~10× neighbours | Comma removed; `price_clean` = NaN; flagged | Cannot tell whether true value is 125, 109 or other | Low — nothing invented |
| L4 | 30 | 2026-01-22 / Sweetener D / City 1 | `9999` sentinel | Set to NaN; flagged | Placeholder, not a price | Low |
| L5 | 5 | 2026-01-01 / Pulse C / City 2 | Price missing in source | Retained, flagged, not imputed | Real slot; value unknown | Low |
| L6 | all | `unit`, `data_label` | Constant metadata | Left alone | Valid; label marks data as illustrative, not PBS | None |
| L7 | all | `price` column | Text type | Converted to float, original in `price_raw` | Needed for numeric work; original preserved | Low |

## 3. Reflection
**Which decisions differed, and which were better supported?** There is no independent human attempt to compare against, so the real comparison is between the naive AI-style one-liners and the final decisions. The naive lines differed on three points. They would drop the missing-price row (`dropna`), relabel `oil b` by capitalisation only (`str.title`), and trim by a quantile cutoff. The final decisions were better supported because each was tested against the expected 12 × 4 × 2 grid and against the item's own neighbours: retaining the row preserves the observation, relabeling is tied to evidence that Staple A was missing that day, and outliers were judged per item series.

**What would I have missed alone?** Most likely the `oil b` row: a quick `value_counts()` shows a stray label, but the right fix depends on noticing that Staple A / City 1 was missing on 2026-02-05, so title-casing it would have produced a duplicate Oil B row. Also the point that `to_numeric(errors='coerce')` hides `1,250` by turning it into a plain NaN, so the missing count would look like 2 with no explanation. And that `9999` and `1,250` need different treatment (sentinel versus an unverifiable value).

**Limits.** L2 is an inference and should be confirmed with the original source. `1,250` was excluded rather than corrected because no value could be justified. The data is illustrative teaching data, so conclusions apply only to this exercise.

---
## Entry for `AI_USE_LOG.md`
| Date | Tool | Task | How used | Checked how | Kept? |
|---|---|---|---|---|---|
| 2026-11-12 | Claude | Assignment 2 audit, cleaning code, write-ups | Wrote diagnosis, cleaning code and documents for the whole assignment, per instructor's instruction to use AI for all of it | Ran notebook; verified row counts, missing counts, describe(), grid completeness and raw-file hash | Yes — all kept, with L2 flagged for source confirmation |
