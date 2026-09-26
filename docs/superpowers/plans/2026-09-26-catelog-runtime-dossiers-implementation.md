# Catelog Runtime + Deep Project Dossiers Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Turn Catelog into a deeper project operating system where every project has a structured technical dossier and any confirmed public web deployment can run inside Catelog in embedded or full-screen mode.

**Architecture:** Keep `data/projects.json` as the canonical public source while extending the project schema with backward-compatible optional dossier and runtime fields. Add pure normalization/discovery models first, then render project-type-aware detail sections and a dedicated `/projects/:slug/run` route. Runtime health/probe work remains server-side, accepts only deployment IDs from canonical project data, and never proxies a site to bypass framing restrictions.

**Tech Stack:** React 19, TanStack Start/Router, TypeScript, Zod, Bun tests, Cloudflare Worker runtime, GitHub-hosted canonical JSON, Higgsfield website deployment.

**Spec:** `docs/superpowers/specs/2026-09-26-catelog-runtime-dossiers-design.md`

## Global Constraints

- Preserve all existing project records; new dossier/runtime fields are optional and backward-compatible.
- Missing facts remain `Not documented yet` or empty values; do not fabricate repository, build, feature, deployment, or testing claims.
- Human detail pages, AI context, JSON, and Markdown must derive from the same normalized `ProjectRecord`.
- Runtime URLs must use `http` or `https`; prefer `https`; reject script/data schemes and private/local targets.
- Never accept an arbitrary runtime URL query parameter; the runner selects canonical deployment IDs only.
- Do not proxy hosted projects to evade `X-Frame-Options` or CSP `frame-ancestors`.
- Do not load project iframes on the dashboard; load one only on the runner route.
- No secrets, GitHub tokens, API keys, or private environment values may enter public project JSON.
- Desktop and modern Android layouts remain first-class; reduced-motion and accessible labels remain supported.
- Existing AI routes `/projects/:slug/ai`, `/projects/:slug/context.json`, and `/projects/:slug/README.md` stay stable.

## Review Focus

1. **Legacy records with none of the new fields** must still validate and render with safe defaults; pinned by Task 1 tests.
2. **Malicious or local runtime URLs** such as `javascript:`, `localhost`, `127.0.0.1`, private IP literals, or malformed URLs must never be probed or framed; pinned by Task 5 tests.
3. **A deployment that is reachable but forbids framing** must render an explicit `Embedding blocked` state with `Open Live Project`, not a blank iframe; pinned by Task 6 and Task 7 tests.
4. **Multiple deployments with a stale preferred ID** must deterministically fall back to the first active valid deployment; pinned by Task 4 tests.
5. **Large or sparsely documented dossiers** must omit empty noise in AI/Markdown output while retaining important unknowns and remain searchable; pinned by Task 2 and Task 3 tests.

---

### Task 1: Backward-Compatible Dossier Types and Normalization

**Files:**
- Modify: Higgsfield `app/src/lib/catalog/catalog-types.ts`
- Create: Higgsfield `app/src/lib/catalog/catalog-schema.ts`
- Create: Higgsfield `app/src/lib/catalog/project-normalize.ts`
- Modify: Higgsfield `app/src/lib/catalog/catalog-fetch.server.ts`
- Test: Higgsfield `app/src/lib/catalog/project-normalize.test.ts`

**Interfaces:**
- Produces: `normalizeProject(raw: unknown): ProjectRecord | null`
- Produces new typed records: `ProjectFeature`, `ProjectScreen`, `ProjectSystem`, `ProjectDataEntity`, `ProjectMilestone`, `ProjectBug`, `ProjectAsset`, `ProjectDeployment`, `ProjectRuntime`, `ProjectRepositoryInfo`, `ProjectAiGuidance`.
- `ProjectRecord` keeps every existing required field and adds dossier fields as normalized arrays/objects with safe defaults.

