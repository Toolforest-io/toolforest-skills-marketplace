Read this file when designing slide layouts. These patterns have been tested in production builds.
## Page Dimensions (EMU)
Standard 10" × 5.63" slide: 9,144,000 × 5,143,500 EMU. Reference conversions: 1 inch = 914,400 EMU. 1 point = 12,700 EMU. 0.5" margin = 457,200 EMU. 0.3" gap = 274,320 EMU.
## Layout Pattern: Header Bar
Use on content slides for consistent visual structure.
- Dark rectangle: full width (9,144,000 EMU) × ~914,400 EMU tall, anchored at top
- Slide title inside bar: white text, bold, 23–26pt
- Green accent line: ~45,720 EMU thick, immediately below the bar

This creates a strong visual anchor. Every content slide should use this unless intentionally breaking the pattern for impact.
## Layout Pattern: Card Grid
- Background shapes: ROUND_RECTANGLE with text boxes overlaid inside
- Inset text boxes ~70,000–114,000 EMU from card edges
- Alternate card colors: dark navy vs white for contrast
- Minimum gap between cards: 228,600 EMU (~0.25")

## Layout Pattern: Big Stat / KPI Callout
- Large number: 40–48pt, accent color (#00B874), bold
- Label below: 10–12pt, bold, body color
- Description below label: 9–10pt, muted color

Use multiple runs in a single text box to achieve this hierarchy — never flatten to one font size. See references/tables-and-formatting.md for the multi-run technique.
## Layout Pattern: Two-Column Split
- Left column: x: 457,200 to ~4,500,000 EMU (stats, cards, data)
- Right column: x: ~4,800,000 to ~8,700,000 EMU (dark panel with text)
- Minimum gap between columns: ~340,000 EMU

## Color System
Map text colors to background for readability:
- Dark (#0D2137): Title/Header #FFFFFF, Body #FFFFFF, Muted/Caption #A8C4D4
- Light (#E8F5EE): Title/Header #0D2137, Body #0D2137, Muted/Caption #5A6B7A
- White card: Title/Header #0D2137, Body #0D2137, Muted/Caption #5A6B7A
- Accent (#00B874): Title/Header #FFFFFF, Body #FFFFFF, Muted/Caption #FFFFFF

## Gradients via create_gradient

Use create_gradient to add gradient backgrounds and rectangular gradient fills to slides. Primary use cases:

- Full-slide gradient backgrounds — position at (0,0) with size matching slide dimensions (9,144,000 × 5,143,500 EMU). This is the main use case since the Slides API doesn't support gradient page backgrounds natively.
- Rectangular gradient fills — place anywhere at any size to simulate a gradient-filled rectangle.
- Limitation: create_gradient produces rectangular images. For non-rectangular shapes (rounded rectangles, circles), the gradient won't clip to the shape's edges. For those, layer the gradient behind a shape with a transparent fill, which only partially works.

For additional visual depth, also consider: alternating dark/light backgrounds, semi-transparent overlays via fillAlpha, accent-colored bars and dividers, and contrasting card colors.
