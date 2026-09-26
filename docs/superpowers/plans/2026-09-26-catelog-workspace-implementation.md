# Catelog Workspace Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build and publicly deploy Catelog as Brad's personal project workspace, with GitHub-backed canonical project data, shareable AI-ready project pages, and owner-only editing.

**Architecture:** The public product is a React 19 + TanStack Start website hosted by Higgsfield. Canonical project content lives in `nothatcher-creator/Catelog-` under `data/`; the website reads those records and derives the human project page, AI handoff, Markdown, and JSON views from one schema. Owner Mode performs browser-side GitHub API writes with a locally stored fine-grained token limited to this repository.

**Tech Stack:** React 19, TanStack Start, TypeScript, Tailwind/CSS, Higgsfield website hosting, GitHub Contents API, JSON project records, Bun tests.

**Spec:** `docs/superpowers/specs/2026-09-26-catelog-workspace-design.md`

## Global Constraints

- Public read access requires no sign-in.
- Owner editing is for Brad only.
- Canonical project data remains in `nothatcher-creator/Catelog-`.
- Never commit or server-store Brad's GitHub token.
- Human, AI, Markdown, and JSON project views must derive from the same project record.
- Missing facts must be shown as unknown/not documented rather than invented.
- Supported statuses are exactly: `IDEA`, `PLANNING`, `BUILDING`, `TESTING`, `WORKING`, `BROKEN`, `PAUSED`, `FINISHED`, `ARCHIVED`.
- Mobile is first-class for modern Android phones including Galaxy S25-sized screens.
- The UI may take structural inspiration from the supplied workspace screenshot but must remain an original design.
- Higgsfield is used as a standalone `website`; no Higgsfield generation/sign-in exists inside Catelog itself.
- The first release is publicly deployed but is not listed in the Higgsfield community feed unless Brad explicitly opts in.

## Review Focus

- GitHub is unreachable or rate-limited: show a clear data-loading error rather than a blank workspace.
- One malformed project JSON file: skip only that record and surface a data warning without crashing the catalog.
- Unknown optional project fields: render `Not documented yet` rather than failing or fabricating content.
- Owner token is missing, invalid, or lacks repo write access: remain read-only and expose no destructive owner actions.
- Mobile viewport around 360-430 px: no squeezed desktop navigator, clipped controls, or inaccessible Share with AI action.

---

### Task 1: Create the canonical Catelog data model and initial records

**Files:**
- Create: `data/categories.json`
- Create: `data/workspace.json`
- Create: `data/projects/worldforge.json`
- Create: `data/projects/roomforge.json`
- Create: `data/projects/lyricforge.json`
- Create: `data/projects/voidforge.json`
- Create: `data/projects/parenting-app.json`
- Create: `data/projects/bass-transcription.json`
- Create: `data/projects/room-scanner.json`
- Create: `data/projects/rust-survival.json`
- Create: `data/projects/zomboid-android-web.json`
- Create: `data/projects/zomboid-mods.json`
- Create: `data/projects/balatro-mods.json`
- Create: `data/projects/web-blender.json`
- Create: `README.md`

**Interfaces:**
- Produces: stable JSON project records keyed by `slug`, with `contextVersion`, `status`, `category`, `platforms`, `tags`, `summary`, `purpose`, `goals`, `nonGoals`, `stack`, `architecture`, `currentState`, `completed`, `inProgress`, `knownBugs`, `decisions`, `constraints`, `repositories`, `links`, `buildInstructions`, `nextTasks`, `notes`, and `history`.
- Produces: categories and workspace metadata consumed by the website loader.

- [ ] **Step 1: Define the project record shape in the first seed record and document required vs optional fields in `README.md`.**
- [ ] **Step 2: Populate `categories.json` and `workspace.json` with the approved categories, display name, and pinned-order fields.**
- [ ] **Step 3: Seed the 12 initial project files using only known information; use empty arrays/nulls for unknown details.**
- [ ] **Step 4: Validate every JSON file parses successfully with `python3 -m json.tool` or equivalent.**
- [ ] **Step 5: Commit the canonical data seed.**

### Task 2: Create the Higgsfield website shell and project-data loader

**Files:**
- Create/modify in Higgsfield website repo: `app/design-brief.md`
- Create/modify: `app/src/lib/catalog/types.ts`
- Create/modify: `app/src/lib/catalog/github.server.ts`
- Create/modify: `app/src/lib/catalog/parse.ts`
- Create/modify: `app/src/lib/catalog/parse.test.ts`
- Modify: `app/src/routes/__root.tsx`
- Modify: `app/src/styles.css`

**Interfaces:**
- Produces: `ProjectRecord`, `WorkspaceConfig`, `CategoryRecord`, `CatalogLoadResult` types.
- Produces: `loadCatalog(): Promise<CatalogLoadResult>` that reads the public Catelog repo.
- Produces: `parseProjectRecord(input: unknown): ParseResult<ProjectRecord>` that isolates malformed records.

