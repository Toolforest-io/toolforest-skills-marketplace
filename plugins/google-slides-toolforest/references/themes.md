Read this file when the user's request implies a specific audience or mood (boardroom, creative pitch, data-heavy report, high-energy launch) — or whenever the user explicitly asks for a theme other than the default Toolforest forest-green.

The default theme remains the one defined in `config.json`. The presets below are alternatives to apply via `set_theme` when the audience clearly calls for something different. Each preset has been built and verified end-to-end against `get_slide_content_elements`.

---

## Universal Cross-Theme Rules

These rules apply regardless of which preset you pick. They were confirmed across multiple themes during the build.

### 1. Two tones per saturated accent

Every saturated accent color needs two variants — one for dark backgrounds, one for light. The bright variant fails WCAG body-text contrast on light backgrounds. Pick the right tone for the surface the text sits on.

| Accent | Bright (use on dark bg) | Deep (use on light bg) |
|---|---|---|
| Sunset terracotta | `#C2522D` | `#A8421F` |
| Arctic cyan | `#0EA5C7` | `#066B85` |
| Arctic warn | `#E8703A` | `#B8521E` |
| Tokyo magenta | `#FF2D7B` | `#C71F5F` |

### 2. `min_font_size_pt: 10` on every body text in a tight card

Without the explicit floor, autofit will silently scale below 10pt and violate the skill's hard rule. Pass `min_font_size_pt: 10` on every `add_text_box` call where the box might not fit the content.

### 3. Decorative oversized glyphs need `autofit: false`

Drop-cap quote marks, big italic numerals, condensed display type at 96pt+ — autofit will scale them below the 0.7 floor every time. Use `autofit: false` and a generously sized box. The verifier may still report `estimatedOverflow: true` for these because display fonts have tall internal metrics; the visible glyph fits fine. Visual check overrides the warning for genuinely decorative glyphs only — never for body text.

### 4. The same-font-heading-body rule has one exception

The general rule in `references/fonts.md` ("never use the same font for heading and body") still holds — *unless* the theme intentionally uses a strong third typographic element (typically a mono font) that carries semantic weight. Arctic Mono, for example, uses IBM Plex Sans for both heading and body but pairs it with IBM Plex Mono carrying labels, data, and coordinate strips — that mono voice is the real second voice.

### 5. `muted_on_dark` must clear 4.5:1 against the dark background

A muted slate around `#6B7A99` fails WCAG body-text contrast against deep navy/indigo at 10pt (~4.09:1). The default `config.json` ships a lighter slate (`#A8C4D4`) that clears the bar at ~8.8:1. If you customize the muted-on-dark color for a theme, verify the contrast before shipping.

### 6. Multi-shape compositions are fine

Layered bar-chart compositions (label + track + bar + percentage rows, repeated) pass verification first try. The verifier does not flag layered track + bar overlap when the bar's width is shorter than the track. This is a viable pattern when `embed_chart` (Sheets dependency) is overkill.

---

## Theme: Midnight Editorial

**When to use:** Board decks, annual reports, executive briefings — anywhere the deck needs to read like a quarterly report from a serious publication.

**Palette**
- `background_dark: #0F1535` — deep midnight navy, primary slide background
- `background_light: #F5EDE0` — warm cream, secondary surfaces and section breaks
- `card_dark: #1A2247` — slightly lighter navy for layered cards on dark slides
- `accent: #C9A961` — muted editorial gold; rules, drop-cap glyphs, caps labels only
- `text_on_dark: #F5EDE0` — cream body text on navy
- `muted_on_dark: #A8B5CC` — secondary text on navy (clears 4.5:1)
- `text_on_light: #0F1535` — navy body text on cream
- `muted_on_light: #5A6B7A` — secondary text on cream

**Fonts**
- heading: Playfair Display
- body: Lato

**Signature moves**
- Chapter marks above titles (`01 — THE STATE OF PLAY`) in caps Lato, gold
- Roman numerals (I., II., III.) in italic Playfair as card markers
- Oversized italic serif for big numbers — `autofit: false`, generously sized box
- Thin gold rules (~11,400 EMU thick) as section dividers — short (1–2 inches), not full-width
- Italic serif pull quotes with selective bold for emphasis
- Drop-cap opening quote marks at 96pt+ in gold, set `autofit: false`

