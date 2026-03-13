---
name: plan-project
description: Plan and scaffold a new project with full Claude Code infrastructure. Use when starting a new project, setting up a new codebase, or when the user says they want to build something new.
disable-model-invocation: true
---

# /plan-project

You are a project planning and scaffolding agent. Your job is to help the user design a new project, generate a comprehensive plan, and scaffold everything — including full Claude Code infrastructure so that future sessions can work autonomously with sub-agents.

Follow these phases in order. Do NOT skip phases. Do NOT scaffold before the user approves the plan.

## Resume detection

Before starting Phase 1, check for existing state in this order:

### 1. Check `.claude/plan-state.json`

If this file exists, the skill was interrupted mid-planning. Read it and resume from the last completed phase:

| `phase` value | Resume from | What's already done |
|---|---|---|
| `discovery_complete` | Phase 2, Step 1 | Discovery answers are saved — skip Phase 1 |
| `stack_complete` | Phase 2, Step 2 | Stack research done — spawn remaining parallel agents |
| `research_complete` | Phase 3 | All research done — synthesize the plan |
| `plan_complete` | Phase 4 | Plan written — save and scaffold |
| `scaffolded` | Done | Tell the user the project is already scaffolded |

Tell the user: **"Found plan state from a previous session (phase: [phase]). Resuming from [next step]."**
Show a brief summary of what's been completed, then continue.

### 2. Check `PROJECT_PLAN.md` (no state file)

If `PROJECT_PLAN.md` exists but `.claude/plan-state.json` does not:

1. Read `PROJECT_PLAN.md`
2. Tell the user: **"Found an existing project plan. Resuming from Phase 5 (scaffolding)."**
3. Show a brief summary of the plan (project name, stack, key features)
4. Ask: **"Ready to scaffold, or do you want to revise the plan first?"**
5. Skip directly to Phase 5 once confirmed

### 3. Neither exists → start fresh from Phase 1

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

## Plan Persistence

After each major milestone, save state to `.claude/plan-state.json` in the project directory so the plan survives `/clear`, `/compact`, interruptions, and new sessions.

```bash
mkdir -p .claude
```

Write/update `.claude/plan-state.json` with:

```json
{
  "project_name": "<name>",
  "created": "<ISO timestamp>",
  "updated": "<ISO timestamp>",
  "phase": "<phase value>",
  "discovery": { "description": "...", "requirements": {}, "checklist_answers": {} },
  "stack_output": {},
  "design_output": {},
  "infrastructure_output": {},
  "dev_workflow_output": {},
  "safety_first_pass": {},
  "plan_draft_path": "PROJECT_PLAN.md"
}
```

**Save points:**
- End of Phase 1 → `phase: "discovery_complete"`, persist `discovery`
- After Stack Research Agent returns → `phase: "stack_complete"`, persist `stack_output`
- After all Phase 2 agents return → `phase: "research_complete"`, persist all outputs
- After plan is written → `phase: "plan_complete"`
- After scaffold is done → `phase: "scaffolded"`

Only populate fields as they become available. Earlier phases will have `null` for later fields.

---

## Sub-Agents

The skill uses 5 specialized sub-agents defined as `.claude/agents/` files. Each agent runs in its own context window, does focused work, and returns structured output to the main agent.

| Agent | File | When | Purpose |
|---|---|---|---|
| `stack-research` | `.claude/agents/stack-research.md` | Phase 2 Step 1 (sequential) | Tech stack, libraries, deployment, architecture |
| `frontend-design` | `.claude/agents/frontend-design.md` | Phase 2 Step 2 (parallel) | UI/UX, layouts, styling, accessibility |
| `claude-infrastructure` | `.claude/agents/claude-infrastructure.md` | Phase 2 Step 2 (parallel) | CLAUDE.md, skills, hooks, permissions |
| `dev-workflow` | `.claude/agents/dev-workflow.md` | Phase 2 Step 2 (parallel) | Local dev, testing, CI/CD, deploy, ops handoff |
| `safety` | `.claude/agents/safety.md` | Phase 2 Step 2 + Phase 3 (runs twice) | Dependency audit, supply chain, security review |

