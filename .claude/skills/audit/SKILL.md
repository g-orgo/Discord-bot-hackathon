---
name: audit
description: "Use for: code quality reviews, consistency audits, security checks. Choose workflow by scope: quick (single project/module) or full (cross-stack)."
---

# Audit — Choose Your Workflow

**Pick the right audit for your task:**

## Quick Audit (30 min - 2 hours)
**When:** Single project, module-level review, or targeted fix  
**File:** [audit-quick.md](audit-quick.md)  
**Includes:** Mandated checklist only, minimal context reads, direct fixes

👉 Use this for:
- Reviewing `raptor-chatbot` or just `/commands`
- Quick security/ESM check on one file
- Fixed scope: "does X follow conventions?"

## Full Audit (4-8 hours)
**When:** Cross-stack consistency, comprehensive review, or reporting needed  
**File:** [audit-full.md](audit-full.md)  
**Includes:** All steps (context reads, code review, 3 summary docs, git commit)

👉 Use this for:
- Entire Raptor mono-repo health check
- Refactoring multiple projects at once
- Stakeholder + developer documentation needed

---

**Quick decision:** If you can describe the audit in one sentence and scope is clear → **Quick**. Otherwise → **Full**.
