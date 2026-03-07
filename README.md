# /plan-project

A Claude Code skill that plans and scaffolds new projects with full infrastructure for autonomous agent workflows.

Give it a project idea, and it will research best practices, generate a comprehensive plan, and scaffold everything — source files, tests, Docker, CLAUDE.md, settings, hooks, and agent workflow skills (/gsd + /ralph).

## Installation

Copy (or symlink) the skill into your global Claude Code skills directory:

```bash
# Clone
git clone https://github.com/dwstein/Project-Planner.git

# Copy skill to Claude Code
cp -r Project-Planner/skill ~/.claude/skills/plan-project
```

Or symlink to keep it synced with the repo:

```bash
ln -s "$(pwd)/Project-Planner/skill" ~/.claude/skills/plan-project
```

## Usage

In any Claude Code session:

```
/plan-project
```

Then describe your project. The skill handles 4 phases:

1. **Discovery** — discusses requirements, asks about gaps
2. **Web Research** (mandatory) — searches for latest versions, best practices, licenses
3. **Plan Generation** — produces a full plan with 17 required sections
4. **Scaffold** — creates all files, installs dependencies, git init

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
