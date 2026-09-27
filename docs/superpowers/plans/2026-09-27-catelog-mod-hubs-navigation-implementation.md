# Catelog Mod Hubs and Navigation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add first-class game hubs and individual mod pages to Catelog, with stable routes, canonical AI context, verified historical mod migration, and a reusable workspace shell without breaking existing project pages.

**Architecture:** Keep `data/projects.json` and `data/project-details.json` unchanged as the main-project source of truth. Add GitHub-backed `data/game-hubs.json` and `data/mods.json`, parse them through a dedicated mod-domain loader, and render `/mods`, game-hub, and individual-mod routes through a shared workspace shell. No D1/R2 dependency is introduced in this plan; operational files/releases arrive in the next plan.

**Tech Stack:** React 19, TanStack Start/Router, TypeScript, Zod, Bun tests, existing Higgsfield website repository, public GitHub raw JSON data.

**Spec:** `docs/superpowers/specs/2026-09-27-catelog-mods-files-architecture-design.md`

## Global Constraints

- Existing `/`, `/projects/:slug`, project AI, JSON, Markdown, and runner routes must continue working.
- Public browsing requires no authentication.
- Main projects and mods remain separate concepts; mods do not flood the Projects list.
- A `ModRecord` belongs to one verified `GameHub`; uncertain associations use the `unsorted` hub instead of guessing.
- Canonical descriptive game/mod data remains GitHub-backed.
- Missing `game-hubs.json` or `mods.json` must degrade to an empty mod library, not break the project catalog.
- Game hubs and mods require stable public routes and canonical AI handoff routes.
- Mobile navigation must remain usable at 390px width with no horizontal overflow.
- Canonical-data writes to `nothatcher-creator/Catelog-` are an external side effect; execution must use the explicit implementation approval for this plan or stop before that write.

## Review Focus

1. Duplicate mod slugs in different games must resolve by `(gameSlug, modSlug)`, never as a global mod slug.
2. A malformed or missing new canonical file must not make existing project pages fail.
3. A game hub that references a missing mod must surface a data warning and omit the bad child rather than crash.
4. Ambiguous historical mods must land in Unsorted Mods, not in an inferred game hub.
5. A mod AI handoff must never include unrelated mods from another game hub.

---

### Task 1: Define the GameHub and ModRecord domain

**Files:**
- Create: `app/src/lib/mods/mod-types.ts`
- Create: `app/src/lib/mods/mod-schema.ts`
- Create: `app/src/lib/mods/mod-normalize.ts`
- Test: `app/src/lib/mods/mod-normalize.test.ts`

**Interfaces:**
- Produces: `GameHubRecord`, `ModRecord`, `ModCatalogPayload`, `normalizeGameHub(raw)`, `normalizeMod(raw)`, `validateModCatalog(gameHubs, mods)`.
- `ModRecord` identity is `(gameSlug, slug)`.
- `validateModCatalog` returns `{ gameHubs, mods, dataErrors }` and moves a valid mod with an unknown `gameSlug` into the synthetic `unsorted` hub only when its association is not already a verified different hub.

- [ ] **Step 1: Write failing normalization tests**

Cover valid game/mod records, duplicate `(gameSlug, slug)`, same mod slug under two different games, unknown game assignment, malformed nested arrays, and missing optional fields.

- [ ] **Step 2: Run the focused tests and verify RED**

Run: `cd app && bun test src/lib/mods/mod-normalize.test.ts`

Expected: FAIL because the mod-domain modules do not exist.

- [ ] **Step 3: Implement the domain types, Zod schemas, and normalization**

Required fields:

`GameHubRecord`: `slug`, `name`, `aliases`, `summary`, `description`, `cover`, `banner`, `notes`, `sharedDocs`, `sharedAssets`, `modOrder`, `history`, `aiGuidance`.

