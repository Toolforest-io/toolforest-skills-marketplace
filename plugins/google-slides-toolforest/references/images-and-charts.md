Read this file when embedding images or charts into slides.
## Image Embedding via add_image
The add_image tool supports three source modes: image_url (public URL), image_data (single encoded string), and image_chunks (array of encoded strings). Position and size are in EMU.
### Critical: Always Use Hex Encoding
Claude/Cowork’s content filter scans outgoing tool parameters for API key-like patterns. Base64 data can trigger false positives, silently corrupting the image with [API_KEY] replacements. Hex encoding (characters 0–9, a–f only) avoids this entirely.
✅ image_encoding: "hex", image_data: "89504e470d0a1a0a..."
❌ image_encoding: "base64", image_data: "iVBORw0KGgo..." (may be corrupted)
### Always Read Hex From File — Never Hand-Type
Early attempts to manually chunk hex strings introduced duplicate data and byte-level corruption, producing invalid PNGs. When splitting into image_chunks, always do it programmatically:
chunk_size = 4000
chunks = [hex_str[i:i+chunk_size] for i in range(0, len(hex_str), chunk_size)]
assert ''.join(chunks) == hex_str  # verify reconstruction
### Size Constraints
The add_image API accepts up to 1MB decoded. However, hex-encoded image data must fit in the LLM’s context window (~25K tokens per Read call ≈ 20–25KB decoded). Recommendations by image type:
- Small icons/decorations (< 20KB): add_image with hex encoding
- Charts and data viz (any size): Google Sheets → embed_chart (see below)
- Photos/logos with public URL (any size): add_image with image_url
- Large local images (20KB–1MB): Use image_url if available, or the Sheets pipeline

### Generating Simple Icons with Pillow
Geometric icons (circles, arrows, checkmarks) can be generated locally at 1–5KB using Pillow, then hex-encoded. Keep to 200–400px dimensions, PNG with transparency.
## Charts — Always Use the Sheets Pipeline
For any chart or data visualization, always use Google Sheets → Google Slides instead of rendering matplotlib charts as images. This approach:
- Avoids the context window bottleneck — image data stays within Google’s infrastructure
- Produces live, linked charts — update the spreadsheet and the presentation chart updates too
- Supports rich chart types: bar, column, line, area, pie, scatter, combo, stacked
- No encoding, no content filter issues, no token limits

### Sheets Chart Workflow
Step 1: create_spreadsheet → returns spreadsheet_id
Step 2: update_values with headers in row 1, data below
Step 3: create_chart with chartType, sourceRange, title, seriesColors matching the deck’s theme
Step 4: embed_chart with linking_mode: "LINKED" (live data) or "NOT_LINKED_IMAGE" (static snapshot)
### Chart Styling Tips
- Use seriesColors to match the presentation’s color palette from config.json
- Set legendPosition: "none" for single-series charts
- Control aspect ratio via width/height in create_chart; control slide size via EMU in embed_chart

### Why Not Matplotlib?
A presentation-quality chart at 100–150 DPI is typically 20–50KB as PNG. Hex-encoded, that’s 40–100K characters — exceeding the context window. Even if it could be passed through, the Sheets approach gives live, editable charts with zero encoding overhead.