- [ ] **Step 1: Resolve Higgsfield's mandatory website animation choice and brand intake before creation.**
- [ ] **Step 2: Create Catelog as a Higgsfield standalone website and write the design brief before product code.**
- [ ] **Step 3: Write failing parser tests for a valid project, missing optional fields, and a malformed project.**
- [ ] **Step 4: Implement typed parsing and public GitHub data loading.**
- [ ] **Step 5: Verify parser tests pass with `bun test`.**
- [ ] **Step 6: Establish global workspace tokens, typography, desktop shell, and mobile shell without owner functionality yet.**
- [ ] **Step 7: Commit the shell and loader.**

### Task 3: Build the public workspace, project browsing, search, and filtering

**Files:**
- Create/modify: `app/src/routes/index.tsx`
- Create: `app/src/routes/projects/index.tsx`
- Create: `app/src/routes/search.tsx`
- Create: `app/src/components/workspace/WorkspaceHeader.tsx`
- Create: `app/src/components/workspace/ProjectCard.tsx`
- Create: `app/src/components/workspace/ProjectNavigator.tsx`
- Create: `app/src/components/workspace/MobileNav.tsx`
- Create: `app/src/components/workspace/ActivityFeed.tsx`
- Create: `app/src/lib/catalog/search.ts`
- Create: `app/src/lib/catalog/search.test.ts`

**Interfaces:**
- Consumes: `CatalogLoadResult` and `ProjectRecord` from Task 2.
- Produces: `searchProjects(projects, query, filters, sort): ProjectRecord[]`.

- [ ] **Step 1: Write failing tests proving search matches name, description, aliases, tags, stack, platforms, and notes.**
- [ ] **Step 2: Write failing tests for category/status/platform/pinned/favorite filters and recently-updated/alphabetical/status/category sorts.**
- [ ] **Step 3: Implement search/filter/sort utilities and pass the tests.**
- [ ] **Step 4: Build Home with pinned projects, recent projects, categories, summary counts, and activity.**
- [ ] **Step 5: Build Projects and Search views with large touch targets and responsive cards.**
- [ ] **Step 6: Implement the desktop right-side project/file navigator and mobile full-screen Files panel.**
- [ ] **Step 7: Verify the mobile layout does not render the desktop navigator inline below 768 px.**
- [ ] **Step 8: Commit public discovery UI.**

### Task 4: Build project detail and canonical AI handoff outputs

**Files:**
- Create: `app/src/routes/projects/$slug.tsx`
- Create: `app/src/routes/projects/$slug/ai.tsx`
- Create: `app/src/routes/projects/$slug/context[.]json.ts`
- Create: `app/src/routes/projects/$slug/README[.]md.ts`
- Create: `app/src/components/project/ProjectOverview.tsx`
- Create: `app/src/components/project/ProjectSections.tsx`
- Create: `app/src/components/project/ShareWithAI.tsx`
- Create: `app/src/lib/catalog/render-context.ts`
- Create: `app/src/lib/catalog/render-context.test.ts`

**Interfaces:**
- Produces: `renderAiContext(project: ProjectRecord): string`.
- Produces: `renderProjectMarkdown(project: ProjectRecord): string`.
- Produces: `buildSharePrompt(project: ProjectRecord, canonicalUrl: string): string`.

- [ ] **Step 1: Write failing tests proving AI, Markdown, and JSON outputs use the same project fields and represent missing values as `Not documented yet`.**
- [ ] **Step 2: Implement context renderers and Share with AI prompt generation.**
- [ ] **Step 3: Build the human project route with Overview, Current State, Roadmap, Files/Links, AI Context, and History sections.**
- [ ] **Step 4: Build the dense AI route with the canonical-source instruction at the top.**
- [ ] **Step 5: Expose valid JSON and Markdown response routes with correct content types.**
- [ ] **Step 6: Implement one-tap Share with AI copy on desktop and mobile.**
- [ ] **Step 7: Commit project detail and handoff routes.**

### Task 5: Build owner-only GitHub authentication and safe write adapter

**Files:**
- Create: `app/src/lib/owner/token.client.ts`
- Create: `app/src/lib/owner/github.client.ts`
- Create: `app/src/lib/owner/paths.ts`
- Create: `app/src/lib/owner/paths.test.ts`
- Create: `app/src/components/owner/OwnerUnlock.tsx`
- Create: `app/src/components/owner/OwnerBar.tsx`
- Modify: `app/src/routes/settings.tsx`

**Interfaces:**
- Produces: `unlockOwnerMode(token: string, remember: boolean): Promise<OwnerSession>`.
- Produces: `lockOwnerMode(): void`.
- Produces: `readOwnerToken(): string | null`, called only in client-safe code.
- Produces: `assertWritableCatalogPath(path: string): string` allowing only approved `data/` project/config paths.
- Produces: `writeCatalogFile({path, content, sha?, message, token})` against `nothatcher-creator/Catelog-` only.

