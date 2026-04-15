# README Updates

**Date:** 2026-04-09

## Summary

All READMEs across the Discord bot ecosystem were rewritten or created from scratch. The mono-repo had no README at all (causing the GitHub page to be empty with no navigation), and the sub-project READMEs were either missing, outdated, or were the default Vite template.

## Files created/modified

| File | Action | Description |
|------|--------|-------------|
| `E:/raptor/README.md` | Created | Mono-repo overview with architecture diagram and links to all 4 sub-repos |
| `raptor-chatbot/README.md` | Rewritten | Full setup guide: ngrok, env vars, slash commands table, architecture |
| `raptor-chatbot-llm/README.md` | Updated | Added `/chat`, `/chat/stream`, `/system-prompt` endpoints and all env vars |
| `raptor-chatbot-server/README.md` | Updated | Added MongoDB, `/auth/register`, `/auth/profile`, history endpoints, SSE stream, `/discord/history` |
| `raptor-chatbot-web/README.md` | Rewritten | Replaced Vite template boilerplate with actual project docs: routes, features, architecture |

## Decisions made

- **Mono-repo README** links directly to the individual GitHub repos by their actual remote URL (not relative paths), so they work from GitHub.com.
- Each README ends with a **Related services** section cross-linking the other repos.
- Branch names matched per-repo: `raptor-chatbot` and both server/web repos use `main`; `raptor-chatbot-llm` uses `master`. Mono-repo uses `master`.
- GPG signing was disabled per user request (`-c commit.gpgsign=false`).

## Known issues / next steps

- `raptor-chatbot` remote (`Discord-bot-studies`) has been renamed to `Discord-bot` on GitHub — the remote URL still works via redirect but could be updated locally with `git remote set-url origin https://github.com/g-orgo/Discord-bot.git`.
- The mono-repo README links to sub-repos by their GitHub URLs — these could be updated if repos are ever renamed.
