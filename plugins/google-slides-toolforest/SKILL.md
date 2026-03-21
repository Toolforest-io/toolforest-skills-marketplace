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

- 1. Create presentation → create_presentation returns presentationId, default slide ID, available layouts, and page dimensions (standard: 9,144,000 × 5,143,500 EMU).
- 2. Set theme → set_theme for master background, default text color, heading + body font families, accent colors. Never skip this step. Never default to Arial — always choose an intentional pairing (see references/fonts.md).
- 3. Build each slide → Use add_slide with layoutId (prefer "p12" / BLANK for full control). Populate with shapes, text boxes, tables. Delete default placeholders (i0, i1) on first slide if building a custom title.
- 4. Verify each slide → After building, call get_slide_content_elements and run the verification checklist (see scripts/verify_slide.md). Fix issues before moving on.
- 5. Iterate → Rework any slide that fails verification. Never ship a slide you haven't checked.

### Critical Gotchas (Read These First)

These are the highest-signal failure points. They are non-obvious and will waste significant time if missed.

### Gotcha 1: autofit Is Boolean, Not String

Always set autofit: true + minFontSize: 8 on content text boxes. Check scaleFactor in the response — values below ~0.7 mean the box is too small.

✅ autofit: true, minFontSize: 8
❌ autofit: "SHAPE_AUTOFIT" → ValidationError

### Gotcha 2: Hex Encoding for Images, Never Base64

Claude/Cowork's content filter scans outgoing tool parameters for API key-like patterns. Base64 image data can trigger this, silently corrupting the image. Always use hex encoding.

✅ imageEncoding: "hex", imageData: "89504e470d0a1a0a..."
❌ imageEncoding: "base64" (may be corrupted by content filter)

Always read hex data from file programmatically. Never hand-type or split hex strings manually.

### Gotcha 3: Gradients via create_gradient

Use create_gradient to add gradient backgrounds and rectangular fills. Supports full-slide backgrounds (position at 0,0 with slide dimensions) and rectangular gradient fills at any position/size. Limitation: gradients are always rectangular — they won't clip to non-rectangular shapes like rounded rectangles or circles.

### Gotcha 4: Font Rendering — Google Fonts Only

Google Slides cannot embed fonts. Licensed fonts (Proxima Nova, Avenir, Helvetica Neue, Futura) fall back silently to a default. Stick to Google Fonts catalog. See references/fonts.md for safe pairings.

## Reference Files

Claude should read these on demand — not all at once.

- references/design-patterns.md — Layout patterns (header bars, card grids, KPI callouts, two-column splits), color palettes for dark/light/accent backgrounds. Read when designing slide layouts.
- references/fonts.md — Font pairing recommendations by audience/tone, common mistakes, Google Fonts availability. Read when choosing typography.
- references/images-and-charts.md — Image embedding via hex encoding, size constraints, Sheets→Slides chart pipeline. Read when adding images or data visualizations.
- references/tables-and-formatting.md — Table creation, multi-run text formatting for visual hierarchy, minimum font size rules, alignment rules. Read when creating tables or text-heavy slides.
- scripts/verify_slide.md — Verification checklist to run after every slide. Read before starting verification.
- config.json — User preferences (theme colors, font pairing, margin style). If this file doesn't exist, ask the user for preferences or use sensible defaults.
