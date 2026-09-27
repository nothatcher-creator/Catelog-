# Catelog Mods, Navigation, and Public Hybrid File Architecture

Date: 2026-09-27
Status: Approved design awaiting implementation plan
Repository: `nothatcher-creator/Catelog-`

## 1. Purpose

Catelog is evolving from a project dashboard into a complete personal developer workspace and public source of truth for projects, mods, files, releases, issues, history, and AI handoff context.

This upgrade has four primary goals:

1. Make every sidebar item open a real, useful workspace screen.
2. Organize mods hierarchically by game, with each mod getting its own permanent page and inner sections.
3. Add a public hybrid file system that combines GitHub-backed small files with Catelog-hosted large/binary/media files.
4. Replace password-based owner editing with GitHub sign-in, while keeping browsing and downloads public.

The design must preserve the existing Catelog project model and existing routes. The new systems must layer onto current project records without forcing a destructive migration.

## 2. Core Principles

- Public by default: all catalog pages, mod pages, files, assets, releases, and downloads are public.
- Owner-only mutation: only the approved GitHub owner may upload, edit, replace, rename, delete, recategorize, publish releases, or change settings.
- No browser-stored GitHub token: OAuth credentials and reusable tokens stay server-side only.
- Hybrid storage is transparent to users: GitHub-backed and Catelog-backed files appear in one unified library.
- No guessing: uncertain mod/game relationships go to an Unsorted Mods bucket until verified.
- Permanent AI-ready links: every project, game hub, and mod must have a canonical context link suitable for handoff to another AI.
- Preserve existing Catelog data and routes wherever possible.
- Mobile-first owner workflows must work well on modern Android phones.

## 3. Information Architecture

The sidebar becomes a real application navigation system instead of a set of page anchors.

### 3.1 Sidebar groups

```text
Dashboard

Workspace
Projects
Mods
Roadmap

Library
Assets
Files / Uploads
Docs

Tracking
Issues
Changelog

AI
AI Handoff

System
Settings
```

Useful items may show count badges, such as total projects, total mods, open issues, pending roadmap items, or recent uploads.

The same hierarchy appears in the mobile slide-out menu.

### 3.2 Dashboard

The Dashboard remains the command center and summarizes the entire workspace.

It should include:

- Pinned projects.
- Recent mod work.
- Open tasks.
- Current milestones.
- Recent uploads.
- Latest releases.
- Known issues.
- Recent changelog activity.
- Quick actions: New Project, New Mod, Upload Files, Create Release, AI Handoff.

Dashboard cards should link into the exact workspace entity they summarize.

### 3.3 Projects

Projects is the complete non-mod project browser.

Capabilities:

- Grid and list views.
- Search.
- Filters by category, status, platform, favorite, and pinned state.
- Project cards show summary, status, progress, open work, recent update, repository/runtime state, and quick actions.
- Individual mods must not be mixed into the main project list.

Existing project routes and AI handoff routes remain supported.

### 3.4 Mods

Mods is hierarchical.

```text
Mods
└── Game Hub
    ├── Mod A
    ├── Mod B
    └── Mod C
```

Examples include game hubs such as Balatro, Project Zomboid, GTA V, Fallout 4, Skyrim, Terraria, Arma 3, Garry's Mod, Minecraft, Unturned, and any other verified game relationships.

Each game hub shows:

- Game banner/cover.
- Game description/notes.
- Number of mods.
- Recent updates.
- Shared files/assets/docs.
- Ordered mod tabs.
- Game-level AI handoff.

Each individual mod gets its own permanent route:

```text
/mods/:gameSlug/:modSlug
```

Examples:

```text
/mods/balatro/deck-talents
/mods/project-zomboid/simple-admin-menu
/mods/gta-v/nothatchers-customs
```

Each mod gets inner sections:

- Overview
- Features
- Files
- Versions
- Bugs
- Changelog
- Downloads
- AI Brief

