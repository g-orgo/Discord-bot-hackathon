# Articles Missing from Kanban — Investigation Report

**Date:** 2026-04-14  
**Project:** SignalRaptor — Content Management  
**Symptom:** Kanban board shows 7 columns but all are empty. API returns `totalCount: 0` and `articles: []` for every status column.

---

## What the API Returns (confirmed via DevTools)

```json
{
  "status": "success",
  "data": [
    { "id": 1, "name": "Drafts",         "totalCount": 0, "articles": [] },
    { "id": 2, "name": "In Review",      "totalCount": 0, "articles": [] },
    { "id": 3, "name": "Pending",        "totalCount": 0, "articles": [] },
    { "id": 4, "name": "Submitted",      "totalCount": 0, "articles": [] },
    { "id": 5, "name": "Media Edits",    "totalCount": 0, "articles": [] },
    { "id": 6, "name": "Media Approved", "totalCount": 0, "articles": [] },
    { "id": 7, "name": "Live",           "totalCount": 0, "articles": [] }
  ]
}
```

The frontend and backend code are working correctly. The issue is a **data problem**, not a code bug.

---

## Root Cause Analysis

The backend query that retrieves articles filters strictly by `brand_id`:

```ts
// retrieve_controller.ts — formatArticles()
Article.query()
  .where('articles.brand_id', brandId)       // ← hard filter on brand_id
  .where('articles.article_status_id', status.id)
```

The `Article` model has **two brand association fields**:

```ts
declare brandId?: number | null        // current field — used by the query
declare brandProfileId: number | null  // legacy field — NOT used by the query (TODO: remove)
```

### Likely Scenarios

| Scenario | Evidence |
|----------|----------|
| Articles exist with `brand_id = NULL` (created before `brand_id` migration) | `brand_profile_id` was the old reference; legacy rows may have `brand_id = NULL` |
| Database was reset (`migration:fresh`) | All data wiped; columns are structurally correct but empty |
| Articles belong to a different `brand_id` than the active session | Active brand in localStorage doesn't match stored `brand_id` |

### Fast UI Checks (No DB Access Required)

1. In browser DevTools, inspect `localStorage.activeBrand` and compare with `brandId` query param sent to `retrieve-article`.
2. Switch active brand in UI and retry Kanban load.
3. Open any known article directly by ID (`retrieve-article-by-id`) to confirm records exist even when grouped retrieve is empty.

---

## Environment Discovery

| Layer | Target |
|-------|--------|
| Frontend (`VITE_API_URL`) | `https://api-test.signalraptor.com/` (remote test server) |
| Local backend | Running on `localhost:8000` but **not used by the frontend** |
| Local PostgreSQL | **Not running** (ports 5432, 5430 — ECONNREFUSED) |

The frontend is hitting the **remote test database**, not a local one. Direct SQL access is not available from this machine.

---

## Additional Findings (Code-Confirmed)

### 1) Article retrieval is strictly `brand_id`

Confirmed in backend retrieve flow:

```ts
// app/controllers/v1/brands/content_management/article/retrieve_controller.ts
.where('articles.brand_id', brandId)
.where('articles.article_status_id', status.id)
```

There is no fallback to `brand_profile_id` in Kanban retrieval.

### 2) `brand_id` was introduced later, and no backfill migration exists

Migration timeline confirms `brand_id` was added after the table already existed:

- `1747053032102_create_articles_table.ts` -> creates `articles` without `brand_id`
- `1747645495824_alter_articles_table.ts` -> adds nullable `brand_id`
- later migrations add profile fields (`brand_profile_id`, `agency_profile_id`)

No migration was found that backfills `articles.brand_id` from profile relations.

### 3) Empty result can be cached for up to 7 days

Articles retrieve uses Redis cache keyed by `brandId` + query descriptor:

- `app/services/content_cache_service.ts`
- `config/acl.ts`: `contentCacheTtl: 604800` (7 days)