- [ ] **Step 1: Write failing tests that reject path traversal, arbitrary repository paths, and non-`data/` writes.**
- [ ] **Step 2: Implement the writable-path allowlist and pass tests.**
- [ ] **Step 3: Implement owner unlock by verifying the token can access the configured repository before enabling writes.**
- [ ] **Step 4: Store the token only in browser storage when Remember is explicitly selected; never render it or transmit it to Higgsfield servers.**
- [ ] **Step 5: Implement Lock Owner Mode to clear locally stored credentials and owner state.**
- [ ] **Step 6: Build Settings owner unlock/lock UI and an Owner Mode indicator visible only after successful verification.**
- [ ] **Step 7: Commit Owner Mode authentication/write foundation.**

### Task 6: Build the project editor and GitHub save flows

**Files:**
- Create: `app/src/components/owner/ProjectEditor.tsx`
- Create: `app/src/components/owner/ProjectEditorSections.tsx`
- Create: `app/src/components/owner/NewProjectDialog.tsx`
- Create: `app/src/lib/owner/project-mutations.ts`
- Create: `app/src/lib/owner/project-mutations.test.ts`
- Modify: public workspace/project components to expose owner controls only while unlocked.

**Interfaces:**
- Produces: `createProject(record: ProjectRecord, token: string): Promise<void>`.
- Produces: `updateProject(record: ProjectRecord, sha: string, token: string): Promise<void>`.
- Produces: `archiveProject(record: ProjectRecord, sha: string, token: string): Promise<void>`.
- Produces: `appendHistoryEntry(record, entry): ProjectRecord` and automatic `contextVersion` increment on meaningful edits.

- [ ] **Step 1: Write failing tests for slug normalization, duplicate/invalid slug rejection, history append, archive behavior, and context-version increment.**
- [ ] **Step 2: Implement mutation helpers and pass tests.**
- [ ] **Step 3: Build New Project and Edit Project flows covering status, category, platform, goals, architecture, current state, bugs, decisions, constraints, links, next tasks, and history.**
- [ ] **Step 4: Implement Pin/Favorite/Status/Add Note/Add Link/Add Changelog controls as focused owner actions.**
- [ ] **Step 5: Save through GitHub Contents API with descriptive commit messages and current SHA checks so concurrent edits do not silently overwrite newer data.**
- [ ] **Step 6: Refresh project/catalog state after a successful save and show useful conflicts/errors when saving fails.**
- [ ] **Step 7: Commit owner editor flows.**

### Task 7: Finish branding, error resilience, metadata, verification, and deployment

**Files:**
- Modify: `app/src/app-meta.json`
- Create/update: `app/public/assets/*` generated website identity/cover assets required by Higgsfield.
- Modify: relevant route error/loading states.
- Modify: `app/design-brief.md` only if an implementation decision changes the approved design contract.

**Interfaces:**
- Consumes all previous tasks.
- Produces the publicly deployed Catelog URL.

- [ ] **Step 1: Complete Higgsfield's required reference-board/asset pipeline consistent with the approved dark workspace concept and user-selected animation mode.**
- [ ] **Step 2: Add loading, partial-data warning, GitHub-unavailable, unknown-field, invalid-token, and save-conflict states.**
- [ ] **Step 3: Verify all generated asset references are used and no placeholders/scaffold copy remain.**
- [ ] **Step 4: Run `bun test` from `app/` and require all tests to pass.**
- [ ] **Step 5: Run Higgsfield's mechanical review gate and `bun run typecheck`; fix every failure before deployment.**
- [ ] **Step 6: Verify required routes exist: `/`, `/projects`, `/projects/:slug`, `/projects/:slug/ai`, `/projects/:slug/context.json`, `/projects/:slug/README.md`, `/search`, `/settings`.**
- [ ] **Step 7: Fill real OG title, description, favicon, OG image, and marketplace cover metadata.**
- [ ] **Step 8: Deploy publicly through Higgsfield.**
- [ ] **Step 9: Publish to the Higgsfield community feed only if Brad explicitly opted in.**
- [ ] **Step 10: Record the live Catelog URL in the GitHub repository README/workspace metadata and commit the final canonical-data update.**

## Self-Review

- Spec coverage: all 12 V1 acceptance criteria map to Tasks 1-7.
- Single-source context rule is enforced by Task 4 renderers.
- Owner-token safety is isolated to client-only modules in Task 5 and never requires a server secret.
- GitHub write scope is constrained by repository constant plus tested data-path allowlisting.
- Partial/malformed data and GitHub failure cases are explicitly tested/handled.
- Mobile navigation and Share with AI remain first-class rather than desktop afterthoughts.
- The only unresolved product choice is Higgsfield's mandatory animation-mode intake; it must be answered before website creation and does not change Catelog's functional architecture.
