# Catelog Hybrid Files and GitHub Owner Authentication Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add public hybrid file storage, Android-friendly uploads, stable public download URLs, release/version management, and GitHub OAuth owner authentication to Catelog.

**Architecture:** Enable one D1 database and one R2 bucket on the existing Higgsfield website. D1 stores operational metadata, sessions, OAuth state, encrypted GitHub credentials, upload sessions, file records, and releases; R2 stores binary/media/build bytes. Small allowlisted UTF-8 text files default to the canonical public GitHub repository, but every file receives a Catelog metadata record and stable Catelog URL so both backends appear as one library.

**Tech Stack:** React 19, TanStack Start/Router server routes, TypeScript, Cloudflare D1/R2 bindings, Web Crypto, GitHub OAuth + REST Contents API, Bun tests, Playwright Chromium for targeted mobile/desktop flows.

**Spec:** `docs/superpowers/specs/2026-09-27-catelog-mods-files-architecture-design.md`

**Depends on:** `docs/superpowers/plans/2026-09-27-catelog-mod-hubs-navigation-implementation.md`

## Global Constraints

- All catalog files/downloads are public; mutations are owner-only.
- Owner identity is the configured stable numeric GitHub user ID, not username/display name.
- GitHub OAuth client secret and reusable access token never enter localStorage/sessionStorage or client bundles.
- OAuth state must be one-time and expire after 10 minutes.
- Owner sessions expire after 7 days of inactivity; cookie is `HttpOnly; Secure; SameSite=Strict; Path=/` after callback completion.
- GitHub OAuth scope is limited to `read:user public_repo` for the public Catelog repository.
- Reusable GitHub OAuth tokens are encrypted at rest in D1 with AES-GCM using a server-only encryption key.
- Required server secret names: `GITHUB_OAUTH_CLIENT_ID`, `GITHUB_OAUTH_CLIENT_SECRET`, `CATELOG_OWNER_GITHUB_ID`, `CATELOG_SESSION_SECRET`, `CATELOG_GITHUB_TOKEN_KEY`.
- Secret values are configured only in Higgsfield website settings; never request or commit values.
- `app/app.manifest.json` enables exactly `db: true` and `r2: true`; no KV/DO is required for this plan.
- All D1 migrations are additive because deployment applies them to the single live database.
- Default GitHub backend eligibility: allowlisted UTF-8 text/code/doc extension AND size ≤ 1 MiB. Everything else defaults to R2.
- R2 multipart chunk size is 8 MiB; application default max file size is 1 GiB and is enforced before upload starts.
- R2 object keys are opaque UUID-based keys, never derived solely from display filenames.
- `Replace file` and `Upload new version` are distinct operations.
- Failed uploads must not create a completed/downloadable file record.
- Public file routes must not execute uploaded active content on the Catelog origin; unsafe/active MIME types are forced to attachment download with `X-Content-Type-Options: nosniff`.
- GitHub writes are hard-scoped to `nothatcher-creator/Catelog-`, branch `main`, and permitted catalog file roots; browser input never selects repository/branch.

## Review Focus

1. Filename/path traversal and active-content uploads must not escape canonical paths or become same-origin executable content.
2. OAuth state replay, wrong-owner login, expired sessions, and revoked credentials must fail closed to public read-only mode.
3. Interrupted multipart uploads must not create completed metadata and must be abortable/retryable without duplicate versions.
4. Concurrent GitHub replace/update must surface SHA conflict instead of overwriting newer content.
5. Stable `/files/:fileId/:displayFilename` links must continue working after rename or backend/version changes.

---

### Task 1: Enable D1/R2 and add the additive operational schema

**Files:**
- Modify: `app/app.manifest.json`
- Modify: `app/src/lib/bindings.server.ts`
- Create: `app/migrations/0002_files_auth.sql`
- Create: `app/src/lib/storage/storage-health.server.ts`
- Test: `app/src/lib/storage/storage-health.test.ts`

**Interfaces:**
- Produces: guarded `db()` and `storage()` helpers returning D1/R2 bindings or `null`; `storageCapabilities()` returning `{ db, r2 }`.

**Migration tables:**