Each section must be addressable through stable navigation state and should support direct linking when practical.

### 3.5 Roadmap

Roadmap is a global planning workspace aggregating milestones and work items from projects and mods.

Views:

- Timeline.
- Kanban.
- List.

Statuses:

- Planned.
- In Progress.
- Blocked.
- Testing.
- Complete.

Filters:

- Project.
- Game hub.
- Mod.
- Status.
- Priority.

Clicking an item navigates to its source project/mod.

### 3.6 Assets

Assets is the visual/media-focused library.

It covers:

- Covers.
- Screenshots.
- UI boards.
- Logos/icons.
- Textures.
- Reference art.
- 3D models.
- Audio.
- Video.

Views:

- Grid.
- List.
- Preview/details panel.

Filters:

- Project.
- Game hub.
- Mod.
- Asset type.
- Tags.
- Upload date.

Owner actions include Use as Cover, Mark as Screenshot, Mark as Reference, Rename, Replace, Delete, and Reassign.

### 3.7 Files / Uploads

Files / Uploads is the full file manager and owner upload center.

It contains:

- Upload button.
- Drag-and-drop zone.
- Android file picker support.
- Multi-file uploads.
- Recent uploads.
- Storage usage.
- GitHub-backed files.
- Catelog-backed files.
- Type, project, game, mod, version, and tag filters.
- Version history.
- Replace file.
- Upload new version.
- Rename.
- Delete.
- Attach/detach to entities.
- Batch selection and download where feasible.

### 3.8 Docs

Docs aggregates documentation across projects, game hubs, and mods.

Examples:

- README files.
- Product specs.
- Build instructions.
- Mod installation instructions.
- Architecture notes.
- Changelogs.
- AI briefs.
- Markdown/text notes.

Docs must be globally searchable and filterable by owner entity.

### 3.9 Issues

Issues is a Catelog-native issue tracker that can optionally link to GitHub Issues.

Fields include:

- Title.
- Description.
- Status.
- Severity.
- Reproduction steps.
- Affected platform/version.
- Workaround.
- Related files.
- Screenshots.
- Project/game/mod association.
- Optional external GitHub issue URL.

Views can filter Open, In Progress, Blocked, Fixed, severity, project, game, and mod.

### 3.10 Changelog

Changelog is the global chronological activity history.

Events include:

- Project created/updated.
- Game hub created.
- Mod created/updated.
- File uploaded/replaced/deleted.
- Release published.
- Bug fixed.
- Issue created/resolved.
- Cover changed.
- Documentation updated.

It must be filterable by project, game hub, mod, event type, and date.

Entity-specific Changelog tabs display the filtered subset of this same history.

### 3.11 AI Handoff

AI Handoff is a dedicated workspace for selecting a Project, Game Hub, or Mod and viewing exactly what context an AI will receive.

It includes:

- Overview.
- Current state.
- Constraints.
- Architecture.
- Features.
- Bugs/issues.
- Next tasks.
- Relevant files.
- Runtime URLs.
- Releases/downloads.
- Owner-authored AI guidance.

Actions:

- Copy Prompt.
- Copy Context URL.
- Open JSON.
- Open Markdown.
- Open full AI brief.

Game-hub handoff may include all mods in that hub. A mod handoff must scope itself to that mod plus only genuinely shared game-level context.

### 3.12 Settings

Settings is owner-only after GitHub sign-in.

Settings include:

- GitHub connection status.
- Owner identity.
- Catalog configuration.
- Upload/storage rules.
- Allowed file types.
- Size limits.
- Default upload backend rules.
- Categories.
- Game hubs.
- Mod ordering/organization.
- Runtime settings.
- AI handoff defaults.
- Dangerous actions such as deletion/removal.

## 4. Mod Data Model

The mod system must not flatten every mod into the main project list.

### 4.1 GameHub

A `GameHub` represents one target game or modding ecosystem.

