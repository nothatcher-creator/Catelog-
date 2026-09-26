# Catelog Runtime + Deep Project Dossiers Design

## Purpose

This upgrade turns Catelog from a project catalog into a full project operating system.

The two defining goals are:

1. Every project page should contain enough structured detail that Brad can understand, manage, resume, or hand the project to an AI without filling in missing context manually.
2. Hosted web projects should be runnable from inside Catelog, with both an embedded workspace view and a full-screen runner.

The existing project catalog, AI handoff routes, and public read-only model remain. This design extends them rather than replacing them.

## Scope

This upgrade adds four major capabilities:

- A much richer project data model.
- Deeper project detail pages with project-type-specific sections.
- A hosted-project runtime system with automatic discovery and manual override.
- A dedicated runner experience with embedded and full-screen modes.

This upgrade does not attempt to run native Android APKs, Godot desktop executables, private local development servers, or arbitrary binaries inside the browser. Those projects can still show build/download/install information, but the in-Catelog runner is only for publicly reachable web-hosted deployments that a browser can load.

## Core Product Model

Each project becomes a structured dossier rather than a single summary record.

The normal project page remains visually polished for humans. The AI page, JSON endpoint, and Markdown endpoint are generated from the same canonical project data so they cannot drift apart.

Missing information must remain explicitly unknown. Catelog must use values such as `Not documented yet` or empty arrays instead of inventing project facts.

## Deep Project Schema

The existing project fields remain valid. The project model is extended with structured groups.

### Identity

- slug
- name
- aliases
- category
- project type
- status
- favorite
- pinned
- version
- context version
- created date when known
- last updated
- short summary
- long description

### Product definition

- purpose
- target users
- problem solved
- goals
- non-goals
- success criteria
- constraints
- do-not-change rules
- terminology / glossary

### Platforms and environment

- platforms
- target devices
- operating systems
- minimum requirements
- browser requirements
- permissions
- offline support
- network requirements

### Technology

- technology stack
- frameworks
- engines
- languages
- major libraries
- services
- APIs
- third-party integrations
- package/dependency notes
- architecture summary

### Repository and source

- repository URL
- default branch
- additional repositories
- important directories
- important files
- entry points
- config files
- environment variables by name only
- build instructions
- local run instructions
- test instructions
- release/build output locations

Secrets must never be stored in Catelog project data.

### Feature model

Features become structured records rather than only strings.

Each feature may include:

- id
- name
- description
- status
- priority
- category/system
- requirements
- acceptance notes
- dependencies
- related files
- related tasks
- known issues

The UI may continue to support simpler string records during migration, but the target model is structured.

### UI and screens

For apps, games, and websites:

- screen/page name
- purpose
- route when applicable
- description
- major controls
- interactions
- navigation relationships
- responsive behavior
- implementation status
- related screenshots/assets
- related files

### Systems

Projects may define system records such as:

- authentication
- save system
- inventory
- combat
- quests
- multiplayer
- mapping
- AR scanning
- export pipeline
- audio processing
- moderation
- asset pipeline
- navigation
- persistence

Each system can include purpose, current implementation, dependencies, data flow, known limitations, and related files.

### Data model

Projects may document:

- entities/models
- major fields
- relationships
- persistence location
- API payloads
- local storage/database usage
- migrations
- sync rules

This is descriptive project documentation, not a requirement that Catelog itself becomes a database modeling tool.

### Runtime and deployments

Each project gains a runtime object.

Suggested shape:

```json
{
  "runtime": {
    "preferredUrl": "",
    "manualUrl": "",
    "autoDetect": true,
    "embedPreference": "auto",
    "preferredDeploymentId": "",
    "lastChecked": "",
    "notes": ""
  },
  "deployments": [
    {
      "id": "production",
      "label": "Production",
      "url": "https://example.com",
      "source": "manual",
      "environment": "production",
      "active": true,
      "embedStatus": "unknown",
      "lastChecked": ""
    }
  ]
}
```

