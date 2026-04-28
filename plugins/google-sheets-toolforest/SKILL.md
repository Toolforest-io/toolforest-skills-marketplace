---
name: google-sheets-toolforest
description: >
  Use when creating or editing Google Sheets via Toolforest MCP tools.
  Triggers on: create_spreadsheet, create_sheet, update_values,
  batch_update_values, fill_formula, add_named_range,
  set_conditional_format, freeze_panes, merge_cells, batch_format_cells,
  batch_create_chart, duplicate_sheet, set_data_validation, set_notes,
  get_notes, set_banding, auto_resize_columns, add_filter_view,
  group_columns, check_spreadsheet_errors, or any request to build,
  design, or modify a multi-tab dashboard, tracker, scorecard, eval
  scorecard, or cross-tab rollup using Google Sheets through the
  Toolforest connector.
---

# google-sheets-toolforest

Best practices for building Google Sheets via the Toolforest MCP. Covers theme presets, formula patterns that survive scale, cross-tab references, dashboard layouts, conditional formatting on categorical data, dynamic ranking tables, and batch-wrapper gotchas — all derived from end-to-end builds that were verified to render and resolve cleanly.

The toolkit has 73 tools. This skill does not enumerate them. It gives you the patterns that recur across real builds and the gotchas the Google Sheets API itself doesn't tell you about.

## When to load this skill

Load this skill whenever a task involves the `google_sheets-*` tools, especially when the user wants:

- A multi-tab dashboard, tracker, or scorecard
- Cross-sheet rollups (a master tab pulling from N data tabs)
- A repeating layout (per-month, per-customer, per-run) — the template-and-duplicate idiom
- Conditional formatting beyond a single threshold
- Charts driven by computed/pivoted data, not the raw table
- Anything that will be shared with non-author readers (formatting matters)

Skip it for one-off `get_values` reads, single-cell updates, or trivial appends — direct tool calls are fine.

## Default theme

Apply the Toolforest brand theme unless the request implies a different mood (boardroom report, ops tracker, eng scorecard, marketing rollup). Brand:

- **Title banner:** background `#1F4E79`, white Montserrat 16pt bold, left-padded
- **Section headers:** background `#B8B8B8`, Montserrat 12pt bold
- **Column headers:** background `#4C4C4C`, white Source Sans 3 bold, centered
- **Banding:** header `#4C4C4C`, alternating `#FFFFFF` / `#F2F2F2`
- **Body:** Source Sans 3 for prose columns, IBM Plex Mono for IDs/timestamps/code-like data
- **Accents:** green `#2E7D32` (positive/accepted), red `#B71C1C` (negative/rejected), amber `#F9A825` (warning/in-progress), blue `#1976D2` (informational)

For audience-specific themes (Boardroom, Eng Scorecard, Ops Tracker, Marketing Rollup) see `references/themes.md`.

## The patterns that matter

### 1. Template-and-duplicate

For any workflow producing structurally identical sheets (eval runs, monthly reports, per-customer tabs):

1. Create a `_template` tab. Apply all formatting, conditional formats, validation, freeze, banding **on the template**.
2. Use `duplicate_sheet` to create each instance. The duplicate carries over freeze rows, conditional formats, validation, tab color, and column widths.
3. Populate values only — formatting is already in place.

Two API calls per new instance instead of fifteen. The template should have a `_` prefix and gray tab color so it reads as infrastructure, not data.

### 2. Cross-tab references with INDIRECT

When a master tab needs to roll up from N data tabs whose names are stored as values in column A:

```
=INDIRECT("'" & A5 & "'!F4:F1000")
```

The single quotes around the sheet name are **required** if any sheet name contains a hyphen, space, or other non-alphanumeric character (e.g. `run_2026-04-26`). Without them, the formula will silently return `#REF!`. Always use the quoted form — it's harmless on plain names and necessary on hyphenated ones.