**All sub-agents inherit the HARD CONSTRAINTS above** (licensing, security, transparency). Include a reminder of these constraints when spawning each agent.

---

## Phase 1: Discovery Chat

Start by letting the user describe their project idea freely. Listen, ask follow-up questions, and build understanding.

Then, before moving to Phase 2, check this **requirements checklist** and ask about any gaps:

- [ ] Project name and one-line description
- [ ] Core functionality / user stories (what does it do?)
- [ ] Target stack (you suggest based on project type — see Stack Opinions below)
- [ ] External integrations (APIs, databases, services)
- [ ] Deployment target (Docker, serverless, local, VPS, etc.)
- [ ] Server operations (if remote server: SSH user, deploy method, Docker services, cron jobs)
- [ ] Auth requirements (if any)
- [ ] Who/what interacts with it (users, bots, cron jobs, other services)
- [ ] Any existing code or repos to build on
- [ ] Preferred agent workflow (GSD, Ralph loop, or both — default: both)

Do NOT proceed to Phase 2 until you have enough information to make good stack decisions.

**Save plan state:** Write `.claude/plan-state.json` with `phase: "discovery_complete"` and the full discovery data (project name, description, requirements checklist answers, integrations, deployment target, etc.).

---

## Phase 2: Agent-Driven Research (MANDATORY)

Research is handled by specialized sub-agents. This is not optional — do NOT skip to plan generation.

### Step 1: Spawn Stack Research Agent (sequential)

Spawn the `stack-research` agent with:
- Project description and requirements from Phase 1
- Focus areas based on project type (e.g., `frontend, backend, infrastructure` for a full-stack web app; `backend, local-service` for a Slack bot)
- The Stack Opinions table from this skill

Wait for the agent to return its structured report. Review the output for completeness and sanity.

**Save plan state:** Update `.claude/plan-state.json` with `phase: "stack_complete"` and persist the `stack_output`.

### Step 2: Spawn parallel agents

Using the Stack Research Agent's output, spawn these agents **in parallel** (use multiple Agent tool calls in a single message):

1. **`frontend-design`** agent — only if the project has a user interface (web, CLI, bot, desktop). Pass the stack decisions and interface type.

2. **`claude-infrastructure`** agent — always. Pass the stack decisions, project type, deployment target, and whether the project has a remote server, auth, or external APIs.

3. **`dev-workflow`** agent — always. Pass the stack decisions, project type, deployment target, whether the project has a remote server, database, external APIs, Docker, and the testing framework from the stack decisions.

4. **`safety`** agent (first pass) — always. Include "FIRST PASS" in the prompt. Pass the dependency table from the Stack Research Agent, deployment target, whether any step requires sudo, and the list of external APIs.

Wait for all parallel agents to return.

**Save plan state:** Update `.claude/plan-state.json` with `phase: "research_complete"` and persist all agent outputs (`design_output`, `infrastructure_output`, `dev_workflow_output`, `safety_first_pass`).

### Step 3: Review and resolve

Review all agent outputs for conflicts or gaps:
- If the Safety Agent flagged dependencies as rejected, find alternatives or remove them from the stack
- If the Frontend Design Agent recommended UI libraries not in the Stack Research output, verify they meet the HARD CONSTRAINTS
- If the Claude Infrastructure Agent's hook config conflicts with the Safety Agent's findings, adjust
- If the Dev Workflow Agent's testing strategy conflicts with the Stack Research Agent's testing framework, prefer the Dev Workflow Agent (it runs later with more context)
- Stack Research Agent decides *what* to deploy to; Dev Workflow Agent decides *how* — reconcile if they conflict

Summarize findings to the user:
- What the agents researched and recommended
- Latest versions of key frameworks
- Any interesting patterns or templates discovered
- Any libraries considered but rejected (and why)
- Any safety concerns flagged

---

## Phase 3: Plan Generation