`source` may be `manual`, `project-link`, `repository`, or another explicitly supported discovery source.

`embedStatus` may be:

- unknown
- embeddable
- blocked
- unreachable

The project may have multiple deployments. One is selected as the preferred runtime.

### Tasks and roadmap

- current milestone
- milestones
- next tasks
- backlog
- completed work
- in-progress work
- blockers
- priorities
- target versions

### Bugs and testing

- known bugs
- regressions
- reproduction notes
- affected platforms
- severity
- workaround
- test status
- automated test notes
- manual QA notes
- last tested build

### Assets

- cover image
- screenshots
- videos
- icons
- logos
- 3D assets
- audio assets
- reference images
- source URL or file reference
- usage notes

### Decisions and history

- architecture decisions
- design decisions
- rejected approaches
- decision date when known
- reason
- changelog entries
- release/version history

### AI-specific guidance

- canonical project instructions
- important context
- do-not-assume list
- preferred next actions
- coding conventions
- output expectations
- AI handoff notes

These fields are part of the canonical project record and appear in AI context automatically.

## Project-Type-Specific Sections

Catelog should expose a common base structure plus additional tabs depending on project type.

### Games

Possible tabs:

- Overview
- Features
- UI
- Systems
- World
- Combat
- Characters
- Items
- Quests
- Progression
- Multiplayer
- Assets
- Tasks
- Roadmap
- Bugs
- Testing
- Deployments
- Runtime
- Files
- Documentation
- History
- AI Brief

Only sections with meaningful data need to be prominent. Empty optional tabs may be hidden or grouped under More.

### Apps

Possible tabs:

- Overview
- Features
- Screens
- Navigation
- UI
- Systems
- Data
- Services
- Permissions
- Assets
- Tasks
- Roadmap
- Bugs
- Testing
- Deployments
- Runtime
- Files
- Documentation
- History
- AI Brief

### Websites

Possible tabs:

- Overview
- Pages
- Components
- UI
- Features
- APIs
- Data
- Hosting
- SEO
- Analytics
- Assets
- Tasks
- Roadmap
- Bugs
- Testing
- Deployments
- Runtime
- Files
- Documentation
- History
- AI Brief

### Mods and tools

Use the common tabs plus domain-specific sections when present. The schema should not require a unique hard-coded page for every project category.

## Project Detail Experience

The project header should become a command area rather than only a title block.

It includes:

- project artwork
- name and summary
- status
- version
- platform badges
- repository button
- preferred deployment selector when applicable
- Run Project button when a runnable deployment exists
- Share with AI button
- Edit control for owner mode

The Overview dashboard should show dense but useful project intelligence:

- completion/progress summary
- current state
- current milestone
- recently changed items
- latest bugs
- next tasks
- runtime status
- deployments
- important links
- repository/source state
- feature status
- recent history
- quick AI handoff

## Runtime Discovery

Catelog supports both automatic detection and manual override.

### Automatic discovery

Automatic discovery may inspect data already available to Catelog, including:

- project links whose labels or URLs indicate live, demo, website, preview, production, GitHub Pages, Vercel, Netlify, Cloudflare Pages, Higgsfield, Render, Railway, or similar hosting
- repository metadata when a known deployment/homepage URL is already exposed
- manually recorded deployments from previous checks

Discovery must not blindly treat the repository URL itself as a runtime URL.

Discovery results are candidates, not unquestionable truth.

### Manual override

Owner mode can:

- add a deployment URL
- remove a deployment URL
- rename a deployment
- choose production/preview/other environment
- mark the preferred deployment
- disable auto detection for a project
- force a preferred runtime URL
- set embed preference

Manual values win over automatic guesses.

### Multiple deployments

If a project has multiple valid deployments, the runner toolbar includes a selector such as:

- Production
- Preview
- GitHub Pages
- Legacy build

The preferred runtime opens by default.

## Runner Experience

### Entry points

Runnable projects expose:

- `Run Project` from the project header
- a runtime status card on Overview
- a Runtime tab
- a direct route such as `/projects/:slug/run`

