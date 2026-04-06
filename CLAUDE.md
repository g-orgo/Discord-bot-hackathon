# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Structure

Mono-repo containing all Raptor projects.

| Folder | Stack | Description |
|--------|-------|-------------|
| `raptor-chatbot/` | Node.js, Discord HTTP interactions | Discord bot (RPS game, slash commands) |
| `raptor-chatbot-llm/` | Python, FastAPI, Ollama | LLM API server |
| `raptor-chatbot-server/` | Node.js, Express, JWT | Auth server (login / register) |
| `raptor-chatbot-web/` | React 19, Vite | Web frontend (chat, personality, auth) |
| `raptor-services/` | Docker Compose | Shared infrastructure services |
| `signalraptor-mobile/` | Quasar / Capacitor | Mobile app |
| `signalraptor-server/` | AdonisJS | Backend API server |
| `signalraptor-web/` | — | Web frontend |

## Active projects

The actively developed projects in this repo are:

- **`raptor-chatbot/`** — Discord bot (HTTP interactions, Node.js/ESM)
- **`raptor-chatbot-llm/`** — Dedicated LLM server for chatbot messages (Python, FastAPI, Ollama)
- **`raptor-chatbot-server/`** — Auth server for the web frontend (Node.js/ESM, Express, JWT)
- **`raptor-chatbot-web/`** — Web frontend for the chatbot (React 19, Vite)

All implementation work, audits, and context files are scoped to these two projects. The other folders (`raptor-services/`, `signalraptor-*`) are separate projects — do not modify them unless explicitly asked.

## Per-project guidance

Each sub-project has its own `CLAUDE.md` with project-specific commands, architecture, and conventions. Always read the relevant `CLAUDE.md` before making changes inside a sub-project folder.