Synthesize all agent outputs into a comprehensive project plan. Read the example at `${CLAUDE_SKILL_DIR}/examples/slack-notion-bot-plan.md` for format reference.

### Inputs available:
- Phase 1 discovery data
- Stack Research Agent output (stack, dependencies, architecture, deployment)
- Frontend Design Agent output (layouts, components, styling — if applicable)
- Claude Infrastructure Agent output (CLAUDE.md, skills, hooks, permissions)
- Dev Workflow Agent output (local dev, testing, CI/CD, deploy, ops handoff)
- Safety Agent first pass output (dependency audit, privilege concerns)

### Required sections:
1. **Overview** — what it does, why, one-paragraph summary
2. **Stack** — chosen technologies with versions, justified by research (from Stack Research Agent)
3. **Changes from defaults** (if any) — what was changed from the Stack Opinions and why
4. **Dependencies** — table with: name, version, license, stars/downloads, rationale. Also list what was NOT chosen and why. (from Stack Research Agent + Safety Agent audit)
5. **Prerequisites** — manual steps the user must do (API keys, accounts, service setup)
6. **Project Structure** — full directory tree with one-line descriptions per file
7. **UI/UX Design** (if applicable) — layouts, component hierarchy, styling approach, UX patterns (from Frontend Design Agent)
8. **Boundaries** — three-tier system (from Claude Infrastructure Agent):
   - **Always do**: safe actions, conventions, required practices
   - **Ask first**: high-impact changes needing review
   - **Never do**: hard stops, security rules
9. **Code Style Example** — one real code snippet (10-15 lines) showing the canonical pattern for this project. Show don't tell.
10. **Implementation Steps** — ordered, with code snippets for key files
11. **Testing Strategy** — how to run tests, what to mock vs. test directly, philosophy (from Dev Workflow Agent)
12. **Error Handling** — startup failures vs. runtime errors vs. rate limits (from Dev Workflow Agent)
13. **Git Workflow** — branch naming, commit format, when to commit (from Dev Workflow Agent)
14. **Build, Run & CI/CD Instructions** — local dev, production, CI/CD pipeline (from Dev Workflow Agent)
15. **Claude Code Infrastructure** — CLAUDE.md, settings, skills (including /gsd and /ralph), hooks with config (from Claude Infrastructure Agent)
16. **Deployment / Handoff Notes** — ops info for whoever runs it (from Dev Workflow Agent). If targeting a remote server, include a **Server Operations** subsection: SSH access, deploy workflow, Docker services/container names, cron schedules. This feeds directly into `/server` skill generation.
17. **Estimated Costs** (if applicable — API costs, hosting)
18. **Known Limitations & Future Phases**
19. **Security Considerations** — from Safety Agent first pass. Attack surface, top risks, mitigations.

### Safety second pass

After writing the plan draft, spawn the **`safety`** agent (second pass) with "SECOND PASS" in the prompt, the complete plan content, and the first pass findings.

If the Safety Agent returns `requires_revision: true`:
1. Apply the revision instructions
2. Re-run the Safety Agent second pass on the revised plan
3. Maximum 2 revision cycles — after that, present the plan with any remaining warnings

Insert the Safety Agent's **Security Considerations** section into the plan.

### Present the plan to the user:
- Show the plan in full
- Ask for approval before scaffolding
- Be prepared to iterate — the user may want changes

**Save plan state:** Update `.claude/plan-state.json` with `phase: "plan_complete"`.

Do NOT proceed to Phase 4 until the user explicitly approves the plan.

---

## Phase 4: Save Plan & Initialize Git

Once the user approves the plan, **immediately** save it to disk and commit. The plan is a deliverable — the user may want to review it later, share it with colleagues, or pick the project back up in a new session. Don't leave it sitting only in the conversation context.

### 4.1 Create the project directory

Ask the user: "Where should I create this project? Please provide the full path."

```bash
mkdir -p <path>
cd <path>
git init
```

### 4.2 Save the plan and commit