- `github_oauth_states(state_hash TEXT PRIMARY KEY, created_at TEXT NOT NULL, expires_at TEXT NOT NULL)`.
- `owner_sessions(id_hash TEXT PRIMARY KEY, github_user_id TEXT NOT NULL, github_login TEXT NOT NULL, created_at TEXT NOT NULL, last_seen_at TEXT NOT NULL, expires_at TEXT NOT NULL)`.
- `github_credentials(github_user_id TEXT PRIMARY KEY, token_ciphertext TEXT NOT NULL, token_iv TEXT NOT NULL, updated_at TEXT NOT NULL)`.
- `catalog_files(id TEXT PRIMARY KEY, logical_id TEXT NOT NULL, owner_type TEXT NOT NULL, project_slug TEXT, game_hub_slug TEXT, mod_slug TEXT, display_name TEXT NOT NULL, original_filename TEXT NOT NULL, role TEXT NOT NULL, mime_type TEXT NOT NULL, extension TEXT NOT NULL, size_bytes INTEGER NOT NULL, version TEXT, tags_json TEXT NOT NULL DEFAULT '[]', notes_json TEXT NOT NULL DEFAULT '[]', storage_backend TEXT NOT NULL, storage_key TEXT, github_path TEXT, checksum_sha256 TEXT NOT NULL, uploaded_at TEXT NOT NULL, uploaded_by_github_user_id TEXT NOT NULL, is_latest INTEGER NOT NULL DEFAULT 1, replaced_at TEXT)`.
- indexes on owner entity, logical_id, uploaded_at, and `is_latest`.
- `upload_sessions(id TEXT PRIMARY KEY, owner_type TEXT NOT NULL, project_slug TEXT, game_hub_slug TEXT, mod_slug TEXT, backend TEXT NOT NULL, filename TEXT NOT NULL, mime_type TEXT NOT NULL, size_bytes INTEGER NOT NULL, role TEXT NOT NULL, version TEXT, storage_key TEXT, multipart_upload_id TEXT, state TEXT NOT NULL, created_at TEXT NOT NULL, expires_at TEXT NOT NULL, actor_github_user_id TEXT NOT NULL, metadata_json TEXT NOT NULL DEFAULT '{}')`.
- `releases(id TEXT PRIMARY KEY, game_hub_slug TEXT NOT NULL, mod_slug TEXT NOT NULL, version TEXT NOT NULL, title TEXT NOT NULL, status TEXT NOT NULL, released_at TEXT, compatibility_json TEXT NOT NULL DEFAULT '[]', changes_json TEXT NOT NULL DEFAULT '[]', notes_json TEXT NOT NULL DEFAULT '[]', installation_notes_json TEXT NOT NULL DEFAULT '[]', is_latest INTEGER NOT NULL DEFAULT 0, created_at TEXT NOT NULL, updated_at TEXT NOT NULL)`.
- `release_files(release_id TEXT NOT NULL, file_id TEXT NOT NULL, kind TEXT NOT NULL, PRIMARY KEY(release_id, file_id))`.

- [ ] **Step 1: Write failing capability tests**

Test missing bindings return disabled capabilities instead of throwing, and mocked DB/R2 bindings report enabled.

- [ ] **Step 2: Verify RED**

Run: `cd app && bun test src/lib/storage/storage-health.test.ts`

- [ ] **Step 3: Add manifest flags, migration, and guarded binding helpers**

Set `db: true`, `r2: true`, leave `kv: false`, `durableObject: null`.

- [ ] **Step 4: Verify GREEN and typecheck**

Run: `cd app && bun test src/lib/storage/storage-health.test.ts && bun run typecheck`

- [ ] **Step 5: Commit**

Commit message: `feat: enable Catelog file storage infrastructure`

---

### Task 2: Replace password owner mode with GitHub OAuth + server sessions

**Files:**
- Create: `app/src/lib/auth/auth-config.server.ts`
- Create: `app/src/lib/auth/crypto.server.ts`
- Create: `app/src/lib/auth/github-oauth.server.ts`
- Create: `app/src/lib/auth/owner-session.server.ts`
- Test: `app/src/lib/auth/github-oauth.test.ts`
- Test: `app/src/lib/auth/owner-session.test.ts`
- Create: `app/src/routes/api/auth/github/start.ts`
- Create: `app/src/routes/api/auth/github/callback.ts`
- Create: `app/src/routes/api/auth/session.ts`
- Create: `app/src/routes/api/auth/logout.ts`
- Modify: `app/src/lib/owner/github-catalog-write.server.ts`
- Delete after migration: `app/src/lib/owner/owner-auth.server.ts`
- Delete after migration: `app/src/routes/api/owner/session.ts`

