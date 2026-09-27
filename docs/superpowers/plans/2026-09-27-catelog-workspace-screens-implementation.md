# Catelog Workspace Screens and Global Activity Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make every Catelog sidebar item open a complete workspace screen and connect Dashboard, Projects, Roadmap, Docs, Issues, Changelog, AI Handoff, Settings, global search, and sidebar badges to the project/mod/file systems.

**Architecture:** Add a small operational workspace layer in D1 for roadmap items, issues, activity events, and settings, then build route-specific aggregate loaders that combine existing GitHub project/mod data with D1 operational records and file/release metadata. Update the shared workspace shell only after each target route exists so every sidebar entry becomes a real route atomically. Global search uses one normalized result model rather than forcing every screen to implement its own search semantics.

**Tech Stack:** React 19, TanStack Start/Router, TypeScript, Cloudflare D1, existing GitHub-backed project/mod loaders, file/release services from Plan 2, Bun tests, targeted Chromium browser verification.

**Spec:** `docs/superpowers/specs/2026-09-27-catelog-mods-files-architecture-design.md`

**Depends on:**
- `docs/superpowers/plans/2026-09-27-catelog-mod-hubs-navigation-implementation.md`
- `docs/superpowers/plans/2026-09-27-catelog-hybrid-files-github-auth-implementation.md`

## Global Constraints

- Every final sidebar entry is a real route: Dashboard, Projects, Mods, Roadmap, Assets, Files / Uploads, Docs, Issues, Changelog, AI Handoff, Settings.
- Public browsing stays available without login; only owner mutations require the GitHub owner session from Plan 2.
- Projects excludes individual mods; Mods remains grouped by GameHub.
- Aggregates must be paginated or scoped; the dashboard must not load every file/issue/activity row.
- Global search indexes projects, game hubs, mods, features, tasks, issues, files, tags/roles, docs, releases, changelog entries, and AI guidance.
- Missing operational data renders empty/unknown states rather than causing route failures.
- Issue and roadmap records may point to Project, GameHub, or Mod; invalid/stale associations remain visible as unavailable references rather than disappearing.
- Existing project/mod AI routes stay canonical and are reused by AI Handoff rather than duplicated.
- Activity/changelog entries are append-only in normal operation; destructive history editing is not part of this upgrade.
- Settings never displays server secret values.
- Mobile drawer, filters, and action controls work at 390px width and do not rely on hover.

## Review Focus

1. Stale entity references in roadmap/issues/activity must remain understandable and must not crash aggregate screens.
2. Combining multiple filters must use AND semantics and deterministic pagination rather than returning inconsistent subsets.
3. Public visitors must never see mutation controls merely because a route loader knows owner configuration exists.
4. Large activity/file/issue histories must paginate; no screen may fetch the entire operational database by default.
5. Search results must navigate to the exact project/game/mod/file/issue location and must not confuse identical names across entity types.

---

### Task 1: Add operational workspace tables and repositories

**Files:**
- Create: `app/migrations/0003_workspace_activity.sql`
- Create: `app/src/lib/workspace/workspace-types.ts`
- Create: `app/src/lib/workspace/roadmap-repository.server.ts`
- Create: `app/src/lib/workspace/issue-repository.server.ts`
- Create: `app/src/lib/workspace/activity-repository.server.ts`
- Create: `app/src/lib/workspace/settings-repository.server.ts`
- Test: `app/src/lib/workspace/workspace-repositories.test.ts`

**Interfaces:**
- Produces: `RoadmapItem`, `WorkspaceIssue`, `ActivityEvent`, `WorkspaceSetting` and paginated CRUD/query repository functions.

**Migration tables:**

`roadmap_items`: `id`, `entity_type`, `project_slug`, `game_hub_slug`, `mod_slug`, `title`, `description`, `status`, `priority`, `target_version`, `target_date`, `depends_on_json`, `related_issue_ids_json`, `created_at`, `updated_at`.

`issues`: `id`, `entity_type`, `project_slug`, `game_hub_slug`, `mod_slug`, `title`, `description`, `status`, `severity`, `reproduction_json`, `affected_platforms_json`, `affected_versions_json`, `workaround`, `related_file_ids_json`, `screenshot_file_ids_json`, `external_github_issue_url`, `created_at`, `updated_at`, `resolved_at`.

