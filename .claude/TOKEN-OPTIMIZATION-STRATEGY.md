# Claude Code Token Optimization Strategy — Global

**Date updated:** 2026-04-15  
**Reason:** Token budget constraints across multi-project workspace; SignalRaptor + Raptor in same mono-repo.

## Key Decision: Workspace Separation

**Rule:** Always use **project-specific workspaces**, never the mono-repo root.

- **Raptor work:** Use `raptor.code-workspace`
- **SignalRaptor work:** Use `signalraptor.code-workspace`  
- **Reason:** Avoids loading cross-project instructions automatically (e.g., SignalRaptor instructions when working on Raptor)

### Before vs After (Global Scenarios across E:/raptor, E:/DataScraper, E:/GoResolve)
| Scenario | Before | After | Savings |
|----------|--------|-------|---------|
| Portfolio static instructions (all 3 roots open) | ~11839 | ~10289 | **~13%** |
| GoResolve backend dev (old broad-context behavior vs new policy) | ~27388 | ~6436 | **~77%** |
| GoResolve frontend dev (old broad-context behavior vs new policy) | ~33686 | ~6348 | **~81%** |
| GoResolve cross-end audit (old broad-context behavior vs new policy) | ~54428 | ~8893 | **~84%** |
| DataScraper routine task (old broad-context behavior vs new policy) | ~12228 | ~2180 | **~82%** |
| Portfolio average per session (1/3 Raptor dev, 1/3 GoResolve frontend dev, 1/3 DataScraper routine) | ~16995 | ~4534 | **~73%** |

**What changed globally so far:**
- SignalRaptor server instructions: 2882 -> 595 tokens (saved 2287)
- SignalRaptor web instructions: 1035 -> 425 tokens (saved 610)
- GoResolve root instructions: 726 -> 350 tokens (saved 376)
- GoResolve backend instructions: 909 -> 262 tokens (saved 647)
- GoResolve frontend instructions: 965 -> 282 tokens (saved 683)
- GoResolve/DataScraper workflows now enforce incremental context loading
- Net static reduction from this phase (GoResolve + DataScraper + policy files): **~1452 tokens/request baseline**
---

## Instruction Compression Completed

### SignalRaptor Instructions (Completed)
- ✅ `signalraptor-server/.github/copilot-instructions.md`: 2882 → 595 tokens (79% reduction)
- ✅ `signalraptor-web/.github/copilot-instructions.md`: 1035 → 425 tokens (59% reduction)

### GoResolve Instructions (Completed)
- ✅ `GoResolve/.github/copilot-instructions.md`: 726 -> 350 tokens (52% reduction)
- ✅ `GoResolve-backend/.github/copilot-instructions.md`: 909 -> 262 tokens (71% reduction)
- ✅ `GoResolve-frontend/.github/copilot-instructions.md`: 965 -> 282 tokens (71% reduction)

### DataScraper Instructions (Updated for context policy)
- ✅ `DataScraper/CLAUDE.md`: policy added to prevent broad context loads
- ✅ `DataScraper/.github/copilot-instructions.md`: policy added to constrain default reads

**Strategy:** Keep instructions short and normative; move historical detail to context files and load it incrementally only when needed.
---

## Audit Skill Split (Completed)

**Files:**
- `SKILL.md` → Smart dispatcher (chooses quick vs full)
- `audit-quick.md` → 30 min–2 hours for targeted reviews  
- `audit-full.md` → 4–8 hours for comprehensive audits

**Savings:** Reduces average audit overhead from ~6300 tokens to ~3500–4000 tokens, depending on scope.

---

## Implementation Checklist (All Done)

- [x] Create `audit-quick.md` for targeted reviews
- [x] Rename `audit/SKILL.md` to `audit-full.md`
- [x] Create smart dispatcher `audit/SKILL.md`
- [x] Compress `signalraptor-server/.github/copilot-instructions.md` (79%)
- [x] Compress `signalraptor-web/.github/copilot-instructions.md` (59%)
- [x] Compress `GoResolve/.github/copilot-instructions.md` (52%)
- [x] Compress `GoResolve-backend/.github/copilot-instructions.md` (71%)
- [x] Compress `GoResolve-frontend/.github/copilot-instructions.md` (71%)
- [x] Add minimal context policy in `GoResolve` skills (`dev` and `audit`)
- [x] Add minimal context policy in `DataScraper` (`CLAUDE.md` + workspace instructions)
- [x] Create `raptor.code-workspace` (excludes SignalRaptor)
- [x] Update `signalraptor.code-workspace` (excludes Raptor, removes root)
- [x] Document strategy (this file)

