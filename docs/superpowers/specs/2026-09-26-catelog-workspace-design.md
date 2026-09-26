# Catelog Workspace Design

## Purpose

Catelog is Brad's public personal development workspace: one place to browse, organize, resume, and share every game, app, mod, experiment, website, AI tool, 3D project, and idea.

The defining requirement is that every project can be shared with an AI using one permanent link with enough canonical context that the AI can continue the project without guessing core requirements, architecture, current state, or prior decisions.

## Product Shape

Catelog should feel like a personal development operating system rather than a normal portfolio. The visual direction is inspired by the supplied dark workspace reference: a large primary workspace, grouped project cards, compact status metadata, and a persistent project/file navigator. The implementation must be original and not copy the reference interface pixel-for-pixel.

The workspace is public. Anyone may browse project information and AI handoff pages. Editing is owner-only.

## Primary Experience

### Home workspace

The home screen contains:

- Greeting header and workspace summary.
- Search across project names, descriptions, tags, technologies, and notes.
- Pinned projects.
- Recently updated projects.
- Category sections.
- Status filters.
- Favorites.
- A compact activity feed.
- Desktop project/file navigator.
- Mobile bottom navigation.

Project cards show:

- Name.
- Short description.
- Category.
- Status.
- Platform.
- Current version when applicable.
- Last updated date.
- Pinned/favorite state.
- Quick access to the project page and AI handoff link.

### Categories

Initial categories include:

- Games
- Apps & Tools
- Mods
- AI
- 3D / AR
- Websites
- Experiments / Ideas
- Archived

Categories remain data-driven so new categories can be added without changing application code.

### Project statuses

Supported statuses:

- IDEA
- PLANNING
- BUILDING
- TESTING
- WORKING
- BROKEN
- PAUSED
- FINISHED
- ARCHIVED

## Project Detail

Each project has a permanent route such as `/projects/worldforge`.

The project page contains these sections:

- Overview
- Current State
- Roadmap
- Files / Links
- AI Context
- History / Activity

Project information includes:

- Project purpose.
- Goals.
- Explicit non-goals.
- Platform and target devices.
- Technology stack.
- Architecture summary.
- UI/art direction.
- Existing features.
- Current implementation state.
- Work in progress.
- Known bugs.
- Important decisions.
- "Do not change" constraints.
- Build/run instructions when available.
- GitHub repository and live/download links.
- Next recommended tasks.
- Changelog/history.
- Screenshots or artwork when available.

## AI Handoff System

Every project exposes a canonical AI handoff route:

- `/projects/:slug/ai`

It must be intentionally dense, plain, and machine-friendly rather than decorative.

It begins with an instruction equivalent to:

> This is the canonical context for this project. Treat it as the current source of truth. Read Current State, Existing Decisions, Constraints, and Next Tasks before making changes. Do not invent missing requirements or silently replace established decisions.

The AI handoff contains:

1. Project identity.
2. Purpose.
3. Goals and non-goals.
4. Platforms.
5. Stack.
6. Architecture.
7. Terminology.
8. Current state.
9. Completed work.
10. In-progress work.
11. Known bugs.
12. Existing decisions.
13. Constraints / do-not-change rules.
14. Repository and important files.
15. Build/run instructions.
16. Links/assets.
17. Next tasks.
18. Last updated timestamp.
19. Context version.

Each project also exposes machine-friendly output derived from the same source data:

- `/projects/:slug/context.json`
- `/projects/:slug/README.md`

The application must not maintain separate hand-edited copies of the normal page and AI context. Both are rendered from the same canonical project record to prevent drift.

### Share with AI

Each project has a prominent `Share with AI` control.

It copies a short handoff prompt containing the canonical AI link and directions to read the context before continuing.

## Versioned Context

Project records include an integer context version. Important project changes increment the version.

The current routes always represent the newest canonical state. History remains visible to humans. Version-specific AI snapshots are a future-compatible extension and the data model must not prevent them.

## Owner Mode

The public site is read-only by default.

Owner Mode is unlocked only by Brad using a fine-grained GitHub personal access token with access limited to the `nothatcher-creator/Catelog-` repository.

Security requirements:

- The token must never be committed to source control.
- The token must never be embedded in shipped HTML or server-side source.
- The token is stored only on Brad's device using browser storage when he explicitly chooses to remember it.
- The UI provides a clear `Lock Owner Mode` action that removes the locally stored token.
- Public visitors never see owner controls.
- The app verifies authenticated repository access before enabling writes.
- Write operations are limited to the configured Catelog repository and the project-data paths used by the application.

Owner controls include:

- New Project
- Edit Project
- Delete / Archive Project
- Pin / Unpin
- Favorite / Unfavorite
- Change Status
- Edit Roadmap
- Add Note
- Add Link
- Add Changelog Entry
- Update AI Context fields
- Update repository/build/live links