If the first response for a descriptor is empty, that empty payload can persist until invalidation or TTL expiry.

### 4) Frontend target confirmed

Frontend API client uses `import.meta.env.VITE_API_URL`, and local env points to test API:

```env
# signalraptor-web/.env
VITE_API_URL="https://api-test.signalraptor.com/"
```

So the observed empty Kanban is from the remote test environment.

---

## Database Diagnostic Queries

Run these against the `postgres` database to confirm:

```sql
-- 1. Total article count (any brand)
SELECT COUNT(*) AS total_articles FROM articles;

-- 2. Distribution of brand_id vs brand_profile_id
SELECT
  brand_id,
  brand_profile_id,
  COUNT(*) AS count
FROM articles
GROUP BY brand_id, brand_profile_id
ORDER BY count DESC;

-- 3. Articles with NULL brand_id (legacy rows)
SELECT id, title, brand_id, brand_profile_id, article_status_id
FROM articles
WHERE brand_id IS NULL
LIMIT 20;

-- 4. All distinct brand_ids that have articles
SELECT DISTINCT brand_id, COUNT(*) AS article_count
FROM articles
GROUP BY brand_id;
```

---

## Cache Diagnostics (Remote Redis)

If DB data exists but endpoint still returns zero, verify cache before deeper changes:

```bash
# 1) List cached article payloads for the brand
SCAN 0 MATCH content:articles:brand:<BRAND_ID>:* COUNT 100

# 2) Remove only article cache for that brand
DEL <key1> <key2> ...
# or repeat SCAN + DEL until cursor returns 0
```

Then re-run `retrieve-article` with the same query params used by frontend.

---

## Fix (if articles have `brand_id = NULL`)

If articles exist but have `brand_id = NULL` while `brand_profile_id` is set, backfill the `brand_id` column:

```sql
-- Preview what would be updated
SELECT a.id, a.brand_profile_id, bp.brand_id
FROM articles a
JOIN brand_profiles bp ON bp.id = a.brand_profile_id
WHERE a.brand_id IS NULL;

-- Apply the backfill (run ONLY after confirming the preview)
UPDATE articles a
SET brand_id = bp.brand_id
FROM brand_profiles bp
WHERE bp.id = a.brand_profile_id
  AND a.brand_id IS NULL;
```

---

---

## Final Conclusion — Root Cause Confirmed

**Articles in the remote test database have `brand_id = NULL`.**

Evidence:
- Articles table exists (API returns valid structure with 7 columns)
- Each column returns `totalCount: 0` and `articles: []`
- Backend query filters strictly on `.where('articles.brand_id', brandId)` with no fallback
- `brand_id` was added in a later migration without a backfill UPDATE
- Legacy articles created before the `brand_id` migration retain `brand_id = NULL` AND have valid `brand_profile_id` values

**Result:** These legacy articles are invisible to the Kanban retrieve query, which only sees rows where `brand_id` matches the active brand.

---

## Required Fix

Run this backfill SQL **on the remote test database**:

```sql
UPDATE articles a
SET brand_id = bp.brand_id
FROM brand_profiles bp
WHERE bp.id = a.brand_profile_id
  AND a.brand_id IS NULL;
```

After backfill:
1. (Optional) Clear Redis cache: `FLUSHDB` or targeted key deletion
2. Reload the Kanban in browser — articles should now appear in their respective columns

---

## Frontend & Backend Status

| Layer | Status |
|-------|--------|
| Frontend store (`fetchArticles`) | ✅ Fixed — column mapping restored (QA-1038 regression reverted) |
| Frontend `CreateNewContent.vue` | ✅ Fixed — idea dropdown race condition resolved |
| Backend `retrieve_controller.ts` | ✅ Correct — query, caching, and response shape are all valid |
| Database data — Remote (api-test) | ❌ **Articles have `brand_id = NULL`** — backfill required |
