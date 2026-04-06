# Audit Report — Raptor Mono-repo

**Date:** 2026-04-06  
**Scope:** All four active projects: `raptor-chatbot`, `raptor-chatbot-llm`, `raptor-chatbot-server`, `raptor-chatbot-web`

---

## Summary

Cross-stack audit of the full Raptor mono-repo. Two security/correctness issues found (one OWASP-class) and one cleanliness issue. All three were fixed inline during the audit. The four projects are otherwise consistent, well-structured, and follow their documented conventions.

---

## Files Reviewed

| File | Notes |
|---|---|
| `raptor-chatbot/app.js` | Interaction dispatch, command handlers |
| `raptor-chatbot/api/api.js` | LLM client, translate logic |
| `raptor-chatbot/api/discord.js` | Discord API helpers |
| `raptor-chatbot/utils.js` | Shared utilities |
| `raptor-chatbot-llm/main.py` | FastAPI app, Ollama proxy, system prompt |
| `raptor-chatbot-server/app.js` | Express auth server, JWT, bcrypt |
| `raptor-chatbot-web/src/App.jsx` | Root, routing, session |
| `raptor-chatbot-web/src/views/Chat.jsx` | Chat view |
| `raptor-chatbot-web/src/views/Playground.jsx` | Personality editor |
| `raptor-chatbot-web/src/views/Auth.jsx` | Login / Register |
| `raptor-chatbot-web/src/components/Nav.jsx` | Navigation |
| `raptor-chatbot-web/vite.config.js` | Proxy config |

---

## Issues Found

### MAJOR — CORS origin hardcoded in `raptor-chatbot-llm/main.py`

**File:** `raptor-chatbot-llm/main.py`  
**Severity:** Major  
**Status:** ✅ Fixed

`allow_origins` was hardcoded to `["http://localhost:5173"]`, making any non-local deployment silently fail for browser clients. Fixed by reading from `CORS_ORIGIN` env var with the local URL as fallback:

```python
allow_origins=[os.getenv("CORS_ORIGIN", "http://localhost:5173")],
```

---

### MAJOR — No rate limiting on auth endpoints (OWASP A07)

**File:** `raptor-chatbot-server/app.js`  
**Severity:** Major (security)  
**Status:** ✅ Fixed

`POST /auth/login` and `POST /auth/register` had no protection against brute-force or credential stuffing attacks — a direct violation of OWASP A07:2021 (Identification and Authentication Failures). Fixed with an in-memory rate limiter: max 10 attempts per IP per 15-minute sliding window, returning HTTP 429 on excess.

```js
const AUTH_LIMIT = 10;
const AUTH_WINDOW_MS = 15 * 60 * 1000;
const authAttempts = new Map();

function rateLimit(req, res, next) { ... }

app.post('/auth/login', rateLimit, ...);
app.post('/auth/register', rateLimit, ...);
```

Note: the map is in-memory and resets on restart. Acceptable for the current single-process deployment model.

---

### MINOR — Unused imports in `raptor-chatbot/app.js`

**File:** `raptor-chatbot/app.js`  
**Severity:** Minor  
**Status:** ✅ Fixed

Four symbols imported but never referenced in `app.js`:
- `getRandomEmoji` from `./utils.js`
- `logChannelMessages` from `./api/discord.js`
- `editInteractionResponse` from `./api/discord.js` (used inside `api/api.js`, not in `app.js`)
- `sendInteractionFollowup` from `./api/discord.js` (unused)

All four removed from the import declarations.

---

## No Issues Found

The following were inspected and found correct:

- **ESM convention** — all four projects use `import`/`export` exclusively ✓
- **Components V2** — `raptor-chatbot` uses `IS_COMPONENTS_V2` + `TEXT_DISPLAY` on all responses ✓
- **Pydantic models** — all `raptor-chatbot-llm` routes use typed request/response models ✓
- **Async routes** — all Ollama-calling routes are `async def` with `httpx.AsyncClient` ✓
- **Input validation** — `raptor-chatbot-server` validates type, format, and length at route level ✓
- **Password hashing** — bcrypt used everywhere, no plain-text passwords stored or logged ✓
- **JWT verification** — `requireAuth` middleware correctly validates Bearer tokens ✓
- **No hardcoded URLs in frontend** — all fetches use relative paths through Vite proxy ✓
- **Route guards** — `/personality` correctly redirects unauthenticated users to `/auth` ✓
- **Session storage** — token and user stored in `sessionStorage` (cleared on tab close) ✓
- **Rate limit note** — `raptor-chatbot-llm` system prompt endpoints are not protected server-side (by design — the Personality page requires auth in the frontend). Acceptable for current architecture.

---

## Recommendations

1. **Add `CORS_ORIGIN` to deployment docs / `.env.example`** for both `raptor-chatbot-llm` and `raptor-chatbot-server` so production deployments know to set it.
2. **Consider a reverse proxy** (nginx/Caddy) in production to expose only ports 5173-equivalent and 3001, keeping port 8000 (LLM) internal-only.
3. **Persist rate-limit state to Redis** if the server ever becomes multi-process.
