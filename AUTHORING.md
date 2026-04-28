# Authoring a new skill

This guide documents the conventions for adding a new skill to the Toolforest Skills Marketplace, so future skill PRs land with the right shape from the start. Existing skills (`google-slides-toolforest`, `google-sheets-toolforest`) are reference implementations.

## Directory layout

Each skill lives under `plugins/<skill-name>/`:

```
plugins/<skill-name>/
├── .claude-plugin/
│   └── plugin.json          # Claude Code plugin manifest
├── SKILL.md                 # Main skill content (with YAML frontmatter)
├── references/              # Optional: progressive-disclosure files
│   ├── topic_a.md
│   └── topic_b.md
└── scripts/                 # Optional: helper scripts the skill references
    └── verify_thing.md
```

The `<skill-name>` should be the same value used everywhere (`name` in `plugin.json`, `name` in the SKILL.md frontmatter, `name` in the marketplace.json plugin entry).

## SKILL.md frontmatter

Every `SKILL.md` starts with a YAML frontmatter block. This is what Claude Code's skill auto-loader reads to decide when to surface the skill.

```yaml
---
name: <skill-name>
description: >
  Use when <one-line task description>. Triggers on: <tool_a>, <tool_b>,
  <tool_c>, or any request to <intent prose — what the user is trying
  to accomplish, in their words> using <toolkit> through the Toolforest
  connector.
---
```

Two parts to the description, both load-bearing:

1. **`Triggers on: <tool list>`** — enumerate the specific tools this skill has guidance for. Be generous: if the skill has even a paragraph of advice on a tool, include it. Missing tools mean the skill won't load when the user is doing work the skill could help with. Excess tools cost little (the dispatcher uses the description as one input among several).
2. **Intent prose** — what the *user* is trying to accomplish, framed in user vocabulary ("multi-tab dashboard", "scorecard", "rollup"), not API vocabulary. This is what catches tasks that don't match a tool name literally — for example, a user asking "build me a tracker" before any tool call has happened.

Include both. The trigger list helps the dispatcher when a tool call is already in flight; the intent prose helps it when the user is describing what they want at the start of a session.

## Marketplace summary

The user-facing one-liner that shows up in `skills-list_skills` lives in two places:

1. **`.claude-plugin/marketplace.json`** — the `description` field of the plugin entry. This is the canonical source for what users see when browsing the marketplace.
2. **`plugins/<skill-name>/.claude-plugin/plugin.json`** — also has a `description` field. Keep these in sync to avoid drift.

The summary should follow the convention established by the Sheets skill:

> Best practices for building <thing> via Toolforest MCP. <Three to six concrete capability phrases, comma-separated>, and <one more>, — all derived from end-to-end builds verified to render cleanly.

Three components:

1. **Anchor:** "Best practices for building <thing> via Toolforest MCP." Tells the reader what the skill is *for*.
2. **Capability list:** five-ish concrete items the skill covers. Specific is better than abstract — "theme presets" beats "design guidance"; "batch-wrapper gotchas" beats "API patterns". If the skill has a notable signature pattern (e.g. "the PERCENT-format trap"), include it by name.
3. **Trust signal:** the "— all derived from end-to-end builds verified to render cleanly" suffix tells the reader the patterns aren't speculative. Keep this exact phrasing for consistency across skills.

Avoid the "Includes X, Y, Z" cadence — it reads as a feature list rather than a positioning statement.

## Marketplace registration

After creating the skill files, register the plugin in `.claude-plugin/marketplace.json`:

```json
{
  "name": "<skill-name>",
  "source": "./plugins/<skill-name>",
  "description": "<the marketplace summary, exactly>",
  "version": "1.0.0",
  "category": "productivity",
  "tags": ["<toolkit>", "<domain>", "toolforest", "mcp"],
  "strict": true
}
```

## README

Add a short bullet section to the top-level `README.md` under "Available Skills" listing what the skill covers. The README format is more verbose than the marketplace summary (a feature bullet list is fine here) — the README is for repo browsers, the marketplace summary is for skill-discovery.

## Body content conventions

The body of `SKILL.md` (everything after the frontmatter) is what `skills-get_skill` returns. A few conventions that have held up across multiple skills:

- **Lead with "When to load this skill."** A short bulleted list of the tasks the skill is for, plus an explicit "Skip it for X" so the model doesn't load on trivial work.
- **A "default theme" section** if the skill produces visual output (slides, sheets, docs). Encode the brand palette plus a small set of audience-mapped presets.
- **Numbered patterns ("The patterns that matter")** rather than alphabetical reference. Order by importance to a typical build.
- **A "common gotchas" quick-reference section** at the end. One-line entries for the footguns that don't merit their own pattern.
- **A verification checklist** as the last section. Final-handoff checks the model should run before declaring the work done.
- **Reference files for depth.** Anything longer than ~200 words on a single topic goes in `references/<topic>.md` and is linked from the SKILL.md by relative path. This keeps the SKILL.md scannable.

## Verifying a new skill

Before opening a PR:

1. JSON files are valid (`python3 -c "import json; json.load(open(...))"` on each).
2. `SKILL.md` frontmatter parses and contains both `name` and `description`.
3. All `references/<file>.md` paths referenced from `SKILL.md` resolve.
4. No personal references, no client/account names, no internal-only URLs.
5. Smoke-test by installing the marketplace locally and confirming `skills-list_skills` shows the new skill with the expected summary.