**Interfaces:**
- Produces: `ownerAuthConfigured()`, `createOAuthState()`, `consumeOAuthState(state)`, `exchangeGithubCode(code)`, `fetchGithubIdentity(token)`, `storeGithubCredential(userId, token)`, `getGithubCredential(userId)`, `createOwnerSession(identity)`, `getOwnerSession(request)`, `requireOwner(request)`, `clearOwnerSession(request)`.
- `requireOwner` returns stable `{ githubUserId, login }` or throws a typed unauthorized error.

- [ ] **Step 1: Write failing auth tests**

Cover missing config, state creation/10-minute expiry, state one-time consumption/replay, token exchange failure, non-owner numeric ID, encrypted credential round-trip, valid session, tampered cookie, expired session, and logout.

- [ ] **Step 2: Verify RED**

Run: `cd app && bun test src/lib/auth`

- [ ] **Step 3: Implement OAuth state and callback flow**

`/api/auth/github/start` creates one-time state and redirects to GitHub. Callback consumes state, exchanges code, fetches `/user`, compares `String(user.id)` with `CATELOG_OWNER_GITHUB_ID`, encrypts token server-side only for the approved owner, creates the owner session, and redirects to `/settings`.

- [ ] **Step 4: Migrate GitHub write helper to `requireOwner` + stored credential**

Keep repository/branch/path hard-coded. Do not accept token, repository, branch, or arbitrary base path from browser payloads.

- [ ] **Step 5: Verify GREEN, client-bundle secret scan, and typecheck**

Run focused auth tests, `bun run typecheck`, and grep built client assets for secret names/token values after build fixtures.

- [ ] **Step 6: Commit**

Commit message: `feat: add GitHub OAuth owner authentication`

---

### Task 3: Define unified file metadata, storage policy, and stable URLs

**Files:**
- Create: `app/src/lib/files/file-types.ts`
- Create: `app/src/lib/files/file-policy.ts`
- Create: `app/src/lib/files/file-url.ts`
- Create: `app/src/lib/files/file-repository.server.ts`
- Test: `app/src/lib/files/file-policy.test.ts`
- Test: `app/src/lib/files/file-url.test.ts`
- Test: `app/src/lib/files/file-repository.test.ts`

**Interfaces:**
- Produces: `CatalogFile`, `FileOwner`, `FileRole`, `chooseStorageBackend(fileMeta)`, `sanitizeDisplayFilename(name)`, `buildStableFileUrl(file)`, `createPendingFileMetadata`, `listFiles(filters)`, `getFile(id)`, `replaceFileMetadata`, `createVersionMetadata`, `renameFileMetadata`, `deleteFileMetadata`.

- [ ] **Step 1: Write failing policy tests**

Assert allowlisted UTF-8 `.md/.txt/.json/.ts/.js/.lua/.gd` ≤1 MiB chooses GitHub; images, archives, APKs, executables, audio/video/models, binary content, unknown extensions, and >1 MiB choose R2. Assert owner override is rejected when unsafe for GitHub.

- [ ] **Step 2: Write failing filename/URL tests**

Cover Unicode names, traversal attempts, control characters, slash/backslash, rename preserving file ID, and cosmetic filename mismatch.

- [ ] **Step 3: Verify RED**

Run: `cd app && bun test src/lib/files`

- [ ] **Step 4: Implement policy and D1 repository**

Use UUIDs for file IDs/logical IDs. Metadata rows become downloadable only after backend completion. Filters paginate by `uploaded_at,id`; do not load the full file table into dashboard requests.

- [ ] **Step 5: Verify GREEN**

Run: `cd app && bun test src/lib/files`

- [ ] **Step 6: Commit**

Commit message: `feat: add unified Catelog file model`

---

### Task 4: Implement the GitHub small-file backend with optimistic concurrency

