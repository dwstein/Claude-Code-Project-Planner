# Slack-Notion Bot — Implementation Plan (Revised)

## Overview

Build a Node.js Slack bot that lets users query Notion databases and wiki pages using natural language. The bot uses Claude Haiku 4.5 with tool use to interpret queries and map them to Notion API calls. It runs in a Docker container for deployment on in-house servers.

**Stack**: Node.js 20+ (ESM), Slack Bolt SDK v4.6.x (Socket Mode), Notion SDK v5.11.x (2025-09-03 API), Anthropic SDK v0.78.x, Docker

---

## Changes from Original Spec

This plan revises the original spec (committed as the first git commit) with improvements discovered during research.

| Area | Original Spec | Revised |
|------|--------------|---------|
| **Module system** | CommonJS (`require`) + p-queue v7 (ESM-only) = broken | ESM throughout (`"type": "module"`) |
| **Database schemas** | Hardcoded in system prompt | Auto-discovered on startup via `dataSources.retrieve()` |
| **Page content** | Manual block-by-block traversal (`utils/blocks.js`) | `pages.retrieveMarkdown()` — single API call |
| **Anthropic SDK** | `^0.39.0` (outdated) | `^0.78.0` (current) |
| **Notion SDK** | `v5.9.0` | `^5.11.0` (adds `retrieveMarkdown`) |
| **Logging** | `console.error` only | `pino` structured JSON logging |
| **Testing** | None | Basic tests using `node:test` (no extra deps) |
| **Config** | Separate DB IDs per database | Single `NOTION_DATA_SOURCE_IDS` comma-separated list |
| **Dev tooling** | None | CLAUDE.md, `.claude/settings.json`, custom skills |

---

## Dependencies

All dependencies are free, open-source, and installed from npm.

| Package | Version | License | Popularity | Why chosen |
|---|---|---|---|---|
| `@slack/bolt` | ^4.6.0 | MIT | 2.7k stars, 300k+ weekly downloads | Official Slack SDK by Salesforce/Slack. Socket Mode support built in. |
| `@notionhq/client` | ^5.11.0 | MIT | 5k stars, 200k+ weekly downloads | Official Notion SDK. v5.11 adds `pages.retrieveMarkdown()`. |
| `@anthropic-ai/sdk` | ^0.78.0 | MIT | 3k stars, 500k+ weekly downloads | Official Anthropic SDK. Tool-use support, prompt caching. |
| `p-queue` | ^8.0.0 | MIT | 3.2k stars, 15M+ weekly downloads | Lightweight promise-based queue. ESM-native in v8. |
| `pino` | ^9.0.0 | MIT | 14k stars, 10M+ weekly downloads | Fastest Node.js JSON logger. Minimal overhead, structured output. |

**Not chosen (and why):**
- **No `dotenv`** — Node 20+ supports `--env-file` flag natively; Docker uses `env_file`
- **No `nodemon`** — Node 20+ has built-in `--watch` mode
- **No `jest` or `vitest`** — `node:test` is built-in, zero dependencies
- **No `axios`** — Node 18+ has built-in `fetch`
- **No `winston`** — pino is faster and produces structured JSON by default
- **No `devDependencies`** — all dev tools are Node built-ins

---

## Prerequisites (manual steps before coding)

These steps require human action in web UIs and cannot be automated by Claude Code.

### 1. Create a Slack App

1. Go to https://api.slack.com/apps -> Create New App -> From Scratch
2. Name it (e.g., "Notion Bot") and select your workspace
3. Under **OAuth & Permissions -> Bot Token Scopes**, add:
   - `app_mentions:read` — receive @mention events
   - `chat:write` — post messages in channels
   - `commands` — register slash commands
   - `im:history` — read DM messages to the bot
   - `reactions:write` — add emoji reactions to show processing status

   **Note**: Do not add `channels:history`, `groups:history`, or `mpim:history` unless you add a feature that requires reading ambient channel messages. Minimizing scopes reduces security review friction and blast radius.
