# Toolforest Skills Marketplace

Agent skills for [Toolforest](https://toolforest.io) MCP toolkits — Google Slides, Docs, Sheets, and more.

These skills teach Claude best practices, gotchas, and verified patterns for building content through Toolforest's MCP connectors.

## Installation

### Via Claude Code / Cowork marketplace:
```
/plugin marketplace add toolforest-io/toolforest-skills-marketplace
/plugin install google-slides-toolforest@toolforest-skills
```

### Manual installation:
```bash
git clone https://github.com/toolforest-io/toolforest-skills-marketplace.git
cp -r toolforest-skills-marketplace/plugins/google-slides-toolforest ~/.claude/skills/
```

## Available Skills

### google-slides-toolforest
Best practices for building Google Slides presentations via Toolforest MCP. Includes:
- Core workflow (create → theme → build → verify → iterate)
- Critical gotchas (autofit booleans, hex encoding, gradient support, font availability)
- Design patterns (header bars, card grids, KPI callouts, two-column splits)
- Font pairing recommendations by audience/tone
- Google Sheets → Slides chart pipeline
- Slide verification checklist

Requires: Toolforest Google Slides MCP connector

## Adding New Skills

To add a new toolkit skill (e.g., Google Docs, Sheets, Gmail):
1. Create a new directory under `plugins/`
2. Add a `.claude-plugin/plugin.json` with the skill metadata
3. Add a `SKILL.md` with frontmatter and the core workflow
4. Add `references/` files for progressive disclosure
5. Update `marketplace.json` to include the new plugin

## License

MIT