Save the approved plan as `PROJECT_PLAN.md`, then commit it:

```bash
git add PROJECT_PLAN.md
git commit -m "Add project plan from /plan-project"
```

This is a checkpoint. The user now has a versioned plan they can revisit, share, or iterate on before scaffolding begins.

### 4.3 GitHub repo setup

Prompt the user to create a GitHub repo so the plan is backed up and shareable:

**"Would you like to create a GitHub repo for this project? I can run `gh repo create` for you (public or private)."**

If the user agrees:
```bash
gh repo create <project-name> --private --source . --push
```

If they decline, that's fine — they can do it later.

### 4.4 Clear context for scaffolding

The conversation context is now full of discovery chat, web research, and plan iteration from Phases 1–3. Scaffolding will be more accurate with a fresh context window that only has the plan.

Tell the user:

**"The plan is saved, committed, and pushed. Before we scaffold, I recommend clearing the context window — it's full of research and discussion that will reduce scaffolding accuracy.**

**Start a new Claude Code session in this project directory and run `/plan-project`. It will detect `PROJECT_PLAN.md` and resume from Phase 5 (scaffolding) with a clean context.**

**Or you can continue here if you prefer — your call."**

Do NOT proceed to Phase 5 until the user confirms they want to scaffold (either here or in a fresh session).

---

## Phase 5: Scaffold Everything

**Note:** The Claude Infrastructure Agent's output from Phase 2 provides the CLAUDE.md outline, skills, hooks, permissions, and boundaries. Use that output as the primary source for sections 5.1–5.4 below, supplementing with the plan details as needed.

Execute in this order:

### 5.1 CLAUDE.md

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
- **Safety Rules** — no modifications outside project dir, no destructive commands without confirmation, no secrets in logs/output, no commits without instruction, prefer small reversible changes, flag risks don't silently fix them
- **Agent Workflow** — how sub-agents should operate in this project

### 5.2 .claude/settings.json

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
      "Bash(rm -rf *)", "Bash(rm -r *)",
      "Bash(git push *)", "Bash(git commit *)",
      "Bash(npm publish *)",
      "Read(.env)"
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
      "Bash(rm -rf *)", "Bash(rm -r *)",
      "Bash(git push *)", "Bash(git commit *)",
      "Read(.env)"
    ]
  }
}
```

Add hooks based on project type (see Hooks section below).

Also create `.claude/settings.local.json` as an empty JSON object `{}` for personal overrides, and add it to `.gitignore`.

### 5.3 .claude/skills/

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

#### /server — Remote server operations (conditional)

Create this skill ONLY when the project deploys to a remote server (VPS, cloud VM, dedicated host). Do NOT create it for serverless, local-only, or managed-platform deployments.

```
.claude/skills/server/SKILL.md
```

The /server skill routes ALL remote operations through sub-agents (Agent tool) to keep SSH output, Docker logs, and build noise out of the main conversation context.

When generating this skill, discover and hardcode these project-specific values:
- Server host/IP and SSH user (from Phase 1 discovery or DEPLOY.md)
- App directory on the server (e.g., `/opt/myapp`)
- Deploy method (rsync, git pull, CI/CD)
- Docker service and container names (from docker-compose.yml)
- Env file paths, rsync exclude patterns
- Cron schedules (if any)

Required subcommands:

| Command | Action | Confirm? |
|---|---|---|
| `/server status` | Container status, health, uptime, cron schedules | No |
| `/server logs [service]` | Last 50 lines, casual name mapping (e.g. "api" -> actual container) | No |
| `/server deploy` | Full deploy: sync code, rebuild, restart containers | Yes |
| `/server deploy --env-only` | Push env files only, restart | Yes |
| `/server run <service> [--dry-run]` | Trigger a manual run inside a container | No |
| `/server ssh <command>` | Pass-through for arbitrary SSH commands | No |

Each subcommand MUST:
1. Spawn a sub-agent (Agent tool) with the full SSH/bash command
2. After the sub-agent returns, print a 2-4 line summary
3. Show errors in full if the command failed

Set `disable-model-invocation: true` — manual only (makes remote changes).

#### /security-review — Security analysis (conditional)

Create this skill when the project has any of: user authentication, external API integrations, stored user data, web endpoints, or database access. Do NOT create it for pure CLI tools, local scripts, or projects with no attack surface.

```
.claude/skills/security-review/SKILL.md
```
```yaml
---
name: security-review
description: Run a security analysis of the project. Use before launch, after major architectural changes, or when adding new endpoints/integrations.
disable-model-invocation: true
---

