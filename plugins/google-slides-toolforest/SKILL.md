---
name: google-slides-toolforest
description: >
  Use when creating or editing Google Slides presentations via Toolforest MCP
  tools. Triggers on: create_presentation, add_slide, add_text_box, set_theme,
  set_page_background, create_shape, add_image, embed_chart, create_table,
  get_slide_content_elements, or any request to build, design, or modify a
  slide deck using Google Slides through the Toolforest connector.
---

### Core Workflow

Follow these steps for every presentation build:

1. **Create presentation** → create_presentation returns presentation_id, default slide ID, available layouts, and page dimensions (standard: 9,144,000 × 5,143,500 EMU).
2. **Set theme** → set_theme for master background, default text color, heading + body font families, accent colors. Never skip this step. Never default to Arial — always choose an intentional pairing (see references/fonts.md).
3. **Build and verify each slide — ONE AT A TIME.** For each slide, complete ALL of the following before moving to the next:

   a. **Add the slide** → Use add_slide with layout_id (prefer "p12" / BLANK for full control). Delete default placeholders (i0, i1) on first slide if building a custom title.
   b. **Populate the slide** → Add shapes, text boxes, tables, images.
   c. **Verify the slide** → Call get_slide_content_elements and check EVERY element against the verification checks below.
   d. **Fix any failures** → If any check fails, fix the issue, then re-verify. Repeat until the slide passes.
   e. **Only then move to the next slide.**

### Mandatory Verification Checks (Run After EVERY Slide)

After calling get_slide_content_elements, check ALL of the following. If ANY check fails, fix and re-verify before proceeding.

**Layout:**
- All right edges (x + width) < 9,144,000 EMU
- All bottom edges (y + height) < 5,143,500 EMU
- No overlapping elements (compare bounding boxes of all element pairs)
- Content ≥ 457,200 EMU from slide edges
- ≥ 150,000 EMU gap between elements (228,600 EMU preferred)

**Text overflow:**
- estimatedOverflow: false on all text boxes and tables
- If autofit was used: scale_factor ≥ 0.7. If below 0.7, the element MUST be rebuilt — enlarge the box, reduce content, or split across elements.

**Font sizes (tiered minimum):**
- Body text, bullets, descriptions: ≥ 14pt
- KPI labels and captions: ≥ 10pt
- Nothing below 10pt — ever. If any text is below 10pt, delete and rebuild the element.

**Formatting quality:**
- Text hierarchy is visible — headings, body, and captions are clearly distinct
- Body text is LEFT-aligned, not centered
- Colors create proper contrast against the background

For the full checklist including common issues and z-order checks, see scripts/verify_slide.md.

### Critical Gotchas (Read These First)

These are the highest-signal failure points. They are non-obvious and will waste significant time if missed.

### Gotcha 1: autofit Is Boolean, Not String

Always set autofit: true + min_font_size_pt: 10 on content text boxes. Check scale_factor in the response — values below 0.7 mean the box is too small and MUST be rebuilt.

✅ autofit: true, min_font_size_pt: 10
❌ autofit: "SHAPE_AUTOFIT" → ValidationError
❌ min_font_size_pt: 8 → allows illegible text

### Gotcha 2: Hex Encoding for Images, Never Base64

Claude/Cowork's content filter scans outgoing tool parameters for API key-like patterns. Base64 image data can trigger this, silently corrupting the image. Always use hex encoding.

✅ image_encoding: "hex", image_data: "89504e470d0a1a0a..."
❌ image_encoding: "base64" (may be corrupted by content filter)

Always read hex data from file programmatically. Never hand-type or split hex strings manually.

### Gotcha 3: Gradients via create_gradient

Use create_gradient to add gradient backgrounds and rectangular fills. Supports full-slide backgrounds (position at 0,0 with slide dimensions) and rectangular gradient fills at any position/size. Limitation: gradients are always rectangular — they won't clip to non-rectangular shapes like rounded rectangles or circles.

### Gotcha 4: Font Rendering — Google Fonts Only

Google Slides cannot embed fonts. Licensed fonts (Proxima Nova, Avenir, Helvetica Neue, Futura) fall back silently to a default. Stick to Google Fonts catalog. See references/fonts.md for safe pairings.

### Font Size Reference (Quick Look)

| Element | Size |
|---------|------|
| Slide title | 24–40pt bold |
| Section header | 20–26pt bold |
| KPI / stat numbers | 28–48pt bold, accent color |
| Body text, bullets | 14–16pt |
| KPI labels, captions | 10–12pt muted |

Nothing below 10pt — ever.

## Reference Files

Claude should read these on demand — not all at once.

- references/design-patterns.md — Layout patterns (header bars, card grids, KPI callouts, two-column splits), color palettes for dark/light/accent backgrounds. Read when designing slide layouts.
- references/fonts.md — Font pairing recommendations by audience/tone, common mistakes, Google Fonts availability. Read when choosing typography.
- references/images-and-charts.md — Image embedding via hex encoding, size constraints, Sheets→Slides chart pipeline. Read when adding images or data visualizations.
- references/tables-and-formatting.md — Table creation, multi-run text formatting for visual hierarchy, alignment rules. Read when creating tables or text-heavy slides.
- scripts/verify_slide.md — Full verification checklist with common issues and edge cases. The critical checks are already inline above — this file has additional detail.
- config.json — User preferences (theme colors, font pairing, margin style). If this file doesn't exist, ask the user for preferences or use sensible defaults.