- [ ] **Step 1: Write failing normalization tests** asserting an existing legacy record becomes a valid `ProjectRecord`, omitted new fields normalize to empty/default values, malformed nested entries are discarded without rejecting the whole project, and `contextVersion`/legacy fields are preserved.
- [ ] **Step 2: Run `cd app && bun test src/lib/catalog/project-normalize.test.ts`** and confirm the new tests fail because the normalizer does not exist.
- [ ] **Step 3: Add the dossier/runtime TypeScript types and Zod schemas** with optional input fields and normalized output defaults. `runtime.autoDetect` defaults to `true`; `runtime.embedPreference` defaults to `auto`; `deployments` defaults to `[]`.
- [ ] **Step 4: Implement `normalizeProject(raw)`** and change `fetchCatalogFromGitHub()` to normalize each project instead of requiring every future field in the raw object.
- [ ] **Step 5: Run the normalization tests plus existing catalog tests** and expect all to pass.
- [ ] **Step 6: Commit** with `feat: add backward compatible project dossier schema`.

### Task 2: Deep Search and Canonical AI/Markdown Serialization

**Files:**
- Modify: Higgsfield `app/src/lib/catalog/catalog-format.ts`
- Modify: Higgsfield `app/src/lib/catalog/catalog-format.test.ts`

**Interfaces:**
- Consumes normalized `ProjectRecord` from Task 1.
- Produces: expanded `matchesProjectSearch(project, query)`, `buildAiContext(project, canonicalUrl?)`, and `buildProjectMarkdown(project)`.

- [ ] **Step 1: Add failing tests** for matches inside feature names/descriptions, screens, systems, bugs, tasks, files, deployment labels/URLs, history, and AI guidance.
- [ ] **Step 2: Add failing serializer tests** proving documented dossier groups appear, empty optional groups are omitted, `Not documented yet` appears for critical unknowns, runtime/deployment data appears when present, and secrets are not introduced by formatting.
- [ ] **Step 3: Run `cd app && bun test src/lib/catalog/catalog-format.test.ts`** and verify failure.
- [ ] **Step 4: Implement flattened dossier search and section-aware AI/Markdown output** using small formatter helpers rather than one monolithic template.
- [ ] **Step 5: Re-run the test file** and expect pass.
- [ ] **Step 6: Commit** with `feat: expand Catelog search and AI handoff detail`.

### Task 3: Project-Type-Aware Detail Model

**Files:**
- Create: Higgsfield `app/src/lib/catalog/project-detail-model.ts`
- Create: Higgsfield `app/src/lib/catalog/project-detail-model.test.ts`
- Modify: Higgsfield `app/src/lib/catalog/dashboard-model.ts`
- Modify: Higgsfield `app/src/lib/catalog/dashboard-model.test.ts`

**Interfaces:**
- Produces: `getProjectKind(project): "game" | "app" | "website" | "mod-tool" | "general"`.
- Produces: `buildProjectSections(project): ProjectSectionDescriptor[]` where descriptors contain stable `id`, `label`, optional `count`, and `visible`.
- Existing `buildProjectTabs()` becomes a thin compatibility wrapper over section descriptors.

- [ ] **Step 1: Write failing tests** for game tabs (`World`, `Combat`, `Characters`, `Items`, `Quests`, `Progression`, `Multiplayer`), app tabs (`Screens`, `Navigation`, `Data`, `Services`, `Permissions`), website tabs (`Pages`, `Components`, `APIs`, `Hosting`, `SEO`, `Analytics`), and hiding empty optional sections.
- [ ] **Step 2: Add the review-focus test** that a sparse dossier still exposes Overview, Tasks/Roadmap when legacy data exists, Files/Documentation, History, and AI Brief without a wall of empty tabs.
- [ ] **Step 3: Run the two detail/dashboard model test files** and confirm failure.
- [ ] **Step 4: Implement the project kind mapper and section descriptor builder** with stable IDs used by the UI and direct anchors.
- [ ] **Step 5: Re-run tests** and expect pass.
- [ ] **Step 6: Commit** with `feat: add project type aware dossier sections`.

### Task 4: Runtime Candidate Detection and Preferred Deployment Selection

**Files:**
- Create: Higgsfield `app/src/lib/catalog/runtime-model.ts`
- Create: Higgsfield `app/src/lib/catalog/runtime-model.test.ts`

