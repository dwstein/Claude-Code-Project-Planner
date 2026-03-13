# Project Planner

This repo is the canonical source for the `/plan-project` Claude Code skill.

## Key Files

- `plan-project/SKILL.md` — the skill definition (source of truth)
- `plan-project/examples/slack-notion-bot-plan.md` — bundled example plan
- `.claude/agents/` — sub-agent definitions (stack-research, frontend-design, claude-infrastructure, dev-workflow, safety)
- `docs/` — build history and readable example copy

## How to Test Changes

1. Edit files in this repo (`plan-project/SKILL.md`, `.claude/agents/`, etc.)
2. Open a new Claude Code session in this directory and run `/plan-project`

## Conventions

- `plan-project/` contains the skill definition and examples
- Sub-agent definitions live in `.claude/agents/` as standalone `.md` files with YAML frontmatter
- Example plans go in `plan-project/examples/`

## Agent Architecture

The skill uses 5 sub-agents defined in `.claude/agents/` during Phases 2-3:

1. **`stack-research`** — researches tech stack, libraries, deployment, architecture (Phase 2, Step 1 — sequential)
2. **`frontend-design`** — designs UI/UX, layouts, styling, navigation (Phase 2, Step 2 — parallel, only if project has a UI)
3. **`claude-infrastructure`** — designs CLAUDE.md, skills, hooks, permissions (Phase 2, Step 2 — parallel)
4. **`dev-workflow`** — designs local dev, testing, CI/CD, deploy procedure, ops handoff (Phase 2, Step 2 — parallel)
5. **`safety`** — audits dependencies and reviews the final plan (Phase 2 Step 2 + Phase 3 — runs twice)

The main agent orchestrates sub-agents and synthesizes their outputs into the final PROJECT_PLAN.md.

### Plan Persistence

Plans survive context breaks via `.claude/plan-state.json` (per-project). The file tracks which phase completed last and stores all agent outputs, so the skill can resume from where it left off after `/clear`, `/compact`, or session interruptions.
