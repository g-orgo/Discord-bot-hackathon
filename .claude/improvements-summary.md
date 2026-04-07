# Raptor Mono-Repo — Improvements Summary

**Date:** April 7, 2026
**Source:** Audit findings and architectural analysis

---

## Quick Wins (Low effort, high value)

1. **Add input length cap on `/ask`** — Currently any message length is forwarded to Ollama. A simple `if (message.length > 2000) return res.send(...)` in `commandHandler.js` prevents accidental timeouts from very large prompts.

2. **Persist `SYSTEM_PROMPT` to disk** — `system_prompt.py` stores the prompt in a module-level list. Any restart resets it to the env-var default. Writing to a local `.prompt` file (read on startup, write on `PUT /system-prompt`) would survive restarts without adding a database dependency.

3. **`/personality` preset editor — add change confirmation** — Clicking a preset card immediately calls `PUT /api/system-prompt`. A simple "Are you sure?" modal would prevent accidental overwrites during exploration.

4. **`History.jsx` — add empty-state message** — When `entries.length === 0`, a "No history yet. Start chatting!" message would improve the UX rather than rendering a blank list.

5. **Rate limit on `/discord/history`** — The bot secret is checked, but the endpoint has no rate limit. A simple `express-rate-limit` rule (e.g., 60 req/min per IP) would limit damage from a leaked `DISCORD_BOT_SECRET`.

6. **SSE ping interval is hardcoded** — `setInterval(..., 25000)` is scattered across `sse.js` and `history.js` as a magic number. Extract to `config.js` as `SSE_PING_INTERVAL_MS`.

---

## Medium Term (Architecture improvements)

7. **Structured logging in `raptor-chatbot`** — All logging is `console.log/console.error`. Replacing with a small structured logger (e.g., `pino`) would make errors grep-able in production logs, especially important since `askAndRespond` swallows errors silently.

8. **Stream error propagation** — `sendMessageStream()` has no `catch` on the `ReadableStream` reader loop. A network drop mid-stream leaves the chat input permanently disabled (`loading` or `streaming` remain `true`). An `onError` callback would allow Chat.jsx to reset state.

9. **Shared secret rotation strategy** — `DISCORD_BOT_SECRET` and `JWT_SECRET` are env vars with no rotation mechanism. At minimum, document a rotation runbook: update the env var, restart the server, re-deploy the bot. For JWT, consider adding a `jti` blacklist for immediate revocation.

10. **MongoDB index on `discordUsername`** — `User.findOne({ discordUsername: /^<name>$/i })` is called on every Discord history save. A case-insensitive collation index would make this sub-millisecond at scale rather than a full collection scan.

11. **`/translatechannel` — expose message limit** — The 50-message batch size is hardcoded. A Discord slash command option (`/translatechannel limit:100`) would let moderators control scope without a code change.

---

## Future Enhancements

12. **Conversation memory in `/ask`** — The Discord `/ask` command sends each message in isolation. A per-channel message buffer (e.g., last 5 turns stored in memory) would let the model maintain context across a conversation thread.

13. **Multi-model support in web chat** — `raptor-chatbot-llm` already supports a `model` parameter on `/generate`. Exposing a model selector in the web UI would allow users to compare outputs from different Ollama models without changing the system prompt.

14. **Auth: email verification** — Registration currently accepts any email format that passes the regex. Adding a verification flow (send token via SMTP, confirm before activation) would prevent abuse of the registration endpoint.

15. **Docker healthcheck for LLM service** — The `llm` Docker service depends on `ollama-init` completing, but there's no healthcheck to confirm FastAPI is accepting requests before `raptor-chatbot` starts. A `curl -f http://llm:8000/` healthcheck would eliminate a race condition on cold starts.

16. **Context file auto-generation** — Context files under `.claude/context/` drifted from reality and required manual correction during the audit. A `yarn context` npm script (or similar) that introspects commands, exports, and routes and regenerates the markdown would eliminate this class of documentation drift.