4. Under **Settings -> Socket Mode**, toggle ON
5. Under **Basic Information -> App-Level Tokens**, generate a token with `connections:write` scope — this produces an `xapp-...` token
6. Under **Features -> Slash Commands**, create `/ask-notion` with description "Ask a question about Notion"
7. Under **Features -> Event Subscriptions**, toggle ON, then under **Subscribe to bot events** add:
   - `app_mention`
   - `message.im`
8. **Install the app** to your workspace (OAuth & Permissions -> Install to Workspace)
9. Collect these values:
   - `SLACK_BOT_TOKEN` — xoxb-... from OAuth & Permissions
   - `SLACK_APP_TOKEN` — xapp-... from Basic Information -> App-Level Tokens

### 2. Create a Notion Integration

1. Go to https://www.notion.so/profile/integrations -> New Integration
2. Name it (e.g., "Slack Bot"), select the workspace, set type to "Internal"
3. Copy the token (starts with `ntn_`)
4. In Notion, open each database the bot should access -> *** menu -> Add connections -> select your integration
5. For each database, get the **data source ID** (use the Notion API or the `/check-notion` skill once built)
6. Collect:
   - `NOTION_API_KEY` — ntn_... from the integration page
   - `NOTION_DATA_SOURCE_IDS` — comma-separated data source UUIDs

### 3. Get an Anthropic API Key

1. Go to https://console.anthropic.com -> API Keys -> Create Key
2. Collect: `ANTHROPIC_API_KEY` — sk-ant-...

### 4. Populate the .env file

After the project is scaffolded, create a `.env` file from `.env.example` and fill in all values collected above.

---

## Project Structure

```
slack-notion-bot/
├── CLAUDE.md                         # Project intelligence for Claude Code
├── .claude/
│   ├── settings.json                 # Team-shared project settings (committed)
│   ├── settings.local.json           # Personal machine settings (gitignored)
│   └── skills/
│       ├── test/
│       │   └── SKILL.md              # /test — run test suite
│       ├── check-notion/
│       │   └── SKILL.md              # /check-notion — verify Notion connection + schemas
│       ├── dev/
│       │   └── SKILL.md              # /dev — start local dev server
│       ├── gsd/
│       │   └── SKILL.md              # /gsd — spec-driven development workflow
│       └── ralph/
│           └── SKILL.md              # /ralph — autonomous agent loop
├── .planning/                        # GSD progress tracking (created by /gsd)
│   └── progress.md
├── .env.example
├── .gitignore
├── .dockerignore
├── Dockerfile
├── docker-compose.yml
├── package.json                      # "type": "module" — ESM everywhere
├── package-lock.json                 # Committed — required by Dockerfile's npm ci
├── test/
│   ├── config.test.js
│   ├── schema.test.js
│   └── claude-loop.test.js
└── src/
    ├── app.js                        # Entry point — Bolt init, schema discovery, start
    ├── config.js                     # Env var validation, typed config export
    ├── logger.js                     # Pino logger setup
    ├── listeners/
    │   ├── commands.js               # /ask-notion slash command handler
    │   └── events.js                 # app_mention and DM event handlers
    ├── services/
    │   ├── notion.js                 # Notion API wrapper (query, search, page markdown)
    │   ├── claude.js                 # Claude tool-use orchestration loop
    │   └── schema.js                 # Schema auto-discovery + system prompt builder
    └── tools/
        └── definitions.js            # Claude tool schemas (built dynamically)
```

**Removed from original spec**: `utils/blocks.js` — replaced by `pages.retrieveMarkdown()`.
**Added**: `services/schema.js`, `tools/definitions.js`, `logger.js`, `test/`, `.claude/skills/`, `CLAUDE.md`, GSD + Ralph skills.

---

## Boundaries

### Always do
- Use ESM imports (`import`/`export`), never CommonJS `require`
- Use `pino` for all logging — no `console.log` or `console.error`
- Run `npm test` before committing
- Use `async`/`await` — no raw `.then()` chains
- Validate env vars at startup via `config.js`

### Ask first
- Modify the Claude tool-use loop (`services/claude.js`) — core orchestration logic
- Change Notion API query patterns or add new tools
- Alter Docker config or deployment setup
- Add new dependencies (must meet OSS + popularity requirements)

