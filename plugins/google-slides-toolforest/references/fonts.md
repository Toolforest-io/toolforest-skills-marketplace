Read this file when choosing typography for a presentation. All fonts must be from the Google Fonts catalog — anything else falls back silently.
## Safe Fonts (Confirmed Working)
- Google Fonts: Montserrat, Inter, Roboto, Lato, Open Sans, Poppins, Playfair Display, Raleway, Oswald, Source Sans 3, DM Sans, Space Grotesk, IBM Plex Sans, IBM Plex Mono, Merriweather, Archivo, Nunito, DM Serif Display, Karla, Roboto Mono
- System fonts (widely available): Arial, Calibri, Trebuchet MS, Georgia
- Will NOT render: Proxima Nova, Avenir, Helvetica Neue, Futura, SF Pro (licensed — fall back silently)

## Recommended Pairings by Audience
Always use contrasting fonts for heading vs body — never the same font at different weights.
- Executive / Boardroom: Playfair Display + Lato — Serif heading adds editorial authority
- Tech / Startup: Poppins + Inter — Round + neutral = modern, approachable
- Creative / Design: DM Serif Display + Karla — Distinctive serif + humanist sans
- Corporate / Consulting: Montserrat + Source Sans 3 — Geometric + humanist = clean, professional
- Data-heavy / Analytical: IBM Plex Sans + IBM Plex Mono — Structured, precise, monospace for data
- Bold / Impactful: Oswald + Roboto — Condensed heading creates weight contrast

## Font Selection Gotchas
- Never use the same font for heading and body — even at different weights, this creates no real visual contrast
- Don’t pair two geometric sans-serifs (e.g., Montserrat + Poppins) — they compete rather than complement
- Display/decorative fonts (Lobster, Pacifico, Permanent Marker) are headings-only — never use for body text
- Body fonts must be legible at 14–16pt — test mentally: would this be comfortable in a paragraph?
- Always set both heading and body fonts via set_theme at the start — never rely on the default

## PPTX Comparison Note
pptxgenjs can embed any font into the .pptx file, so recipients see the correct typeface without installing the font. Google Slides can’t do this — but Google Fonts render identically for every viewer since they’re served from Google’s CDN. This is a tradeoff, not a limitation.