**Interfaces:**
- Produces: `discoverDeploymentCandidates(project: ProjectRecord, repositoryHomepage?: string | null): ProjectDeployment[]`.
- Produces: `selectPreferredDeployment(project: ProjectRecord, candidates?: ProjectDeployment[]): ProjectDeployment | null`.
- Produces: `mergeDeployments(manual, discovered): ProjectDeployment[]` where manual IDs/URLs win over automatic guesses.

- [ ] **Step 1: Write failing tests** covering manual preferred URL, project links labeled Live/Demo/Production/Preview, known hosting domains, a GitHub repository homepage candidate, repository URL exclusion, deduplication, multiple environments, and stale preferred IDs falling back to the first active valid deployment.
- [ ] **Step 2: Run `cd app && bun test src/lib/catalog/runtime-model.test.ts`** and confirm failure.
- [ ] **Step 3: Implement pure discovery/merge/selection functions** without network access so behavior is deterministic and unit-testable.
- [ ] **Step 4: Re-run the runtime model tests** and expect pass.
- [ ] **Step 5: Commit** with `feat: detect and select project deployments`.

### Task 5: Safe Server-Side Deployment Metadata and URL Validation

**Files:**
- Create: Higgsfield `app/src/lib/catalog/runtime-security.server.ts`
- Create: Higgsfield `app/src/lib/catalog/runtime-discovery.server.ts`
- Create: Higgsfield `app/src/lib/catalog/runtime-security.test.ts`

**Interfaces:**
- Produces: `validateRuntimeUrl(value: string): { ok: true; url: URL } | { ok: false; reason: string }`.
- Produces: `getRepositoryHomepage(repositoryUrl: string, fetcher?: typeof fetch): Promise<string | null>` for public GitHub repository metadata only.
- Runtime discovery may pass that homepage into Task 4 pure candidate discovery.

- [ ] **Step 1: Write failing URL-security tests** rejecting `javascript:`, `data:`, credentials-in-URL, `localhost`, `.local`, loopback, private IPv4 literals, IPv6 loopback/link-local, invalid ports, and malformed values; accept normal public HTTPS URLs.
- [ ] **Step 2: Write failing repository-homepage tests** with an injected fake fetcher for GitHub repo URLs, missing homepage, non-GitHub repository strings, and API failure.
- [ ] **Step 3: Run both tests** and verify failure.
- [ ] **Step 4: Implement validation and public GitHub homepage lookup** with no secret requirement and a short server cache where the runtime permits it.
- [ ] **Step 5: Re-run tests** and expect pass.
- [ ] **Step 6: Commit** with `feat: add safe deployment discovery metadata`.

### Task 6: Runtime Health and Frame-Policy Probe

**Files:**
- Create: Higgsfield `app/src/lib/catalog/runtime-probe.server.ts`
- Create: Higgsfield `app/src/lib/catalog/runtime-probe.test.ts`

**Interfaces:**
- Produces: `probeDeployment(url: string, fetcher?: typeof fetch): Promise<RuntimeProbeResult>`.
- `RuntimeProbeResult.status`: `online | unreachable | unknown`.
- `RuntimeProbeResult.embedStatus`: `embeddable | blocked | unknown`.
- Includes `httpStatus?`, `finalUrl?`, and human-readable `reason` without exposing response bodies.

- [ ] **Step 1: Write failing tests** for success, redirect, timeout/network failure, `X-Frame-Options: DENY`, `X-Frame-Options: SAMEORIGIN`, CSP `frame-ancestors 'none'`, CSP excluding Catelog origin, and inconclusive/no-policy headers.
- [ ] **Step 2: Run `cd app && bun test src/lib/catalog/runtime-probe.test.ts`** and confirm failure.
- [ ] **Step 3: Implement a conservative HEAD-first probe with bounded redirect/timeout behavior**; fall back to a minimal GET only where needed for headers; call Task 5 URL validation before any network request.
- [ ] **Step 4: Re-run tests** and expect pass.
- [ ] **Step 5: Commit** with `feat: probe hosted project runtime status`.

### Task 7: Dedicated Embedded and Full-Screen Project Runner