**Files:**
- Create: `app/src/lib/files/github-file-store.server.ts`
- Test: `app/src/lib/files/github-file-store.test.ts`

**Interfaces:**
- Produces: `githubPathForOwner(owner, filename)`, `putGithubFile({ owner, filename, text, existingSha? }, credential)`, `deleteGithubFile(...)`.
- Canonical roots:
  - project: `files/projects/<projectSlug>/`
  - game hub: `files/game-hubs/<gameHubSlug>/`
  - mod: `files/mods/<gameHubSlug>/<modSlug>/`
  - workspace: `files/workspace/`

- [ ] **Step 1: Write failing GitHub-store tests**

Assert canonical path construction, traversal rejection, `public_repo` token use only server-side, create success, replace requires current SHA, 409/422 conflict mapping, and repository/branch cannot be overridden.

- [ ] **Step 2: Verify RED**

Run: `cd app && bun test src/lib/files/github-file-store.test.ts`

- [ ] **Step 3: Implement GitHub Contents API adapter**

Hard-code `nothatcher-creator/Catelog-` and `main`; return typed `{ path, blobSha, rawUrl }` or conflict/failure.

- [ ] **Step 4: Verify GREEN**

- [ ] **Step 5: Commit**

Commit message: `feat: add GitHub-backed catalog files`

---

### Task 5: Implement resumable R2 multipart upload sessions

**Files:**
- Create: `app/src/lib/files/r2-file-store.server.ts`
- Create: `app/src/lib/files/upload-session.server.ts`
- Test: `app/src/lib/files/upload-session.test.ts`
- Create: `app/src/routes/api/files/upload/init.ts`
- Create: `app/src/routes/api/files/upload/$sessionId/part.ts`
- Create: `app/src/routes/api/files/upload/$sessionId/complete.ts`
- Create: `app/src/routes/api/files/upload/$sessionId/abort.ts`

**Interfaces:**
- Produces: `startUploadSession(input, owner)`, `uploadPart(sessionId, partNumber, bytes, owner)`, `completeUploadSession(sessionId, parts, owner)`, `abortUploadSession(sessionId, owner)`.
- `init` returns `{ sessionId, backend, chunkSize: 8388608 }`; GitHub-eligible files may return backend `github` and use Task 6’s small-file completion route.

- [ ] **Step 1: Write failing upload-session tests**

Cover 1 GiB preflight rejection, 8 MiB chunk contract, unauthorized calls, invalid/out-of-order part numbers, retrying an already uploaded part, completion checksum/size mismatch, abort, expiry, and failure rollback.

- [ ] **Step 2: Verify RED**

Run: `cd app && bun test src/lib/files/upload-session.test.ts`

- [ ] **Step 3: Implement R2 multipart storage**

Store only upload-session state until byte completion. On complete, verify total byte count and SHA-256 supplied/derived by the client protocol, create the final `catalog_files` row, then mark session complete. A failed completion leaves no public completed file row.

- [ ] **Step 4: Add cleanup behavior**

Expired sessions are marked abandoned; abort R2 multipart uploads when possible. Cleanup can run opportunistically on upload API calls; no scheduler is required in this plan.

- [ ] **Step 5: Verify GREEN**

- [ ] **Step 6: Commit**

Commit message: `feat: add resumable Catelog object uploads`

---

### Task 6: Add owner file mutation APIs for GitHub/R2, replace, version, rename, delete, and reassignment

**Files:**
- Create: `app/src/lib/files/file-service.server.ts`
- Test: `app/src/lib/files/file-service.test.ts`
- Create: `app/src/routes/api/files/github-upload.ts`
- Create: `app/src/routes/api/files/$fileId/index.ts`
- Create: `app/src/routes/api/files/$fileId/replace.ts`
- Create: `app/src/routes/api/files/$fileId/new-version.ts`

**Interfaces:**
- Produces: `uploadSmallGithubFile`, `replaceCatalogFile`, `createCatalogFileVersion`, `renameCatalogFile`, `reassignCatalogFile`, `deleteCatalogFile`.

- [ ] **Step 1: Write failing service tests**

