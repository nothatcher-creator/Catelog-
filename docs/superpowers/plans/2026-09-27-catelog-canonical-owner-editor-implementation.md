# Catelog Canonical Owner Editor Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Give the GitHub-authenticated Catelog owner safe in-app controls to create and edit Projects, Game Hubs, Mods, categories, and canonical cover metadata without accepting arbitrary GitHub repository/path writes.

**Architecture:** Reuse the GitHub OAuth credential from the hybrid-files plan, but place all canonical writes behind a typed server-only writer whose target is an enum mapped to fixed files in `nothatcher-creator/Catelog-` on `main`. Each mutation reads the latest file + SHA, validates the entire resulting document with the same schemas the public loader uses, then writes with optimistic concurrency. Owner-facing editors are reusable dialogs/sheets opened from Dashboard quick actions and Settings.

**Tech Stack:** React 19, TanStack Start server routes, TypeScript, Zod, GitHub REST Contents API, existing Catelog project/mod schemas, Bun tests.

**Spec:** `docs/superpowers/specs/2026-09-27-catelog-mods-files-architecture-design.md`

**Depends on:**
- `docs/superpowers/plans/2026-09-27-catelog-mod-hubs-navigation-implementation.md`
- `docs/superpowers/plans/2026-09-27-catelog-hybrid-files-github-auth-implementation.md`

**Execution order:** Run this plan after the two dependencies and before `2026-09-27-catelog-workspace-screens-implementation.md`, so Dashboard/Settings quick actions have real owner editor destinations.

## Global Constraints

- Every write requires `requireOwner()` from GitHub OAuth authentication.
- Repository is always `nothatcher-creator/Catelog-`, branch always `main`; browser payloads cannot choose either.
- Allowed canonical targets are exactly `data/projects.json`, `data/project-details.json`, `data/categories.json`, `data/game-hubs.json`, and `data/mods.json`.
- Every write uses latest blob SHA and returns HTTP 409 on concurrent change rather than overwriting.
- The complete resulting JSON document must pass its domain schema before the GitHub write occurs.
- Create/Edit forms must not invent unknown project/mod facts; empty facts stay explicit empty/not-documented values.
- Deletion is a dangerous action with exact-name confirmation. Prefer archive/status change when the entity schema supports it; hard deletion is only the explicit destructive action.
- Uncertain mod game association defaults to the existing `unsorted` game hub.
- Changing display name must not silently change a stable slug/URL. Slug changes are a separate explicit action with broken-link warning.
- Server caches for changed canonical files are invalidated after successful writes so the owner sees the update without waiting for the normal fetch TTL.

## Review Focus

1. A forged client target/path must never write outside the five fixed canonical JSON files.
2. Concurrent edits must return a conflict with refresh/retry UI instead of last-write-wins.
3. Renaming a title must not change the stable slug or route unless the owner explicitly performs a slug migration.
4. Deleting a game hub with child mods must be blocked until children are moved/removed; moving them to Unsorted is the safe default.
5. “Use as cover” must only accept a valid public CatalogFile owned by the same entity and must not produce a broken private/internal URL.

---

### Task 1: Add typed canonical GitHub writer

**Files:**
- Create: `app/src/lib/catalog/canonical-targets.ts`
- Create: `app/src/lib/catalog/canonical-writer.server.ts`
- Test: `app/src/lib/catalog/canonical-writer.test.ts`
- Modify: `app/src/lib/catalog/catalog-fetch.server.ts`
- Modify: `app/src/lib/mods/mod-fetch.server.ts`

**Interfaces:**
- Produces: `CanonicalTarget = 'projects' | 'projectDetails' | 'categories' | 'gameHubs' | 'mods'`, `readCanonicalTarget(target, credential)`, `writeCanonicalTarget(target, nextDocument, expectedSha, credential)`, `invalidateCanonicalTarget(target)`.

- [ ] **Step 1: Write failing writer tests**

Assert fixed target→path mapping, fixed repo/branch, full-document validation, wrong expected SHA conflict, GitHub 409/422 conflict mapping, no arbitrary URL/path/repo fields, and cache invalidation only after successful write.

- [ ] **Step 2: Verify RED**

Run: `cd app && bun test src/lib/catalog/canonical-writer.test.ts`

