Run this checklist after building EVERY slide. Call get_slide_content_elements and inspect the result against each check below. Do not proceed to the next slide until all checks pass.
## Layout Checks
- No overlapping elements: Compare x/y positions + widths/heights of all elements. Flag any pair where bounding boxes intersect.
- Elements within page bounds: All right edges < 9,144,000 EMU. All bottom edges < 5,143,500 EMU.
- Sufficient margins: Content should be ≥ 457,200 EMU (0.5") from slide edges.
- Sufficient gaps: At least 150,000 EMU (~0.16") between elements; 228,600 EMU (~0.25") is better.

## Text and Table Overflow Checks
- Check estimatedOverflow: false on all text boxes and tables.
- Compare estimatedContentHeight to the element’s actual height.
- If autofit was used (text boxes or tables), check scale_factor — values below 0.7 mean the element is too small for the content. **This is a hard failure: delete and rebuild the element** (enlarge the box, reduce content, or split across elements).

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
Fix the issue and re-run verification. If a text box overflows, try: (1) enlarging the box, (2) reducing content, (3) splitting across two boxes. If elements overlap, adjust positions. Never ship a slide that fails verification.