# Security Review

Perform a structured security analysis of this project. Read PROJECT_PLAN.md and CLAUDE.md for context.

## Step 1: Attack Surface Inventory

Map every point where data enters or exits the system:

**Inbound:** API endpoints, web forms, file uploads, webhooks, WebSocket connections, CLI inputs, scheduled jobs consuming external data.

**Outbound:** Third-party API calls, database connections, email/SMS services, external auth providers, logging services, payment processors.

**Trust boundaries:** Identify every point where user-controlled or external data crosses into a trusted context (browser→server, server→database, service→service).

**Data flows:** What sensitive data exists (PII, credentials, financial), where it's stored, how it moves, who can access it, how long it's retained.

## Step 2: Threat Assessment

For each entry point and trust boundary, evaluate:
- **Injection** — SQL/NoSQL, XSS (stored/reflected/DOM), command injection, template injection, path traversal
- **Auth flaws** — session management, token handling, credential storage, account enumeration, brute force
- **Authorization** — privilege escalation, IDOR, missing function-level access control
- **Data exposure** — sensitive data in logs/errors/URLs, over-permissive API responses, client-side storage
- **Infrastructure** — CORS, CSP, rate limiting, TLS configuration
- **Business logic** — race conditions, replay attacks, workflow bypass, abuse of intended functionality

## Step 3: Risk Classification

Rate each finding:

| Field | Values |
|---|---|
| Severity | Critical / High / Medium / Low |
| Likelihood | High / Medium / Low |
| Impact | Data breach / Service disruption / Financial loss / Reputation / Regulatory |
| Priority | Block (fix before launch) / Post-launch (next cycle) |

## Step 4: Mitigations

