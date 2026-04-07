# 🛠️ Raptor Mono-Repo — Developer Learning Guide

**Date:** April 7, 2026
**Audience:** Developers new to the codebase
**Purpose:** Understand how the system actually works — algorithms, data flows, and design patterns

---

## Service Topology

```
┌─────────────┐    slash command     ┌──────────────────┐    POST /chat          ┌────────┐
│   Discord   │ ──────────────────▶  │  raptor-chatbot  │ ──────────────────▶   │ Ollama │
└─────────────┘                      │  (Node.js/ESM)   │   (via llm server)     └────────┘
                                     └────────┬─────────┘
                                              │ POST /discord/history
                                              ▼
┌─────────────┐    /api/* (proxy)    ┌──────────────────┐    POST /auth/*    ┌──────────────────────┐
│   Browser   │ ──────────────────▶  │raptor-chatbot-web│ ─────────────────▶ │raptor-chatbot-server │
└─────────────┘                      │  (React 19/Vite) │                    │  (Express + MongoDB) │
                                     │                  │ ◀───── SSE ──────  │                      │
                                     └────────┬─────────┘                    └──────────────────────┘
                                              │ /api/* (proxy)
                                              ▼
                                     ┌──────────────────┐
                                     │raptor-chatbot-llm│
                                     │ (FastAPI/Python) │
                                     └──────────────────┘
```

All inter-service communication is plain HTTP. In development, the Vite dev server proxies `/api/*` → port 8000 and `/auth/*` → port 3001. In production, an nginx reverse proxy performs the same role.

| Service | Stack | Port |
|---|---|---|
| raptor-chatbot | Node.js/ESM, Express | 3000 |
| raptor-chatbot-llm | Python, FastAPI, Ollama | 8000 |
| raptor-chatbot-server | Node.js/ESM, Express, MongoDB | 3001 |
| raptor-chatbot-web | React 19, Vite | 5173 (dev) / 80 (prod) |

---

## Flow 1: Web Chat Message → LLM → History

This is the primary flow for a logged-in user typing in the web interface.

```
User types message → presses Enter
       │
       ▼
Chat.jsx: send()
  ├─ Appends { role: 'user', text } to messages state
  ├─ Sets loading=true  →  shows "Thinking…" spinner
  └─ Calls sendMessageStream(text, onToken, onDone)
             │
             ▼
         chatApi.js: sendMessageStream()
           └─ POST /api/chat/stream  (Vite proxy → port 8000)
                       │
                       ▼
               raptor-chatbot-llm: POST /chat/stream
                 ├─ Prepends SYSTEM_PROMPT to user message:
                 │    "{SYSTEM_PROMPT}\n\nMessage: {text}"
                 ├─ Calls ollama_generate_stream(model, prompt)
                 │         │
                 │         ▼
                 │     ollama.py: POST http://ollama:11434/api/generate
                 │     { model, prompt, stream: True }
                 │         │
                 │         ▼ (streaming NDJSON, one JSON object per line)
                 │     Yields token strings as they are generated
                 │
                 └─ StreamingResponse emits SSE lines:
                    "data: {"token": "Hello"}\n\n"
                    "data: {"token": " world"}\n\n"
                    "data: {"done": true, "model": "llama3.2:3b"}\n\n"
                               │
                               ▼ (ReadableStream in browser)
               chatApi.js: reader.read() loop
                 ├─ Decodes bytes, buffers incomplete lines
                 ├─ Parses "data: ..." lines as JSON
                 ├─ Calls onToken(token) for each fragment
                 └─ Calls onDone(model) on done event
                               │
               ┌───────────────┴────────────────┐
               ▼                                ▼
         onToken(token):                  onDone(model):
           First token?                    setStreaming(false)
             └─ setLoading(false)          Update last bot message
                setStreaming(true)           with model label
                Append bot message         If user logged in:
           Subsequent tokens?               → saveHistoryEntry(text, fullText, model)
             └─ Update last bot message               │
                in-place (no new element)             ▼
                                           POST /auth/history { userMessage, botResponse, model }
                                           Bearer token in Authorization header
                                                      │
                                                      ▼
                                           HistoryEntry.create({ userId, ... })
                                           emitToUser(userId, 'history:new')
                                                      │
                                                      ▼
                                           SSE push to all open /auth/history/stream
                                           connections for that userId
                                                      │
                                                      ▼
                                           useHistoryStream fires 'history:new' listener
                                           → onUpdate() → refresh() → GET /auth/history
                                           → Sidebar shows new entry
```

