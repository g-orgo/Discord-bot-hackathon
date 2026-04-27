# Improvements Summary

This backlog captures opportunities from the current codebase state.

## Quick Wins (1-2 hours)

### 1. Add automated docs-contract checks
What: Validate README env vars and endpoint contracts against `config`/route definitions.
Why: Prevent drift between implementation and docs.
Effort: ~1-2 hours.

```js
// scripts/validate-doc-contracts.mjs (example)
import fs from 'node:fs';
const readme = fs.readFileSync('README.md', 'utf8');
if (!readme.includes('DISCORD_BOT_SECRET')) {
  throw new Error('README missing DISCORD_BOT_SECRET');
}
```

### 2. Add startup config validation for required secrets
What: Fail fast when critical env vars are missing in non-dev environments.
Why: Reduces silent runtime failures.
Effort: ~1 hour.

```js
if (process.env.NODE_ENV === 'production' && !process.env.JWT_SECRET) {
  throw new Error('JWT_SECRET must be set in production');
}
```

### 3. Normalize historical docs language to English
What: Translate remaining Portuguese in DECISIONS/operations docs.
Why: Consistent onboarding for distributed teams.
Effort: ~1-2 hours.

## Medium Term (1-2 days)

### 1. Add integration tests for cross-service flows
What: Test end-to-end scenario from bot/web request to history persistence.
Why: Increases confidence in service contracts and prevents regressions.
Effort: ~1-2 days.

```text
Test scenario:
web chat -> /api/chat/stream -> final response -> /auth/history -> GET /auth/history includes new entry
```

### 2. Introduce shared API contract package
What: Share request/response schemas across services where practical.
Why: Reduces mismatch risk and duplicate validation logic.
Effort: ~1-2 days.

```ts
// packages/contracts/history.ts
export interface HistoryEntryDTO {
  userMessage: string;
  botResponse: string;
  source: 'web' | 'discord';
}
```

### 3. Improve observability with structured logs
What: Move to structured logging at boundaries (request id, route, latency).
Why: Faster troubleshooting in production-like deployments.
Effort: ~1-2 days.

```js
console.info(JSON.stringify({
  route: '/auth/register',
  latencyMs,
  status: res.statusCode,
}));
```

## Future Enhancements

### 1. Prompt/version management with persistence
What: Store system prompt versions and allow rollback.
Why: Safer prompt experiments and reproducible AI behavior.
Effort: Multi-sprint.

### 2. Event-driven history pipeline
What: Move history sync to queue/event bus for better decoupling.
Why: More resilient ingestion under spikes and clearer retry semantics.
Effort: Multi-sprint.

### 3. Multi-model routing strategy
What: Route requests to specialized models by task (translation, tone rewrite, summarization).
Why: Better quality/latency per use case.
Effort: Multi-sprint.

```python
# pseudo-router
if task == 'translate':
    model = 'qwen2.5:7b'
elif task == 'summarize':
    model = 'llama3.1:8b'
```