`ModRecord`: `slug`, `gameSlug`, `name`, `aliases`, `status`, `summary`, `description`, `features`, `requirements`, `supportedGameVersions`, `modLoaderRequirements`, `installationInstructions`, `compatibilityNotes`, `repository`, `sourceReferences`, `tasks`, `bugs`, `releases`, `history`, `aiGuidance`, `coverKey`.

Use safe defaults for optional collections; reject only identity-breaking records.

- [ ] **Step 4: Run tests and verify GREEN**

Run: `cd app && bun test src/lib/mods/mod-normalize.test.ts`

Expected: PASS.

- [ ] **Step 5: Commit**

Commit message: `feat: add Catelog game hub and mod domain`

---

### Task 2: Add the GitHub-backed mod catalog loader

**Files:**
- Create: `app/src/lib/mods/mod-fetch.server.ts`
- Test: `app/src/lib/mods/mod-fetch.server.test.ts`
- Modify: `app/src/lib/catalog/catalog.functions.ts`

**Interfaces:**
- Consumes: Task 1 normalization.
- Produces: `fetchModCatalogFromGitHub(fetcher?: typeof fetch): Promise<ModCatalogPayload>` and `getWorkspaceCatalog()` returning `{ catalog, modCatalog }` for routes that need both domains.

- [ ] **Step 1: Write failing loader tests**

Use injected fake fetch responses for: both JSON files valid, either file 404, malformed JSON, invalid child records, and a hub referencing a missing mod.

- [ ] **Step 2: Verify RED**

Run: `cd app && bun test src/lib/mods/mod-fetch.server.test.ts`

Expected: FAIL because the loader is missing.

- [ ] **Step 3: Implement the loader**

Read:

- `https://raw.githubusercontent.com/nothatcher-creator/Catelog-/main/data/game-hubs.json`
- `https://raw.githubusercontent.com/nothatcher-creator/Catelog-/main/data/mods.json`

A 404 for either new file becomes an empty array plus a non-fatal data warning. Preserve the existing catalog loader untouched except for the small shared `getWorkspaceCatalog()` composition function.

- [ ] **Step 4: Verify GREEN and run existing catalog tests**

Run: `cd app && bun test src/lib/mods/mod-fetch.server.test.ts src/lib/catalog`

Expected: all focused tests pass.

- [ ] **Step 5: Commit**

Commit message: `feat: load canonical mod catalog data`

---

### Task 3: Extract a reusable workspace shell without changing existing project behavior

**Files:**
- Create: `app/src/components/workspace/workspace-shell.tsx`
- Create: `app/src/components/workspace/workspace-sidebar.tsx`
- Create: `app/src/components/workspace/workspace-topbar.tsx`
- Create: `app/src/components/workspace/workspace-nav.ts`
- Test: `app/src/components/workspace/workspace-nav.test.ts`
- Modify: `app/src/components/catalog/command-center.tsx`
- Modify: `app/src/styles.css`

**Interfaces:**
- Produces: `WorkspaceShell({ children, activeSection, counts? })`, `WORKSPACE_NAV`, `getWorkspaceNavState(pathname)`.
- Plan 3 later changes every remaining legacy anchor into a route only after the corresponding screens exist.

- [ ] **Step 1: Write failing navigation-model tests**

Assert stable entries for Dashboard, Projects, Mods, Roadmap, Assets, Files / Uploads, Docs, Issues, Changelog, AI Handoff, and Settings; assert `/mods/...` selects Mods and `/projects/...` selects Projects.

- [ ] **Step 2: Verify RED**

Run: `cd app && bun test src/components/workspace/workspace-nav.test.ts`

- [ ] **Step 3: Extract the shell**

Move only shell/sidebar/topbar responsibilities out of `command-center.tsx`; keep the current dashboard/project content and behavior intact. In this plan, wire Dashboard, Projects/current project behavior, and Mods as real routes. Leave not-yet-built entries visually present but pointing to their existing safe destinations until Plan 3 replaces them atomically.

- [ ] **Step 4: Verify tests, typecheck, and project-page rendering**

