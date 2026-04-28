# Batch wrapper argument shapes

The pattern: singular wrappers accept a friendly simplified shape that the toolkit translates internally; `batch_*` wrappers forward directly to the Google Sheets API and require the raw API form. Below are the specific differences with verified payloads.

## batch_create_chart

`create_chart` accepts snake_case (`chart_type`, `source_range`, `anchor_cell`, `series_colors`, `header_row`, `legend_position`, `x_axis_title`, `y_axis_title`).

`batch_create_chart` requires camelCase: `chartType`, `sourceRange`, `anchorCell`, `seriesColors`, `headerRow`, `legendPosition`, `xAxisTitle`, `yAxisTitle`.

```python
# WRONG — fails with: charts[0] missing 'chartType'
batch_create_chart(charts=[{
  "sheet": "Charts",
  "source_range": "A4:E10",
  "chart_type": "line",
  "title": "BTC Funding Rate",
  "anchor_cell": "G3"
}])

# RIGHT
batch_create_chart(charts=[{
  "sheet": "Charts",
  "sourceRange": "A4:E10",
  "chartType": "line",
  "title": "BTC Funding Rate",
  "anchorCell": "G3",
  "seriesColors": ["#F0B90B", "#F7A600", "#1976D2", "#00B874"],
  "legendPosition": "right",
  "headerRow": True
}])
```

## batch_set_data_validation

`set_data_validation` accepts `{type: "dropdown", values: [...]}`, `{type: "number_between", min, max}`, `{type: "checkbox"}`.

`batch_set_data_validation` requires the raw API form with `condition` wrapping:

```python
# Dropdown backed by a Reference range
{
  "range": "F4:F100",
  "rule": {
    "condition": {
      "type": "ONE_OF_RANGE",
      "values": [{"userEnteredValue": "=Reference!$C$2:$C$9"}]
    },
    "showCustomUi": True,
    "strict": True
  },
  "sheet": "Pipeline"
}

# Dropdown with hardcoded values
{
  "range": "K4:K100",
  "rule": {
    "condition": {
      "type": "ONE_OF_LIST",
      "values": [
        {"userEnteredValue": "High"},
        {"userEnteredValue": "Medium"},
        {"userEnteredValue": "Low"}
      ]
    },
    "showCustomUi": True,
    "strict": True
  },
  "sheet": "Pipeline"
}

# Checkbox
{
  "range": "F4:F1000",
  "rule": {
    "condition": {"type": "BOOLEAN"},
    "strict": True
  },
  "sheet": "_run_template"
}

# Number between
{
  "range": "J4:J1000",
  "rule": {
    "condition": {
      "type": "NUMBER_BETWEEN",
      "values": [
        {"userEnteredValue": "1"},
        {"userEnteredValue": "5"}
      ]
    },
    "strict": False
  },
  "sheet": "_run_template"
}

# Date is valid
{
  "range": "C4:C100",
  "rule": {
    "condition": {"type": "DATE_IS_VALID"},
    "inputMessage": "Enter a valid date (YYYY-MM-DD)",
    "strict": False
  },
  "sheet": "Pipeline"
}
```

Common condition types: `ONE_OF_LIST`, `ONE_OF_RANGE`, `NUMBER_BETWEEN`, `NUMBER_GREATER_THAN_EQ`, `NUMBER_LESS_THAN_EQ`, `DATE_IS_VALID`, `DATE_AFTER`, `BOOLEAN`, `TEXT_CONTAINS`, `TEXT_EQ`, `CUSTOM_FORMULA`.

## batch_set_column_width

`set_column_width` takes `start_index`, `end_index`, `pixel_size`.

`batch_set_column_width` takes a `widths` array where each entry has its own column range:

```python
batch_set_column_width(
  spreadsheet_id="...",
  widths=[
    {"sheet": "Pipeline", "startIndex": 0, "endIndex": 1, "pixelSize": 240},  # col A
    {"sheet": "Pipeline", "startIndex": 12, "endIndex": 13, "pixelSize": 320},  # col M
    {"sheet": "Dashboard", "startIndex": 0, "endIndex": 5, "pixelSize": 150},   # cols A-E
  ]
)
```

Note: `endIndex` is **exclusive**. To set width on column A only, use `startIndex: 0, endIndex: 1`. To set columns A through E, use `startIndex: 0, endIndex: 5`.

## batch_add_conditional_format

`set_conditional_format` exposes a high-level API with `rule_type` of `color_scale`, `threshold`, `text_contains`, `top_n`, `custom_formula`. This is by far the friendlier interface.

`batch_add_conditional_format` requires the raw `booleanRule` or `gradientRule` shape:

```python
# Threshold (boolean rule)
{
  "sheet": "Pipeline",
  "range": "D4:D100",
  "rule": {
    "booleanRule": {
      "condition": {
        "type": "NUMBER_LESS",
        "values": [{"userEnteredValue": "14"}]
      },
      "format": {
        "backgroundColor": {"red": 1.0, "green": 0.80, "blue": 0.82},
        "textFormat": {"bold": True, "foregroundColor": {"red": 0.72}}
      }
    }
  }
}

# Color scale (gradient rule)
{
  "sheet": "Live Dashboard",
  "range": "B6:E10",
  "rule": {
    "gradientRule": {
      "minpoint": {"color": {"red": 0.10, "green": 0.46, "blue": 0.82}, "type": "MIN"},
      "midpoint": {"color": {"red": 1, "green": 1, "blue": 1}, "type": "NUMBER", "value": "15"},
      "maxpoint": {"color": {"red": 0.83, "green": 0.18, "blue": 0.18}, "type": "MAX"}
    }
  }
}
```

**Recommendation: prefer `set_conditional_format` over `batch_add_conditional_format` even for multi-rule workflows.** Calling the singular tool 5–10 times is rarely measurable in user-perceived latency, and the friendly interface (especially `match_type: "exact"` on `text_contains`, or named presets like `red_white_green` on `color_scale`) eliminates a class of bugs.

## When to batch and when not to

Default to **singular** for ≤4 operations of a kind. The cleaner API more than pays for the extra calls.

Default to **batch** for 5+ operations of a kind. The throughput win matters once you're loading a sheet with several dozen formats or formulas.

If the operation is one-time setup that runs in the background, singular is almost always fine. If it's part of a hot path the user is waiting on, batch carefully.