**Files:**
- Create: Higgsfield `app/src/components/catalog/project-runner.tsx`
- Create: Higgsfield `app/src/components/catalog/project-runner.test.tsx` if the existing test setup supports DOM rendering; otherwise test runner state helpers in `runtime-model.test.ts`.
- Create: Higgsfield `app/src/routes/projects/$slug/run.tsx`
- Modify: Higgsfield `app/src/styles.css`

**Interfaces:**
- Route loader resolves the project, discovered/manual deployments, selected preferred deployment, and probe result using canonical project data only.
- `ProjectRunner` consumes `{ project, deployments, selectedDeployment, probe }`.

- [ ] **Step 1: Add failing runner-state tests** proving no deployment shows `No hosted build documented`, blocked embedding exposes `Open Live Project`, unknown probe state still permits a cautious embed attempt, and deployment switching only accepts IDs in the canonical deployment list.
- [ ] **Step 2: Run tests** and verify failure.
- [ ] **Step 3: Implement `/projects/$slug/run` loader** with no unrestricted URL parameter. If a deployment selector is encoded in search params, validate it strictly against known deployment IDs before use.
- [ ] **Step 4: Implement the runner UI** with Back to Project, deployment selector, reload key, Open Externally, Enter/Exit Full Screen, status text, meaningful iframe title, lazy iframe creation, and explicit blocked/unreachable/unknown states.
- [ ] **Step 5: Add iframe permissions as a canonical allow-list**; default to only the minimal documented permissions and never interpolate arbitrary `allow` text from project JSON.
- [ ] **Step 6: Add responsive styles** so desktop uses a command toolbar plus large frame and mobile becomes near-full-screen with one-tap return. Browser Fullscreen API is optional enhancement; route-level full-screen styling must work even where the API is unavailable.
- [ ] **Step 7: Run targeted tests, `bun run typecheck`, and `bun run build`** and expect zero failures.
- [ ] **Step 8: Commit** with `feat: add embedded project runner`.

### Task 8: Deep Project Detail UI and Runtime Integration

**Files:**
- Create: Higgsfield `app/src/components/catalog/project-overview.tsx`
- Create: Higgsfield `app/src/components/catalog/project-detail-sections.tsx`
- Create: Higgsfield `app/src/components/catalog/runtime-card.tsx`
- Modify: Higgsfield `app/src/components/catalog/command-center.tsx`
- Modify: Higgsfield `app/src/styles.css`

**Interfaces:**
- Consumes section descriptors from Task 3 and runtime selection from Task 4.
- Project header exposes `Run Project` only when a canonical deployment exists.

- [ ] **Step 1: Add model/component tests or pure view-model tests** for summary counts, runtime CTA visibility, empty-state wording, feature/system/screen lists, and native projects correctly showing deployment as undocumented rather than runnable.
- [ ] **Step 2: Extract the current large project overview from `command-center.tsx`** into focused components while preserving the approved command-center visual language.
- [ ] **Step 3: Implement detailed sections** for Overview, Features, Screens/UI, Systems, Architecture/Technical, Data/APIs/Services, Tasks/Roadmap, Bugs/Testing, Deployments/Runtime, Files/Documentation, History, and AI Brief; render project-kind-specific groups from Task 3.
- [ ] **Step 4: Add `Run Project` and runtime-status cards** linking to `/projects/$slug/run`; do not render an iframe in the detail page.
- [ ] **Step 5: Verify desktop and Android-width CSS behavior** with the existing responsive breakpoints; long technical material uses collapsible or compact structured presentation rather than one wall of text.
- [ ] **Step 6: Run targeted tests, typecheck, and build** and expect pass.
- [ ] **Step 7: Commit** with `feat: deepen Catelog project workspace`.

### Task 9: Canonical Project Data Enrichment and Runtime Population

**Files:**
- Modify: GitHub `data/projects.json`

**Interfaces:**
- Data must validate through Task 1 normalization and drive all human/AI/runtime views.