This pattern lets you add a new run/month/customer by appending one row + duplicating the template tab. No formula edits needed on the overview tab.

### 3. 2D matrix lookups

`fill_formula` only fills along one axis (vertical OR horizontal). For a 2D matrix where the formula varies by both row and column (assets × venues, dates × metrics):

**Option A — call `fill_formula` once per column**, with the column letter hard-coded in the formula and `{row}` as the placeholder. This is the cleanest approach for a few columns.

**Option B — hand-author the formula array** and push via `update_values`. Generate it programmatically in code.

The lookup formula itself, for an arbitrary (row_key, col_key) pair against a 3-column table:

```
=INDEX(SourceData,
  MATCH(1, (INDEX(SourceData,,col_key_idx)=row_key) * (INDEX(SourceData,,row_key_idx)=col_key), 0),
  return_col)
```

This is an array formula that handles non-sorted source data without requiring helper columns. Wrap in `IFERROR` for empty cells.

### 4. Long formulas across many rows

Three rules for keeping long formulas maintainable:

1. **Use named ranges for source data**, not raw addresses. `FundingData` instead of `'Funding Rates'!A2:F121`. Update the range once via `add_named_range`, every dependent formula picks it up.
2. **Anchor the lookup keys with `$`**, not the lookup ranges. `$A6` and `B$5` so the formula can be filled in either direction without breaking.
3. **Verify with `formulaResults`, not a separate read.** Every `update_values` and `batch_update_values` response includes a `formulaResults` field showing the resolved values. Use that to confirm formulas evaluated as intended — saves a `get_values` round-trip.

### 5. Dynamic sort/rank tables

Use `QUERY` for a top-N table that updates as the source changes:

```
=IFERROR(
  QUERY(Pipeline!A4:M100,
    "select A, B, C, D, F where D >= 0 and L = '' order by D asc limit 5",
    0),
  "No matching rows")
```

Two gotchas the Sheets docs are quiet about:

- The QUERY result **does not inherit number/date formatting from the source**. If the source column is a date, the QUERY output will show raw serial numbers (e.g. 46143). Format the destination range explicitly after.
- The QUERY column references are positional letters within the source range, not the spreadsheet's absolute columns. If the source range starts at column C, then "select A" pulls from C.

For ranking against a calculated metric, `LARGE`/`SMALL` + `INDEX/MATCH` works inline:

```
=INDEX(SourceRange, MATCH(LARGE(MetricColumn, 1), MetricColumn, 0), name_col)
```

### 6. Conditional formatting on categorical data

When categories share substrings (like "Submitted" vs "Ready to Submit", or "Drafting" vs "Re-drafting"), the `text_contains` rule type with default match mode will mis-fire — "Submitted" matches both. **Always pass `match_type: "exact"` for categorical/discrete state values.** Only use the default `contains` match for true substring searches (free-text notes columns).

The `set_conditional_format` tool exposes this directly. Worked example: see `references/categorical_formatting.md`.

### 7. Layout safety: freeze, merge, and column-freeze

Two boundary rules that the API enforces but the Sheets UI does not:

1. **Freeze before merging.** If you freeze rows 1–5 and then merge a title across `A1:I1`, that's fine. If you merge first and then freeze, the merge can break the freeze silently.
2. **Don't merge across a column-freeze.** If you freeze column A and merge `A1:I1` for a title banner, the merge fails with "can't merge frozen and non-frozen columns". Title banners want only a row freeze, not both. Freeze column A is for label-column scrolling, which is incompatible with a wide top-row banner.

The recommended order: freeze rows → format → merge title within the frozen zone → freeze column (only if needed and no banner crosses it).

### 8. Number formatting: PERCENT expects fractions

The PERCENT number format multiplies by 100 for display. So `0.0261` formatted as PERCENT shows `2.61%`. If you have a percentage value that's already in percentage units (e.g. `2.61` meaning 2.61%), use a NUMBER format with pattern `"0.00\"%\""` instead — appending a literal `%` character.