**Streaming state machine:**
`loading` (spinner visible) flips to `streaming` (text printing) on the very first token received. This means the "Thinking…" indicator disappears the moment the model starts generating — no wait for the full response. The `firstToken` flag handles this transition inside the `onToken` callback.

---

## Flow 2: Discord `/ask` → LLM → History

```
User runs /ask message:"Hello"
       │
       ▼
Discord POSTs to https://<ngrok>/interactions
       │
       ▼
app.js: verifyKeyMiddleware validates Ed25519 signature (PUBLIC_KEY)
       │  ← 401 if invalid
       ▼
handleCommand(req, res) in commandHandler.js
       │
       ├─ IMMEDIATELY: res.send(DEFERRED_CHANNEL_MESSAGE_WITH_SOURCE)
       │    Discord interaction acknowledged < 3s  ← hard Discord requirement
       │
       └─ FIRE-AND-FORGET: askAndRespond(message, token, discordUsername)
                    │
                    ▼
             api.js: askLLM(message)
               └─ POST http://llm:8000/chat  { message }
                            │
                            ▼
                  raptor-chatbot-llm: POST /chat  (non-streaming for bot)
                    ├─ Prepends SYSTEM_PROMPT
                    └─ ollama_generate(model, prompt)  ← stream: False, full wait
                            │
                            ▼
                  Returns { model, response }
                            │
                            ▼
             askAndRespond:
               ├─ PATCH /webhooks/{APP_ID}/{token}/messages/@original
               │   Edits the deferred message with the real response
               │
               └─ If discordUsername provided (user ran /ask from a guild):
                   saveDiscordHistory(discordUsername, message, llmResponse)
                             │  (fire-and-forget — any error is logged, never thrown)
                             ▼
                   POST http://auth-server:3001/discord/history
                   Headers: X-Bot-Secret: {DISCORD_BOT_SECRET}
                   Body: { discordUsername, userMessage, botResponse }
                             │
                             ▼
                   requireBotKey validates the shared secret
                   User.findOne({ discordUsername: /^<name>$/i })
                     ├─ Not found → 204 silent skip
                     └─ Found:
                         HistoryEntry.create({ userId, source: 'discord', ... })
                         emitToUser(userId, 'history:new', { source: 'discord' })
                         → Web browser sidebar updates if user is online
```

**Why two separate auth mechanisms?**
The web client uses JWTs (Bearer tokens) — the user is identified by their login session. The Discord bot cannot log in as a web user. Instead it authenticates via a shared secret (`DISCORD_BOT_SECRET`) and the server maps `discordUsername` → `userId` with a case-insensitive regex lookup. A Discord user only gets history linked if they've entered their Discord username in their web profile.

**Why the bot uses `/chat` (non-streaming) but the web uses `/chat/stream`?**
Discord's webhook API expects a single PATCH call with the final content — there's no way to "stream" edits progressively. The web UI owns the rendering pipeline and can update state incrementally.

---

## Flow 3: Real-Time History Sidebar (SSE)

```
raptor-chatbot-server: in-memory Map
  clients: { userId → Set<res> }

User loads web app:
  useHistoryStream(user, onUpdate) effect runs
    └─ new EventSource("/auth/history/stream?token=<jwt>")
            │
            ▼
       GET /auth/history/stream?token=<jwt>
         ├─ jwt.verify(token, JWT_SECRET)  ← token in query param because
         │   EventSource API has no way to set headers
         ├─ res.setHeader('Content-Type', 'text/event-stream')
         ├─ addClient(userId, res)
         └─ setInterval(() => res.write(': ping\n\n'), 25000)
              Keeps connection alive through proxies/load balancers
              │
              ▼
       Connection stays open indefinitely
              │
       When history entry is created (web OR discord):
              │
       emitToUser(userId, event, data)
         └─ For each res in clients.get(userId):
              res.write("event: history:new\ndata: {"source":"web"}\n\n")
              │
              ▼
       Browser EventSource fires named listener
       → onUpdate()
       → refresh() fetches GET /auth/history
       → Sidebar re-renders with new entry

User closes tab:
  req.on('close') fires → clearInterval(ping), removeClient(userId, res)
  If Set becomes empty → Map entry deleted
```