Run: `cd app && bun test src/components/workspace/workspace-nav.test.ts src/components/catalog src/lib/catalog && bun run typecheck`

Expected: PASS.

- [ ] **Step 5: Commit**

Commit message: `refactor: extract Catelog workspace shell`

---

### Task 4: Build the Mods index and game-hub routes

**Files:**
- Create: `app/src/lib/mods/mod-view-model.ts`
- Test: `app/src/lib/mods/mod-view-model.test.ts`
- Create: `app/src/components/mods/mod-library.tsx`
- Create: `app/src/components/mods/game-hub.tsx`
- Create: `app/src/routes/mods/index.tsx`
- Create: `app/src/routes/mods/$gameSlug/index.tsx`
- Modify: `app/src/styles.css`

**Interfaces:**
- Produces: `buildGameHubCards(payload)`, `getOrderedHubMods(hub, mods)`, `findGameHub(payload, gameSlug)`.

- [ ] **Step 1: Write failing view-model tests**

Assert mod counts, explicit `modOrder`, deterministic fallback order, Unsorted Mods placement, and omission of invalid child references with a warning.

- [ ] **Step 2: Verify RED**

Run: `cd app && bun test src/lib/mods/mod-view-model.test.ts`

- [ ] **Step 3: Implement index and game hub UIs**

`/mods` shows game cards with mod count, summary, latest documented update, and cover/banner fallback.

`/mods/:gameSlug` shows game header, shared notes/docs/assets summaries, recent history, and one tab/card per individual mod. Every mod link is `/mods/:gameSlug/:modSlug`.

- [ ] **Step 4: Verify focused tests and build**

Run: `cd app && bun test src/lib/mods && bun run typecheck && bun run build`

Expected: PASS.

- [ ] **Step 5: Commit**

Commit message: `feat: add Catelog game hub library`

---

### Task 5: Build individual mod pages and inner sections

**Files:**
- Create: `app/src/lib/mods/mod-detail-model.ts`
- Test: `app/src/lib/mods/mod-detail-model.test.ts`
- Create: `app/src/components/mods/mod-page.tsx`
- Create: `app/src/components/mods/mod-tabs.tsx`
- Create: `app/src/components/mods/mod-detail-section.tsx`
- Create: `app/src/routes/mods/$gameSlug/$modSlug/index.tsx`
- Modify: `app/src/styles.css`

**Interfaces:**
- Produces: `MOD_SECTIONS`, `buildModSections(mod)`, `getModDetailEntries(mod, section)`.
- Stable sections: Overview, Features, Files, Versions, Bugs, Changelog, Downloads, AI Brief.
- Files, Versions, and Downloads render explicit “No uploaded files/releases yet” states until Plan 2 provides operational data.

- [ ] **Step 1: Write failing section-model tests**

Assert all eight inner sections, documented feature/bug/install/compatibility mapping, and explicit empty states for data owned by Plan 2.

- [ ] **Step 2: Verify RED**

Run: `cd app && bun test src/lib/mods/mod-detail-model.test.ts`

- [ ] **Step 3: Implement the mod page**

Use URL search `?section=` for direct-linkable inner state; default to `Overview`; ignore invalid section values and fall back to Overview. Keep touch targets phone-friendly.

- [ ] **Step 4: Verify tests, typecheck, and production build**

Run: `cd app && bun test src/lib/mods src/components/mods && bun run typecheck && bun run build`

- [ ] **Step 5: Commit**

Commit message: `feat: add individual mod dossier pages`

---

### Task 6: Add game-hub and mod AI handoff serializers and routes

**Files:**
- Create: `app/src/lib/mods/mod-format.ts`
- Test: `app/src/lib/mods/mod-format.test.ts`
- Create: `app/src/routes/mods/$gameSlug/ai.tsx`
- Create: `app/src/routes/mods/$gameSlug/context[.]json.ts`
- Create: `app/src/routes/mods/$gameSlug/README[.]md.ts`
- Create: `app/src/routes/mods/$gameSlug/$modSlug/ai.tsx`
- Create: `app/src/routes/mods/$gameSlug/$modSlug/context[.]json.ts`
- Create: `app/src/routes/mods/$gameSlug/$modSlug/README[.]md.ts`

