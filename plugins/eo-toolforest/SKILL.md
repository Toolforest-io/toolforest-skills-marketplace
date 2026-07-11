---
name: eo-toolforest
description: >
  Use when working with satellite imagery or earth-observation data via
  Toolforest MCP tools. Triggers on: satellite image/photo of a place,
  Sentinel-2, NDVI, NDWI, vegetation or water trends, "show me X from space",
  clearest/least cloudy scene, earth observation, eo-search_catalog,
  eo-render_image, eo-get_statistics, or any request that combines satellite
  imagery with geo AOIs or Google Docs/Slides output through the Toolforest
  connector.
---

### Hard Rules (Read First)

1. **Never use a native image generator for satellite requests.** If the user
   asks for a satellite image, aerial view, or map of a real place, the ONLY
   correct source is the Toolforest `eo` toolkit (`eo-render_image`). Do not
   use image_gen, DALL-E, or any drawing tool — a generated picture of
   "Mount Rainier from space" is fabrication, not observation.
2. **Verify connections before starting**: `eo` is required; `geo` for
   place-based areas of interest; `google_docs` / `google_slides` only if the
   user wants a document. Use `list_toolkits` first.
3. Sentinel-2 pixels are **10 m** — suggest field-scale or larger subjects
   (farms, lakes, parks, mountains, shorelines). A house is one pixel.

### Workflow A — Clearest-scene snapshot of a place

1. Locate the place: `geo-geocode` (or use a known bbox directly).
2. Build the area: `geo-make_aoi` with `op: point_buffer` and a sensible
   radius (5–15 km for towns, more for regions). Pass its `geojson` output
   directly as `aoi` to the eo tools — this is the designed handoff.
3. Find the clearest scene: `eo-search_catalog` with the AOI/bbox, a season
   or month range, and `max_cloud_cover` 15–30. Pick the scene with the
   lowest `cloud_cover` whose bbox actually covers the area (scenes are
   tiles; two results with the same timestamp are adjacent tiles).
4. Render it: `eo-render_image` with a **narrow time window around the chosen
   scene's date** (e.g. that day ±1 day), `visualization: true_color`,
   `mosaicking_order: leastCC`. Report the acquisition date and cloud cover
   with the image.

### Workflow B — Vegetation / water index statistics

For "how green was X this season", "when did the lake peak", trend questions:

1. Build the AOI as in Workflow A.
2. `eo-get_statistics` with `index: ndvi` (vegetation) or `ndwi` (water),
   `interval: P1M` for seasons/years (`P1D` only for ranges ≤ 400 days).
3. Intervals start at `time_from`; the trailing partial interval is included,
   shortened to `time_to` (March 1 – June 30 monthly = four intervals).
4. Answer from the `mean` series; cite the peak interval and value. Offer a
   rendered image of the peak period (Workflow A step 4) as a follow-up.

### Workflow C — Satellite image into a Google Doc

The composability flow: real pixels, captioned, in a document.

1. Run Workflow A to get `image_url`.
2. `google_docs`: create the document, insert the image **by URL** using
   `image_url` — the URL is short (img.toolforest.io) and passes Google's
   fetcher. Google copies the image bytes into the document, so the doc
   remains intact after the source URL expires.
3. Insert promptly: `image_url` expires ~24 h after rendering (`expires_at`
   field). Re-render if stale.
4. Caption format: place, "true color, captured by Sentinel-2 on {date}",
   cloud cover %. Pull both values from the search result you rendered from.

### Critical Gotchas

1. **The image-generator hijack** (most common failure): assistants reach for
   native image generation despite instructions. If the user says "use
   Toolforest", every image MUST come from `eo-render_image`. Re-read Hard
   Rule 1 before any imagery step.
2. **Large AOI polygons can be blocked client-side** before the call reaches
   Toolforest (platform argument screening). If a tool call with a big
   `geojson` fails without a Toolforest error, simplify first:
   `geo-make_aoi` `op: simplify` targeting ≤ 20 vertices, or fall back to a
   plain `bbox`.
3. **Free accounts are Sentinel-2 only** for render/statistics. Requests for
   Landsat, MODIS, or commercial collections return a structured error
   telling the user to switch their eo Account type to Planet Insights —
   relay it; do not retry.
4. **Blank or mostly-transparent render** means no scene at or below
   `max_cloud_cover` covered the AOI in the window (the result's `note` says
   so). Widen the window or raise `max_cloud_cover` — do not lower the width.
5. **Pixels outside a polygon AOI render transparent** (a circular AOI yields
   a circular image). Use `bbox` when the user wants a full rectangle.
6. **Costs are metered in processing units** (renders ~1–4 PU, statistics
   ~1 PU, searches free; free tier 10,000 PU/month). Render deliberately,
   not speculatively.
7. **Widget caching**: in MCP Apps hosts (ChatGPT), the inline image card is
   served from a versioned `ui://` bundle the client caches along with the
   tool list. If widget behavior looks stale after a server update, refresh
   or reconnect the connector — a new chat is not enough.

### Reporting conventions

- Always name the satellite and date: "Sentinel-2, June 18 2026".
- Quote `cloud_cover` from the catalog scene, not a guess.
- NDVI ranges: <0 water/rock, 0–0.2 bare, 0.2–0.4 sparse, 0.4–0.6 healthy,
  >0.6 dense vegetation.
- Mention that render links expire in ~24 h when handing the raw URL to the
  user.