**Reconnection logic:**
`useHistoryStream` handles `onerror` by closing and reconnecting after a 3s delay. A `closed` boolean in the closure prevents reconnection after intentional unmount (React cleanup function sets `closed = true` before calling `es.close()`).

---

## Algorithm: Channel Translation Detection

`/translatechannel` scans the last 50 messages and uses the LLM itself to detect and translate non-English text.

```javascript
async function detectAndTranslate(text) {
  const prompt =
    'You are a translation tool. Follow these rules exactly:\n' +
    '1. If the message is written entirely in English, respond with: ENGLISH\n' +
    '2. If the message contains any non-English text, translate the entire message to English.\n\n' +
    `Message: ${text}`;

  // Uses /generate (raw proxy) — NOT /chat
  // /chat injects the system prompt that rewrites messages warmly
  // For translation we need raw instruction-following, no persona
  const res = await fetch(`${LLM_URL}/generate`, { method: 'POST', ... });
  const json = await res.json();

  if (json.response.trim().toUpperCase() === 'ENGLISH') return null; // skip
  return json.response; // the translation
}
```

**Result chunking:**
Discord `TEXT_DISPLAY` has a 4000-character limit. Results are accumulated and split into multiple follow-up messages when needed:

```javascript
const LIMIT = 4000;
let current = header; // "**Non-English messages found:**"
for (const entry of entries) {
  const line = `\n\n${entry}`;
  if (current.length + line.length > LIMIT) {
    chunks.push(current); // flush current chunk
    current = entry;      // start new chunk
  } else {
    current += line;
  }
}
chunks.push(current);
// First chunk: editInteractionResponse (edits the deferred message)
// Extra chunks: sendInteractionFollowup (new webhook messages)
```

---

## Algorithm: SSE Streaming Response Parser

`sendMessageStream` in `chatApi.js` parses a raw `ReadableStream` of SSE data from FastAPI. The key challenge is that network packets don't align with SSE message boundaries.

```javascript
const reader = res.body.getReader();
const decoder = new TextDecoder();
let buffer = '';

while (true) {
  const { done, value } = await reader.read(); // raw Uint8Array chunk
  if (done) break;

  buffer += decoder.decode(value, { stream: true });
  const lines = buffer.split('\n');
  buffer = lines.pop(); // last item may be an incomplete line — keep it

  for (const line of lines) {
    if (!line.startsWith('data: ')) continue;
    const data = JSON.parse(line.slice(6));
    if (data.done) { onDone?.(data.model); return; }
    onToken(data.token);
  }
}
```

`buffer = lines.pop()` is the core pattern: `split('\n')` always produces at least one trailing empty string, so `pop()` safely removes the last potentially-incomplete line and carries it into the next chunk.

---

## Auth Flow: Registration → JWT → Session

```
POST /auth/register { email, password, displayName }
         │
         ▼
  Validation (all at route level, before any DB operation):
    ├─ typeof checks on all fields
    ├─ email: /^[^\s@]+@[^\s@]+\.[^\s@]+$/  (no regex injection — safe pattern)
    ├─ password: length >= 6
    └─ displayName: trimmed length >= 2
         │
         ▼
  bcrypt.hash(password, 10)
    ← Cost factor 10 ≈ 100ms on modern hardware; defeats bulk cracking
  User.create({ email: email.toLowerCase(), passwordHash, displayName })
         │
         ├─ Mongoose unique index on email → code 11000 → 409 Conflict
         │
         ▼
  jwt.sign({ sub: userId, email, displayName }, JWT_SECRET, { expiresIn: '7d' })
         │
         ▼
  { token, email, displayName, discordUsername: null }

POST /auth/login { email, password }
         │
         ▼
  User.findOne({ email: email.toLowerCase() })
  bcrypt.compare(password, user.passwordHash)
  ├─ User not found  →  401 "Invalid credentials."  ← same message
  └─ Hash mismatch   →  401 "Invalid credentials."  ← prevents user enumeration
```

