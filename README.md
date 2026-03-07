# /plan-project

A Claude Code skill that plans and scaffolds new projects with full infrastructure for autonomous agent workflows.

Give it a project idea, and it will research best practices, generate a comprehensive plan, and scaffold everything — source files, tests, Docker, CLAUDE.md, settings, hooks, and agent workflow skills (/gsd + /ralph).

## Installation

Clone the repo and copy the skill into your global Claude Code skills directory:

```bash
git clone https://github.com/dwstein/Project-Planner.git
cp -r Project-Planner/skill ~/.claude/skills/plan-project
```

## Usage

Type `/plan-project` in any Claude Code session — it's a global skill, so it works from any directory.

```
/plan-project
```

Then describe your project idea (e.g., "I want to build a CLI tool that tracks expenses"). The skill walks you through 4 phases:

1. **Discovery** — asks follow-up questions to fill gaps (stack, deployment, integrations, auth, etc.)
2. **Web Research** (mandatory) — searches for latest versions, best practices, and licenses
3. **Plan Generation** — produces a full plan with 17 required sections and asks for your approval
4. **Scaffold** — asks where to create the project, then creates the folder, scaffolds all files, installs dependencies, and makes the initial git commit

You don't need to create a project folder first — the skill asks you for a path and creates it for you. Just bring an idea, the skill handles the rest.

### Updating the skill

To improve the skill over time:

1. Edit `skill/SKILL.md` in this repo
2. Copy it to the installed location: `cp -r skill/ ~/.claude/skills/plan-project/`
3. Test by running `/plan-project` in a new Claude Code session

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
