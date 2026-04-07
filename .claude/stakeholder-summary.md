# 🦖 Raptor — Project Health Summary

**Date:** April 7, 2026

---

## What is Raptor?

Raptor is a suite of connected tools that help people communicate better — powered by AI. It has two main interfaces: a **Discord bot** and a **web app**, both backed by the same AI engine.

---

## 🤖 raptor-chatbot — Discord Bot

The Discord bot lets users interact with the AI directly from any Discord server or DM. It has three slash commands:

- **`/ask`** — Send a message to the AI and get a rewritten, more empathetic version back
- **`/translatechannel`** — Automatically detects and translates all non-English messages in a channel
- **`/clearchannel`** — Debug tool to wipe all messages from a channel

History: The bot started out with a Rock-Paper-Scissors game (`/challenge`), which has since been fully removed to keep things clean and focused on the communication tools.

**Current state:** ✅ Healthy — clean codebase, no dead code, all commands working correctly.

---

## 🧠 raptor-chatbot-llm — AI Engine

The brain of the operation. A Python API server that wraps a locally-running Ollama AI model (`llama3.2:3b`). It takes messages, applies a personality prompt, and returns a rewritten version.

Recent additions:
- **Streaming responses** — the AI now types its answer in real-time instead of waiting for the full response
- **Startup model check** — the server now automatically downloads the AI model if it's missing, preventing silent failures

**Current state:** ✅ Healthy — stable, well-structured, handles errors gracefully.

---

## 🔐 raptor-chatbot-server — Auth & History Server

Handles user accounts, login, and conversation history. Backed by MongoDB for persistent storage.

Features:
- Register/login with email and password
- JWT-based authentication (tokens expire after 7 days)
- Full conversation history per user
- **Real-time sidebar** — when a new conversation is saved (from the web OR Discord), the browser sidebar updates instantly via a live connection (SSE)
- **Discord linking** — users can connect their Discord username to their web account so bot conversations appear in the web history

**Current state:** ✅ Healthy — persistent storage, secure passwords, rate limiting on login attempts.

---

## 🌐 raptor-chatbot-web — Web App

The web frontend. A clean, responsive React app that lets users:

- Chat with the AI in real-time (with token-by-token streaming)
- Switch between AI "personalities" (Empathetic, Professional, Casual, Concise, Creative, or Custom)
- Browse and restore conversation history
- Link their Discord account in the profile page

**Current state:** ✅ Healthy — all API calls use the proxy correctly, session is properly ephemeral, route guards in place.

---

## 🐳 raptor-services — Infrastructure

Docker Compose setup that wires everything together. All service ports are now configurable via environment variables (e.g. if port 8000 is already in use, just set `LLM_PORT=8001` in `.env`).

---

## 📱 signalraptor — Mobile & Server

Separate project — mobile app (`Quasar/Capacitor`) with its own backend (`AdonisJS`). Not part of the active development cycle.

---

## 🧹 What Was Cleaned Up Recently

- Removed the `/challenge` RPS game completely (code, docs, context files)
- Removed `logChannelMessages` dead code
- Fixed all documentation that still described the auth server as "in-memory" (it's been MongoDB for a while)
- Updated all AI context files to reflect the current split-file architecture of the LLM server
- Added streaming support to the web chat interface
- Added automatic Ollama model pull on startup (prevents "no messages found" silent failure)
- All port conflicts in Docker are now resolvable via `.env` — no need to edit compose files

---

## 🟢 Overall Health: Strong

The codebase is clean, well-structured, and following consistent patterns across all four services. Security fundamentals (passwords hashed, JWT, rate limiting, input validation) are solid throughout.