### Never do
- Commit `.env` files or read secrets in code reviews
- Hardcode database schemas — always use auto-discovery
- Use CommonJS `require()` — breaks p-queue and our ESM setup
- Log tool results verbatim — may contain internal Notion content
- Skip `await ack()` in Slack handlers — causes 3-second timeout errors

---

## Code Style Example

This is the canonical pattern for this project. One real example is worth more than paragraphs of description:

```javascript
import { logger } from '../logger.js';

export async function queryDataSource(client, queue, dsId, options = {}) {
  const { filter, sorts } = options;

  try {
    const results = await queue.add(() =>
      client.dataSources.query({
        data_source_id: dsId,
        ...(filter && { filter }),
        ...(sorts && { sorts }),
      })
    );

    logger.info({ dsId, count: results.results.length }, 'Query complete');
    return results;
  } catch (err) {
    logger.error({ dsId, err: err.message }, 'Notion query failed');
    throw err;
  }
}
```

Key patterns shown: ESM import, async/await, pino structured logging with context objects, destructuring options, queue-based rate limiting, error logging with context before re-throwing.

---

## Implementation Steps

### Step 1: Git Init + Commit Original Plan
- `git init`, create `.gitignore`, commit original plan
- Preserves the original spec as historical reference

### Step 2: CLAUDE.md
Create `CLAUDE.md` at project root. Keep concise (<150 lines). Sections:

- **Overview** — 2-3 sentence project description
- **Stack** — Runtime, dependencies with versions, key choices (ESM, Socket Mode)
- **Project Structure** — directory tree with one-line descriptions
- **Key Patterns**:
  - ESM (`import`/`export`, no `require`)
  - Schema auto-discovery on startup (not hardcoded)
  - Claude tool-use loop pattern
  - Rate limiting via p-queue for Notion API (3 req/sec)
  - Page content via `pages.retrieveMarkdown()` (not block traversal)
- **Commands** — `npm start`, `npm test`, `npm run dev`, `docker compose up`
- **Environment Variables** — list all with descriptions, no actual values
- **File Responsibilities** — what each `src/` file does, one line each
- **Coding Conventions** — ESM imports, pino for logging, node:test for tests
- **Gotchas** — Slack ack() 3-second deadline, Notion 3 req/sec limit, etc.

### Step 3: `.claude/settings.json` (team-shared)
Create `.claude/settings.json` with:

- **Permissions — allow**: common safe operations
  - `Bash(npm run *)`, `Bash(npm test)`, `Bash(node *)`
  - `Bash(docker compose *)`, `Bash(git status)`, `Bash(git diff)`, `Bash(git log *)`
  - `Read`, `Grep`, `Glob` (unrestricted for code exploration)
- **Permissions — deny**: dangerous operations
  - `Bash(rm -rf *)` — prevent accidental deletion
  - `Read(.env)` — never read real secrets
- **Hooks** — see Hooks section below

Update `.gitignore` to include `settings.local.json` and `CLAUDE.local.md`.

### Step 4: `.claude/skills/` — Custom Commands
Five skills:

**`/test`** — Run test suite
- Runs `npm test` (node:test runner)
- Options: `/test` (all), `/test config` (specific file)
- `disable-model-invocation: false` — Claude can auto-run after code changes

**`/check-notion`** — Verify Notion connection and inspect schemas
- Authenticates with Notion API
- Retrieves and displays schema for each configured data source
- Lists property names, types, and select options
- `disable-model-invocation: true` — manual only (makes external API calls)

**`/dev`** — Start local development
- Starts bot with `npm run dev` (node --watch)
- Reminds user to have `.env` configured
- `disable-model-invocation: true` — manual only

**`/gsd`** — GSD spec-driven development
- Breaks work into atomic plans from PROJECT_PLAN.md
- Spawns fresh sub-agents per task to prevent context rot
- Groups independent tasks into parallel waves
- Tracks progress in `.planning/progress.md`
- `disable-model-invocation: true` — manual only

**`/ralph`** — Ralph autonomous loop
- Iterates against a task list until all items complete
- Each iteration in a fresh context, memory via git + progress file
- Configurable max iterations (default 10)
- `disable-model-invocation: true` — manual only

### Step 5: Save to Auto-Memory
Write project context to Claude Code auto-memory so future sessions can pick up where we left off.

