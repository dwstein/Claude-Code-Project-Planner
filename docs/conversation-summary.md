# Building the /plan-project Skill — Conversation Summary

**Date:** March 7, 2026

---

## What We Built

A reusable Claude Code skill (`/plan-project`) that automates the full lifecycle of setting up a new project. It lives at `~/.claude/skills/plan-project/` and is available globally across all projects.

### The Workflow

1. **Discovery Chat** — Discuss requirements, ask about gaps using a requirements checklist
2. **Web Research (mandatory)** — Search for best practices, latest versions, starter templates, and license verification
3. **Plan Generation** — Produce a comprehensive project plan with 17 required sections
4. **Scaffold Everything** — Create all folders, files, configs, skills, hooks, git init, and make an initial commit

---

## Key Files Created/Modified

| File | Purpose |
|---|---|
| `~/.claude/skills/plan-project/SKILL.md` | Main skill file (~490 lines) driving the 4-phase workflow |
| `~/.claude/skills/plan-project/examples/slack-notion-bot-plan.md` | Reference example plan that Claude follows when generating new plans |
| `~/Dropbox/.../Project Planer/project plan example.md` | Synced copy of the example plan |
| `~/.claude/skills/skill-creator/` | Anthropic's official skill-creator (18 files) for testing/iterating on skills |

---

## Hard Constraints Baked Into the Skill

- **Free and open source ONLY** — every dependency must have a permissive or copyleft OSS license
- **Popular, well-known repos only** — prefer 1k+ GitHub stars, active maintenance, high download counts
- **Favor built-ins** — `node:test` over Jest, `fetch` over axios, `node --watch` over nodemon
- **Supply chain safety** — verify package names, check for CVEs, avoid typosquats
- **Transparency** — every plan must include a dependency table with license, popularity, and rationale

---

## The 17 Required Plan Sections

Every generated plan must include:

1. Overview
2. Stack (justified by web research)
3. Changes from defaults (if any)
4. Dependencies table (name, version, license, stars/downloads, rationale + "not chosen" list)
5. Prerequisites
6. Project Structure
7. Boundaries (three-tier: Always do / Ask first / Never do)
8. Code Style Example (real code snippet, 10-15 lines)
9. Implementation Steps (with code snippets)
10. Testing Strategy (commands, mocking approach, philosophy)
11. Error Handling (startup vs. runtime vs. rate limits)
12. Git Workflow (branching, commit format)
13. Build & Run Instructions
14. Claude Code Infrastructure (CLAUDE.md, settings, skills including GSD + Ralph, hooks)
15. Deployment / Handoff Notes
16. Estimated Costs (if applicable)
17. Known Limitations & Future Phases

---

## What Gets Scaffolded

When a plan is approved, the skill creates:

- **Project directory + git init**
- **`PROJECT_PLAN.md`** — the full plan, committed first
- **`CLAUDE.md`** — project intelligence file (under 150 lines)
- **`.claude/settings.json`** — permissions scoped to the stack + hooks
- **`.claude/settings.local.json`** — empty, gitignored, for personal overrides
- **Skills:**
  - `/test` — run the test suite
  - `/dev` — start local development
  - `/gsd` — spec-driven development (breaks work into atomic plans, runs each in fresh sub-agent)
  - `/ralph` — autonomous agent loop (iterates against task list until all items complete)
  - Project-specific skills as needed (`/deploy`, `/lint`, `/db-migrate`, etc.)
- **Hooks:**
  - `PreToolUse` on Bash — blocks destructive commands (`rm -rf`, `DROP TABLE`, `--force`)
  - `PostToolUse` on Edit/Write — auto-format (Prettier for JS/TS, ruff for Python)
- **Source files, configs, `.env.example`, `.gitignore`, test scaffolding**
- **Dependencies installed** (`npm install` or `uv sync`)
- **Initial git commit** with descriptive message

---

## Improvements Made to the Example Plan

The example plan (`slack-notion-bot-plan.md`) was improved with 8 additions based on research from:

- Addy Osmani's guide (synthesizing GitHub's analysis of 2,500+ agent config files)
- Claude Code spec-driven workflow patterns
- AI-optimized PRD best practices

### Improvements:

1. **Dependency table** — license, stars, downloads, rationale for every package
2. **Three-tier boundaries** — Always/Ask First/Never (from GitHub's 2,500-repo analysis)
3. **Code style example** — real 15-line JavaScript snippet showing canonical pattern
4. **Testing strategy** — commands, mock vs. direct, philosophy
5. **Error handling strategy** — table of failure types with behavior and examples
6. **Git workflow** — branch naming, conventional commits
7. **GSD + Ralph skills** — added to project structure and infrastructure
8. **Hooks configuration** — PreToolUse destructive command blocker with full JSON config

---

## Anthropic Skill-Creator (Installed)

Installed the official [anthropics/skills](https://github.com/anthropics/skills) `skill-creator` at `~/.claude/skills/skill-creator/`. It provides:

- A structured workflow for testing and iterating on skills (Draft > Test > Grade > Improve)
- 3 specialized agents: analyzer, comparator, grader
- 9 Python scripts for eval, benchmarking, and reporting
- Can be used to evaluate `/plan-project` and any future skills

---

## Stack Opinions (Defaults)

| Project Type | Default Stack |
|---|---|
| API / Bot | Node.js 22+, ESM, pino, node:test |
| Web Frontend | Next.js + React, TypeScript, Tailwind |
| Data Pipeline | Python 3.12+, uv, pytest |
| CLI Tool | Node.js (simple) or Python (complex) |
| Automation | Python + cron or Node.js |
| Full-stack Web | Next.js (frontend + API routes) |

These are overridable — web research may surface better options for specific projects.

---

## How to Use

```
/plan-project
```

Then describe your project idea. The skill handles the rest.