For each finding, specify:
- The specific library, middleware, or pattern to use (tied to this project's stack)
- Which file/module/route it applies to
- Effort estimate: trivial (< 1hr) / moderate (hours) / significant (days)
- How to verify the mitigation works (test case or check)

## Step 5: Output

Save results to `docs/security/`:
- `attack-surface.md` — inventory from Step 1
- `findings.md` — threat findings with classifications from Steps 2-3
- `mitigations.md` — action plan from Step 4

Report a summary to the user with critical/high findings highlighted.
```

#### Additional project-specific skills

Based on the project type, also create relevant skills such as:
- `/deploy` — for projects with deployment targets (use `/server` instead for remote servers)
- `/check-api` — for projects that integrate with external APIs
- `/lint` — for projects with linters configured
- `/db-migrate` — for projects with databases

### 5.3.5 .claude/agents/ — Custom Agents

Create project-specific agents based on the Claude Infrastructure Agent's output. These are `.md` files with YAML frontmatter that sub-agents (from `/gsd`, `/ralph`, or manual Agent calls) can use as specialized debugging and operational tools.

Each agent file should follow this format:

```yaml
---
name: <agent-name>
description: <one-line — when to spawn this agent>
tools: <comma-separated list, scoped to what the agent needs>
---

<System prompt: what to check, what to report, what NOT to do. Keep under 20 lines.>
```

**Guidelines:**
- Scope tools narrowly — a read-only diagnostic agent shouldn't have Write/Edit
- Focus each agent on ONE failure mode or domain
- Keep prompts short — the spawner provides runtime context
- Only create agents that address real, project-specific needs (not generic "code reviewer" agents)

**Common patterns by project type:**

| Project type | Likely agents |
|---|---|
| API / Bot | API debugger, event/webhook tracer, auth inspector |
| Web Frontend | Build debugger, hydration inspector, accessibility auditor |
| Data Pipeline | Pipeline debugger, data validator, schedule inspector |
| Full-stack | Combination of the above |

### 5.4 Hooks

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
            "command": "jq -r '.tool_input.command' | grep -qE 'rm -rf|rm -r |DROP TABLE|TRUNCATE|--force|--hard' && echo '{\"hookSpecificOutput\":{\"hookEventName\":\"PreToolUse\",\"permissionDecision\":\"deny\",\"permissionDecisionReason\":\"Destructive command blocked by safety hook\"}}' || true"
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

### 5.5 Source Files

Create all source files, config files, and test scaffolding as specified in the plan:
- Package manager config (`package.json` with `"type": "module"`, `pyproject.toml`, etc.)
- `.env.example` with all required variables documented
- `.env` — copy of `.env.example` so the user can fill in real values immediately
- `.gitignore` appropriate to the stack — **MUST include `.env`** to prevent committing secrets
- Docker setup if applicable (`Dockerfile`, `docker-compose.yml`, `.dockerignore`)
- Source files matching the project structure
- At least one working test

### 5.6 Install Dependencies

Run the appropriate install command:
- Node.js: `npm install`
- Python: `uv sync` or `pip install -e .`

### 5.7 CHANGELOG.md & Final Git Commit

Create `CHANGELOG.md`:

```markdown
# Changelog

## [Unreleased]

### Added
- Initial project scaffold from /plan-project
- Project plan, CLAUDE.md, settings, skills, hooks
- Source files and test scaffolding
```

The changelog is a human-readable summary of major changes. Sub-agents (`/gsd`, `/ralph`) and developers should update it when completing significant work — add a brief entry under the appropriate heading (`Added`, `Changed`, `Fixed`, `Removed`). This avoids digging through git history to understand what happened.

Then commit everything:

```bash
git add -A
git commit -m "Initial project scaffold from /plan-project

Includes:
- Project plan and source files
- CLAUDE.md with project intelligence
- .claude/ config (settings, skills, hooks)
- GSD and Ralph workflow skills
- Test scaffolding
- README.md with full setup and usage docs

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>"
```

### 5.8 README.md

Create a comprehensive `README.md` at the project root. This is the public face of the project — it should let a new developer (or future-you) clone the repo and get productive fast.

**Required sections:**

1. **Title & badges** — project name, one-line description
2. **Overview** — 2-3 paragraph explanation of what the project does and why it exists
3. **Features** — bullet list of key capabilities
4. **Prerequisites** — system requirements, accounts, API keys needed before setup
5. **Quick Start** — numbered steps from clone to running (copy-pasteable commands)
6. **Configuration** — environment variables table (name, description, required/optional, example value). Reference `.env.example`.
7. **Usage** — how to use the project once it's running (CLI commands, API endpoints, UI walkthrough — whatever applies)
8. **Development** — how to run tests, lint, format, and develop locally
9. **Project Structure** — directory tree with brief descriptions (pull from the plan)
10. **Deployment** — how to deploy to production (if applicable)
11. **Architecture** — brief description of how the system works internally, data flow, key design decisions
12. **Contributing** — how to contribute (branch workflow, commit format, PR process)
13. **License** — state the license (default to MIT if not specified in the plan)

**Guidelines:**
- Write for someone who has never seen the project before
- Every command should be copy-pasteable — use code blocks with the correct language tag
- Keep it practical, not aspirational — only document what actually exists in the scaffold
- If the project has an API, include example requests/responses
- If the project has a CLI, show example usage with expected output
- Link to `PROJECT_PLAN.md` for the full design rationale

### 5.9 Final state save

Update `.claude/plan-state.json` with `phase: "scaffolded"`. This marks the project as fully set up.

### 5.10 Summary

Tell the user:
1. What was created (list key files)
2. What hooks are active and what they do
3. What skills are available (`/test`, `/dev`, `/gsd`, `/ralph`, `/server` if applicable, etc.)
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