**Interfaces:**
- Produces: `buildGameHubAiContext`, `buildGameHubMarkdown`, `buildModAiContext`, `buildModMarkdown`, `buildModSharePrompt`.

- [ ] **Step 1: Write failing serializer tests**

Assert a mod handoff contains its parent game, summary, current facts, features, requirements, compatibility, installation, bugs, tasks, history, source references, and AI guidance. Assert it excludes unrelated mods. Assert a game-hub handoff contains shared context plus only its own mod index.

- [ ] **Step 2: Verify RED**

Run: `cd app && bun test src/lib/mods/mod-format.test.ts`

- [ ] **Step 3: Implement serializers and routes**

JSON routes return canonical structured data; Markdown and AI views derive from the same records. Missing facts say `Not documented yet` instead of being invented.

- [ ] **Step 4: Verify focused routes through production preview**

Run unit tests, `bun run typecheck`, `bun run build`, then preview and request one hub AI/JSON/README route and one mod AI/JSON/README route.

- [ ] **Step 5: Commit**

Commit message: `feat: add AI handoff for game hubs and mods`

---

### Task 7: Populate canonical game hubs and individual historical mod records

**Files in canonical data repo `nothatcher-creator/Catelog-`:**
- Create: `data/game-hubs.json`
- Create: `data/mods.json`

**Interfaces:**
- Consumes: Tasks 1–6 schemas.
- Produces: verified initial public mod catalog.

- [ ] **Step 1: Build a migration inventory from existing Catelog data and verified project history**

For every candidate mod, record `name`, verified target game, known status/summary, source/repository if documented, and whether the association is verified. Do not add inferred facts.

- [ ] **Step 2: Put uncertain associations into `unsorted`**

Create a real `unsorted` GameHub. Any ambiguous historical mod goes there until owner review.

- [ ] **Step 3: Write `game-hubs.json` and `mods.json`**

Use one ModRecord per individual mod. Preserve any existing grouped project records during migration, but do not render duplicate mod cards when a matching individual ModRecord exists.

- [ ] **Step 4: Validate the live raw files through the website parser**

Run a one-shot validation against the raw GitHub URLs using the Task 1 schemas; expected: zero invalid records, zero duplicate composite identities, and no missing game hubs except intentionally normalized Unsorted entries.

- [ ] **Step 5: Commit canonical data**

Commit message: `data: add Catelog game hubs and individual mods`

---

### Task 8: End-to-end mod-library verification and release

**Files:**
- Modify only if verification exposes a bug.

- [ ] **Step 1: Run the complete Catelog-focused test gate**

Run: `cd app && bun test src/components/catalog src/components/workspace src/components/mods src/lib/catalog src/lib/mods src/lib/owner src/lib/security-headers.test.ts`

Expected: zero failures.

- [ ] **Step 2: Run typecheck, targeted lint, and production build**

Run: `bun run typecheck`; targeted ESLint on changed Catelog/mod/workspace files; `bun run build`.

Expected: typecheck/build exit 0 and no ESLint errors.

- [ ] **Step 3: Browser verify desktop and Android widths**

At 1440×900 and 390×844 verify `/mods`, one multi-mod game hub, one individual mod, all eight mod tabs, and mod AI/JSON/Markdown routes. Assert no horizontal overflow and no page errors.

- [ ] **Step 4: Regression-check existing project routes**

Verify `/`, one project page, project AI, context JSON, README, and a runnable project route still behave as before.

- [ ] **Step 5: Commit any verification fixes**

Use TDD for every product-code fix discovered during verification.

- [ ] **Step 6: Push and deploy**

Push the verified website branch through the authorized website repository tool, deploy once, and check deployment status. Do not publish to the community feed unless separately requested.
