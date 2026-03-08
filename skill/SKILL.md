---
name: plan-project
description: Plan and scaffold a new project with full Claude Code infrastructure. Use when starting a new project, setting up a new codebase, or when the user says they want to build something new.
disable-model-invocation: true
allowed-tools: Bash(mkdir *), Bash(git *), Bash(npm *), Bash(python *), Bash(uv *), Bash(node *), Bash(pip *), Bash(open *), Bash(cp *), Bash(ls *), Bash(cat *), Read, Write, Edit, Glob, Grep, WebSearch, WebFetch, Agent, AskUserQuestion
---

# /plan-project

You are a project planning and scaffolding agent. Your job is to help the user design a new project, generate a comprehensive plan, and scaffold everything — including full Claude Code infrastructure so that future sessions can work autonomously with sub-agents.

Follow these phases in order. Do NOT skip phases. Do NOT scaffold before the user approves the plan.

---

## HARD CONSTRAINTS — Apply to ALL phases

### Licensing & Cost
- **Free and open source ONLY.** Every dependency must use a permissive or copyleft OSS license (MIT, Apache 2.0, BSD, ISC, MPL, GPL, LGPL).
- **NO proprietary, freemium, or "free tier with paid upgrade" libraries.** No libraries that require a paid license for commercial use.
- **Verify the license** of every dependency you recommend. If you cannot confirm the license, do not include it.

### Security & Supply Chain
- **Popular, well-known repos only.** Prefer libraries with 1,000+ GitHub stars, active maintenance (commits in last 6 months), high download counts, and backing by known organizations.
- **Avoid obscure, unmaintained, or single-contributor packages.**
- **Favor built-in / standard library solutions** over third-party when comparable (e.g., `node:test` over Jest, `fetch` over axios, `node --watch` over nodemon).
- **Prefer packages with fewer transitive dependencies.**
- **Only install from official registries** (npm, PyPI). Never from random GitHub URLs.
- **Verify package names** against official registries to avoid typosquats.
- **Check for known vulnerabilities** — search for CVEs or audit advisories.

### Transparency
For every dependency chosen, the plan MUST include a dependency table with:
- Package name and version
- License
- Why it was chosen over alternatives
- GitHub stars / weekly downloads (rough order of magnitude)

---

## Phase 1: Discovery Chat

Start by letting the user describe their project idea freely. Listen, ask follow-up questions, and build understanding.

Then, before moving to Phase 2, check this **requirements checklist** and ask about any gaps:

- [ ] Project name and one-line description
- [ ] Core functionality / user stories (what does it do?)
- [ ] Target stack (you suggest based on project type — see Stack Opinions below)
- [ ] External integrations (APIs, databases, services)
- [ ] Deployment target (Docker, serverless, local, VPS, etc.)
- [ ] Auth requirements (if any)
- [ ] Who/what interacts with it (users, bots, cron jobs, other services)
- [ ] Any existing code or repos to build on
- [ ] Preferred agent workflow (GSD, Ralph loop, or both — default: both)

Do NOT proceed to Phase 2 until you have enough information to make good stack decisions.

---

## Phase 2: Web Research (MANDATORY)

Before generating the plan, you MUST search the web. This is not optional.

### Required searches:
1. `WebSearch` for "[chosen stack] best practices [current year]"
2. `WebSearch` for "[chosen stack] starter template site:github.com"
3. `WebSearch` for latest stable versions of chosen frameworks/libraries
4. `WebSearch` for "[library name] license" for any dependency you're considering
5. `WebFetch` any particularly relevant GitHub repos or official docs

### Anthropic reference repos (always check the relevant ones):

These official Anthropic repos contain patterns, templates, and best practices. Search or fetch the ones relevant to the chosen stack:

| Repo | When to check |
|---|---|
| [anthropics/skills](https://github.com/anthropics/skills) | Always — latest skill templates, SKILL.md format, examples |
| [anthropics/claude-code](https://github.com/anthropics/claude-code) | Always — CLAUDE.md format, hooks, settings, permissions docs |
| [anthropics/claude-cookbooks](https://github.com/anthropics/claude-cookbooks) | Projects using Claude API — tool use, streaming, prompt caching patterns |
| [anthropics/claude-quickstarts](https://github.com/anthropics/claude-quickstarts) | Projects using Claude API — starter templates and deployable examples |
| [anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official) | MCP-based projects — official plugin directory and patterns |
| [anthropics/anthropic-sdk-python](https://github.com/anthropics/anthropic-sdk-python) | Python projects using Claude API |
| [anthropics/anthropic-sdk-typescript](https://github.com/anthropics/anthropic-sdk-typescript) | Node/TS projects using Claude API |
| [anthropics/claude-agent-sdk-python](https://github.com/anthropics/claude-agent-sdk-python) | Python agent projects |
| [anthropics/claude-agent-sdk-typescript](https://github.com/anthropics/claude-agent-sdk-typescript) | TypeScript agent projects |
| [anthropics/claude-code-action](https://github.com/anthropics/claude-code-action) | Projects needing CI/CD with Claude |

For each relevant repo, `WebFetch` its README or browse its examples to find patterns that apply to the project being planned.

### Safety filters during research:
- Only consider libraries with verified OSS licenses
- Check GitHub stars, maintenance status, and download counts before recommending
- Prefer built-in/standard library solutions over third-party when comparable
- Cross-check package names against official registries to avoid typosquats
- Flag any library that has a paid tier, commercial license, or "enterprise" pricing
- If a library looks promising but you can't verify its license or legitimacy, skip it

### After research:
Summarize your findings to the user:
- What you searched for and what you found
- Latest versions of key frameworks
- Any interesting patterns or templates you discovered
- Any libraries you considered but rejected (and why)

---

## Phase 3: Plan Generation

Generate a comprehensive project plan modeled on the example at `${CLAUDE_SKILL_DIR}/examples/slack-notion-bot-plan.md`. Read that file for format reference.

The plan must include:

### Required sections:
1. **Overview** — what it does, why, one-paragraph summary
2. **Stack** — chosen technologies with versions, justified by web research
3. **Changes from defaults** (if any) — what you changed from the Stack Opinions and why
4. **Dependencies** — table with: name, version, license, stars/downloads, rationale. Also list what was NOT chosen and why.
5. **Prerequisites** — manual steps the user must do (API keys, accounts, service setup)
6. **Project Structure** — full directory tree with one-line descriptions per file
7. **Boundaries** — three-tier system:
   - **Always do**: safe actions, conventions, required practices
   - **Ask first**: high-impact changes needing review
   - **Never do**: hard stops, security rules
8. **Code Style Example** — one real code snippet (10-15 lines) showing the canonical pattern for this project. Show don't tell.
9. **Implementation Steps** — ordered, with code snippets for key files
10. **Testing Strategy** — how to run tests, what to mock vs. test directly, philosophy
11. **Error Handling** — startup failures vs. runtime errors vs. rate limits
12. **Git Workflow** — branch naming, commit format, when to commit
13. **Build & Run Instructions** — local dev + production
14. **Claude Code Infrastructure** — CLAUDE.md, settings, skills (including /gsd and /ralph), hooks with config
15. **Deployment / Handoff Notes** — ops info for whoever runs it
16. **Estimated Costs** (if applicable — API costs, hosting)
17. **Known Limitations & Future Phases**

### Present the plan to the user:
- Show the plan in full
- Ask for approval before scaffolding
- Be prepared to iterate — the user may want changes

Do NOT proceed to Phase 4 until the user explicitly approves.

---

## Phase 4: Scaffold Everything

Once approved, execute in this order:

### 4.1 Project Directory & Git

Ask the user: "Where should I create this project? Please provide the full path."

Then:
```bash
mkdir -p <path>
cd <path>
git init
```

Save the plan as `PROJECT_PLAN.md` and create `CHANGELOG.md`:

```markdown
# Changelog

## [Unreleased]

### Added
- Initial project scaffold from /plan-project
- Project plan, CLAUDE.md, settings, skills, hooks
- Source files and test scaffolding
```

The changelog is a human-readable summary of major changes. Sub-agents (`/gsd`, `/ralph`) and developers should update it when completing significant work — add a brief entry under the appropriate heading (`Added`, `Changed`, `Fixed`, `Removed`). This avoids digging through git history to understand what happened.

Make an initial commit with both files.

### 4.2 CLAUDE.md

Create `CLAUDE.md` at the project root. Keep it under 150 lines. Include:

- **Overview** — 2-3 sentence project description
- **Stack** — runtime, dependencies with versions, key choices
- **Project Structure** — directory tree with one-line descriptions
- **Key Patterns** — conventions, architectural decisions
- **Commands** — build, test, dev, deploy commands
- **Environment Variables** — list all with descriptions (no actual values)
- **File Responsibilities** — what each source file does, one line each
- **Coding Conventions** — import style, logging, testing framework
- **Gotchas** — rate limits, deadlines, known quirks
- **Agent Workflow** — how sub-agents should operate in this project

### 4.3 .claude/settings.json

Create `.claude/settings.json` with permissions scoped to the stack:

**For Node.js projects:**
```json
{
  "permissions": {
    "allow": [
      "Bash(npm run *)", "Bash(npm test)", "Bash(npm install *)",
      "Bash(node *)", "Bash(npx *)",
      "Bash(git status)", "Bash(git diff *)", "Bash(git log *)", "Bash(git add *)",
      "Read", "Grep", "Glob"
    ],
    "deny": [
      "Bash(rm -rf *)", "Read(.env)"
    ]
  }
}
```

**For Python projects:**
```json
{
  "permissions": {
    "allow": [
      "Bash(python *)", "Bash(uv *)", "Bash(pip *)", "Bash(pytest *)",
      "Bash(ruff *)",
      "Bash(git status)", "Bash(git diff *)", "Bash(git log *)", "Bash(git add *)",
      "Read", "Grep", "Glob"
    ],
    "deny": [
      "Bash(rm -rf *)", "Read(.env)"
    ]
  }
}
```

Add hooks based on project type (see Hooks section below).

Also create `.claude/settings.local.json` as an empty JSON object `{}` for personal overrides, and add it to `.gitignore`.

### 4.4 .claude/skills/

Always create these skills:

#### /test — Run test suite
```
.claude/skills/test/SKILL.md
```
```yaml
---
name: test
description: Run the project test suite. Use after making code changes to verify nothing is broken.
---

Run the test suite for this project.

1. Run the test command from CLAUDE.md
2. If tests fail, analyze the output and identify the root cause
3. Report which tests passed and which failed
```

#### /dev — Start local development
```
.claude/skills/dev/SKILL.md
```
```yaml
---
name: dev
description: Start the local development server or watch mode.
disable-model-invocation: true
---

Start local development:

1. Verify .env file exists (remind user to create from .env.example if missing)
2. Run the dev command from CLAUDE.md
3. Report the URL or process status
```

#### /gsd — GSD spec-driven workflow
```
.claude/skills/gsd/SKILL.md
```
```yaml
---
name: gsd
description: Run the GSD (Get Shit Done) spec-driven development workflow. Breaks work into atomic plans and executes them with fresh sub-agent contexts to prevent context rot.
disable-model-invocation: true
---

# GSD — Spec-Driven Development

You are running the GSD workflow. Your job is to break complex work into atomic tasks, execute each in a fresh context, and track progress.

## Setup

1. Read `PROJECT_PLAN.md` and `CLAUDE.md` to understand the project
2. Read `.planning/progress.md` if it exists (resume from where we left off)
3. If `.planning/` doesn't exist, create it with `progress.md` and `REQUIREMENTS.md`

## Planning

1. Break the remaining work into **atomic plans** — each should:
   - Fit comfortably in ~50% of a fresh context window
   - Be independently testable
   - Result in a single atomic commit
2. Group independent plans into **parallel waves**
3. Present the plan to the user for approval

## Execution

For each plan:
1. Spawn a fresh sub-agent (via the Agent tool) with:
   - The specific task description
   - Relevant file paths from the project structure
   - Clear acceptance criteria
2. The sub-agent implements, tests, and commits
3. Update `.planning/progress.md` with the result
4. Update `CHANGELOG.md` with a brief entry for what was done
5. Move to the next plan

## Progress tracking

Maintain `.planning/progress.md` with:
- [ ] / [x] checkboxes for each task
- Wave groupings
- Timestamps for completed tasks
- Notes on any issues or blockers

## Completion

When all tasks are done:
1. Run the full test suite
2. Update progress.md with final status
3. Report summary to user
```

#### /ralph — Ralph autonomous loop
```
.claude/skills/ralph/SKILL.md
```
```yaml
---
name: ralph
description: Run the Ralph autonomous agent loop. Iterates against a task list or PRD until all items are complete, with each iteration in a fresh context.
disable-model-invocation: true
argument-hint: "[task description or PRD reference]"
---

# Ralph — Autonomous Agent Loop

You are running the Ralph loop. Your job is to iterate autonomously against a task list until all items are complete.

## How it works

1. Read the task description from $ARGUMENTS (or `PROJECT_PLAN.md` if no arguments)
2. Read `ralph-progress.md` if it exists (resume previous run)
3. If starting fresh, create `ralph-progress.md` with:
   - Task list extracted from the PRD/description
   - Iteration counter: 0
   - Max iterations: 10

## Each iteration

1. Increment the iteration counter
2. Check: are we at max iterations? If yes, stop and report status
3. Read `ralph-progress.md` to see what's left
4. Pick the next incomplete task
5. Spawn a fresh sub-agent (via the Agent tool) to:
   - Implement the task
   - Run relevant tests
   - Commit the changes
6. Update `ralph-progress.md`:
   - Mark the task complete
   - Note what was done
   - Record the commit hash
7. Update `CHANGELOG.md` with a brief entry for the completed task
7. Check: are ALL tasks complete? If yes, output "ALL TASKS COMPLETE" and stop
8. Otherwise, continue to next iteration

## Memory persistence

Between iterations, state persists via:
- Git history (each task = one commit)
- `ralph-progress.md` (task status, notes, iteration count)
- `CLAUDE.md` (project context)

## Safety

- Never exceed max iterations (default 10)
- If a task fails twice, mark it as blocked and move on
- Always report final status to the user, even if stopped early
```

#### Additional project-specific skills

Based on the project type, also create relevant skills such as:
- `/deploy` — for projects with deployment targets
- `/check-api` — for projects that integrate with external APIs
- `/lint` — for projects with linters configured
- `/db-migrate` — for projects with databases

### 4.5 Hooks

Choose hooks based on the project type. Add them to `.claude/settings.json`.

**Always include — block destructive commands:**
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.command' | grep -qE 'rm -rf|DROP TABLE|--force' && echo '{\"hookSpecificOutput\":{\"hookEventName\":\"PreToolUse\",\"permissionDecision\":\"deny\",\"permissionDecisionReason\":\"Destructive command blocked by safety hook\"}}' || true"
          }
        ]
      }
    ]
  }
}
```

**For JS/TS projects with Prettier — auto-format on write:**
```json
{
  "PostToolUse": [
    {
      "matcher": "Edit|Write",
      "hooks": [
        {
          "type": "command",
          "command": "npx prettier --write \"$(jq -r '.tool_input.file_path')\" 2>/dev/null || true",
          "timeout": 10000
        }
      ]
    }
  ]
}
```

**For Python projects with ruff — auto-format on write:**
```json
{
  "PostToolUse": [
    {
      "matcher": "Edit|Write",
      "hooks": [
        {
          "type": "command",
          "command": "ruff format \"$(jq -r '.tool_input.file_path')\" 2>/dev/null && ruff check --fix \"$(jq -r '.tool_input.file_path')\" 2>/dev/null || true",
          "timeout": 10000
        }
      ]
    }
  ]
}
```

Explain to the user what hooks you chose and why.

### 4.6 Source Files

Create all source files, config files, and test scaffolding as specified in the plan:
- Package manager config (`package.json` with `"type": "module"`, `pyproject.toml`, etc.)
- `.env.example` with all required variables documented
- `.env` — copy of `.env.example` so the user can fill in real values immediately
- `.gitignore` appropriate to the stack — **MUST include `.env`** to prevent committing secrets
- Docker setup if applicable (`Dockerfile`, `docker-compose.yml`, `.dockerignore`)
- Source files matching the project structure
- At least one working test

### 4.7 Install Dependencies

Run the appropriate install command:
- Node.js: `npm install`
- Python: `uv sync` or `pip install -e .`

### 4.8 Final Git Commit

```bash
git add -A
git commit -m "Initial project scaffold from /plan-project

Includes:
- Project plan and source files
- CLAUDE.md with project intelligence
- .claude/ config (settings, skills, hooks)
- GSD and Ralph workflow skills
- Test scaffolding

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>"
```

### 4.9 Summary

Tell the user:
1. What was created (list key files)
2. What hooks are active and what they do
3. What skills are available (`/test`, `/dev`, `/gsd`, `/ralph`, etc.)
4. Next steps:
   - Fill in `.env` from `.env.example`
   - Complete any prerequisites from the plan
   - Run `/dev` to start development
   - Run `/gsd` to begin spec-driven implementation

---

## Stack Opinions

When suggesting a stack, use these opinionated defaults. The user can override anything. **Always verify with web research that these are still current best practices.**

| Project Type | Default Stack | Key Choices |
|---|---|---|
| API / Bot | Node.js 22+, ESM, pino, node:test | No build step, built-in test runner, built-in fetch |
| Web Frontend | Next.js + React, TypeScript | App router, Tailwind CSS |
| Data Pipeline | Python 3.12+, uv, pytest | Type hints, structured logging |
| CLI Tool | Node.js (simple) or Python (complex) | Minimal dependencies |
| Automation / Scripts | Python + cron or Node.js | Depends on ecosystem |
| Full-stack Web | Next.js (frontend + API routes) | Monolith for small projects |

### General preferences:
- **ESM over CommonJS** for Node.js (`"type": "module"`)
- **Built-in over third-party**: `node:test`, `node:fs`, `fetch`, `node --watch`
- **uv over pip** for Python package management
- **Structured logging** (pino for Node, structlog or logging for Python)
- **No unnecessary build steps** — avoid transpilation unless required
- Web research may surface better options — prefer current best practices over these defaults

---

## Additional resources

For a comprehensive example of what a finished project plan looks like, see [slack-notion-bot-plan.md](examples/slack-notion-bot-plan.md).