- [ ] **Step 3: Implement typed reader/writer and cache invalidation hooks**

Reuse stored OAuth credential server-side. Keep the existing public raw fetch path for ordinary reads.

- [ ] **Step 4: Verify GREEN with existing catalog/mod tests**

- [ ] **Step 5: Commit**

Commit message: `feat: add safe canonical Catelog writer`

---

### Task 2: Add Project, GameHub, Mod, and Category mutation services

**Files:**
- Create: `app/src/lib/catalog/catalog-editor.server.ts`
- Test: `app/src/lib/catalog/catalog-editor.test.ts`
- Create: `app/src/routes/api/catalog/projects.ts`
- Create: `app/src/routes/api/catalog/projects/$slug.ts`
- Create: `app/src/routes/api/catalog/game-hubs.ts`
- Create: `app/src/routes/api/catalog/game-hubs/$slug.ts`
- Create: `app/src/routes/api/catalog/mods.ts`
- Create: `app/src/routes/api/catalog/mods/$gameSlug/$modSlug.ts`
- Create: `app/src/routes/api/catalog/categories.ts`

**Interfaces:**
- Produces owner-only `createProject`, `updateProject`, `archiveProject`, `deleteProject`, `createGameHub`, `updateGameHub`, `deleteGameHub`, `createMod`, `updateMod`, `moveMod`, `deleteMod`, `updateCategories`.

- [ ] **Step 1: Write failing service tests**

Cover owner authorization, duplicate slug, same mod slug in different game allowed, unknown game→Unsorted default, move between hubs, `modOrder` maintenance, blocked parent deletion with children, project/details merge, and exact-name hard-delete confirmation.

- [ ] **Step 2: Verify RED**

- [ ] **Step 3: Implement services using Task 1 writer**

Project creation writes a valid base record to `projects.json`; optional dossier fields go to `project-details.json`. Mod/game creation writes only documented input; never generate implementation facts from names.

- [ ] **Step 4: Verify GREEN and 409 conflict propagation**

- [ ] **Step 5: Commit**

Commit message: `feat: add canonical project and mod editing services`

---

### Task 3: Support stable public file covers in canonical entity schemas

**Files:**
- Modify: `app/src/lib/catalog/catalog-types.ts`
- Modify: `app/src/lib/catalog/catalog-schema.ts`
- Modify: `app/src/lib/catalog/project-normalize.ts`
- Modify: `app/src/lib/catalog/dashboard-model.ts`
- Modify: `app/src/lib/mods/mod-types.ts`
- Modify: `app/src/lib/mods/mod-schema.ts`
- Test: existing/new schema and cover-resolution tests.

**Interfaces:**
- Add optional `coverUrl` to project dossier data and ModRecord where necessary; GameHub already has `cover`/`banner` URL-capable fields.
- Produce `getEntityCover(entity)` that prefers a validated HTTPS/Catelog stable file URL then falls back to existing `coverKey` artwork.

- [ ] **Step 1: Write failing backward-compatibility and cover tests**

Legacy records with only `coverKey` still render unchanged. A valid `/files/<id>/<name>` or HTTPS cover URL wins. Dangerous schemes are rejected.

- [ ] **Step 2: Verify RED**

- [ ] **Step 3: Implement optional cover URL support and resolver**

- [ ] **Step 4: Verify GREEN across existing project/mod UI tests**

- [ ] **Step 5: Commit**

Commit message: `feat: support uploaded entity covers`

---

### Task 4: Build reusable owner catalog editor UI and Dashboard quick actions

**Files:**
- Create: `app/src/components/catalog-editor/catalog-editor-dialog.tsx`
- Create: `app/src/components/catalog-editor/project-form.tsx`
- Create: `app/src/components/catalog-editor/game-hub-form.tsx`
- Create: `app/src/components/catalog-editor/mod-form.tsx`
- Create: `app/src/components/catalog-editor/category-editor.tsx`
- Create: `app/src/lib/catalog/catalog-editor-model.ts`
- Test: `app/src/lib/catalog/catalog-editor-model.test.ts`
- Modify: Dashboard component from workspace-screens plan when present; until then expose editor triggers through existing Dashboard quick-action area.
- Modify: `app/src/styles.css`

**Interfaces:**
- Produces quick actions New Project, New Game Hub, New Mod and owner edit actions on entity pages.