### Step 6: Project Scaffold
- Create `package.json` with `"type": "module"`, all dependencies, scripts
- Create `.env.example`, `.dockerignore`
- `npm install` to generate `package-lock.json`

**package.json:**
```json
{
  "name": "slack-notion-bot",
  "version": "1.0.0",
  "type": "module",
  "description": "Slack bot that queries Notion using natural language via Claude",
  "engines": { "node": ">=20.0.0" },
  "scripts": {
    "start": "node src/app.js",
    "dev": "node --watch src/app.js",
    "test": "node --test test/*.test.js"
  },
  "dependencies": {
    "@slack/bolt": "^4.6.0",
    "@notionhq/client": "^5.11.0",
    "@anthropic-ai/sdk": "^0.78.0",
    "p-queue": "^8.0.0",
    "pino": "^9.0.0"
  }
}
```

**.env.example:**
```
# Slack (Socket Mode)
SLACK_BOT_TOKEN=xoxb-your-bot-token
SLACK_APP_TOKEN=xapp-your-app-level-token

# Notion
NOTION_API_KEY=ntn_your-notion-token
NOTION_DATA_SOURCE_IDS=data-source-id-1,data-source-id-2

# Anthropic
ANTHROPIC_API_KEY=sk-ant-your-api-key

# Optional
LOG_LEVEL=info
MAX_TOOL_ITERATIONS=5
```

### Step 7: Config + Logger

**`src/config.js`** — Validate required env vars, fail fast, export typed config:
```javascript
function requireEnv(name) {
  const value = process.env[name];
  if (!value) throw new Error(`Missing required env var: ${name}`);
  return value;
}

export const config = {
  slack: {
    botToken: requireEnv('SLACK_BOT_TOKEN'),
    appToken: requireEnv('SLACK_APP_TOKEN'),
  },
  notion: {
    apiKey: requireEnv('NOTION_API_KEY'),
    dataSourceIds: requireEnv('NOTION_DATA_SOURCE_IDS')
      .split(',').map(id => id.trim()).filter(Boolean),
  },
  anthropic: {
    apiKey: requireEnv('ANTHROPIC_API_KEY'),
    model: process.env.ANTHROPIC_MODEL || 'claude-haiku-4-5-20251001',
    maxToolIterations: parseInt(process.env.MAX_TOOL_ITERATIONS || '5', 10),
  },
  logLevel: process.env.LOG_LEVEL || 'info',
};
```

**`src/logger.js`** — Pino with JSON output:
```javascript
import pino from 'pino';
import { config } from './config.js';

export const logger = pino({ level: config.logLevel });
```

### Step 8: Notion Service (`src/services/notion.js`)
- Initialize `Client` with auth token
- Rate limiter via `p-queue` (3 req/sec to match Notion limits)
- `queryDataSource(dsId, { filter, sorts })` — calls `dataSources.query()`
- `searchPages(query)` — calls `notion.search()`
- `getPageContent(pageId)` — calls `pages.retrieveMarkdown()` (replaces block traversal)
- `formatPageSummary(page)` — extracts slim, token-efficient summaries

### Step 9: Schema Discovery (`src/services/schema.js`)

This is the key improvement over the original spec. Instead of hardcoding database property names/types/values in the system prompt:

- `discoverSchemas(dataSourceIds)` — calls `dataSources.retrieve()` per ID
  - Extracts property names, types, select/status options
  - Caches result in memory
