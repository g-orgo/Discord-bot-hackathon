# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project identification — folder prefix = project

This mono-repo hosts **two completely separate projects**. The folder prefix identifies which project a folder belongs to:

| Prefix | Project | Owner |
|--------|---------|-------|
| `raptor-*` | **Raptor Chatbot** | This repo's owner — actively developed here |
| `signalraptor-*` | **SignalRaptor** | Third-party company — do NOT modify |

> **Rule:** If a folder starts with `raptor-`, it belongs to Raptor Chatbot. If it starts with `signalraptor-`, it belongs to SignalRaptor and is out of scope for any work, audit, or context.

---

## Raptor Chatbot (`raptor-*`)

The owner's project. All implementation work, audits, and context files are scoped here.

| Folder | Stack | Description |
|--------|-------|-------------|
| `raptor-chatbot/` | Node.js, Discord HTTP interactions | Discord bot (RPS game, slash commands) |
| `raptor-chatbot-llm/` | Python, FastAPI, Ollama | LLM API server |
| `raptor-chatbot-server/` | Node.js, Express, JWT | Auth server (login / register) |
| `raptor-chatbot-web/` | React 19, Vite | Web frontend (chat, personality, auth) |

Each sub-project has its own `CLAUDE.md` with project-specific commands, architecture, and conventions. Always read the relevant `CLAUDE.md` before making changes inside a sub-project folder.

---

## SignalRaptor (`signalraptor-*`)

A third-party company's project. These folders exist in this repo but are **not owned by the repo owner**.

| Folder | Stack | Description |
|--------|-------|-------------|
| `signalraptor-mobile/` | Quasar / Capacitor | Mobile app |
| `signalraptor-server/` | AdonisJS | Backend API server |
| `signalraptor-services/` | Docker Compose | Shared infrastructure |
| `signalraptor-web/` | — | Web frontend |

**Do not read, edit, audit, or reference any `signalraptor-*` folder unless the user explicitly asks.**

---

## PT-BR filler map (agent parsing)

- "talvez até" (hedge/filler) -> (drop)