`activity_events`: `id`, `type`, `entity_type`, `project_slug`, `game_hub_slug`, `mod_slug`, `file_id`, `release_id`, `issue_id`, `title`, `description`, `created_at`, `actor_github_user_id`, `metadata_json` plus indexes on time/entity/type.

`workspace_settings`: `key TEXT PRIMARY KEY`, `value_json TEXT NOT NULL`, `updated_at TEXT NOT NULL`, `updated_by_github_user_id TEXT NOT NULL`.

- [ ] **Step 1: Write failing repository tests**

Cover CRUD, filters, stale entity fields, pagination cursors, chronological activity order, issue resolve/reopen, roadmap status changes, and settings upsert.

- [ ] **Step 2: Verify RED**

Run: `cd app && bun test src/lib/workspace/workspace-repositories.test.ts`

- [ ] **Step 3: Add additive migration and repositories**

All writes accept an already-verified owner identity; authorization remains in service/API layers.

- [ ] **Step 4: Verify GREEN and typecheck**

- [ ] **Step 5: Commit**

Commit message: `feat: add Catelog workspace operational records`

---

### Task 2: Add a normalized entity resolver and global aggregate/search model

**Files:**
- Create: `app/src/lib/workspace/entity-resolver.ts`
- Create: `app/src/lib/workspace/workspace-aggregate.server.ts`
- Create: `app/src/lib/workspace/global-search.ts`
- Test: `app/src/lib/workspace/entity-resolver.test.ts`
- Test: `app/src/lib/workspace/global-search.test.ts`

**Interfaces:**
- Produces: `WorkspaceEntityRef`, `resolveEntity(ref, workspaceCatalog)`, `getWorkspaceSummary()`, `SearchResult`, `searchWorkspace(query, sources)`.
- SearchResult: `{ id, entityType, title, subtitle, href, matchedField, updatedAt? }`.

- [ ] **Step 1: Write failing resolver tests**

Cover Project, GameHub, Mod composite identity, missing/stale references, identical display names across types, and Unsorted mods.

- [ ] **Step 2: Write failing search tests**

Index project/mod aliases, features, tasks, issues, file display names/tags/roles, docs, releases, changelog text, and AI guidance. Exact entity name/alias outranks body text; result href points to exact route.

- [ ] **Step 3: Verify RED**

- [ ] **Step 4: Implement normalized resolver, summary queries, and search model**

`getWorkspaceSummary()` returns bounded counts/recent slices only: project/mod counts, open issue count, active roadmap count, latest 10 activities, latest 8 uploads, latest 6 releases.

- [ ] **Step 5: Verify GREEN**

- [ ] **Step 6: Commit**

Commit message: `feat: add Catelog workspace aggregates and search`

---

### Task 3: Build the Projects workspace and upgrade Dashboard aggregation

**Files:**
- Create: `app/src/components/projects/projects-workspace.tsx`
- Create: `app/src/lib/workspace/project-browser-model.ts`
- Test: `app/src/lib/workspace/project-browser-model.test.ts`
- Create: `app/src/routes/projects/index.tsx`
- Create: `app/src/components/dashboard/workspace-dashboard.tsx`
- Modify: `app/src/routes/index.tsx`
- Modify: `app/src/styles.css`

**Interfaces:**
- Produces: project-only grid/list browser and bounded dashboard summary.

- [ ] **Step 1: Write failing browser-model tests**

Cover search, category/status/platform/favorite/pinned filters, combined filter AND semantics, sort by recent/name/status, and guarantee that ModRecords are not included.

- [ ] **Step 2: Verify RED**

- [ ] **Step 3: Implement `/projects`**

Grid/list switch; filters; progress/open-work/recent-update/repository/runtime summary; quick links to project page, AI context, repository, and runner when available.

- [ ] **Step 4: Upgrade Dashboard**

Show pinned projects, recent mod work, open tasks/issues, active milestones, recent uploads, latest releases, known issues, recent activity, and quick actions. Every card links to exact source entity.

- [ ] **Step 5: Verify unit/browser behavior**