### Embedded mode

Default behavior opens the preferred deployment in a large embedded browser frame inside Catelog.

Desktop runner chrome contains:

- Back to Project
- project name
- deployment selector
- reload
- open in new tab
- full-screen button
- runtime status

Below or beside the frame, Catelog may expose collapsed supporting panels for:

- project details
- deployments
- files
- AI context
- known issues

The embedded frame should occupy most of the usable viewport.

### Full-screen mode

Full-screen mode hides the normal Catelog sidebar and most workspace chrome.

A compact floating or fixed runner toolbar remains available with:

- exit full screen / return to project
- deployment selector
- reload
- open externally

The hosted project itself receives the maximum practical viewport.

### Mobile runner

On mobile:

- the project fills nearly the whole screen
- controls move into a compact top or bottom toolbar
- the deployment selector is touch friendly
- full-screen mode removes almost all Catelog chrome
- returning to Catelog must require only one tap

## Embedding Failure and Browser Security

Catelog cannot force arbitrary websites to allow framing.

A site can block embedding using `X-Frame-Options` or Content Security Policy `frame-ancestors`. Catelog must handle this as a normal state, not as a broken blank panel.

Behavior:

1. Attempt or preflight embedding where technically possible.
2. If embedding is known to be blocked, show a clear blocked state.
3. Preserve deployment details and status.
4. Provide `Open Live Project` in a new tab.
5. Do not proxy the target site merely to bypass anti-framing restrictions.

Catelog must not inject scripts into third-party hosted projects.

## Runtime Health

The runtime system may perform lightweight checks for public deployments.

Useful status values:

- Online
- Unreachable
- Embedding blocked
- Unknown

Checks should be conservative. A failed preflight from Catelog does not always mean the project is actually offline because CORS and browser policies can interfere with inspection.

The UI must distinguish `unreachable` from `could not verify`.

## AI Handoff Expansion

The AI context becomes substantially richer and is generated from the expanded dossier.

It should include, when documented:

1. Identity and current version.
2. Purpose, target users, goals, and non-goals.
3. Platforms and environment requirements.
4. Technology stack and dependencies.
5. Architecture and data flow.
6. Feature inventory with statuses.
7. UI/screens/pages.
8. Major systems.
9. Data model/API/service details.
10. Current implementation state.
11. Completed, in-progress, and next work.
12. Bugs and testing state.
13. Decisions and constraints.
14. Repository, branch, important files, and entry points.
15. Build/run/test instructions.
16. Deployment/runtime URLs.
17. Assets and related links.
18. History and versions.
19. AI-specific instructions and do-not-assume rules.
20. Context version and last updated timestamp.

The generated AI document should omit empty noise while still explicitly flagging important unknowns.

The human page, AI page, JSON output, and Markdown output continue to use the same canonical source.

## Detail Density Without Clutter

The interface should become more detailed without becoming one enormous wall of text.

Use:

- summary cards for current-state information
- collapsible sections for long technical material
- tables for structured inventories such as deployments and files
- status chips only where status matters
- nested detail drawers or panels for features/systems
- project-type-aware tabs
- compact counts in tab labels where helpful

The Overview page remains scannable. Deep detail lives one click away.

## Search Expansion

Global search should eventually index:

- project names and aliases
- features
- screens/pages
- systems
- tasks
- bugs
- files
- technologies
- deployments
- history
- notes
- terminology

Search results should identify both the project and the section that matched.

## Owner Editing

The owner editor must be expanded to edit the deeper schema without requiring raw JSON for normal use.

Recommended editor structure:

- General
- Product
- Features
- UI/Screens
- Systems
- Technical
- Repository
- Runtime/Deployments
- Tasks/Roadmap
- Bugs/Testing
- Assets
- Decisions
- AI Context

Advanced/raw JSON editing may exist as an expert option, but should not be the primary workflow.

Owner authentication and GitHub write-back must remain secure. Catelog must not expose credentials to public visitors or store secrets inside project JSON.