---

## Future Guidance

### Starting a new session?
1. **Identify project:** Raptor or SignalRaptor?
2. **Open correct workspace:** `raptor.code-workspace` or `signalraptor.code-workspace`
3. **Choose audit scope:** Quick for targeted, Full for comprehensive
4. **Read copilot instructions:** Only for projects being modified

### Adding a new project?
1. Create a project-specific `CLAUDE.md` following patterns in `raptor/CLAUDE.md`
2. Create `.github/copilot-instructions.md` — keep under 1000 tokens by using tables + links
3. Add new workspace file in repo root: `[projectname].code-workspace`
4. Document in this file (this strategy guide)

### Reducing tokens further?
1. **Archive old context:** Move completed feature docs to `archived/` folder
2. **Use session memory:** For task-specific notes (don't reload every session)
3. **Link to external:** For lengthy how-to guides, link to repo README/wiki instead of embedding

---

## Measured Overhead (Before & After)

### Current measured instruction footprint
- E:/raptor (all CLAUDE/copilot instruction files): ~4407 tokens
- E:/DataScraper (all CLAUDE/copilot instruction files): ~1046 tokens
- E:/GoResolve (all CLAUDE/copilot instruction files): ~4836 tokens
- Portfolio static total (3 roots together): **~10289 tokens**

### Skills and context pressure (high impact for real sessions)
- Raptor skills:
  - dev: ~666
  - audit dispatcher: ~282
  - audit quick: ~524
  - audit full: ~1863
- GoResolve skills:
  - dev: ~464
  - audit: ~518
- Context volumes:
  - E:/DataScraper/.claude/contexts: ~11338 tokens (20 files)
  - E:/GoResolve/GoResolve-backend/.claude/context: ~20429 tokens (36 files)
  - E:/GoResolve/GoResolve-frontend/.claude/context + contexts: ~26727 tokens (51 files)
  - E:/raptor context trees (root + subprojects): ~8123 tokens

### Operational scenarios (with context policy)
- Mega-workspace with all 3 roots open: ~10289 static tokens before task-specific skill/context
- Raptor isolated dev session: ~5073 tokens (~4407 + dev skill)
- GoResolve isolated dev session (instructions + dev skill only): ~5300 tokens
- DataScraper isolated routine session (instructions only): ~1046 tokens
- Highest residual risk: audit flows that still require broad historical reconstruction
---

## Notes for Teams

**Why this matters (portfolio-wide):**
- Opening all three roots together still starts above ~10k tokens before code context.
- GoResolve and DataScraper were the biggest avoidable spend due to broad context reads.
- The new policy shifts default behavior from bulk history loading to incremental reads.
- Savings now scale with session volume, not only with instruction compression.

## Dollar Benchmark (Monthly)

**Assumptions:**
- Token pricing for context/input: **$3.00 per 1M tokens** (Claude-style input reference)
- Portfolio average savings per session (from table above): **~12461 tokens**

| Sessions / month | Tokens saved / month | Estimated USD / month |
|------------------|----------------------|-----------------------|
| 100 | ~1,246,100 | **~$3.74** |
| 300 | ~3,738,300 | **~$11.21** |
| 1000 | ~12,461,000 | **~$37.38** |

For higher input-price tiers (e.g., $15 per 1M), multiply values above by 5.

**Cultural guideline:**
- Always use project-specific workspace unless you're genuinely auditing cross-project
- If context feels bloated mid-session, close and reopen with correct workspace
- Don't treat `.code-workspace` files as aspirational — use them as operational structure

---

**Last reviewed:** 2026-04-15  
**Next review:** After adding new projects or when instruction files grow beyond 1000 tokens