- [ ] **Step 6: Commit**

Commit message: `feat: add Projects workspace and global dashboard`

---

### Task 4: Build Roadmap workspace and owner mutations

**Files:**
- Create: `app/src/lib/workspace/roadmap-service.server.ts`
- Test: `app/src/lib/workspace/roadmap-service.test.ts`
- Create: `app/src/components/roadmap/roadmap-workspace.tsx`
- Create: `app/src/components/roadmap/roadmap-list.tsx`
- Create: `app/src/components/roadmap/roadmap-kanban.tsx`
- Create: `app/src/components/roadmap/roadmap-timeline.tsx`
- Create: `app/src/routes/roadmap/index.tsx`
- Create: `app/src/routes/api/roadmap/index.ts`
- Create: `app/src/routes/api/roadmap/$itemId.ts`

**Interfaces:**
- Produces: public filtered reads and owner-only create/update/delete/status changes using `requireOwner` from Plan 2.

- [ ] **Step 1: Write failing service tests**

Cover allowed statuses Planned/In Progress/Blocked/Testing/Complete, priority, entity association, stale association rendering, owner auth, and combined filters.

- [ ] **Step 2: Verify RED**

- [ ] **Step 3: Implement list, Kanban, and timeline views**

Filter by Project/GameHub/Mod, status, priority. Clicking an item resolves back to its entity. Timeline orders dated items; undated items remain in an explicit “No target date” group.

- [ ] **Step 4: Implement owner editor actions**

Public users see read-only views; owner gets New/Edit/Status/Delete controls.

- [ ] **Step 5: Verify GREEN and mobile drag/tap safety**

Kanban movement may use explicit status controls instead of drag-only behavior so phone/keyboard users are not excluded.

- [ ] **Step 6: Commit**

Commit message: `feat: add global Catelog roadmap`

---

### Task 5: Build Issues workspace and entity issue panels

**Files:**
- Create: `app/src/lib/workspace/issue-service.server.ts`
- Test: `app/src/lib/workspace/issue-service.test.ts`
- Create: `app/src/components/issues/issues-workspace.tsx`
- Create: `app/src/components/issues/issue-detail.tsx`
- Create: `app/src/routes/issues/index.tsx`
- Create: `app/src/routes/api/issues/index.ts`
- Create: `app/src/routes/api/issues/$issueId.ts`
- Modify: `app/src/components/mods/mod-detail-section.tsx`
- Modify: `app/src/components/catalog/project-detail-sections.tsx`

**Interfaces:**
- Produces public issue reads and owner-only create/update/resolve/reopen/delete.

- [ ] **Step 1: Write failing issue-service tests**

Cover Open/In Progress/Blocked/Fixed, severity filters, entity association, reproduction/platform/version arrays, file/screenshot references, optional GitHub issue URL validation, resolve timestamp, reopen behavior, and owner auth.

- [ ] **Step 2: Verify RED**

- [ ] **Step 3: Implement Issues workspace**

Filters: status, severity, project/game/mod; issue detail drawer shows all documented fields and unavailable markers for stale related files.

- [ ] **Step 4: Merge operational issues into Project and Mod Bugs views**

Canonical known bugs remain visible; D1 issues appear alongside them with source badges and stable `/issues?...` links.

- [ ] **Step 5: Verify GREEN**

- [ ] **Step 6: Commit**

Commit message: `feat: add Catelog issue tracking workspace`

---

### Task 6: Build Docs workspace

**Files:**
- Create: `app/src/lib/workspace/docs-index.server.ts`
- Test: `app/src/lib/workspace/docs-index.test.ts`
- Create: `app/src/components/docs/docs-workspace.tsx`
- Create: `app/src/routes/docs/index.tsx`

**Interfaces:**
- Produces `listDocs(filters)` from project/mod README/AI/Markdown routes, canonical important files, and CatalogFiles with role Documentation/source extensions.

- [ ] **Step 1: Write failing docs-index tests**

Cover project README, game-hub README, mod README, GitHub-backed Markdown file, R2 documentation attachment, duplicate canonical/raw reference de-duplication, filters, and search text.

- [ ] **Step 2: Verify RED**