**Session storage (web):**
On login, `useAuth.js` writes two `sessionStorage` entries:
- `raptor_token` — the JWT string (sent as Bearer on every API call)
- `raptor_user` — `{ email, displayName, discordUsername }` JSON (used for display without a round-trip)

`sessionStorage` (not `localStorage`) means credentials are lost on tab close — intentional. On app boot, `getStoredUser()` reads from `sessionStorage` to restore state without a login page.

---

## LLM Startup Model Check

`main.py` uses a FastAPI `lifespan` context to ensure the Ollama model is installed before accepting any requests:

```python
async def _ensure_model() -> None:
    for attempt in range(1, 4):  # max 3 attempts
        tags = await client.get(f"{OLLAMA_BASE_URL}/api/tags")
        installed = [m["name"].split(":")[0] for m in tags.json()["models"]]

        if DEFAULT_MODEL in installed:
            return  # already there — start immediately

        # Pull with long timeout (large models can take minutes)
        await client.post(f"{OLLAMA_BASE_URL}/api/pull",
            json={"name": DEFAULT_MODEL, "stream": False},
            timeout=OLLAMA_PULL_TIMEOUT)  # default 600s
        return

    print(f"WARNING: Could not ensure model after 3 attempts")
    # Server still starts — requests will fail gracefully at the route level
```

Without this, if Ollama is running but the model isn't downloaded, `ollama_generate` returns an error that propagates as `null` in `detectAndTranslate`, silently skipping all messages and returning "No non-English messages found in the channel."

---

## Design Patterns Reference

| Pattern | Where | Why |
|---|---|---|
| **Fire-and-forget** | `askAndRespond`, `saveDiscordHistory`, `saveHistoryEntry` | Background work that must not block the primary response |
| **Deferred interaction** | All Discord commands except `/ask` (sync) | Discord's 3-second acknowledgment requirement |
| **SSE streaming** | LLM `/chat/stream` → web chat | Progressive rendering; avoids full-response wait |
| **SSE push** | Auth server `/auth/history/stream` → web sidebar | Real-time updates across tabs and from Discord |
| **Module-level mutable state** | `system_prompt.py` `_prompt` list | Allows `PUT /system-prompt` to update in-memory without restart |
| **Shared secret auth** | `requireBotKey`, `X-Bot-Secret` | Bot→server auth without exposing user JWTs |
| **Regex case-insensitive lookup** | `User.findOne({ discordUsername: /^<name>$/i })` | Discord username linking tolerates capitalization differences |

---

## Strengths

- **Consistent ESM** — All Node.js services use `import`/`export` throughout; no `require` anywhere.
- **Async/await everywhere** — No callback pyramids; background tasks use fire-and-forget with internal error handling that never throws upward.
- **Pydantic I/O contracts** — Every FastAPI endpoint has typed request/response models; Field constraints enforce length limits before Ollama is called.
- **Security by default** — Bcrypt (cost 10), JWT with env-configured secret, rate limiting on auth endpoints (OWASP A07), shared secret for bot-to-server communication.
- **Components V2 compliance** — Discord bot exclusively uses `IS_COMPONENTS_V2` + `TEXT_DISPLAY`; no legacy top-level `content` field.
- **Graceful degradation** — `askLLM` returns a fallback string on any error; `saveDiscordHistory` and `saveHistoryEntry` are fire-and-forget and never surface errors to users.
- **Session intentionally ephemeral** — `sessionStorage` means credentials clear on tab close by design; no accidental persistence.
- **Startup model guarantee** — `_ensure_model()` fires before the server accepts traffic, preventing the silent null-response failure from a missing Ollama model.
- **Real-time history** — SSE keeps both the web sidebar and the full history view live without polling.
