Run this checklist after building EVERY slide. Call get_slide_content_elements and inspect the result against each check below. Do not proceed to the next slide until all checks pass.
## Layout Checks
- No overlapping elements: Compare x/y positions + widths/heights of all elements. Flag any pair where bounding boxes intersect.
- Elements within page bounds: All right edges < 9,144,000 EMU. All bottom edges < 5,143,500 EMU.
- Sufficient margins: Content should be ≥ 457,200 EMU (0.5") from slide edges.
- Sufficient gaps: At least 150,000 EMU (~0.16") between elements; 228,600 EMU (~0.25") is better.
- Variable stacked text: For text placed below a headline, title, stat, or any other variable-height text box, compute the upper box's occupied bottom as `y + max(height, estimatedContentHeight)`. The lower element must start after that bottom plus the required gap. A fixed `y` for subtitles or captions is a failure if the upper text's measured content height changes.

## Text and Table Overflow Checks
- Check estimatedOverflow: false on all text boxes and tables (vertical overflow).
- For text boxes, compare estimatedContentHeight to estimatedUsableHeight. Do not compare against raw element height; Google Slides reserves internal vertical text inset.
- Check estimatedHorizontalOverflow: false on all text boxes (horizontal bleed past the left/right edges — the vertical checks cannot detect this). Compare estimatedContentWidth (widest rendered line) to estimatedUsableWidth (box width minus the horizontal inset). It fires only for unbreakable content wider than the box (oversized drop-cap glyphs, big numerals); normally-wrapping text never trips it. Fix by widening the box. A decorative glyph may overflow vertically yet must not overflow horizontally.
- For tables, compare estimatedContentHeight to the element's actual height.
- If autofit was used (text boxes or tables), check scale_factor — values below 0.7 mean the element is too small for the content. **This is a hard failure: delete and rebuild the element** (enlarge the box, reduce content, or split across elements).

## Outline (border) Check
- get_slide_content_elements returns outline_visible on shapes and text boxes: true (border renders), false (hidden), or null (inherited from theme). If a text box was meant to be borderless but reports outline_visible: true, hide it with update_element_style (outline_visible: false).

## Contrast Checks (from get_slide_content_elements)
- Text elements include `contrastRatio` (float) and `contrastSufficient` (boolean) when a background color is available
- Minimum `contrastRatio`: 4.5:1 for body text (WCAG AA), 3.0:1 for large text (>=18pt or >=14pt bold)
- If `contrastSufficient` is false, the text may be invisible or hard to read — change the text color or background
- Common problems: default black text (#000000) on dark backgrounds, white text on light backgrounds, muted gray text on medium backgrounds
- `set_theme` will warn if the default text color has poor contrast against the master background — fix the theme before building slides

## Formatting Quality Checks
- Text hierarchy is visible: Can you immediately distinguish headings from body text from captions?
- No single-color/single-size text boxes where hierarchy should exist (e.g., KPI cards, bullet lists with lead-in phrases).
- Font sizes are readable: Body text ≥ 14pt. KPI labels and captions ≥ 10pt. Nothing below 10pt — ever.
- Alignment is appropriate: Body text LEFT-aligned, not center-aligned.
- Colors create contrast: Accent colors for emphasis, muted colors for secondary info, correct text color for the background.

## Common Issues to Watch For
- Text box autofit reported as googleAutofitType: "NONE" on read-back even when applied during creation — this is expected because Toolforest pre-scales font sizes at creation time
- Table autofit is reported correctly via the autofit field (autofitApplied, scale_factor) stored as presentation metadata
- Default placeholder elements (i0, i1) not deleted on first slide — these overlap custom content
- z-order: get_slide_content_elements returns elements in z-order, later elements render on top — verify layering is correct
- Master element injection via set_master_elements may add decorative shapes to every new slide — account for these in layout

## If Verification Fails
Fix the issue and re-run verification. If a text box overflows, try: (1) enlarging the box, (2) reducing content, (3) splitting across two boxes. If stacked text overlaps, fix the upper text box first, then recompute the lower element's `y` from the measured upper bottom plus gap. Never ship a slide that fails verification.