- [ ] **Step 3: Implement Docs workspace**

Show document type, owner entity, source backend, updated date, preview/open/download actions, and exact entity link. Do not fetch entire document bodies until opened.

- [ ] **Step 4: Verify GREEN and lazy-loading behavior**

- [ ] **Step 5: Commit**

Commit message: `feat: add Catelog documentation workspace`

---

### Task 7: Add activity recording and build Changelog workspace

**Files:**
- Create: `app/src/lib/workspace/activity-service.server.ts`
- Test: `app/src/lib/workspace/activity-service.test.ts`
- Create: `app/src/components/changelog/changelog-workspace.tsx`
- Create: `app/src/routes/changelog/index.tsx`
- Modify owner mutation services from Plan 2 and Tasks 4–5 to call `recordActivity()` after successful commits only.
- Modify mod/project changelog sections to merge canonical history with operational events.

**Interfaces:**
- Produces: `recordActivity(event)`, `listActivity(filters, cursor)`, `mergeCanonicalHistory(entity, activity)`.

- [ ] **Step 1: Write failing event tests**

Assert successful upload, replace, new version, release publish, issue create/resolve, roadmap change, cover/role change, and documentation update create one event; failed operations create none; pagination is newest-first.

- [ ] **Step 2: Verify RED**

- [ ] **Step 3: Implement activity service and wire mutation hooks**

Activity is append-only. Metadata JSON may hold before/after labels but never OAuth tokens/secrets.

- [ ] **Step 4: Build global Changelog**

Filter by project, game, mod, event type, date. Entity-specific changelog tabs display canonical history + filtered activity in one timeline with source markers.

- [ ] **Step 5: Verify GREEN**

- [ ] **Step 6: Commit**

Commit message: `feat: add Catelog activity and changelog`

---

### Task 8: Build the AI Handoff workspace and expand AI context with operational data

**Files:**
- Create: `app/src/lib/workspace/ai-handoff.server.ts`
- Test: `app/src/lib/workspace/ai-handoff.test.ts`
- Create: `app/src/components/ai-handoff/ai-handoff-workspace.tsx`
- Create: `app/src/routes/ai-handoff/index.tsx`
- Modify: `app/src/lib/catalog/catalog-format.ts`
- Modify: `app/src/lib/mods/mod-format.ts`

**Interfaces:**
- Produces: `buildWorkspaceHandoff(entityRef)` returning public canonical context URL, Markdown URL, AI URL, latest relevant files/releases/issues, and copy prompt.

- [ ] **Step 1: Write failing handoff tests**

Cover Project, GameHub, Mod, unrelated-data exclusion, latest public files/releases, operational issues/roadmap next work, broken file references marked unavailable, and no invented facts.

- [ ] **Step 2: Verify RED**

- [ ] **Step 3: Implement selector + context preview**

User chooses Project/GameHub/Mod and sees exactly what the canonical handoff contains. Actions: Copy Prompt, Copy Context URL, JSON, Markdown, full AI brief.

- [ ] **Step 4: Extend canonical serializers**

Add relevant public file URLs/releases/issues while preserving existing static project/mod facts and omitting secrets/internal session data.

- [ ] **Step 5: Verify GREEN**

- [ ] **Step 6: Commit**

Commit message: `feat: add global AI handoff workspace`

---

### Task 9: Build owner Settings workspace

**Files:**
- Create: `app/src/lib/workspace/settings-service.server.ts`
- Test: `app/src/lib/workspace/settings-service.test.ts`
- Create: `app/src/components/settings/settings-workspace.tsx`
- Create: `app/src/routes/settings/index.tsx`
- Create: `app/src/routes/api/settings/index.ts`

**Interfaces:**
- Produces owner-only settings reads/writes plus public-safe configuration status.

- [ ] **Step 1: Write failing settings tests**

Cover unauthenticated read-only summary, owner-only writes, GitHub auth configured/unconfigured flags without secret values, upload max size, default backend rules, allowed extension groups, game hub/mod organization links, runtime/AI defaults, and invalid config rejection.

- [ ] **Step 2: Verify RED**

- [ ] **Step 3: Implement Settings UI**