## Data Migration

Existing records in `data/projects.json` must continue to load during migration.

The upgrade should use backward-compatible optional fields so Catelog can progressively enrich projects rather than requiring every dossier to be completed before deployment.

Migration rules:

- preserve all existing fields
- add new fields with safe empty/default values
- convert simple lists to structured records only when needed
- never discard undocumented legacy notes
- increment context version when a project's canonical meaning changes materially

A later cleanup may split the monolithic project list into one file per project, but this upgrade does not require that refactor unless the implementation plan shows it materially simplifies editing and loading.

## Hosted Project Population

During implementation, existing projects should be checked for known public runtime URLs from their canonical links/repositories.

Do not fabricate deployments.

Projects with no confirmed public runtime remain non-runnable and show deployment information as `Not documented yet`.

Projects with a confirmed hosted web version receive a Run Project control automatically.

## Error States

The runtime system needs explicit states for:

- no deployment documented
- deployment URL invalid
- deployment unreachable
- embedding blocked
- deployment verification unknown
- frame loading
- frame loaded

The project page itself must still work even when runtime loading fails.

## Accessibility

- Runner controls must be keyboard accessible.
- Embedded frames require meaningful titles.
- Deployment state cannot rely on color alone.
- Full-screen entry/exit controls require accessible labels.
- Mobile controls require practical touch targets.
- Reduced-motion behavior remains supported.

## Security

- Validate runtime URLs and allow only `http`/`https`, preferring `https` for public projects.
- Never render `javascript:` or arbitrary executable URL schemes.
- Do not proxy third-party pages to bypass frame restrictions.
- Do not place GitHub tokens, API keys, private environment values, or other secrets into public project data.
- Treat text loaded from project data as untrusted display content and avoid raw HTML injection.
- External links use appropriate new-tab isolation where needed.

## Performance

The main dashboard must not load every project runtime iframe.

Only load a hosted project when the user opens its runner.

Project detail data may be lazy-rendered by section if the dossier becomes large.

Artwork and screenshots should use responsive loading and avoid making project pages unnecessarily heavy.

## Routing

New/expanded routes include:

- `/projects/:slug`
- `/projects/:slug/run`
- `/projects/:slug/ai`
- `/projects/:slug/context.json`
- `/projects/:slug/README.md`

The runner route should support selecting a deployment without creating insecure arbitrary-URL query behavior. Deployment IDs from canonical project data are preferred over accepting an unrestricted URL query parameter.

## Acceptance Criteria

This upgrade is complete when:

1. Existing Catelog project data still loads without manual conversion.
2. The project schema supports deep project dossiers with the groups defined above.
3. Project pages expose substantially more structured detail than the current version.
4. Game, app, website, and general projects can expose project-type-specific sections.
5. AI context includes the deeper documented data and remains derived from the same canonical source as the human page.
6. Runtime URLs can be detected automatically from supported canonical sources.
7. Brad can manually add or override deployment/runtime information.
8. Projects can store multiple deployments and select one as preferred.
9. Runnable hosted projects expose a Run Project control.
10. The runner supports embedded mode.
11. The runner supports a near-full-screen/full-screen workspace mode.
12. Mobile runner controls are usable on modern Android phones.
13. Sites that prohibit embedding fall back cleanly to Open Live Project instead of showing a broken empty frame.
14. Non-web/native projects remain fully documented even when they cannot run inside Catelog.
15. No fake project facts or fake deployment URLs are introduced.
16. Runtime errors never prevent the normal project page or AI handoff from loading.
17. Public project data contains no secrets.
18. Search and project navigation remain responsive as dossier data grows.

## Implementation Principle

Catelog should feel like a developer command center where the project page, the source-of-truth documentation, the AI handoff, and the live project itself are connected parts of one workspace.

The result should let Brad move from:

`Find project -> understand project -> inspect current state -> run hosted build -> return to tasks -> share with AI`

without leaving Catelog unless the external project explicitly prevents embedding.
