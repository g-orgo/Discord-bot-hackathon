---
name: audit-quick
description: "Fast audit checklist for reviewing specific projects or modules. Use for targeted reviews. Switch to audit-full.md for comprehensive cross-stack audits."
---

# Audit Quick Checklist

**When to use:** Single project, module scope, or 2-hour time-box.  
**When to escalate:** Cross-stack inconsistencies, security patterns, or full codebase review → use `audit-full.md`.

## Quick Steps

### 1. Select scope
- Single project: `raptor-chatbot`, `raptor-chatbot-llm`, `raptor-chatbot-server`, or `raptor-chatbot-web`
- Or: specific module/file being reviewed

### 2. Read minimal context
- One relevant `copilot-instructions.md` for the project
- One `.claude/context/*.md` file if directly relevant to changes
- Skip full context tree — read only what touches the area

### 3. Review only mandated checklist below

**All stacks:**
- [ ] No `require()` — ESM only (`import`/`export`)
- [ ] No unused imports or variables
- [ ] No hardcoded secrets, URLs, or API keys (use env vars)
- [ ] Error handling exists (try/catch or try/return in async routes)
- [ ] All messages/logs/comments in English

**Project-specific:**

| Project | Must-Check |
|---------|-----------|
| `raptor-chatbot` | Components V2 flag present; no `activeGames` persistence added |
| `raptor-chatbot-llm` | Ollama calls use `stream: false`; CORS env var, not hardcoded |
| `raptor-chatbot-server` | JWT from env; rate limiting on `/auth/*`; bcrypt on passwords |
| `raptor-chatbot-web` | No hardcoded URLs; `sessionStorage` not `localStorage`; route guards on `/personality` |

### 4. Apply fixes directly
- Edit files immediately; don't comment suggestions
- Commit with clear message

### 5. Optional: Save brief report
If findings are non-trivial:
```
// e:/raptor/.claude/audits/quick-audit-YYYYMMDD-project.md
- **Date:** YYYY-MM-DD
- **Project:** name
- **Scope:** what was reviewed
- **Issues:** bullet list
- **Files changed:** list
```

---

**Need deeper review?** → Read `audit-full.md` for comprehensive workflow.
