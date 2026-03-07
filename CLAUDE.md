# Project Planner

This repo is the canonical source for the `/plan-project` Claude Code skill.

## Key Files

- `skill/SKILL.md` — the skill definition (source of truth)
- `skill/examples/slack-notion-bot-plan.md` — bundled example plan
- `docs/` — build history and readable example copy

## How to Test Changes

1. Edit `skill/SKILL.md` in this repo
2. Copy to the installed location: `cp -r skill/ ~/.claude/skills/plan-project/`
3. Open a new Claude Code session and run `/plan-project`

If using a symlink (`ln -s`), changes are reflected immediately.

## Conventions

- The `skill/` directory mirrors `~/.claude/skills/plan-project/` exactly
- Keep `SKILL.md` self-contained — no external file dependencies beyond the examples dir
- Example plans go in `skill/examples/`