Suggested fields:

```text
slug
name
aliases[]
summary
description
cover
banner
notes[]
sharedDocs[]
sharedAssets[]
modOrder[]
history[]
aiGuidance
```

Canonical game-hub metadata is GitHub-backed.

### 4.2 ModRecord

A `ModRecord` belongs to exactly one `GameHub`.

Suggested fields:

```text
slug
gameSlug
name
aliases[]
status
summary
description
features[]
requirements[]
supportedGameVersions[]
modLoaderRequirements[]
installationInstructions[]
compatibilityNotes[]
repository
sourceReferences[]
tasks[]
bugs[]
releases[]
history[]
aiGuidance
coverKey
```

A mod record should be able to represent projects with or without a source repository.

### 4.3 Unsorted Mods

If a mod-to-game association is uncertain, Catelog must not guess.

Such entries are placed into an `unsorted` hub or equivalent review state until the owner assigns them.

## 5. Canonical GitHub Data

Existing project files remain valid.

Existing sources:

```text
data/projects.json
data/project-details.json
```

New canonical sources:

```text
data/game-hubs.json
data/mods.json
```

GitHub remains the source of truth for structured project/game/mod definitions, small documentation/code files, and portable AI context.

The website loader must tolerate missing new files during migration and preserve backward compatibility.

## 6. Hybrid File Architecture

The file system presents a single `CatalogFile` model regardless of storage backend.

### 6.1 Storage rules

Small text/code/docs default to GitHub.

Examples:

- `.md`
- `.txt`
- `.json`
- `.ts`
- `.js`
- `.lua`
- `.gd`
- configuration files

Large/binary/media/build files default to Catelog object storage.

Examples:

- `.zip`
- `.apk`
- `.exe`
- `.png`
- `.jpg`
- `.webp`
- `.mp4`
- `.wav`
- `.mp3`
- `.glb`
- `.gltf`
- `.obj`
- archives
- scans
- release builds

Catelog may use file size and MIME type in addition to extension.

Owner may override the automatic decision through an Advanced control when technically safe.

### 6.2 CatalogFile

Suggested metadata:

```text
id
displayName
originalFilename
ownerType
projectSlug
gameHubSlug
modSlug
role
mimeType
extension
size
version
tags[]
notes[]
storageBackend
storageKey
githubPath
publicUrl
checksum
uploadedAt
uploadedByGithubUserId
releaseId
isLatest
```

`ownerType` identifies whether the file belongs to a project, game hub, mod, or global workspace.

Exactly one canonical entity association should be primary. Additional references can be modeled separately if needed.

### 6.3 File roles

Supported semantic roles should include at least:

- Source
- Build
- APK
- ZIP
- Screenshot
- Cover
- UI Asset
- Texture
- 3D Model
- Audio
- Video
- Documentation
- Save/Test File
- Mod Release
- Patch
- Other

Roles drive display behavior but do not determine storage backend by themselves.

## 7. Catelog Storage Backend

The existing Higgsfield website should enable:

- R2/object storage for binary/media/build bytes.
- D1/database for operational metadata.

### 7.1 R2/object storage

R2 stores actual uploaded bytes.

Object keys should be opaque/stable and must not depend entirely on the user-facing filename.

Changing a display name must not break a permanent public file URL.

### 7.2 D1/database

D1 stores operational/live metadata such as:

- CatalogFile metadata.
- Releases.
- Issues.
- Activity events.
- Upload sessions/state.
- Owner session references where appropriate.
- Operational game/mod metadata that should not require a Git commit for every action.

Canonical descriptive project/game/mod records remain GitHub-backed.

Large files must never be stored directly inside D1 blobs.

## 8. Public File URLs

All uploaded Catelog-storage files are public.

Public file routes use stable IDs, for example:

```text
/files/:fileId/:displayFilename
```

The stable `fileId` is authoritative. The display filename is cosmetic and human-readable.

