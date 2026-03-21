Read this file when creating tables or text-heavy slides. The multi-run technique is essential for professional-quality output.
## Table Creation
- Use create_table with values for initial content, headerFillColorHex for styled headers
- alternateRowColorHex adds automatic row striping
- Table dimensions in get_slide_content_elements may show defaults (3,000,000 × 3,000,000) even when created differently — this is a reporting quirk, not a layout bug

## Multi-Run Text Formatting — The Key to Visual Hierarchy
The add_text_box tool supports multiple runs per paragraph, each with independent font size, color, bold/italic, and font family. Always use this to create proper hierarchy. Never default to a single font size and color for an entire text box.
### Minimum Font Size Rules
- Never set any text below 14pt — reduce content or increase the text box instead
- Preferred body text: 16pt
- Title/header: 24–40pt
- KPI/stat numbers: 28–48pt (accent color, bold)
- Labels and captions: 14–16pt

### Example: KPI Card with Multi-Run Hierarchy
❌ Bad pattern (flat, uniform):
paragraphs: [{"runs": [{"text": "17M+\nEVs Sold",
  "fontSize": 14, "textColorHex": "#00D4AA"}]}]
✅ Good pattern (visual hierarchy):
paragraphs: [{"runs": [
  {"text": "17M+", "fontSize": 36, "bold": true,
   "textColorHex": "#00D4AA"},
  {"text": "\nEVs Sold in 2024", "fontSize": 14,
   "textColorHex": "#A0AEC0"}
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