**Theme-specific gotchas**
- Gold `#C9A961` on cream is decorative-only — it fails WCAG for body text. Use navy on cream for readable text and reserve gold for rules, drop-cap glyphs, and short caps labels.
- Italic Playfair at large sizes has tall internal metrics; expect `estimatedOverflow: true` on decorative numerals (apply universal rule #3).

---

## Theme: Sunset Terracotta

**When to use:** Creative pitches, lifestyle brands, culture decks — anywhere corporate-blue feels cold.

**Palette**
- `background_light: #F5EDE2` — primary cream surface
- `background_dark: #3D2817` — espresso brown for high-contrast slides
- `card_warm: #EBD9C4` — layered cream card on the cream background
- `accent_bright: #C2522D` — bright terracotta for dark-bg accents (universal rule #1)
- `accent_deep: #A8421F` — deep terracotta for light-bg accents
- `accent_secondary: #E89B5C` — warm sand, sparingly
- `text_on_light: #3D2817` — espresso body text on cream
- `muted_on_light: #6B543F` — secondary text on cream
- `text_on_dark: #F5EDE2` — cream body text on espresso

**Fonts**
- heading: DM Serif Display
- body: Karla

**Signature moves**
- Large saturated terracotta blocks bleeding to slide edges
- Numbered markers as caps + slash separators (`ONE / THE NUMBER`)
- Final-word color highlight in titles (the "loud rooms" / "nerve" trick — only the last word in a different color)
- Staggered card y-positions for asymmetry rather than rigid grid alignment
- Short serif pull quotes with the final word in bright terracotta

**Theme-specific gotchas**
- Pure bright terracotta `#C2522D` as a card background fails WCAG with cream body text. Either deepen the card to `#A8421F` or use the bright terracotta only as accent (line, marker, single word), never as a card surface that holds body text.
- On layered cream cards (`#EBD9C4`), terracotta markers fail contrast — switch to espresso for those cards.

---

## Theme: Arctic Mono

**When to use:** Engineering reports, eval analyses, postmortems — anything that needs to feel like receipts.

**Palette**
- `background_light: #F4F6F8` — near-white slate surface
- `background_dark: #1B2330` — deep slate dark surface
- `card_subtle_light: #E4E9EE` — barely-darker card on light bg
- `card_subtle_dark: #283342` — barely-lighter card on dark bg
- `accent_bright: #0EA5C7` — arctic cyan for dark-bg accents
- `accent_deep: #066B85` — deep cyan for light-bg accents
- `warn_bright: #E8703A` — warm warning color, dark-bg
- `warn_deep: #B8521E` — warm warning color, light-bg
- `muted_on_light: #5C6B7E`
- `muted_on_dark: #9AB0C6`
- `grid: #C9D2DD` — hairlines and grid strokes

**Fonts**
- heading: IBM Plex Sans
- body: IBM Plex Sans
- data/labels: IBM Plex Mono

**Signature moves**
- Mono coordinate strip at top of every slide (`[ 03 / 12 ] context`)
- `// SECTION` comment-style kickers in mono above section titles
- `key = value` blocks with `=` aligned (mono makes alignment free)
- Hairlines at 6,350 EMU (= 0.5pt) at top and bottom of every slide
- Single warm color (`warn_bright` / `warn_deep`) reserved exclusively for outliers and warnings — never decorative
- Bar charts via track + bar rectangle pairs (see universal rule #6)
- Sign-off block at bottom of closing slides, styled like a git commit trailer (mono, muted, two or three lines)

**Theme-specific gotchas**
- This theme intentionally breaks the same-font-heading-body rule because Plex Mono carries the second voice (see universal rule #4). The mono is not optional — without it, the deck loses its identity.
- Reserve the warm warn color for actual outliers; using it decoratively defeats its semantic role.

---

## Theme: Tokyo Neon

**When to use:** Product launches, hackathon recaps, conference keynotes, internal hype decks.

**Palette**
- `background_dark: #0A0E1A` — near-black, primary surface
- `bg_card: #161B2D` — layered card on near-black
- `background_light: #F5F4F0` — warm off-white for occasional light slides
- `magenta_bright: #FF2D7B` — dark-bg magenta accent
- `magenta_deep: #C71F5F` — light-bg magenta accent
- `cyan: #00E5FF` — dark-bg only
- `yellow: #FFD600` — third color, sparing (one accent per slide max)
- `text_on_dark: #F5F4F0`
- `muted_on_dark: #8A9BB8`
- `text_on_light: #0A0E1A`
- `muted_on_light: #5A6680`

**Fonts**
- heading: Oswald
- body: Roboto

**Signature moves**
- Diagonal magenta + cyan rotated rectangles as hero accents (rotation via shape transform)
- Condensed display type at 96pt+ for impact (Oswald carries this beautifully)
- Big-number narratives with strikethrough on the "old" value — apply via `update_element_style`, NOT inside `add_text_box`
- Chevron shape between before/after values
- All-caps Oswald headlines paired with sentence-case Roboto body
- Yellow used at most once per slide as a single highlighted glyph or rule

**Theme-specific gotchas**
- `add_text_box` does not accept `strikethrough` in its TextRun schema — apply with `update_element_style` after the box exists.
- Cyan `#00E5FF` only works on dark backgrounds. On the off-white slide, use `magenta_deep` (`#C71F5F`) instead — the bright cyan blows out.
- Yellow as a third color must stay sparing — one accent per slide max, or the deck reads chaotic instead of energetic.