- [ ] **Step 1: Write failing form-model tests**

Cover stable slug generation on create only, slug immutability on ordinary rename, required fields, Unsorted default, game chooser, no duplicated mod composite identity, conflict state, and destructive confirmation text.

- [ ] **Step 2: Verify RED**

- [ ] **Step 3: Implement owner-only forms**

Public users do not render edit triggers. Forms show save progress, conflict/refresh, validation errors, and successful navigation to the new entity.

- [ ] **Step 4: Add quick actions and per-entity Edit controls**

New Mod allows choosing verified game or Unsorted. New Project and New Game Hub open their dedicated forms.

- [ ] **Step 5: Verify GREEN at desktop and 390px mobile width**

- [ ] **Step 6: Commit**

Commit message: `feat: add owner catalog editor UI`

---

### Task 5: Add asset “Use as Cover” and canonical metadata actions

**Files:**
- Create: `app/src/lib/catalog/entity-asset-service.server.ts`
- Test: `app/src/lib/catalog/entity-asset-service.test.ts`
- Create: `app/src/routes/api/catalog/entity-cover.ts`
- Modify: `app/src/components/assets/asset-workspace.tsx`
- Modify: `app/src/components/files/file-detail.tsx`

**Interfaces:**
- Produces `setEntityCover(entityRef, fileId, owner)`, plus role/tag metadata updates that stay in file metadata when they do not belong in canonical descriptive JSON.

- [ ] **Step 1: Write failing cover-action tests**

Require owner; file must exist, be public, be image-compatible, and belong to the same entity; stable Catelog public URL is written to the correct fixed canonical target; conflict propagates.

- [ ] **Step 2: Verify RED**

- [ ] **Step 3: Implement service/API and Assets/File actions**

- [ ] **Step 4: Verify GREEN**

- [ ] **Step 5: Commit**

Commit message: `feat: manage entity covers from Catelog assets`

---

### Task 6: Integrate catalog editor into Settings and public discovery

**Files:**
- Modify: `app/src/components/settings/settings-workspace.tsx` when Plan 3 is present, or create a reusable `CatalogManagementPanel` consumed by it later.
- Create: `app/src/components/catalog-editor/catalog-management-panel.tsx`
- Modify: `app/src/routes/sitemap[.]xml.ts`
- Test: sitemap/catalog management tests.

**Interfaces:**
- Settings Catalog section lists Projects, Game Hubs, Mods, categories, source sync state, and owner edit/create actions.

- [ ] **Step 1: Write failing sitemap tests**

Assert public sitemap includes existing project routes plus `/mods`, every valid game-hub route, and every valid mod route; invalid/Unsorted records still use valid public routes and no duplicates are emitted.

- [ ] **Step 2: Verify RED**

- [ ] **Step 3: Implement catalog management panel and sitemap expansion**

Do not expose secret values or GitHub OAuth token state beyond configured/connected status.

- [ ] **Step 4: Verify GREEN**

- [ ] **Step 5: Commit**

Commit message: `feat: integrate Catelog catalog management`

---

### Task 7: Verify canonical editor and hand off to workspace-screens plan

- [ ] **Step 1: Run catalog/mod/auth/editor tests**

Run: `cd app && bun test src/lib/catalog src/lib/mods src/lib/auth src/lib/files src/components/catalog-editor src/lib/owner`

Expected: zero failures.

- [ ] **Step 2: Run typecheck, targeted lint, and production build**

Expected: zero type errors, zero changed-file lint errors, build exit 0.

- [ ] **Step 3: Browser verify owner/public states**

Public: no create/edit/delete controls. Owner: create test Game Hub/Mod forms render, conflict state works with mocked/integration service, mobile forms fit 390px without overflow.

- [ ] **Step 4: Verify destructive gates**

GameHub with children cannot be hard-deleted. Ordinary rename preserves slug. Hard-delete requires exact entity name/slug confirmation.

- [ ] **Step 5: Verify sitemap and AI/project regressions**

- [ ] **Step 6: Commit verification fixes using TDD, push, and deploy if the execution phase is shipping this plan separately**

Then continue with `2026-09-27-catelog-workspace-screens-implementation.md` so final Dashboard/Settings/sidebar wiring can consume these editor actions.
