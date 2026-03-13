# /plan-project

A Claude Code slash command that plans and scaffolds new projects from scratch.

## Quick Start

1. Open a terminal in any directory
2. Start Claude Code
3. Type `/plan-project`
4. Describe your project idea

That's it. The skill handles everything else — research, planning, scaffolding, git init.

## How It Works

This repo is the **development workspace** for the `/plan-project` skill. The skill itself is installed globally at `~/.claude/skills/plan-project/`, which means you can run `/plan-project` from any Claude Code session, in any folder. You don't even need to create a project folder first — the skill asks where you want it and creates it for you.

When you run it, the skill walks you through 5 phases:

1. **Discovery** — asks follow-up questions to fill gaps (stack, deployment, integrations, auth, etc.)
2. **Agent-Driven Research** — 5 specialized sub-agents research stack, UI/UX, Claude infrastructure, dev workflow, and safety
3. **Plan Generation** — synthesizes agent outputs into a 19-section plan with a safety audit second pass
4. **Save Plan & Init Git** — saves `PROJECT_PLAN.md`, commits, optionally creates a GitHub repo
5. **Scaffold** — creates all files, installs dependencies, and makes the initial git commit

Plans survive context breaks via `.claude/plan-state.json` — the skill can resume from where it left off after `/clear`, `/compact`, or session interruptions.

## Sub-Agent Architecture

The skill uses 5 specialized sub-agents defined in `.claude/agents/`:

| Agent | Purpose | When |
|---|---|---|
| `stack-research` | Tech stack, libraries, deployment, architecture | Phase 2 Step 1 (sequential) |
| `frontend-design` | UI/UX, layouts, styling, navigation | Phase 2 Step 2 (parallel, only if project has a UI) |
| `claude-infrastructure` | CLAUDE.md, skills, hooks, permissions | Phase 2 Step 2 (parallel) |
| `dev-workflow` | Local dev, testing, CI/CD, deploy, ops handoff | Phase 2 Step 2 (parallel) |
| `safety` | Dependency audit, supply chain, security review | Phase 2 Step 2 + Phase 3 (runs twice) |

The main agent orchestrates sub-agents and synthesizes their outputs into the final `PROJECT_PLAN.md`.

## Installation

Clone and copy into your global Claude Code skills and agents directories:

```bash
git clone https://github.com/dwstein/Claude-Code-Project-Planner.git
cp -r Claude-Code-Project-Planner/plan-project ~/.claude/skills/plan-project
cp -r Claude-Code-Project-Planner/.claude/agents/* ~/.claude/agents/
```

## Developing the Skill

1. Edit files in this repo (`plan-project/SKILL.md`, `.claude/agents/`, etc.)
2. Open a new Claude Code session in this directory and run `/plan-project`

## What Gets Generated

Every scaffolded project includes:

- `PROJECT_PLAN.md` — the full implementation plan
- `CLAUDE.md` — project intelligence (under 150 lines)
- `.claude/settings.json` — permissions + safety hooks
- `.claude/skills/` — /test, /dev, /gsd (spec-driven), /ralph (autonomous loop)
- `.claude/skills/server/` — remote server operations (conditional — only for VPS/VM deploys)
- `.claude/skills/security-review/` — security analysis (conditional — only for projects with auth/APIs/user data)
- Source files, configs, `.env.example`, tests, Docker setup
- Dependencies installed, git initialized

## Hard Constraints

- Free and open source libraries only (MIT, Apache 2.0, BSD, etc.)
- Popular repos only (1,000+ GitHub stars preferred)
- Built-in solutions over third-party (node:test > Jest, fetch > axios)
- Every dependency justified with license, stars, and rationale
- Package names verified against registries to prevent typosquats

## Plan Sections

Every generated plan includes these 19 sections:

1. Overview
2. Stack
3. Changes from defaults
4. Dependencies table
5. Prerequisites
6. Project Structure
7. UI/UX Design (if applicable)
8. Boundaries (Always / Ask First / Never)
9. Code Style Example
10. Implementation Steps
11. Testing Strategy
12. Error Handling
13. Git Workflow
14. Build, Run & CI/CD Instructions
15. Claude Code Infrastructure
16. Deployment / Handoff Notes
17. Estimated Costs (if applicable)
18. Known Limitations & Future Phases
19. Security Considerations

## Stack Defaults

| Project Type | Default Stack |
|---|---|
| API / Bot | Node.js 22+, ESM, pino, node:test |
| Web Frontend | Next.js + React, TypeScript, Tailwind |
| Data Pipeline | Python 3.12+, uv, pytest |
| CLI Tool | Node.js (simple) or Python (complex) |
| Full-stack Web | Next.js monolith |

Defaults are overridable — web research may surface better options.

## Repo Structure

```
├── README.md              # This file
├── CLAUDE.md              # Project intelligence for this repo
├── .claude/
│   └── agents/
│       ├── stack-research.md       # Stack research sub-agent
│       ├── frontend-design.md      # Frontend design sub-agent
│       ├── claude-infrastructure.md # Claude Code infrastructure sub-agent
│       ├── dev-workflow.md         # Dev workflow sub-agent
│       └── safety.md              # Safety audit sub-agent
├── plan-project/
│   ├── SKILL.md           # The /plan-project skill definition
│   └── examples/
│       └── slack-notion-bot-plan.md  # Example plan output
└── docs/
    ├── conversation-summary.md       # How the skill was built
    └── example-plan.md              # Readable copy of the example plan
```

## Example Output

See [docs/example-plan.md](docs/example-plan.md) for a complete example of a generated plan (Slack-Notion bot with Claude tool use).

## License

MIT