Sections: GitHub connection/owner identity, Catalog, Upload & Storage, Categories/Game Hubs/Mods, Runtime, AI Handoff defaults, Dangerous Actions. Never render secret values; only configured/missing status and callback URL/setup guidance.

- [ ] **Step 4: Wire safe settings into upload/search/default view behavior**

Do not make secret availability a client-trusted authorization decision; server still calls `requireOwner` for every mutation.

- [ ] **Step 5: Verify GREEN**

- [ ] **Step 6: Commit**

Commit message: `feat: add Catelog owner settings workspace`

---

### Task 10: Convert every sidebar item to real route navigation with badges

**Files:**
- Modify: `app/src/components/workspace/workspace-nav.ts`
- Modify: `app/src/components/workspace/workspace-sidebar.tsx`
- Modify: `app/src/components/workspace/workspace-shell.tsx`
- Modify: `app/src/components/workspace/workspace-topbar.tsx`
- Test: `app/src/components/workspace/workspace-nav.test.ts`
- Modify: `app/src/styles.css`

**Interfaces:**
- `WORKSPACE_NAV` final destinations:
  - Dashboard `/`
  - Projects `/projects`
  - Mods `/mods`
  - Roadmap `/roadmap`
  - Assets `/assets`
  - Files / Uploads `/files`
  - Docs `/docs`
  - Issues `/issues`
  - Changelog `/changelog`
  - AI Handoff `/ai-handoff`
  - Settings `/settings`

- [ ] **Step 1: Update tests first and verify RED**

Assert no final nav entry uses `#...` anchors or external GitHub edit URLs; route-active matching works for nested paths; badges are bounded aggregate counts, not full-query results.

- [ ] **Step 2: Implement final route nav**

Desktop and mobile share the same nav model. Mobile drawer closes after route selection and restores focus appropriately.

- [ ] **Step 3: Add sidebar badges**

Projects total, Mods total, open Issues, active Roadmap, recent Upload count where useful. Hide zero/noisy badges where appropriate.

- [ ] **Step 4: Upgrade global topbar search**

Use Task 2 search results with keyboard/touch result list; selecting a result navigates to exact entity/file/issue. `Escape` closes results; input has accessible combobox semantics.

- [ ] **Step 5: Verify GREEN**

- [ ] **Step 6: Commit**

Commit message: `feat: make every Catelog sidebar route functional`

---

### Task 11: Full workspace verification and release

**Files:**
- Modify only for TDD-backed verification fixes.

- [ ] **Step 1: Run complete Catelog-focused tests**

Run: `cd app && bun test src/components/catalog src/components/workspace src/components/mods src/components/projects src/components/roadmap src/components/issues src/components/docs src/components/changelog src/components/ai-handoff src/components/settings src/components/files src/components/assets src/lib/catalog src/lib/mods src/lib/auth src/lib/files src/lib/releases src/lib/workspace src/lib/owner src/lib/security-headers.test.ts`

Expected: zero failures.

- [ ] **Step 2: Run typecheck, targeted lint, and production build**

Expected: zero type errors, zero lint errors in changed Catelog files, build exit 0.

- [ ] **Step 3: Browser test every sidebar route at desktop and Android widths**

At 1440×900 and 390×844 click every sidebar item and assert the intended route/page heading. Verify mobile drawer closes after navigation and there is no horizontal overflow.

- [ ] **Step 4: Verify global cross-workspace flows**

Search for one project, one mod, one file, one issue; navigate from Roadmap/Issue/Changelog cards back to source entity; open Docs; open AI Handoff; confirm owner controls hidden logged out.

- [ ] **Step 5: Verify owner flows when GitHub OAuth is configured**

Owner can create/edit roadmap item and issue, publish activity-producing actions, edit settings, and see owner quick actions. Public session after logout returns to read-only immediately.

- [ ] **Step 6: Regression-check existing project/mod/file routes and public downloads**

Existing project AI/JSON/Markdown/runner, mod AI/JSON/Markdown, Files/Assets, and public stable file links must still work.

- [ ] **Step 7: Commit any verification fixes, push, and deploy**

Use TDD for product fixes. Deploy only pushed commits. Do not publish to the Higgsfield community feed unless separately requested.