Assert every mutation requires owner session; replace preserves stable file ID/logical ID; new version creates new file ID with same logical ID and flips `is_latest`; rename keeps stable URL by ID; reassignment validates entity existence; delete removes/invalidates the backend object and metadata consistently; GitHub conflicts remain conflicts.

- [ ] **Step 2: Verify RED**

- [ ] **Step 3: Implement service and API routes**

All browser payloads use validated IDs/slugs and semantic metadata only. Backend/repository/path are selected server-side.

- [ ] **Step 4: Verify GREEN and auth regression tests**

- [ ] **Step 5: Commit**

Commit message: `feat: add owner file management operations`

---

### Task 7: Add stable public file delivery

**Files:**
- Create: `app/src/lib/files/public-file.server.ts`
- Test: `app/src/lib/files/public-file.test.ts`
- Create: `app/src/routes/files/$fileId/$filename.ts`

**Interfaces:**
- Produces: `serveCatalogFile(fileId, requestedFilename, request)`.

- [ ] **Step 1: Write failing public-delivery tests**

Cover logged-out access, missing file 404, GitHub backend redirect to canonical raw URL, R2 streaming, rename/cosmetic filename mismatch, safe raster/media inline behavior, active HTML/SVG/JS/XML/executable forced attachment, `nosniff`, and range requests for R2 media where supported.

- [ ] **Step 2: Verify RED**

- [ ] **Step 3: Implement public file route**

The file ID is authoritative. If requested filename differs, either serve by ID or 302 to the current cosmetic URL; never 404 solely because of rename.

- [ ] **Step 4: Verify GREEN**

- [ ] **Step 5: Commit**

Commit message: `feat: add stable public Catelog file URLs`

---

### Task 8: Build Files / Uploads and Assets workspaces

**Files:**
- Create: `app/src/components/files/file-workspace.tsx`
- Create: `app/src/components/files/upload-dialog.tsx`
- Create: `app/src/components/files/upload-queue.tsx`
- Create: `app/src/components/files/file-list.tsx`
- Create: `app/src/components/files/file-detail.tsx`
- Create: `app/src/components/assets/asset-workspace.tsx`
- Create: `app/src/lib/files/file-view-model.ts`
- Test: `app/src/lib/files/file-view-model.test.ts`
- Create: `app/src/routes/files/index.tsx`
- Create: `app/src/routes/assets/index.tsx`
- Modify: `app/src/styles.css`

**Interfaces:**
- Produces public library views plus owner upload/mutation controls when `/api/auth/session` reports owner authenticated.

- [ ] **Step 1: Write failing view-model tests**

Cover unified GitHub/R2 rows, role grouping, asset/media detection, project/game/mod filters, pagination cursors, version grouping, and storage-backend badges.

- [ ] **Step 2: Verify RED**

- [ ] **Step 3: Build public file/asset browsing**

Files view: list/grid, owner entity, role/type, tags, version, backend, date, size, search, recent uploads, stable public links.

Assets view: thumbnail-first grid for cover/screenshot/UI/texture/model/audio/video roles with lazy loading and preview metadata.

- [ ] **Step 4: Build owner upload UX**

Multi-file input and drag/drop; Android native file picker; project/game/mod target selector; role/version/tags/notes; automatic backend choice with Advanced override; upload queue with queued/uploading/verifying/failed/complete statuses; retry/abort; no accidental tap while scrolling.

- [ ] **Step 5: Add owner file actions**

Rename, Replace, Upload New Version, Delete, Reassign, Use as Cover/role changes where applicable. Public visitors never render these controls.

- [ ] **Step 6: Verify focused browser behavior at 1440×900 and 390×844**

Use mocked/local server bindings where appropriate; assert no horizontal overflow and multi-file queue state is keyboard/touch accessible.

- [ ] **Step 7: Commit**

Commit message: `feat: add Catelog files and assets workspaces`

---

### Task 9: Add mod releases, Versions, Files, and Downloads integration

**Files:**
- Create: `app/src/lib/releases/release-repository.server.ts`
- Create: `app/src/lib/releases/release-service.server.ts`
- Test: `app/src/lib/releases/release-service.test.ts`
- Create: `app/src/components/mods/mod-files-section.tsx`
- Create: `app/src/components/mods/mod-versions-section.tsx`
- Create: `app/src/components/mods/mod-downloads-section.tsx`
- Modify: `app/src/components/mods/mod-page.tsx`
- Create: `app/src/routes/api/releases/index.ts`
- Create: `app/src/routes/api/releases/$releaseId.ts`