Saving changes writes the canonical project file back to GitHub and creates a descriptive commit.

## GitHub as Canonical Data Store

The public GitHub repository `nothatcher-creator/Catelog-` is the canonical store for project information.

Recommended structure:

```text
data/
  projects/
    worldforge.json
    roomforge.json
    lyricforge.json
  workspace.json
  categories.json
```

Each project JSON record has a stable slug and all information required to render the human page, AI page, Markdown output, JSON output, and activity metadata.

`workspace.json` contains workspace-level configuration such as display name, pinned order, and global links.

`categories.json` contains category metadata.

## Initial Project Population

The first release should include initial records for major projects already discussed, including at minimum:

- Worldforge
- RoomForge / Gameify
- LyricForge
- Voidforge
- Parenting app
- Bass transcription app
- AR / 3D room scanner experiments
- Rust-like Android survival game
- Project Zomboid Android/web experiments
- Project Zomboid mods/admin tools
- Balatro mods
- Web Blender / Eevee model viewer

The initial records may be marked as incomplete where exact implementation details are unavailable. Missing facts must be represented as missing/unknown rather than invented.

## Responsive Layout

### Desktop

- Wide main content area.
- Right-side project/file navigator similar in spirit to the reference image.
- Optional narrow utility rail.
- Multiple project cards per row when space allows.

### Mobile

Designed first-class for modern Android phones, including Galaxy S25-sized screens.

- Single-column project feed.
- Bottom navigation: Home, Projects, Files, Search, Settings.
- File/project tree opens as a full-screen panel or sheet.
- Large touch targets.
- No desktop panel squeezed into the viewport.
- Share-with-AI remains one-tap accessible.

## Visual Direction

- Dark charcoal workspace.
- Soft elevated cards.
- Restrained borders and shadows.
- Purple/magenta accent family with category-specific secondary accents.
- Strong typography and generous spacing.
- Minimal decorative motion; transitions should communicate navigation/state, not distract.
- Original workspace branding rather than copying the reference mascot or exact component styling.

Working product name: `Catelog`.

## Search and Filtering

Search must match project name, description, aliases, tags, technologies, platforms, and notes.

Filters include:

- Category
- Status
- Platform
- Pinned
- Favorite

Sorting includes:

- Recently updated
- Alphabetical
- Status
- Category

## Activity and History

Each project stores changelog entries with:

- Date/time.
- Short title.
- Description.
- Optional related link/version.

The workspace can surface recent activity across projects.

## Technical Architecture

### Public website

- React 19 + TanStack Start via Higgsfield website hosting.
- Server-rendered where appropriate.
- Responsive custom UI.
- No Higgsfield image/video generation inside the product.
- Public deployment.

### Project data

- Read from the public GitHub Catelog repository.
- Project records are JSON.
- Human, AI, Markdown, and JSON views derive from the same record.

### Owner writes

- Browser-side GitHub API calls using Brad's locally stored fine-grained token.
- No token stored on the public server.
- Owner actions create/update only approved files in `data/`.

### Resilience

- If GitHub data cannot be loaded, the site shows a clear error state instead of an empty workspace.
- Invalid project records are skipped individually and surfaced as data errors rather than crashing the whole catalog.
- Unknown or missing fields display as `Not documented yet` where appropriate.

## Routing

Required public routes:

- `/`
- `/projects`
- `/projects/:slug`
- `/projects/:slug/ai`
- `/projects/:slug/context.json`
- `/projects/:slug/README.md`
- `/search`
- `/settings`

Owner editing can be implemented through modal/sheet editor flows rather than separate public routes.

## Deployment

The site is intended to be publicly accessible by link. Higgsfield provides the hosted deployment. The Catelog GitHub repository remains the project-data source of truth.

The first release does not require publishing to the Higgsfield community feed unless Brad explicitly chooses to do so.

## Acceptance Criteria

Version 1 is complete when:

1. The public workspace loads on desktop and Android-sized mobile screens.
2. Projects are read from canonical JSON data in the Catelog GitHub repository.
3. Projects can be searched, filtered, and opened.
4. At least the major known projects are represented without inventing unknown details.
5. Every project has a permanent human-readable route.
6. Every project has a permanent AI handoff route.
7. Every project exposes JSON and Markdown context derived from the same source data.
8. `Share with AI` copies a useful canonical handoff prompt.
9. Owner Mode can authenticate repository write access from Brad's device without exposing the token publicly.
10. Brad can create and edit project records from the website and save them back to GitHub.
11. The interface matches the intended dark personal-workspace feel while remaining an original design.
12. The site is deployed publicly and usable without sign-in for reading.
