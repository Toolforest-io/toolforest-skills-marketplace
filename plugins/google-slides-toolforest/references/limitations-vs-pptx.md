This file is informational context for the skill author, not loaded during normal use. Include in the skill folder if you want Claude to understand the tradeoffs when a user asks "should this be a Google Slides deck or a PPTX?"
## What Google Slides Cannot Do (via Toolforest)
- Gradients on non-rectangular shapes: create_gradient produces rectangular gradient images. For rounded rectangles or circles, the gradient won't clip to the shape's edges.
- Shadows and effects: No drop shadows, glow, soft edges, or 3D transforms.
- Font embedding: Cannot embed licensed fonts — Google Fonts only.
- Finer typography: No character spacing or text box margin/padding control.

## What Google Slides Does Better
- Live and collaborative: Editable by anyone, no file to manage.
- Consistent fonts: Google Fonts served from CDN to all viewers.
- Real-time updates: Changes are instant, no re-export.
- Live Sheets charts: embed_chart links to spreadsheet data.

## What PPTX Does Better
- Richer gradient support: Radial and path gradients with multiple color stops (Toolforest supports linear gradients via create_gradient, but not radial/path).
- Shadows and effects on shapes.
- Single-pass generation: No round-trip API calls.
- Image embedding: Local files, no public URL needed.
- Any font can be embedded in the file.
