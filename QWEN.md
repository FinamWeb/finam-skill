# Finam Trade API — Qwen Code Extension

## Project Purpose

This repository contains skills for Finam Trade API — a trading API for accessing market data, managing orders and portfolios, and developing algorithmic trading scripts through Finam broker.

This repository also serves as the Claude marketplace (`finam-skill`), Codex marketplace (`finam-skill-marketplace`), and Cursor marketplace (`finam-cursor-marketplace`). GitHub source: `FinamWeb/finam-skill`.

## Language

Official language: **English**.

## Structure

- `.claude-plugin/marketplace.json` — Claude Code marketplace registry (plugin catalog)
- `.claude-plugin/plugin.json` — Claude Code plugin manifest
- `.codex-plugin/plugin.json` — Codex plugin manifest
- `.agents/plugins/marketplace.json` — Codex marketplace registry
- `.cursor-plugin/plugin.json` — Cursor plugin manifest
- `.cursor-plugin/marketplace.json` — Cursor marketplace registry
- `qwen-extension.json` — Qwen Code extension manifest
- `skills/{skill-name}/SKILL.md` — skill definitions
- Each skill = one directory with a `SKILL.md` file

## Namespace

Extension name: `finam`. Skills appear as `finam:{skill-name}`.

GitHub source: `FinamWeb/finam-skill`.

## Adding New Skills

1. Create `skills/{skill-name}/SKILL.md`
2. Include YAML frontmatter: `name`, `description`
3. Update README.md skills table
4. Bump version in `qwen-extension.json`, `.claude-plugin/plugin.json`, `.codex-plugin/plugin.json`, `.cursor-plugin/plugin.json`, `.cursor-plugin/marketplace.json`, and `.claude-plugin/marketplace.json` (both top-level and plugin entry)
5. Update marketplace plugin entry in `.claude-plugin/marketplace.json` if skill name or description changed

## Updating Skills

1. Edit `skills/{skill-name}/SKILL.md`
2. Preserve YAML frontmatter structure
3. Bump patch version in `qwen-extension.json`, `.claude-plugin/plugin.json`, `.codex-plugin/plugin.json`, `.cursor-plugin/plugin.json`, `.cursor-plugin/marketplace.json`, and `.claude-plugin/marketplace.json` (both top-level and plugin entry)
4. If skill name or description changed — update README.md skills table and marketplace plugin entry in `.claude-plugin/marketplace.json`

## Skill Description Quality

The `description` field in SKILL.md frontmatter is the PRIMARY auto-activation signal. Always include: all name variants, technology keywords, task verbs, "Use when..." trigger, and scope markers.