A rename may redirect or continue serving through the stable ID so existing AI/context links do not break.

Public users do not need authentication to download files.

## 9. Upload Flow

Owner upload flow:

```text
Choose/drop files
→ Select owner entity (Project / Game Hub / Mod / Workspace)
→ Catelog inspects size, extension, MIME type
→ Catelog selects GitHub or Catelog storage automatically
→ Owner selects role, version, tags, notes
→ Upload with progress
→ Verify upload/checksum
→ Persist metadata
→ Create activity event
→ Immediately show in Files/Assets/Downloads
```

### 9.1 Mobile requirements

- Must work with Android file picker.
- Must support multiple file selection.
- Must avoid accidental actions while scrolling.
- Upload progress must be visible.
- Failed uploads must be retryable.

### 9.2 Large files

Large uploads should avoid reading the entire file into browser memory.

The implementation should use direct/chunk-friendly upload behavior where supported.

Metadata must not be committed until the upload has succeeded enough to create a valid downloadable asset.

### 9.3 Replace vs new version

These are separate actions.

`Replace file` updates the current logical asset while preserving stable identity where reasonable.

`Upload new version` creates a new historical file/version record and preserves older releases.

## 10. Releases

Mods may publish releases independent of raw upload actions.

Suggested release model:

```text
id
modSlug
version
title
status
releasedAt
compatibility
changes[]
fileIds[]
sourceFileIds[]
notes[]
isLatest
```

A release page must show:

- Version.
- Date.
- Status.
- Download files.
- Changelog.
- Compatibility.
- Installation notes when available.

Older releases remain visible/downloadable.

## 11. GitHub Owner Authentication

Password-based owner mode is replaced by GitHub sign-in.

### 11.1 OAuth flow

```text
Sign in with GitHub
→ OAuth authorization
→ Server callback
→ Server retrieves GitHub identity
→ Verify stable GitHub user ID against configured owner ID
→ Create signed secure session
→ Unlock owner controls
```

### 11.2 Security requirements

- Verify stable GitHub numeric user ID, not only username.
- GitHub OAuth client secret stays server-side.
- Reusable GitHub token must never be stored in localStorage/sessionStorage.
- Owner session cookie is HttpOnly, Secure, SameSite=Strict or an equivalent safe configuration compatible with the callback flow.
- CSRF/state validation is required for OAuth callback.
- Owner mutation endpoints must verify the owner session on every request.
- Public read/download routes must not require sign-in.
- GitHub write operations must be scoped to the canonical Catelog repository/path rules rather than accepting arbitrary repository/branch/path parameters from the browser.

### 11.3 Required website secrets/settings

The deployed site will require server-side OAuth configuration values, including GitHub OAuth application credentials and the configured owner GitHub user ID.

Secret values must be configured through Higgsfield website settings, never pasted into chat or committed to source.

## 12. Mutation Permissions

Owner-only actions:

- Upload file.
- Replace file.
- Upload new version.
- Rename file.
- Delete file.
- Reassign file.
- Create project.
- Create game hub.
- Create mod.
- Edit project/game/mod metadata.
- Create release.
- Update roadmap.
- Create/resolve issue.
- Change cover.
- Edit settings.

Public actions:

- Browse.
- Search.
- Preview.
- Download.
- Open AI handoff.
- Copy public links/context.

## 13. Activity / Changelog Model

Catelog should generate structured activity entries for meaningful owner actions.

Suggested event fields:

```text
id
type
entityType
projectSlug
gameHubSlug
modSlug
fileId
releaseId
issueId
title
description
createdAt
actorGithubUserId
metadata
```

Examples:

- Uploaded `DeckTalents-v1.4.zip`.
- Published Deck Talents v1.4.
- Added a new Balatro mod.
- Fixed issue #12.
- Changed LyricForge cover.
- Updated Project Zomboid Simple Admin Menu documentation.

The global Changelog and entity-specific history views use the same event stream.

