# Raptor Platform Snapshot

## What this product is 🚀
Raptor is a multi-app AI communication platform that helps people rewrite, translate, and manage conversations across Discord and the web.

It is organized as a mono-repo with four active Raptor services:
- `raptor-chatbot`: Discord bot experience
- `raptor-chatbot-llm`: AI inference gateway powered by local Ollama
- `raptor-chatbot-server`: account, auth, and history backend
- `raptor-chatbot-web`: browser UI for chat, profile, and personality controls

## What users can do today ✅
- Chat with the assistant from Discord slash commands and from a web app.
- Rewrite content into professional English tone.
- Stream AI responses in real time for faster feedback.
- Create an account and keep chat history tied to their profile.
- Link Discord username to bring bot conversations into web history.
- Update the active AI system prompt from the web personality area.

## Why it matters 💡
- It centralizes communication improvement workflows in one product.
- It supports both community-first usage (Discord) and personal workflows (web app).
- It combines AI assistance with identity, history, and continuity across channels.

## Service-by-service overview 🧩

### Discord Bot (`raptor-chatbot`) 🤖
Receives Discord interactions over HTTP, handles slash commands, and sends requests to the LLM service. It can also send Discord-originated history records to the auth server.

### LLM Service (`raptor-chatbot-llm`) 🧠
FastAPI service that wraps a local Ollama model and exposes chat/generate/stream endpoints. It also supports runtime system prompt management.

### Auth + History Server (`raptor-chatbot-server`) 🔐
Manages registration, login, JWT authentication, profile updates, and persistent history storage. Also exposes bot-secured endpoints for Discord history ingestion.

### Web Frontend (`raptor-chatbot-web`) 🎨
React SPA where users chat, authenticate, view history, and manage prompt/personality behavior. Uses responsive navigation for desktop and mobile.

## Current maturity 🏁
- Core end-to-end flow is operational: user input -> AI response -> optional history persistence.
- Real-time updates are supported through streaming and server-sent events.
- Security baseline is in place with JWT, bot secret headers, and input validation.
- The architecture supports iterative improvements without major rewrites.

## Mono-repo scope note 📦
The repository also contains non-Raptor folders (`signalraptor-*`) owned externally and intentionally out of the current product development scope.
