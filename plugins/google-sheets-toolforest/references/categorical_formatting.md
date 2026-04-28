# Conditional formatting on categorical text

The `set_conditional_format` tool's `text_contains` rule type defaults to substring matching. For categorical state columns where values share substrings, this mis-fires.

## The trap

Status values in a college application tracker include:

- Researching
- Drafting
- Ready to Submit
- **Submitted**
- Decision Pending
- Accepted
- Rejected

A naïve rule with `rule_type: "text_contains"`, `text: "Submitted"` will match both "Submitted" and "Ready to Submit". The intended highlight on submitted-only applications instead highlights drafts.

## The fix

Pass `match_type: "exact"`:

```python
set_conditional_format(
  sheet="Pipeline",
  range="F4:F100",
  rule_type="text_contains",
  text="Submitted",
  match_type="exact",  # <-- this is the critical bit
  background_color="#FFF9C4",
  text_color="#5D4037"
)
```

`match_type` options: `contains` (default), `not_contains`, `starts_with`, `ends_with`, `exact`.

## When to use which

- **`exact`** — categorical state values, status enums, priority tiers (Low/Medium/High), boolean-like text. Default to this for any column with a finite set of allowed values, especially when those values share words.
- **`contains`** — free-text columns where you're flagging keywords (notes containing "blocker", titles containing "draft").
- **`starts_with` / `ends_with`** — IDs with prefixes (`ERR-` for errors, `_test` for test artifacts).

## A complete worked layer

For a status column with 5+ states, write one rule per state. Use the brand accent palette:

```python
# Researching → no highlight (default state, lets banding show through)

set_conditional_format(sheet="Pipeline", range="F4:F100",
  rule_type="text_contains", text="Drafting", match_type="exact",
  background_color="#FFE0B2", text_color="#E65100", bold=True)

set_conditional_format(sheet="Pipeline", range="F4:F100",
  rule_type="text_contains", text="Submitted", match_type="exact",
  background_color="#FFF9C4", text_color="#5D4037")

set_conditional_format(sheet="Pipeline", range="F4:F100",
  rule_type="text_contains", text="Accepted", match_type="exact",
  background_color="#C8E6C9", text_color="#1B5E20")

set_conditional_format(sheet="Pipeline", range="F4:F100",
  rule_type="text_contains", text="Rejected", match_type="exact",
  background_color="#FFCDD2", text_color="#B71C1C")
```

These do not need to be in a particular order — the conditions are mutually exclusive when `match_type: "exact"`. If two rules can match the same value, the lower index (`index: 0` is highest priority) wins; pass `index` explicitly when ordering matters.

## Row-level highlighting from a category

To highlight an entire row (not just the status cell) based on a categorical value, use `rule_type: "custom_formula"`:

```python
# Strike-through entire row when decision is in (accepted or rejected)
set_conditional_format(
  sheet="Pipeline",
  range="A4:M100",  # all data columns
  rule_type="custom_formula",
  formula='=OR($F4="Accepted",$F4="Rejected")',
  background_color="#FAFAFA",
  text_color="#9E9E9E",
  italic=True
)
```

The `$F4` is row-relative (no `$` before the row number) so the rule walks down the data block correctly. The `$` before the column letter is required; omitting it makes the formula evaluate against each cell's column instead of the status column.
