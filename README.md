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

When you run it, the skill walks you through 4 phases:

1. **Discovery** — asks follow-up questions to fill gaps (stack, deployment, integrations, auth, etc.)
2. **Web Research** (mandatory) — searches for latest versions, best practices, and licenses
3. **Plan Generation** — produces a full plan with 17 required sections and asks for your approval
4. **Scaffold** — creates the folder, scaffolds all files, installs dependencies, and makes the initial git commit

## Installation

Clone and copy the skill into your global Claude Code skills directory:

```bash
git clone https://github.com/dwstein/Claude-Code-Project-Planner.git
cp -r Claude-Code-Project-Planner/skill ~/.claude/skills/plan-project
```

## Developing the Skill

This repo is where you edit and test changes to the skill:

1. Edit `skill/SKILL.md` in this repo
2. Copy to the installed location: `cp -r skill/ ~/.claude/skills/plan-project/`
3. Test by running `/plan-project` in a new Claude Code session

Or symlink so edits are reflected immediately: `ln -s "$(pwd)/skill" ~/.claude/skills/plan-project`

## What Gets Generated

Every scaffolded project includes:

- `PROJECT_PLAN.md` — the full implementation plan
- `CLAUDE.md` — project intelligence (under 150 lines)
- `.claude/settings.json` — permissions + safety hooks
- `.claude/skills/` — /test, /dev, /gsd (spec-driven), /ralph (autonomous loop)
- Source files, configs, `.env.example`, tests, Docker setup
- Dependencies installed, git initialized

## Hard Constraints

- Free and open source libraries only (MIT, Apache 2.0, BSD, etc.)
- Popular repos only (1,000+ GitHub stars preferred)
- Built-in solutions over third-party (node:test > Jest, fetch > axios)
- Every dependency justified with license, stars, and rationale
- Package names verified against registries to prevent typosquats

## Plan Sections

Every generated plan includes: Overview, Stack, Dependencies table, Prerequisites, Project Structure, Boundaries (Always/Ask First/Never), Code Style Example, Implementation Steps, Testing Strategy, Error Handling, Git Workflow, Build & Run, Claude Code Infrastructure, Deployment Notes, Estimated Costs, Known Limitations & Future Phases.

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
├── skill/
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
