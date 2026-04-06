# Raptor — Workspace Guidelines

Mono-repo containing all Raptor projects.

## Projects

| Folder | Stack | Description |
|---|---|---|
| `raptor-chatbot/` | Node.js, Discord HTTP interactions | Discord bot (RPS game, slash commands) |
| `raptor-chatbot-llm/` | Python, FastAPI, Ollama | LLM API server |
| `raptor-chatbot-server/` | Node.js, Express, JWT | Auth server (login / register) |
| `raptor-chatbot-web/` | React 19, Vite | Web frontend (chat, personality, auth) |
| `raptor-services/` | Docker Compose | Shared infrastructure services |
| `signalraptor-mobile/` | Quasar / Capacitor | Mobile app |
| `signalraptor-server/` | AdonisJS | Backend API server |
| `signalraptor-web/` | — | Web frontend |

## Skills

- `dev` — load before any code change in this workspace
