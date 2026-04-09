# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

An open-source collection of Claude Code skills published by [青枫数科 (MapleAI-Tech)](https://github.com/MapleAI-Tech). Skills are self-contained `SKILL.md` files that define behavior for the Claude Code CLI. There is no build system, no package manager, and no tests — each skill is a single Markdown file with structured XML-style sections.

## Repository Layout

```
skills/
├── {skill-name}/
│   └── SKILL.md      # The entire skill definition
```

Users install by copying skill directories into `~/.claude/skills/`.

## Adding a New Skill

1. Create `skills/{name}/SKILL.md`
2. Add a row to the skills table in `README.md`
3. Add the `cp` line to the install section in `README.md`
4. Add usage examples under a new `## {name}` heading in `README.md`

## SKILL.md Structure Convention

Every skill file uses this structure (see existing skills for reference):

- **Frontmatter** (`name`, `description`) — used for skill registry
- `<Purpose>` — one-sentence mission
- `<Use_When>` / `<Do_Not_Use_When>` — trigger conditions
- `<Project_Structure>` — files and directories the skill manages
- `<Commands>` — each slash command with step-by-step instructions
- `<Tool_Usage>` — which Claude Code tools the skill uses
- `<Examples>` — `<Good>` and `<Bad>` with "Why good/bad" annotations
- `<Escalation_And_Stop_Conditions>` — edge cases and guardrails
- `<Final_Checklist>` — verification items

## Existing Skills

| Skill | Purpose |
|-------|---------|
| **chuxin** | Product brain — roadmap init, daily briefings, competitive research |
| **doc-align** | Document alignment — scan, index, and archive all `.md` files into `/docs` |