- [ ] **Step 1: Enrich every existing project record** with the strongest already-documented details available: project kind, target users/problem where known, feature inventory, screens/pages where known, major systems, repository/source details, tasks, bugs/testing, decisions, AI guidance, and runtime/deployment structure.
- [ ] **Step 2: Keep unknown facts explicit** rather than guessing. Native Android/Godot projects receive `deployments: []` unless a confirmed browser-hosted build exists.
- [ ] **Step 3: For projects with documented repositories, inspect public repository metadata/links for confirmed hosted URLs** and record only verified candidates. Use `source: "repository"` or `source: "manual"` accurately.
- [ ] **Step 4: Add meaningful feature/system descriptions rather than duplicating the same sentence across every section.** Preserve all existing notes/history and increment `contextVersion` only for projects whose canonical meaning/detail materially changes.
- [ ] **Step 5: Fetch the updated raw `data/projects.json` through the same URL Catelog uses and run it through the app normalizer**; expect no skipped records or data errors.
- [ ] **Step 6: Commit** with `data: enrich Catelog project dossiers and deployments`.

### Task 10: Optional Secure Owner Runtime Editing Hook

**Files:**
- Create: Higgsfield `app/src/lib/owner/owner-auth.server.ts`
- Create: Higgsfield `app/src/lib/owner/github-catalog-write.server.ts`
- Create: Higgsfield `app/src/lib/owner/owner-auth.test.ts`
- Create: Higgsfield `app/src/components/catalog/runtime-editor.tsx`
- Create server routes under Higgsfield `app/src/routes/api/owner/` for login/session and project update.

**Interfaces:**
- Server-only secrets: `CATELOG_OWNER_PASSWORD`, `CATELOG_SESSION_SECRET`, `GITHUB_CATALOG_TOKEN`.
- Secrets are configured in Higgsfield website settings; they are never requested in chat and never returned to the browser.
- Write adapter is hard-coded to repository `nothatcher-creator/Catelog-`, branch `main`, path `data/projects.json`.

- [ ] **Step 1: Write failing auth/session tests** for wrong password, expired/invalid signature, missing secrets, and successful signed HttpOnly session creation.
- [ ] **Step 2: Write failing write-adapter tests** proving attempts to change repository/branch/path are rejected and GitHub optimistic SHA conflicts surface as a conflict rather than overwriting silently.
- [ ] **Step 3: Implement server-only owner session and GitHub write adapter** with Secure, HttpOnly, SameSite=Strict cookies and no token exposure to client code.
- [ ] **Step 4: Implement Runtime editor controls** for deployment URL, label, environment, active/preferred state, auto-detect toggle, and embed preference. If owner secrets are absent, hide built-in mutation UI and retain the GitHub edit-link fallback.
- [ ] **Step 5: Run owner tests, typecheck, and build** and expect pass.
- [ ] **Step 6: Commit** with `feat: add secure owner runtime editing`.

### Task 11: End-to-End Verification, Route Checks, and Deployment

**Files:**
- Modify only if verification exposes defects.

**Interfaces:**
- No new public API; verifies all prior tasks as a release candidate.

- [ ] **Step 1: Run the complete catalog test suite**: `cd app && bun test src/lib/catalog src/lib/owner` and expect zero failures.
- [ ] **Step 2: Run `cd app && bun run typecheck`** and expect exit code 0.
- [ ] **Step 3: Run targeted ESLint on all modified/new Catelog files** and expect zero errors.
- [ ] **Step 4: Run `cd app && bun run build`** and expect both client and SSR production builds to succeed.
- [ ] **Step 5: Start the production/dev server in the Higgsfield sandbox and use Playwright** to verify `/`, a detailed project route, `/ai`, `/context.json`, `/README.md`, and a runnable `/run` route at desktop and mobile viewport sizes.
- [ ] **Step 6: Verify one known embeddable hosted project** loads inside the runner and one synthetic/test blocked-frame target renders the fallback state. Do not claim iframe compatibility for projects that have not been checked.
- [ ] **Step 7: Verify no iframe is created on the dashboard or normal project overview before Run Project is opened.**
- [ ] **Step 8: Review rendered public JSON/Markdown/AI output for accidental secrets or invented values.**
- [ ] **Step 9: Push committed Higgsfield website changes, deploy the website, and confirm production status reports `deployed`.**
- [ ] **Step 10: Commit any verification-only fixes separately** with `fix: resolve Catelog runtime verification issues`.
