# Catelog Workspace Upgrade Plan Index

This index coordinates the implementation of the approved design in `docs/superpowers/specs/2026-09-27-catelog-mods-files-architecture-design.md`.

The spec was intentionally split into four implementation plans because it contains independently testable subsystems with different risk profiles. Execute them in this order:

1. `2026-09-27-catelog-mod-hubs-navigation-implementation.md`
2. `2026-09-27-catelog-hybrid-files-github-auth-implementation.md`
3. `2026-09-27-catelog-canonical-owner-editor-implementation.md`
4. `2026-09-27-catelog-workspace-screens-implementation.md`

## Cross-plan interface contracts

- Plan 1 owns `GameHubRecord`, `ModRecord`, the mod catalog loader, mod routes, and mod AI/Markdown/JSON serialization.
- Plan 2 owns `requireOwner()`, stored GitHub OAuth credentials, `CatalogFile`, stable public file URLs, upload/version/release services, and Files/Assets routes.
- Plan 3 in the execution order (canonical owner editor) owns safe writes to fixed GitHub canonical JSON targets and create/edit Project/GameHub/Mod/category flows.
- Plan 4 in the execution order (workspace screens) owns D1 roadmap/issues/activity/settings records, global search/aggregates, final Dashboard, all remaining sidebar routes, AI Handoff aggregation, and final sidebar wiring.
- No later plan may redefine an earlier plan’s public interface without updating its tests and dependent plan references.

## Spec coverage check

- Sidebar/navigation/screens: workspace-screens plan.
- Dashboard and Projects browser: workspace-screens plan.
- Game hubs and individual mod tabs/routes: mod-hubs plan.
- Historical mod migration and Unsorted behavior: mod-hubs plan.
- Public hybrid GitHub/R2 files: hybrid-files plan.
- D1/R2 infrastructure: hybrid-files plan.
- Android/multi-file/resumable upload: hybrid-files plan.
- Replace vs New Version: hybrid-files plan.
- Releases and preserved older downloads: hybrid-files plan.
- GitHub OAuth owner auth: hybrid-files plan.
- Safe create/edit Project/GameHub/Mod/category controls: canonical-owner-editor plan.
- Use as Cover and canonical asset metadata: canonical-owner-editor plan.
- Roadmap: workspace-screens plan.
- Issues: workspace-screens plan.
- Docs: workspace-screens plan.
- Changelog/activity: workspace-screens plan, with hooks into successful hybrid-file/editor mutations.
- AI Handoff: mod-hubs plan for canonical game/mod routes; workspace-screens plan for global selection and operational file/release/issue enrichment.
- Settings: workspace-screens plan, consuming auth/storage/editor services from earlier plans.
- Global search: workspace-screens plan.
- Public sitemap for game/mod pages: canonical-owner-editor plan.
- Performance/pagination/accessibility/mobile verification: owned by each plan’s release gate and final workspace gate.
- Existing project-route backward compatibility: regression gates in all relevant plans.

## Security/configuration gates

Implementation can build and deploy public/read-only functionality without OAuth secret values. Live owner sign-in/upload success requires the following server secret names to be configured through Higgsfield website settings:

- `GITHUB_OAUTH_CLIENT_ID`
- `GITHUB_OAUTH_CLIENT_SECRET`
- `CATELOG_OWNER_GITHUB_ID`
- `CATELOG_SESSION_SECRET`
- `CATELOG_GITHUB_TOKEN_KEY`

The GitHub OAuth application callback must point to the deployed Catelog callback route:

`https://catelog-workspace.higgsfield.app/api/auth/github/callback`

Secret values must never be requested in chat, committed to Git, returned from Settings, or stored in browser storage.

## Final release gate

After all four plans are complete, rerun the final Catelog-focused unit suite, TypeScript, changed-file ESLint, production build, and Chromium at 1440×900 and 390×844. Verify every sidebar route, one Project, one GameHub with multiple mods, one Mod’s eight inner sections, public Files/Assets, public download, logged-out owner-control hiding, global search, Roadmap, Issues, Docs, Changelog, AI Handoff, and Settings. When OAuth is configured, additionally verify GitHub owner login, GitHub-backed text upload, R2-backed binary/media upload, Replace, New Version, release publication, owner catalog editing, and logout.

## Self-review result

The four-plan decomposition covers every acceptance criterion in the approved spec. No TODO/TBD placeholders are intentionally left in the plans. The only external dependency that cannot be completed from source code alone is owner-provided GitHub OAuth application/secret configuration; public/read-only Catelog remains functional when that configuration is absent.