- `buildSystemPrompt(schemas)` — generates Claude system prompt dynamically
  - Lists each database with its properties and valid option values
  - Includes date interpretation rules (today's date, "this week", "overdue", etc.)
  - Includes behavioral guidelines (format for Slack, don't fabricate, etc.)
- Called once on startup from `app.js`

This eliminates the "schema coupling" limitation — when someone renames a property or adds a new status option in Notion, the bot picks it up on next restart.

### Step 10: Tool Definitions (`src/tools/definitions.js`)
- `buildToolDefinitions(schemas)` — generates Claude tool schemas dynamically
- Three tools: `query_database`, `search_pages`, `get_page_content`
- Tool descriptions include discovered database names and IDs
- The `data_source_id` parameter lists available databases in its description

### Step 11: Claude Service (`src/services/claude.js`)

Core orchestration loop:

1. Build messages array with user question
2. Call `anthropic.messages.create()` with system prompt, tools, messages
3. If `stop_reason === 'tool_use'`:
   - Extract `tool_use` blocks
   - Execute each tool via Notion service
   - Build `tool_result` messages
   - Append assistant + tool results to messages
   - Call API again
4. Repeat until `stop_reason === 'end_turn'` or max iterations
5. Extract and return final text

Key details:
- Prompt caching on system prompt (`cache_control: { type: 'ephemeral' }`)
- Tool results truncated to ~8000 chars for page content
- Error handling: tool failures return `is_error: true` result (Claude adapts)
- Max iterations guard from config (default 5)

### Step 12: Slack Listeners

**`src/listeners/commands.js`** — `/ask-notion` handler:
- `await ack()` immediately (3-second Slack deadline)
- Post "Searching Notion..." thinking message
- Process query via Claude service
- Update thinking message with answer (or error)

**`src/listeners/events.js`**:
- `app_mention` — strip @mention, add eyes reaction, reply in thread
- `message` (DM) — filter to `channel_type === 'im'`, ignore bots/subtypes

### Step 13: App Entry Point (`src/app.js`)
```javascript
import { App } from '@slack/bolt';
import { config } from './config.js';
import { logger } from './logger.js';
import { discoverSchemas } from './services/schema.js';
import { registerCommands } from './listeners/commands.js';
import { registerEvents } from './listeners/events.js';

const app = new App({
  token: config.slack.botToken,
  socketMode: true,
  appToken: config.slack.appToken,
});

registerCommands(app);
registerEvents(app);

(async () => {
  logger.info('Discovering Notion database schemas...');
  const schemas = await discoverSchemas(config.notion.dataSourceIds);
  logger.info({ databases: schemas.map(s => s.title) }, 'Schema discovery complete');

  await app.start();
  logger.info('Slack bot running in Socket Mode');
})();
```

### Step 14: Docker Setup

**Dockerfile:**
```dockerfile
FROM node:20-slim
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --omit=dev
COPY src/ ./src/
USER node
CMD ["node", "src/app.js"]
```

**docker-compose.yml:**
```yaml
services:
  slack-notion-bot:
    build: .
    env_file: .env
    restart: unless-stopped
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
```

Outbound HTTPS (port 443) required to:
- `wss-primary.slack.com`, `wss-backup.slack.com`, `wss-mobile.slack.com` — Slack Socket Mode
- `api.notion.com` — Notion API
- `api.anthropic.com` — Claude API

No inbound ports needed.

### Step 15: Tests

See Testing Strategy section below.

---

## Testing Strategy

**Framework**: `node:test` (built-in, zero dependencies)

**Commands:**
- Run all tests: `npm test`
- Run one test: `node --test test/config.test.js`
- Run with verbose output: `node --test --test-reporter spec test/*.test.js`

**Test files and what they cover:**

| File | Tests | Mocking |
|---|---|---|
| `test/config.test.js` | Env var validation: missing vars throw, valid vars parse, comma-splitting for data source IDs | Set `process.env` directly, clean up in `afterEach` |
| `test/schema.test.js` | Schema-to-prompt builder: mock Notion response -> verify prompt includes properties and options | Mock `@notionhq/client` with `node:test` `mock` |
| `test/claude-loop.test.js` | Tool-use loop: verify `tool_use` -> `tool_result` -> `end_turn` flow, max iterations guard | Mock `@anthropic-ai/sdk` client |

**What to mock vs. test directly:**
- **Mock**: External API clients (Notion SDK, Anthropic SDK) — never make real API calls in tests
- **Test directly**: Config parsing, prompt building, tool definition generation — pure logic, no side effects

**Philosophy**: Unit tests for pure logic, integration-style tests for the tool-use loop with mocked API clients. No coverage target — focus on testing behavior that matters (startup validation, the Claude loop, schema parsing).

---

## Error Handling Strategy

| Failure type | Behavior | Example |
|---|---|---|
| **Startup** | Fail fast with clear error message. Process exits. | Missing `SLACK_BOT_TOKEN` -> `Error: Missing required env var: SLACK_BOT_TOKEN` |
| **Notion auth** | Fail fast on startup during schema discovery. | Invalid `NOTION_API_KEY` -> log error, exit |
| **Notion API (runtime)** | Return `is_error: true` tool result to Claude. Claude adapts and tells the user. | Rate limit, invalid filter -> Claude says "I couldn't query that database" |
| **Claude API** | Catch error, reply to user with generic error message. Log full error. | 500 from Anthropic -> "Sorry, I'm having trouble processing your request" |
| **Slack ack timeout** | Always `await ack()` first. Post thinking message. Update with result or error. | Handler takes >3s -> ack already sent, no timeout |
| **Max iterations** | Stop the tool-use loop, return whatever Claude has so far. | 5 iterations hit -> return partial answer with note |

---

## Git Workflow

- **Branch naming**: `feature/<description>`, `fix/<description>`, `chore/<description>`
- **Commit format**: Conventional commits — `feat:`, `fix:`, `chore:`, `test:`, `docs:`
- **When to commit**: After each implementation step passes its tests. One logical change per commit.
- **Main branch**: `main` — direct commits during initial development, PRs after first release

---

## Claude Code Infrastructure

### Hooks

Configured in `.claude/settings.json`:

**Block destructive commands** (`PreToolUse` on `Bash`):
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

**Note**: No auto-format hook for this project since we have no linter/formatter configured. If Prettier is added later, add a `PostToolUse` hook on `Edit|Write` to auto-format.

### Agent Workflows

- **`/gsd`**: Use for structured implementation of the plan. Breaks Steps 6-15 into atomic tasks, runs them in parallel waves with fresh sub-agent contexts.
- **`/ralph`**: Use for iterative tasks like "fix all failing tests" or "implement all remaining TODOs". Loops autonomously until done.

---

## Build and Run

### Local development

```bash
npm install
cp .env.example .env
# Fill in .env with real values
npm run dev
```

### Docker

```bash
docker compose build
docker compose up -d
docker compose logs -f
```

### Verify

You should see "Slack bot running in Socket Mode" and "Schema discovery complete" with database names. Then:
- `/ask-notion what tasks are in progress?`
- `@Notion Bot what projects are active?`
- DM the bot: "search for meeting notes"

---

## IT Handoff Notes

- Container needs outbound HTTPS (443) to: `wss-primary.slack.com`, `wss-backup.slack.com`, `wss-mobile.slack.com`, `api.notion.com`, `api.anthropic.com`
- No inbound ports required
- `restart: unless-stopped` handles crash recovery and server reboots
- Secrets in `.env` file via `env_file` — replace with Docker secrets or Vault if applicable
- Logs capped at 10MB x 3 files; output is structured JSON (pino) for log aggregators
- To update: pull new code, `docker compose build && docker compose up -d`
- **Do not log tool results verbatim** — they may contain internal Notion content

---

## Estimated Costs

Using Claude Haiku 4.5:
- Input: $1 per million tokens
- Output: $5 per million tokens
- Typical query: ~1,500 input tokens, ~300 output tokens
- **~$0.003 per query**
- 1,000 queries/month ~ **$3/month**

Prompt caching (`cache_control: { type: 'ephemeral' }`) reduces system prompt costs for queries within a 5-minute window.

---

## Known Limitations (Phase 1)

- **Read-only** — no ability to update Notion from Slack
- **Title search only** — Notion search API matches titles, not page body content
- **No conversation memory** — each query is independent; no thread continuity
- **People properties** — filtering by assignee requires Notion user UUID. No Slack->Notion user mapping yet
- **Rate limits** — 3 req/sec per Notion integration, enforced by p-queue. Concurrent users queue, don't fail
- **Schema on startup only** — schema changes require bot restart to pick up (Phase 2: periodic refresh)

---

## Deferred to Phase 2

- Conversation memory (per-thread follow-ups)
- Write operations (create/update Notion pages from Slack)
- Slack user -> Notion user mapping ("tasks assigned to me")
- Schema refresh command or periodic timer
- Rich Slack Block Kit formatting
- Prettier/ESLint + auto-format hook
