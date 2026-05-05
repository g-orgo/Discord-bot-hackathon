# Context - Documentation sync to current state

Date: 2026-05-05

## Summary
Updated cross-project documentation to reflect the current architecture, staged Discord message workflow, and active route contracts.

## Files created/modified
- Dev-process.md
  - Rewrote as a technical step-by-step of the current production flow and component responsibilities.
- docs/DEVLOG-gerado-por-ia.md
  - Replaced with a consolidated current-state overview and active endpoint/contracts summary.
- raptor-chatbot-llm/OPERATIONS_GUIDE.md
  - Replaced outdated one-pass and old-default-model content with staged pipeline operations and current env contract.
- raptor-chatbot-server/README.md
  - Added missing history/session endpoints and documented session-aware history restore behavior.
- raptor-chatbot-web/README.md
  - Added `/history` and `/profile` routes and documented session restore/profile linking features.

## Decisions made
- Prioritized operational accuracy over historical narrative in docs touched.
- Kept existing README structure where possible and only expanded sections that were stale.
- Standardized references to the current default model (`qwen2.5:1.5b`) and staged pipeline semantics.

## Known issues / next steps
- CLAUDE/internal instruction files in some subprojects still reference older architecture notes and may need a dedicated instruction refresh pass.
- If desired, the same current-state consolidation can be mirrored into presentation docs.
