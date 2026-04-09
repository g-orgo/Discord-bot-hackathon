# Discord Bot Hackathon — Raptor Chatbot

Mono-repo containing all services that make up the **Raptor Chatbot** ecosystem: a Discord bot powered by a local LLM, with a web frontend and authentication server.

## Repositories

| Repo | Description | Stack |
|------|-------------|-------|
| [Discord-bot-studies](https://github.com/g-orgo/Discord-bot-studies) | Discord bot — slash commands, HTTP interactions | Node.js · ESM · Express |
| [Discord-bot-LLM](https://github.com/g-orgo/Discord-bot-LLM) | LLM API server — Ollama proxy + system prompt | Python · FastAPI · Ollama |
| [Discord-bot-web-server](https://github.com/g-orgo/Discord-bot-web-server) | Auth & history server for the web frontend | Node.js · ESM · Express · MongoDB |
| [Discord-bot-web](https://github.com/g-orgo/Discord-bot-web) | Web frontend — chat interface + personality editor | React 19 · Vite |

## Architecture overview

```
Discord
  └── POST /interactions
        └── raptor-chatbot (port 3000)
              └── POST /chat ──► raptor-chatbot-llm (port 8000)
                                        └── Ollama (port 11434)

Browser
  └── raptor-chatbot-web (port 5173)
        ├── /api/* ──► raptor-chatbot-llm (port 8000)
        └── /auth/* ──► raptor-chatbot-server (port 3001)
                              └── MongoDB
```

## Quick start

Clone all sub-projects and run each service:

```bash
# 1. LLM server (requires Ollama)
cd raptor-chatbot-llm
pip install -r requirements.txt
uvicorn main:app --reload

# 2. Auth server (requires MongoDB)
cd raptor-chatbot-server
npm install
npm run dev

# 3. Web frontend
cd raptor-chatbot-web
npm install
npm run dev

# 4. Discord bot (requires ngrok)
cd raptor-chatbot
yarn install
ngrok http 3000   # copy HTTPS URL → Discord Developer Portal
yarn dev
```

## Environment variables

Each sub-project has its own `.env`. See the individual READMEs for the full list of required variables.
