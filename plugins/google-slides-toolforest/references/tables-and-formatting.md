Read this file when creating tables or text-heavy slides. The multi-run technique is essential for professional-quality output.
## Table Creation
- Use create_table with values for initial content, header_fill_color for styled headers
- alternate_row_color adds automatic row striping
- Set autofit: true to automatically shrink header and body font sizes to fit within the specified table height (assumes Arial metrics). Set min_font_size_pt: 10 to control the floor. Check autofitApplied and scale_factor in the response — values below 0.7 mean the table is too dense for the space and MUST be rebuilt (enlarge the table, reduce content, or split across slides).
- get_slide_content_elements returns estimatedContentHeight and estimatedOverflow for tables, plus an autofit field showing whether autofit was applied and the scale factor used

## Multi-Run Text Formatting — The Key to Visual Hierarchy
The add_text_box tool supports multiple runs per paragraph, each with independent font size, color, bold/italic, and font family. Always use this to create proper hierarchy. Never default to a single font size and color for an entire text box.
### Minimum Font Size Rules (Tiered)
- Title/header: 24–40pt bold
- Section header: 20–26pt bold
- KPI/stat numbers: 28–48pt (accent color, bold)
- Body text, bullets, descriptions: 14–16pt (14pt is the floor for body content)
- KPI labels, captions, muted annotations: 10–12pt
- Nothing below 10pt — ever. If autofit scales any text below 10pt, the element must be rebuilt.

### Example: KPI Card with Multi-Run Hierarchy
❌ Bad pattern (flat, uniform):
paragraphs: [{"runs": [{"text": "17M+\nEVs Sold",
  "font_size_pt": 14, "text_color": "#00D4AA"}]}]
✅ Good pattern (visual hierarchy):
paragraphs: [{"runs": [
  {"text": "17M+", "font_size_pt": 36, "bold": true,
   "text_color": "#00D4AA"},
  {"text": "\nEVs Sold in 2024", "font_size_pt": 12,
   "text_color": "#A0AEC0"}
], "alignment": "LEFT"}]
### Where to Apply Multi-Run Formatting
- Bullet lists: Bold lead-in phrase in accent color, followed by regular-weight detail in body color
- Section headers within text boxes: Larger bold text on line 1, smaller body text below
- Callout/insight boxes: "Key Takeaway:" in bold accent, followed by the insight in body color
- Signpost cards: Large bold number ("01") in accent, bold title in white, description in muted gray

### Text Alignment Rules
- Titles and single-line headings: CENTER is appropriate
- Body text, bullets, observations, descriptions: LEFT — never center multi-line body text
- KPI cards with number + label: CENTER is acceptable
- Mixed content (heading + bullets): LEFT for everything

### Fill the Space
Size text to fill the available box. If a text box has room, make the headline bigger rather than leaving whitespace. The goal is to use the full shape, not leave empty space with small text.
