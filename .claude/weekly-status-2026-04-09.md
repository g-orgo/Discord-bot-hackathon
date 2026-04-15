# Weekly Status — Discord Bot Ecosystem
**Date:** 2026-04-09

---

## 1) What was completed this week

- Full cross-stack audit completed across all 4 active sub-projects (`raptor-chatbot`, `raptor-chatbot-llm`, `raptor-chatbot-server`, `raptor-chatbot-web`) — no critical security issues found
- Fixed major documentation mismatch in `raptor-chatbot-server`: docs still described an in-memory users array; the server had already been migrated to MongoDB + Mongoose with a multi-file structure (`src/routes/`, `src/models/`, `src/middleware/`). Full endpoint table (10 endpoints) now documented
- Removed dead code: `logChannelMessages()` was exported from `api/discord.js` but never called — cleaned up along with stale context references to `/logchannel` and `/test` commands that were never registered
- Updated `raptor-chatbot-llm` docs to include `POST /chat/stream` (added in a prior session but undocumented) and two missing env vars (`OLLAMA_TIMEOUT`, `OLLAMA_PULL_TIMEOUT`)
- Rewrote all READMEs from scratch: mono-repo root, `raptor-chatbot`, `raptor-chatbot-llm`, `raptor-chatbot-server`, and `raptor-chatbot-web` (was still showing the default Vite template boilerplate)
- All stale `.claude/context/` files synced to match actual code

---

## 2) What is blocked or at risk

- **System prompt resets on every restart** — `system_prompt.py` stores the active prompt in a module-level variable. Any `raptor-chatbot-llm` container restart wipes any runtime changes made via `PUT /system-prompt` back to the env-var default. No persistence layer in place yet
- **Stream error state not recoverable** — `sendMessageStream()` in the web frontend has no `catch` on the `ReadableStream` reader loop. A network drop mid-response leaves the chat input permanently stuck in a loading/streaming state, requiring a page reload
- **`/discord/history` has no rate limit** — The bot secret is validated, but there's no `express-rate-limit` on that route. A leaked `DISCORD_BOT_SECRET` would give unrestricted access to all user history
- **Docker cold-start race condition** — No healthcheck on the LLM service container. `raptor-chatbot` can start before FastAPI is ready to accept requests, causing silent failures on the first `/ask` commands after a cold deploy

---

## 3) What is planned for next week

- Add input length cap on `/ask` in `commandHandler.js` — prevent accidental Ollama timeouts from very large Discord prompts
- Persist system prompt to disk in `raptor-chatbot-llm` — read from a `.prompt` file on startup, write on `PUT /system-prompt`, survives restarts without adding a DB dependency
- Add rate limit to `GET /discord/history` route (`express-rate-limit`, ~60 req/min per IP)
- Fix stream error propagation in `raptor-chatbot-web` — add `onError` callback to `sendMessageStream()` so `Chat.jsx` can reset loading state on mid-stream network failures
- Add empty-state UI to `History.jsx` — "No history yet. Start chatting!" when the list is empty
- Add a Docker healthcheck to the LLM service to eliminate the cold-start race condition