## 14. Issues Model

Catelog-native issues remain lightweight and independent of GitHub Issues.

Suggested fields:

```text
id
entityType
projectSlug
gameHubSlug
modSlug
title
description
status
severity
reproduction[]
affectedPlatforms[]
affectedVersions[]
workaround
relatedFileIds[]
screenshotFileIds[]
externalGithubIssueUrl
createdAt
updatedAt
resolvedAt
```

## 15. Roadmap Model

Global roadmap records may target projects or mods.

Suggested fields:

```text
id
entityType
projectSlug
gameHubSlug
modSlug
title
description
status
priority
targetVersion
targetDate
dependsOn[]
relatedIssueIds[]
createdAt
updatedAt
```

## 16. AI Handoff Architecture

Existing project AI context remains supported.

Add canonical routes for game hubs and mods, for example:

```text
/mods/:gameSlug/ai
/mods/:gameSlug/context.json
/mods/:gameSlug/README.md
/mods/:gameSlug/:modSlug/ai
/mods/:gameSlug/:modSlug/context.json
/mods/:gameSlug/:modSlug/README.md
```

A mod handoff includes:

- Identity.
- Parent game hub.
- Purpose/summary.
- Current state.
- Features.
- Requirements.
- Compatibility.
- Installation instructions.
- Bugs/issues.
- Next work.
- Releases.
- Latest public downloads.
- Relevant source/docs/assets.
- Constraints and AI guidance.

A game-hub handoff includes shared game context plus an index of its mods.

Public file URLs should be included when relevant so another AI can fetch artifacts directly.

## 17. Migration Strategy

Migration must be additive and non-destructive.

### 17.1 Preserve existing data

Keep:

```text
data/projects.json
data/project-details.json
```

Add:

```text
data/game-hubs.json
data/mods.json
```

The site must continue rendering even before every historical mod is migrated.

### 17.2 Historical mod migration

Known historical mod projects should be converted into individual mod records grouped by verified game hub.

No game association should be inferred when uncertain.

Unknown/ambiguous items go to Unsorted Mods until owner review.

### 17.3 Existing grouped mod records

Any existing generic/grouped mod catalog record can remain during migration, but individual mod records become the preferred detail source once created.

The implementation should avoid duplicate visible entries after migration.

## 18. Error Handling

### 18.1 Upload failures

- Failed byte upload must not create a downloadable catalog record.
- Retry should reuse the same pending upload where safe.
- Partial/abandoned objects should be cleaned up or marked for cleanup.
- UI must clearly distinguish queued, uploading, verifying, failed, and complete states.

### 18.2 GitHub write failures

- Preserve optimistic concurrency behavior.
- SHA conflicts must produce a conflict state instead of overwriting newer data.
- No arbitrary repository/path writes from client input.

### 18.3 OAuth failures

- Invalid/missing OAuth state fails closed.
- Non-owner GitHub users may sign in only if desired for diagnostics, but must never receive owner mutation permissions. Prefer immediately showing read-only state.
- Expired/revoked sessions return to public/read-only mode.

### 18.4 Missing data

- Missing optional game/mod/file metadata should render explicit unknown/not-documented states.
- Broken file references should be visible as unavailable instead of silently disappearing.

## 19. Search

Global search should index:

- Project names/aliases.
- Game hubs.
- Mod names/aliases.
- Features.
- Tasks.
- Issues.
- Files.
- File tags/roles.
- Docs.
- Releases.
- Changelog entries.
- AI guidance.

Search results should identify entity type and navigate to the exact location.

## 20. Testing Strategy

### 20.1 Unit tests

Cover:

- GameHub/Mod normalization.
- Backward-compatible project loading.
- Unsorted mod handling.
- File backend selection by type/size.
- Stable file URL construction.
- Release/version logic.
- AI context serialization.
- Search indexing.
- Sidebar/workspace routing models.
- Permission checks.
- OAuth state/session validation.

