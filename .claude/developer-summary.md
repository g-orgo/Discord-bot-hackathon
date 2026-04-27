# Raptor Developer Summary

This document explains how the current system works end-to-end.

## 1. System Map

```text
Discord User
   |
   v
raptor-chatbot (/interactions)
   |  POST /chat or /chat/stream
   v
raptor-chatbot-llm (FastAPI -> Ollama)
   |
   +--> response tokens / final response
   |
   +--> optional bot history POST /discord/history
            v
       raptor-chatbot-server (Express + Mongo)
            |
            +--> SSE history:new
                     v
               raptor-chatbot-web
```

For browser users:

```text
Browser (React web app)
   |  /api/chat/stream, /auth/*
   v
Vite proxy
   |-- /api/*  -> raptor-chatbot-llm
   '-- /auth/* -> raptor-chatbot-server
```

## 2. Core Data Flows

### Flow A: Discord `/message` command
1. Discord sends interaction to `raptor-chatbot` `/interactions`.
2. Bot verifies signature with `verifyKeyMiddleware`.
3. Command handler returns deferred response immediately.
4. Bot calls LLM service `/chat` (or other route depending on command).
5. LLM service composes prompt and forwards to Ollama `/api/generate`.
6. Bot receives final text and edits original Discord interaction message.
7. If Discord username is available, bot posts to server `/discord/history`.
8. Server links history to existing user or stores pending entry by normalized username.

### Flow B: Web chat streaming
1. User submits message in web `Chat` view.
2. Frontend calls `/api/chat/stream`.
3. LLM service streams SSE token chunks (`data: { token }`).
4. Frontend appends tokens to the latest bot message in-place.
5. On completion, frontend persists full exchange to `/auth/history` when authenticated.
6. Server emits `history:new`; frontend refreshes recent history via hooks.

### Flow C: Discord username linking + pending claim
1. Bot sends history for unknown `discordUsername`.
2. Server stores pending entry with TTL expiration.
3. User registers or updates profile with matching `discordUsername`.
4. Server claims pending entries and writes them to main history collection.
5. Server emits SSE event so web UI updates.

## 3. Key Algorithms and Behaviors

### 3.1 Discord username conflict check (server)
Auth routes use explicit case-insensitive checks before writes, then still rely on DB unique index as final guard.

```js
const existing = await User.findOne({ discordUsername })
  .collation({ locale: 'en', strength: 2 })
  .select('_id')
  .lean();
```

This avoids false success paths when uniqueness should return HTTP 409.

### 3.2 Session grouping (web)
History entries are grouped by `sessionId` for multi-exchange restoration. Legacy entries without a session ID are handled as standalone sessions.

### 3.3 Translation detection loop (bot)
`translateChannelMessages` fetches recent channel messages, runs `detectAndTranslate`, and builds chunked response blocks respecting Discord payload limits.

### 3.4 Streaming assembly (web)
The chat view keeps `fullText` as tokens arrive and updates the last bot message progressively, then tags final metadata (`model`) when stream is done.

## 4. Component Relationships

- `raptor-chatbot` delegates long-running behavior to `api/` modules and responds quickly to Discord interaction timing constraints.
- `raptor-chatbot-llm` isolates Ollama HTTP calls and schema validation via Pydantic models.
- `raptor-chatbot-server` isolates concerns via route + model + middleware folders.
- `raptor-chatbot-web` uses hooks for auth/history concerns and views for route-level UX composition.

## 5. State Management

### Backend
- Server persistence: MongoDB collections for users/history/pending entries.
- LLM runtime prompt state: in-memory mutable prompt manager (resets on restart).
- Bot runtime state: request-scoped operations with deferred interaction editing.

### Frontend
- Auth/session: `sessionStorage` keys for token + user snapshot.
- UI session threading: `sessionIdRef` and grouped history in hooks.
- Live updates: SSE hook triggers refresh on history events.

## 6. Patterns Currently Used

- ESM modules across JS projects.
- Async/await at network and DB boundaries.
- Deferred interaction responses for Discord timeout safety.
- SSE for low-latency output and history notifications.
- Environment-based config with sensible local defaults.

## 7. Strengths in Current Architecture

- Clear service boundaries between bot, LLM, auth server, and frontend.
- Good test coverage on critical backend routes.
- Practical real-time UX with streaming and SSE refresh.
- Extensible model/prompt architecture for future AI behavior tuning.
