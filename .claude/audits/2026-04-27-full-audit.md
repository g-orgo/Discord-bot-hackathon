# Full Audit Report - Raptor Mono-Repo

## Date
2026-04-27

## Summary
Completed a full cross-stack audit for `raptor-chatbot`, `raptor-chatbot-llm`, `raptor-chatbot-server`, and `raptor-chatbot-web`.

The audit found one functional bug (duplicate Discord username conflict not enforced reliably in register flow tests), one security/config hardening issue (hardcoded secret fallback in bot history sync), and multiple documentation consistency gaps.

All high-impact code issues discovered during this audit were fixed directly and validated with tests/lint.

## Files Reviewed
- `raptor-chatbot/app.js` - interaction entrypoint and dispatch rules
- `raptor-chatbot/commands.js` - slash command definitions
- `raptor-chatbot/api/api.js` - LLM integration, translation flow, history sync
- `raptor-chatbot/api/discord.js` - Discord API integration helpers
- `raptor-chatbot/utils.js` - shared Discord request utility
- `raptor-chatbot/handlers/commandHandler.js` - command routing behavior
- `raptor-chatbot/handlers/commandHandler.test.js` - command behavior tests
- `raptor-chatbot/api/api.test.js` - LLM/translation behavior tests
- `raptor-chatbot/README.md` - bot documentation
- `raptor-chatbot-llm/main.py` - FastAPI entrypoint and route wiring
- `raptor-chatbot-llm/config.py` - environment/config constants
- `raptor-chatbot-llm/ollama.py` - Ollama client boundary
- `raptor-chatbot-llm/schemas.py` - Pydantic models
- `raptor-chatbot-llm/routes/chat.py` - chat and streaming routes
- `raptor-chatbot-llm/routes/generate.py` - raw generation route
- `raptor-chatbot-llm/routes/system_prompt.py` - prompt management routes
- `raptor-chatbot-llm/tests/test_routes.py` - route-level tests
- `raptor-chatbot-llm/tests/test_system_prompt.py` - prompt state tests
- `raptor-chatbot-llm/README.md` - LLM service documentation
- `raptor-chatbot-server/src/config.js` - auth/server config and env defaults
- `raptor-chatbot-server/src/models/User.js` - user model and username uniqueness
- `raptor-chatbot-server/src/routes/auth.js` - register/login/profile flows
- `raptor-chatbot-server/src/routes/history.js` - history CRUD routes
- `raptor-chatbot-server/src/routes/discord.js` - bot->server ingestion
- `raptor-chatbot-server/src/routes/auth.test.js` - auth route tests
- `raptor-chatbot-server/src/routes/discord.test.js` - discord route tests
- `raptor-chatbot-server/src/routes/history.test.js` - history route tests
- `raptor-chatbot-server/README.md` - server documentation
- `raptor-chatbot-web/src/App.jsx` - route guards and app composition
- `raptor-chatbot-web/src/components/Nav.jsx` - nav/dropdown behavior
- `raptor-chatbot-web/src/views/Chat.jsx` - chat streaming/session behavior
- `raptor-chatbot-web/src/views/History.jsx` - history list/actions
- `raptor-chatbot-web/src/views/Auth.jsx` - auth view behavior
- `raptor-chatbot-web/src/views/Profile.jsx` - profile integration behavior
- `raptor-chatbot-web/src/api/*.js` - API wrappers (auth/chat/history)
- `raptor-chatbot-web/src/hooks/*.js` - auth/history stream hooks
- `raptor-chatbot-web/README.md` - frontend documentation

## Issues Found

### Critical
#### `raptor-chatbot`
- `api/api.js`: hardcoded fallback secret (`DISCORD_BOT_SECRET`) existed in source. This creates accidental credential coupling and unsafe defaults.
- Status: fixed.

### Major
#### `raptor-chatbot-server`
- `src/routes/auth.test.js` failure reproduced: duplicate `discordUsername` register path returned `201` instead of `409` in test environment.
- Root cause: flow depended on DB unique index enforcement only, which is not sufficient as sole guard in all test/runtime conditions.
- Status: fixed via explicit pre-check with case-insensitive collation in auth route.

#### `raptor-chatbot-server`
- `README.md`: env var names and behavior docs were stale (`MONGODB_URI`/`BOT_SECRET` no longer reflected active config contract).
- Status: fixed (`DB_PATH`, `DISCORD_BOT_SECRET`, pending Discord history behavior documented).

### Minor
#### `raptor-chatbot-llm`
- `README.md`: non-English prompt/response JSON examples reduced doc consistency with English-only conventions.
- Status: fixed.

#### Cross-stack documentation
- Some historical decision/context docs still contain non-English prose and should be normalized over time for long-term consistency.
- Status: open (non-runtime docs only).

## Recommendations

1. Keep route-level uniqueness checks for user-facing constraints, even when unique DB indexes exist.

```js
if (await isDiscordUsernameTaken(trimmedDiscordUsername)) {
  return res.status(409).json({ error: 'Discord username already linked to another account.' });
}
```

2. Never fallback to production-like secrets in source defaults.

```js
const DISCORD_BOT_SECRET = process.env.DISCORD_BOT_SECRET;
if (!DISCORD_BOT_SECRET) return;
```

3. Add a lightweight docs quality gate to prevent env var drift.

```json
{
  "scripts": {
    "check:docs": "node scripts/validate-doc-contracts.mjs"
  }
}
```

4. Continue moving menu/list behavior toward single-config-driven patterns where practical (especially action groups with duplicate wiring in navigation/history views).

## Validation Executed
- `raptor-chatbot`: `yarn test` -> passed (11/11)
- `raptor-chatbot-server`: `npm test -- --run` -> passed (33/33)
- `raptor-chatbot-web`: `npm run lint` -> passed
- `raptor-chatbot-llm`: `python -m pytest -q` -> passed (11/11)
