# Theme presets

Like the Slides skill, four audience-mapped themes plus the default Toolforest brand. Apply via the formatting calls in `set_up_theme()` patterns shown for each.

## Default — Toolforest Brand

For internal tools, dashboards, anything not pitched at a specific audience.

| Element | Value |
|---|---|
| Title banner bg | `#1F4E79` |
| Title banner text | white Montserrat 16pt bold |
| Section header bg | `#B8B8B8` |
| Section header text | Montserrat 12pt bold (default color) |
| Column header bg | `#4C4C4C` |
| Column header text | white Source Sans 3 bold, centered |
| Banding header | `#4C4C4C` |
| Banding 1st band | `#FFFFFF` |
| Banding 2nd band | `#F2F2F2` |
| Body font | Source Sans 3 |
| Mono font | IBM Plex Mono (for IDs, timestamps, code) |
| Positive accent | `#2E7D32` |
| Negative accent | `#B71C1C` |
| Warning accent | `#F9A825` |
| Info accent | `#1976D2` |

## Boardroom — for financial/exec reports

Heavier serif feel, restrained palette, formal.

| Element | Value |
|---|---|
| Title banner bg | `#0D2137` (deep navy) |
| Title banner text | cream `#F5F1E8`, Playfair Display 18pt bold |
| Section header bg | `#E8E4D8` |
| Section header text | Playfair Display 13pt |
| Column header bg | `#0D2137` |
| Column header text | cream, Lato bold, centered |
| Banding bands | `#FFFFFF` / `#F8F5EE` |
| Body font | Lato |
| Mono font | Source Code Pro |
| Accents | gold `#A47B3F`, sage `#5C6E5A` |

Signature moves: roman-numeral row numbers in the leftmost column for executive summaries; right-aligned Lato bold for all currency totals; chapter-mark divider rows (single full-width merged cell) between sections.

## Eng Scorecard — for evals, postmortems, latency reports

Hyper-utilitarian, mono-heavy, contrast-led.

| Element | Value |
|---|---|
| Title banner bg | `#1A1F2E` |
| Title banner text | white IBM Plex Sans 14pt bold |
| Section header bg | `#2E3440` |
| Section header text | `#88C0D0`, IBM Plex Mono 11pt bold |
| Column header bg | `#3B4252` |
| Column header text | `#ECEFF4`, IBM Plex Mono bold |
| Banding bands | `#FFFFFF` / `#ECEFF4` |
| Body font | IBM Plex Sans |
| Mono font | IBM Plex Mono (heavy use — IDs, durations, hashes) |
| Accents | cyan `#5E81AC` (info), warm orange `#D08770` (warn), red `#BF616A` (fail), green `#A3BE8C` (pass) |

Signature moves: status columns rendered as text (`PASS` / `FAIL` / `WARN`) in IBM Plex Mono, never icons; latency columns formatted `#,##0" ms"`; git-sha-style 7-char abbreviations in fixed-width columns.

## Ops Tracker — for project management, task tracking

Soft, approachable, color-led, dropdown-heavy.

| Element | Value |
|---|---|
| Title banner bg | `#7B1FA2` |
| Title banner text | white Source Sans 3 16pt bold |
| Section header bg | `#E1BEE7` |
| Column header bg | `#4A148C` |
| Column header text | white Source Sans 3 bold |
| Banding bands | `#FFFFFF` / `#F3E5F5` |
| Body font | Source Sans 3 |
| Status colors | `#C8E6C9`/`#1B5E20` (Done), `#FFE0B2`/`#E65100` (In Progress), `#FFCDD2`/`#B71C1C` (Blocked), `#E1F5FE`/`#01579B` (Backlog) |

Signature moves: data validation dropdowns on every state column (status, priority, owner) backed by a `Reference` tab — easier to update lists than to hand-edit dropdowns; collapsible column groups for long-tail metadata (assignee, links, comments, history); filter views named for the questions they answer ("Blocked > 7 days", "Mine and due this week").

## Marketing Rollup — for campaigns, growth dashboards

Brighter, denser type hierarchy, chart-forward.

| Element | Value |
|---|---|
| Title banner bg | gradient feel: solid `#FF6B35` |
| Title banner text | white Oswald 18pt bold uppercase |
| Section header bg | `#FFE0B2` |
| Section header text | Oswald 12pt bold |
| Column header bg | `#1A1A1A` |
| Column header text | white Roboto bold, centered |
| Banding bands | `#FFFFFF` / `#FFF8E1` |
| Body font | Roboto |
| Number font | Oswald (for big-stat cells) |
| Accents | magenta `#E91E63`, yellow `#FFC107`, teal `#00ACC1` |

Signature moves: oversized Oswald numbers (24–32pt) in dedicated single-row "scorecard" sections at the top; chart titles styled to match section headers; sparkline-in-row patterns for trend visibility without dedicated chart tabs.

## Cross-theme rules

These apply regardless of which theme you pick:

1. **Apply banding before conditional formatting on the same range.** Banding paints the whole range; conditional formats paint over banding only on matched cells. Reverse order means the banding repaints over your highlights.
2. **Two tones per accent.** A bright tone for use on dark backgrounds (column headers, dark banners) and a deep tone for use on light backgrounds (banded body cells). Single-tone accents fail WCAG body-text contrast on one of the two surfaces.
3. **Min font size in tight columns.** Sheets autofit shrinks freely. If a column is narrow and content is long, fix the column width via `set_column_width` rather than relying on autofit — there's no `min_font_size_pt` in Sheets.
4. **No mono fonts for prose, no proportional fonts for IDs.** Sticking to this consistently is what makes a sheet look like infrastructure rather than a document.