**Interfaces:**
- Produces: `listModReleases`, `publishModRelease`, `updateModRelease`, `setLatestRelease`.

- [ ] **Step 1: Write failing release tests**

Cover owner-only publication, unique mod/version, multiple downloadable files, source files, compatibility, changes, installation notes, one latest release per mod, and preservation of old releases.

- [ ] **Step 2: Verify RED**

- [ ] **Step 3: Implement release repository/service and APIs**

A release may reference only files owned by the same mod. Publishing a new latest release clears previous `is_latest` in one transaction.

- [ ] **Step 4: Replace Plan 1 placeholder mod sections with real operational data**

Files shows unified file records; Versions shows release history; Downloads shows latest plus older releases.

- [ ] **Step 5: Verify GREEN and mod-route integration**

- [ ] **Step 6: Commit**

Commit message: `feat: add Catelog mod releases and downloads`

---

### Task 10: Migrate the existing runtime editor from password mode to GitHub owner session

**Files:**
- Modify: `app/src/components/catalog/runtime-editor.tsx`
- Modify: `app/src/routes/api/owner/project.ts`
- Modify: `app/src/lib/owner/github-catalog-write.server.ts`
- Modify/Delete old password tests under: `app/src/lib/owner/owner-auth.test.ts`
- Add tests: `app/src/lib/owner/github-catalog-write.test.ts`

**Interfaces:**
- Consumes: Task 2 `requireOwner()` and stored GitHub credential.

- [ ] **Step 1: Write failing migration tests**

Assert runtime editor mutation is unavailable logged out, works only for owner, uses the stored OAuth credential, remains hard-scoped to `data/project-details.json`, and preserves SHA conflict handling.

- [ ] **Step 2: Verify RED**

- [ ] **Step 3: Remove password UI/route behavior and wire GitHub owner session**

When OAuth secrets are not configured, mutation UI stays hidden and public browsing remains functional.

- [ ] **Step 4: Verify GREEN**

- [ ] **Step 5: Commit**

Commit message: `refactor: use GitHub owner auth for catalog editing`

---

### Task 11: Full hybrid-storage/auth verification and deploy

**Files:**
- Modify only for TDD-backed verification fixes.

- [ ] **Step 1: Run Catelog-focused unit tests**

Run: `cd app && bun test src/lib/auth src/lib/files src/lib/releases src/lib/mods src/lib/catalog src/lib/owner src/components/files src/components/assets src/components/mods src/lib/security-headers.test.ts`

Expected: zero failures.

- [ ] **Step 2: Run typecheck, targeted ESLint, and production build**

Expected: zero type errors, zero lint errors in changed Catelog files, build exit 0.

- [ ] **Step 3: Verify public/read-only mode with OAuth secrets absent**

Public pages and downloads must render; owner mutation APIs return configured/read-only states rather than exposing controls.

- [ ] **Step 4: Deploy additive D1/R2 migration**

Push and deploy. Confirm deployment applies the migration and the website reports D1/R2 capabilities. Do not run destructive SQL.

- [ ] **Step 5: Configuration handoff for GitHub OAuth secrets**

If any required secret name is absent, stop before claiming live owner upload success. Tell the user to configure the five secret values in Higgsfield website settings; never request the values in chat.

- [ ] **Step 6: After secret names are present, redeploy and perform owner smoke verification**

Verify GitHub sign-in, owner session, one small GitHub-backed text upload, one R2-backed public binary/media upload, public logged-out download, rename stable URL, Replace, New Version, release publication, and logout. Use dedicated temporary test names; clean up only through the tested owner UI/API and only after explicit permission if cleanup would delete public data.

- [ ] **Step 7: Browser verify desktop and Android upload flows**

At 1440×900 and 390×844 verify upload queue/progress, file filters, asset previews, mod Files/Versions/Downloads, owner controls hidden logged out, and no horizontal overflow.

- [ ] **Step 8: Commit verification fixes, push, and deploy once more if code changed**

Do not publish to the Higgsfield community feed unless separately requested.
