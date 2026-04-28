# Worked example: multi-tab financial dashboard

End-to-end build for a multi-venue funding-rate arbitrage monitor. Four tabs, named-range cross-references, formula-driven dashboards, conditional formatting, charts, error verification. This example was used to verify every pattern in `SKILL.md` and produces a sheet that resolves with zero formula errors.

## Specification

- **Spreadsheet title:** "Funding Rate Arb Monitor"
- **Tabs:** `Live Dashboard`, `Funding Rates` (raw), `Positions` (raw), `Charts` (computed pivots + visualizations)
- **Source data:** 8h funding rates across 4 venues (Binance, Bybit, OKX, Hyperliquid) × 5 assets (BTC, ETH, SOL, ARB, AVAX) × 6 recent timestamps = 120 rows
- **Computed:** latest rate matrix per (asset, venue), median rate per asset, best long/short venue per asset, annualized spread in bps, mark-to-funding PnL per open position

## Build sequence

The full sequence (12 distinct stages) lives in this skill's parent test runs. The skeleton:

1. `create_spreadsheet` → delete default `Sheet1` (sheet_id: 0)
2. `create_sheet` × 4 with tab colors (Live Dashboard pink, Funding Rates blue, Positions green, Charts purple)
3. Push raw funding data with `update_values` (120 rows)
4. Push positions with `update_values` (4 rows)
5. `batch_add_named_range` for `FundingData`, `PositionsData`, `AssetsList`, `VenuesList`
6. `batch_update_values` for the dashboard formula matrix (5 assets × 4 venues + median/best-long/best-short/spread columns)
7. `fill_formula` × 3 for venue columns of the rate matrix (one per venue, since `fill_formula` is single-axis)
8. `freeze_panes` (5 rows, 0 columns — see SKILL.md pattern #7)
9. `batch_format_cells` for title, headers, number formats
10. `merge_cells` for title banner
11. `set_conditional_format` × 4 (color scale on rate matrix, threshold on spread, green/red on PnL)
12. Charts: pivot helper tables on Charts tab (`batch_update_values`), then `batch_create_chart` (5 line charts, camelCase keys)
13. `auto_resize_columns` with `max_width_px: 240` to guard against the merged-banner inflation
14. `check_spreadsheet_errors` with `detect_circular_refs: true` — must return zero

## Key formulas

Latest annualized funding rate for (asset, venue) from the long source table:

```
=IFERROR(INDEX(FundingData,
  MATCH(1,
    (INDEX(FundingData,,1)=MAX(IF(INDEX(FundingData,,2)=B$5,
      IF(INDEX(FundingData,,3)=$A6,
        INDEX(FundingData,,1)))))
    * (INDEX(FundingData,,2)=B$5)
    * (INDEX(FundingData,,3)=$A6),
    0),
  5))
```

The `MAX(IF(...))` finds the latest timestamp for that (asset, venue) pair; the outer `MATCH(1, (...)*(...)*(...))` finds the row matching all three keys. Wrapped in `IFERROR` so missing combinations return blank rather than `#N/A`.

Best long venue (lowest rate among 4):

```
=INDEX($B$5:$E$5, MATCH(MIN(B6:E6), B6:E6, 0))
```

Annualized spread in bps:

```
=(MAX(B6:E6) - MIN(B6:E6)) * 100
```

(Source data is already annualized as percentage, so multiplying the difference by 100 converts pct-points to bps.)

Position PnL accrual:

```
=Notional * EntryDiff * 3 * DaysOpen
```

Where `EntryDiff` is the per-8h funding rate diff at entry, `3` is fundings/day, `DaysOpen` from `DATEDIF` against today. For mark-to-funding PnL, replace `EntryDiff` with `CurrentDiff` looked up via `INDEX/MATCH` against the latest rate matrix.

## Number formats used

| Range | Format | Why |
|---|---|---|
| Annualized rate cells | `0.00"%"` (NUMBER) | Source data is already in pct units (12.0860 = 12.0860%); PERCENT format would multiply ×100 again → 1208.60% |
| Notional/PnL | `$#,##0` (CURRENCY) | Standard |
| 8h funding rate (raw) | `0.0000000` (NUMBER) | 7 decimal places — funding rates are 1e-4 to 1e-3 typical |
| Spread (bps) | `#,##0` (NUMBER) | Integer bps |
| Days Open | (default, centered) | Just an integer |

The percent-format trap is the most common misstep. PERCENT type is for fractions (0.025 → 2.5%); use `0.00"%"` NUMBER pattern for already-multiplied values.

## Verification

After the full build, `check_spreadsheet_errors` must return:

```
{ "total_errors": 0, "total_warnings": 0, "error_counts": {}, "sheets_affected": [] }
```

If it returns a `#REF!` count > 0, the most likely cause is one of:

- Named range references a sheet that's been renamed or deleted
- `INDIRECT` formula not single-quoting a hyphenated sheet name
- A `MATCH` lookup against a column that's empty or contains the wrong dtype

Spot-check `formulaResults` from the Live Dashboard build response to confirm the rate matrix populated with realistic values (single-digit to low-double-digit annualized %, not zeros or very large numbers indicating a unit mismatch).
