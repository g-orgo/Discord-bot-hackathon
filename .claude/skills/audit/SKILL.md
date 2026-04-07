---
name: audit
description: "Use when: reviewing code quality, consistency, and adherence to conventions across all stacks in the raptor mono-repo. Always load this skill before performing an audit."
---

# Audit workflow — Ensure code quality and consistency

Validate code quality and consistency across all four active stacks of the raptor mono-repo: **`raptor-chatbot`**, **`raptor-chatbot-llm`**, **`raptor-chatbot-server`**, and **`raptor-chatbot-web`**. Do not audit other folders (`raptor-services/`, `signalraptor-*`) unless explicitly asked.

## 1. Read Copilot instructions for all stacks

- `e:/raptor/.github/copilot-instructions.md` — Mono-repo overview
- `e:/raptor/raptor-chatbot/.github/copilot-instructions.md` — Discord bot conventions
- `e:/raptor/raptor-chatbot-llm/.github/copilot-instructions.md` — LLM server conventions
- `e:/raptor/raptor-chatbot-server/.github/copilot-instructions.md` — Auth server conventions
- `e:/raptor/raptor-chatbot-web/.github/copilot-instructions.md` — Web frontend conventions

## 2. Read existing context files

**raptor-chatbot** (`e:/raptor/raptor-chatbot/.claude/context/`):
- `app.md` — Express server / interaction dispatch
- `game.md` — RPS game logic
- `commands.md` — slash command registration
- `utils.md` — shared utilities
- `ask-llm.md` — `/ask` command and LLM integration

**raptor-chatbot-llm** (`e:/raptor/raptor-chatbot-llm/.claude/context/`) — if present:
- `main.md` — FastAPI app, endpoints, Ollama integration, system prompt

## 3. Perform the audit

Review all stacks for adherence to their conventions and patterns. Key areas per stack:

**raptor-chatbot:**
- ESM module usage (`import`/`export`, never `require`)
- Components V2 compliance (`IS_COMPONENTS_V2` + `TEXT_DISPLAY`, no top-level `content`)
- In-memory state (`activeGames`) — no accidental persistence introduced
- Error handling in command handlers
- LLM integration correctness (`LLM_URL/chat` call, error fallback)
- No unused imports

**raptor-chatbot-llm:**
- FastAPI conventions (Pydantic models, async routes, response models)
- System prompt injection in `/chat` — correct format
- Ollama proxy correctness (`stream: false`, timeout, error propagation)
- Security: no unvalidated inputs passed directly to Ollama
- CORS origin configured via env var (`CORS_ORIGIN`), not hardcoded

**raptor-chatbot-server:**
- ESM module usage (`import`/`export`, never `require`)
- Rate limiting on `/auth/login` and `/auth/register` (OWASP A07)
- Passwords always hashed with bcrypt — never stored or logged as plain text
- Input validation (type, format, length) at route level before business logic
- JWT signed with `JWT_SECRET` from env; default only acceptable in dev
- CORS origin configured via `CORS_ORIGIN` env var

**raptor-chatbot-web:**
- No hardcoded URLs — all fetches use relative paths (`/api/...`, `/auth/...`)
- No mock data — all state from real backend calls
- All styles in `App.css` only — no per-component CSS, no inline styles
- Props over context — `user` and `onLogout` flow from `App.jsx` as props
- ESM only — `import`/`export`, never `require`
- Route guards: `/personality` requires auth, redirects to `/auth` if not logged in
- Session stored in `sessionStorage` (not `localStorage`)

**Cross-stack consistency:**
- Remove unused imports or variables.
- Ensure environment variables are documented and used via env, not hardcoded.
- Check integration points: frontend proxy → LLM server → Ollama; frontend → auth server.
- Everything should be in English for consistency — change all messages, logs, and comments to English.
- All root folder project `README.md` files should be updated too.

Identify inconsistencies, potential bugs, or areas for improvement. Update relevant files where issues are found. Suggestions should not be left as comments — apply them directly to the code and count this as part of the audit. Also, for auditions only the agent should do changes without asking.

## 4. Save audit report

Create a report in `.claude/audits/` (use the most relevant project folder, or `e:/raptor/.claude/audits/` for cross-stack audits). The report must include:
- **Date:** today's date (YYYY-MM-DD)
- **Summary:** brief overview of findings
- **Files reviewed:** list of all files audited with a short description
- **Issues found:** categorized by severity (critical, major, minor) for each stack
- **Recommendations:** actionable suggestions with code snippets where relevant

## 5. Create stakeholder summary

Save a non-technical summary to `e:/raptor/.claude/stakeholder-summary.md`.

Write a non-technical summary of findings for stakeholders. Keep it concise, friendly, and engaging — use emojis, avoid jargon, and focus on impact. it should include all projects in the mono-repo, not just the audited one (not even just the changes it recently did, it should be about all projects history).

## 6. Create summary for the developers learning from the whole project.

Save a very technical summary to `e:/raptor/.claude/developer-summary.md`.

Write a very technical summary of findings for developers. Keep it concise, friendly, and engaging — use emojis, avoid jargon, and focus on impact. it should include all projects in the mono-repo, not just the audited one (not even just the changes it recently did, it should be about all projects history).

## 7. Commit and push changes

Commit all changes with a clear message summarizing the audit and any fixes applied, then push to the remote repository.


---

$ARGUMENTS