The annualized funding rate columns in a financial dashboard are usually already-multiplied (`12.0860` for 12.0860%), so they need the `0.00"%"` pattern, not PERCENT type. Reaching for PERCENT on already-percentage data turns 2.61% into 261%.

### 9. Banding interacts with conditional formatting

Banding paints the whole range. Conditional formatting paints over banding only on matched cells. This is usually what you want — the banding gives the table its baseline rhythm; the conditional formats highlight signal on top.

Apply in this order: banding first, conditional formats second. Reversing it doesn't break anything but can leave you debugging why a "highlighted" cell looks no different from its neighbors (banding repainted on top).

### 10. The verification trifecta

Before declaring a sheet done, run these three reads:

1. `check_spreadsheet_errors` with `detect_circular_refs: true`. Catches `#REF!`, `#N/A`, `#VALUE!`, broken named ranges, circular dependencies. Should return zero errors.
2. `get_notes` over any range you wrote notes to. Confirms unicode/multiline content survived round-trip.
3. Spot-check `formulaResults` from the most recent batch updates. Reading the response is free; a separate `get_values` is a round-trip.

## Batch wrapper gotchas

Several `batch_*` tools accept a different argument shape than their singular counterparts. The pattern is consistent: **singular wrappers accept a friendly simplified shape; batch wrappers forward to the raw Google API and require the raw shape.** Until the toolkit harmonizes these, use the raw form for any batch call. See `references/batch_wrappers.md` for the specific differences and worked-example payloads for:

- `batch_create_chart` (camelCase keys instead of snake_case)
- `batch_set_data_validation` (raw `{condition: {...}}` form, not `{type, values}`)
- `batch_set_column_width` (the `widths` property is shaped differently from singular `set_column_width`)
- `batch_add_conditional_format` (raw `{booleanRule}` / `{gradientRule}`, not the high-level `rule_type` shape)

Defaulting to the singular tools where possible avoids this entirely, at the cost of more API calls. For 1–4 operations, prefer singular. For 5+, accept the raw shape and batch.

## Common gotchas (quick reference)

- **`delete_sheet` takes a numeric `sheet_id`, not a name** — get it from `create_sheet` response or `get_spreadsheet_metadata`. Every other write tool takes the name.
- **`create_spreadsheet` always creates a default `Sheet1`** — delete it (sheet_id is typically 0) before creating named tabs, or you'll have an orphan.
- **`fill_formula` is single-axis only** — see pattern #3 above for 2D fills.
- **`auto_resize_columns` interacts badly with merged title banners** — the merged width gets attributed to individual columns, blowing them up. Pass `max_width_px: 240` (or similar) as a guard, or auto-resize before merging.
- **`min_font_size_pt` is not a Sheets concept**, only Slides — Sheets autofit shrinks freely. Use explicit column widths via `set_column_width` if you need a hard floor on rendered text size.

## End-to-end worked example

A complete walk-through (multi-tab financial dashboard with synthetic data, named ranges, formulas, conditional formats, charts, sharing) is in `references/worked_example_financial_dashboard.md`. The example builds a multi-venue funding-rate monitor used to verify this skill.

## Verification checklist

Before handing off a finished sheet:

- [ ] `check_spreadsheet_errors` returns zero errors and zero broken refs
- [ ] At least one read confirms `INDIRECT` cross-tab refs resolve (not silently `#REF!`)
- [ ] PERCENT-formatted cells contain fractional values, not pre-multiplied percentages
- [ ] Categorical conditional formats use `match_type: "exact"`
- [ ] Title banner exists within the frozen zone, no column-freeze if a wide banner is merged
- [ ] Banding applied before conditional formatting on the same range
- [ ] All notes round-trip via `get_notes`
- [ ] If shared link generated, it's `reader` unless the user explicitly asked for `writer`