### 20.2 Integration tests

Cover:

- GitHub-backed small file registration.
- R2 upload + D1 metadata persistence.
- Upload failure rollback.
- Replace file.
- Upload new version.
- Public download.
- Release publication.
- Activity event creation.
- Mod creation under game hub.
- AI handoff includes latest relevant files/releases.

### 20.3 Browser tests

Desktop and Android-sized Chromium flows:

- Every sidebar item opens the correct workspace.
- Mods → Balatro → individual mod navigation.
- Inner mod tabs work.
- GitHub sign-in success/failure states.
- Owner-only controls hidden for public visitors.
- Upload button and drag/drop/file picker behavior.
- Multi-file upload progress.
- Files appear after upload.
- Public download works logged out.
- No horizontal overflow at common phone widths.
- Search navigates to projects/mods/files.

## 21. Performance Requirements

- Avoid loading all file metadata into the initial dashboard payload.
- Paginate or virtualize large file, asset, issue, and changelog collections.
- Thumbnail-heavy asset views should use lazy loading.
- Large upload progress should not block the rest of the UI.
- Game/mod hub navigation should not require downloading every binary/file record up front.

## 22. Accessibility Requirements

- Sidebar and mod tabs are keyboard navigable.
- Upload controls have accessible labels and status announcements.
- Progress and error states do not rely only on color.
- File actions are usable without hover.
- Mobile drawer has correct focus/close behavior.

## 23. Out of Scope for This Upgrade

- Private per-file visibility. All files are public by design.
- Multi-user collaboration/teams.
- Arbitrary third-party Git repository writes.
- Virus/malware scanning beyond basic file validation unless an external scanner is added later.
- Billing or paid downloads.
- Public user comments/ratings.

## 24. Acceptance Criteria

The upgrade is successful when all of the following are true:

1. Every sidebar item opens a real workspace screen.
2. Mods are grouped by game hub instead of flooding the Projects page.
3. Each mod has a stable permanent URL and inner tabs for Overview, Features, Files, Versions, Bugs, Changelog, Downloads, and AI Brief.
4. Historical mods are migrated into verified game hubs; uncertain ones are not guessed and remain Unsorted.
5. Public visitors can browse and download all files without signing in.
6. Owner signs in through GitHub OAuth using a verified stable GitHub user ID.
7. Owner-only controls stay hidden/inaccessible to unauthenticated/public users.
8. Upload flow works on Android and desktop, including multi-file selection and visible progress.
9. Small code/docs can be GitHub-backed while binaries/media/builds use Catelog object storage.
10. Both storage backends appear together in one Files/Assets UI.
11. Replace File and Upload New Version are distinct operations.
12. Mod releases preserve older downloadable versions.
13. Uploads and releases generate activity/changelog entries.
14. AI handoff routes for projects, game hubs, and mods include relevant public files/releases without inventing missing facts.
15. Existing Catelog project pages and project AI routes remain functional.
16. Desktop and phone layouts have no horizontal overflow in the tested primary flows.
17. Unit, integration, typecheck, lint, build, and targeted browser verification pass for the new Catelog code.

## 25. Recommended Delivery Order

Implementation should proceed in dependency order:

1. Add GameHub/Mod canonical schema and loaders.
2. Build Mods/game-hub/mod routes and navigation.
3. Convert sidebar anchors into route-based workspace navigation.
4. Enable D1/R2 infrastructure and operational data models.
5. Implement GitHub OAuth owner authentication.
6. Implement unified CatalogFile abstraction.
7. Implement upload/download/replace/version flows.
8. Build Assets and Files / Uploads workspaces.
9. Build releases, activity/changelog, issues, and roadmap workspaces.
10. Expand global search and AI handoff.
11. Migrate known historical mods.
12. Run full mobile/desktop verification and deploy.

This ordering keeps the canonical content model stable before adding owner mutation and storage features.
