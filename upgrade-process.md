# Major Version Upgrade Process

## 1. Purpose & Scope

This document describes the end-to-end process for planning, implementing, and releasing a **major version** of `@elyra/canvas`. It serves as both a repeatable process guide and the planning input for the next major cycle.

A major release is the coordinated opportunity to remove accumulated deprecations, raise platform minimums, and make API improvements that are not backward-compatible — all delivered in one well-communicated batch rather than incrementally.

> **Note for reviewers:** Items marked `<!-- TODO: ... -->` are placeholders that require team input before this document is finalised. A full list of needed information appears in [Section 7](#7-information-needed-to-finalise-this-document).

---

## 2. What Constitutes a Major Version

Per [Semantic Versioning](https://semver.org), a major bump is required when **any** of the following occur:

- A public API is **removed** (config prop, `CanvasController` method, callback signature)
- A public API **changes behaviour** in a backward-incompatible way
- A **peer dependency minimum is raised** (e.g., dropping React 16/17, bumping Carbon)
- The **pipeline-flow JSON schema** changes in a way that breaks forward compatibility
- An **architectural change** requires consumers to alter their integration code

> **Best practice:** Collect all deprecations that have been accumulating and remove them in a single major release — not one at a time across multiple majors. Announce the deprecations in advance so consumers have at least one minor version of warning before removal.

---

## 3. Major Upgrade Flow — 7 Phases

```
Phase 1: Discovery & Scoping
        ↓
Phase 2: API Design & Contract
        ↓
Phase 3: Migration Guide (written alongside implementation)
        ↓
Phase 4: Implementation on `next` branch
        ↓
Phase 5: Alpha / Beta / RC Publishing
        ↓
Phase 6: Validation & Stabilisation
        ↓
Phase 7: GA Release, Announce & Support Transition
```

---

## 4. Known Deprecations Queued for Next Major

The following are confirmed by reading the source and docs. All of these should be removed in the next major release.

**Config props** (`canvas-config`):

| Item | Migration path |
|------|----------------|
| `enableStraightLinksAsFreeform` | Use `enableLinkMethod: "Freeform"` |
| `paletteInitialState` | Use `CanvasController.openPalette()` |
| `enableInteractionType: "Trackpad"` | Use alternative zoom configuration |

**Toolbar**:

| Item | Migration path |
|------|----------------|
| Legacy toolbar config as an array | Use the object format (already the default) |

**CanvasController methods** (`canvas-controller.js`):

| Method | Migration path |
|--------|----------------|
| `deleteObject(id, pipelineId)` | Use `deleteNode()` or `deleteComment()` as appropriate |
| `setInstanceId(instanceId)` | Pass `instanceId` via constructor: `new CanvasController(instanceId)` |
| `addCustomAttrToNodes(nodeIds, attrName, pipelineId)` | <!-- TODO: Confirm replacement --> |
| `removeCustomAttrFromNodes(nodeIds, attrName, pipelineId)` | <!-- TODO: Confirm replacement --> |
| `addCustomAttrToComments(comIds, attrName, pipelineId)` | <!-- TODO: Confirm replacement --> |
| `removeCustomAttrFromComments(comIds, attrName, pipelineId)` | <!-- TODO: Confirm replacement --> |

**Properties / UI hints** (`ui-hints`):

| Item | Migration path |
|------|----------------|
| `display_chars` number UI hint | Remove — no replacement needed |

**Exported components / APIs**:

| Item | Migration path |
|------|----------------|
| `ContextMenuWrapper` component | Carbon 11 has its own context menu component |
| `FlowValidation` API | Validation should move to back-end; remove front-end flow validation |

**Tables** (common-properties):

| Item | Migration path |
|------|----------------|
| React-virtualized table (legacy mode via `enableTanstackTable: false`) | TanStack React-Table is now the default and only supported option |

---

## 5. Phase-by-Phase Checklist

### Phase 1 — Discovery & Scoping

- [ ] Audit all `@deprecated` / `@Deprecated` annotations in source (see Section 4 for known list — verify it is complete before the cycle starts)
- [ ] Review all "will be removed in next major" notes in docs (`03.02.01-canvas-config.md`, `03.02.02-toolbar-config.md`, `03.04-canvas-controller.md`, `04.03-ui-hints.md`, `03.30.01`, `03.30.02`)
- [ ] Review open GitHub issues labelled `Major release` <!-- TODO: Confirm exact label name with Craig/Matt -->
- [ ] Identify peer dependency upgrades needed — current peers: `react`, `react-dom`, `react-intl`, `@carbon/react` (note: `redux` is a direct bundled dependency, not a peer)
- [ ] Survey SPSS Modeler and Watsonx teams for blockers and wishlist items
- [ ] Create GitHub milestone for the major version
- [ ] Agree on a written scope: which items from Section 4 are in, which are deferred

### Phase 2 — API Design & Contract

- [ ] Update TypeScript type definitions in `canvas_modules/common-canvas/types/` (`common-canvas.d.ts`, `canvas-controller.d.ts`, `common-properties.d.ts`, `common-properties-controller.d.ts`, `action.d.ts`) to reflect new API
- [ ] Define pipeline-flow schema version bump if JSON format changes — coordinate with [`elyra-ai/pipeline-schemas`](https://github.com/elyra-ai/pipeline-schemas) maintainers
- [ ] Agree on removal list vs. compatibility-shim list for items in Section 4
- [ ] Circulate API design for review with at least one downstream team before code freeze
- [ ] Document all renamed props with old-name → new-name mapping

### Phase 3 — Migration Guide

- [ ] Create a new migration guide page in `docs/pages/` (e.g. `migration-v{N}.md`) and add it to `docs/mkdocs.yml`
- [ ] Add before/after code samples for every breaking change in Section 4
- [ ] Document minimum Node.js version (currently `22.x` for the harness; `24.x` for CI) and peer dependency versions (`react`, `react-dom`, `react-intl`, `@carbon/react`)
- [ ] Link migration guide from `README.md` and the MkDocs nav in `docs/mkdocs.yml`

### Phase 4 — Implementation

- [ ] Create `next` branch from `main`; apply branch protection rules
- [ ] Remove all items listed in Section 4
- [ ] Update harness sample app to use new APIs
- [ ] Update all Jest unit tests (`npm run test:jest` from `canvas_modules/common-canvas/`)
- [ ] Update all Cypress E2E tests (`npx cypress run --headed --browser chrome` from `canvas_modules/harness/`)
- [ ] Update SASS / stylelint if Carbon version bumped (`npx stylelint '**/*.scss'` runs as part of `npm run build`)
- [ ] Continue patch releases on current major branch in parallel

### Phase 5 — Alpha / Beta / RC Publishing

- [ ] Extend `publish.yml` to publish `@next` tag from the `next` branch (currently only triggers on a GitHub Release; a pre-release tag strategy needs to be defined)
- [ ] Publish first `@next` build; notify known consumers
- [ ] Triage and resolve feedback from pre-release adopters
- [ ] Publish `@rc.1` and freeze the API surface
- [ ] Confirm migration guide is accurate against the RC build

### Phase 6 — Validation & Stabilisation

- [ ] All Jest tests passing with no skips
- [ ] All Cypress E2E tests passing
- [ ] TypeScript validation (`npm run test:typescript`) passing
- [ ] Accessibility audit (WCAG 2.1 AA) — Carbon components and canvas interactions
- [ ] Performance regression check (render times for large pipeline flows)
- [ ] Security dependency audit (`npm audit`) — zero high/critical findings
- [ ] Docs site builds cleanly — run `mike deploy --push --update-aliases v{N} latest` from the `docs/` directory (requires `pip3 install mkdocs-material mike`)
- [ ] Harness app builds and runs cleanly end-to-end — `functional_test.sh` starts the Express server and runs `npx cypress run --headed --browser chrome`

### Phase 7 — Release & Announce

- [ ] Merge `next` → `main`; create a GitHub Release tagged `v<!-- TODO: version -->.0.0` — this automatically triggers `publish.yml`
- [ ] Confirm `publish.yml` ran successfully and `@elyra/canvas@latest` on npm points to the new version
- [ ] Update the `mike deploy` command in `.github/workflows/deploy-docs.yml` (currently hardcoded to `v13 latest`) to deploy the new version label
- [ ] GitHub Release notes published (summary + link to migration guide in docs)
- [ ] Update `SECURITY.md` — set new supported version as v{N}, mark v{N-1} as unsupported (currently v13 is supported, `< 13.0` is not)
- [ ] Announce in the [elyra-ai/canvas](https://github.com/elyra-ai/canvas) GitHub repository (Releases page and Discussions)
- [ ] Archive / lock old major version branch

---

## 6. Rollback & Emergency Procedures

If a critical regression is discovered after GA:

1. **Immediate:** Re-point the npm `@latest` tag to the previous version without unpublishing:
   ```bash
   npm dist-tag add @elyra/canvas@<previous-version> latest
   ```
2. **Short-term:** Keep the previous major's branch alive for backports. Consumers can pin to it (e.g., if rolling back from v14 to v13):
   ```json
   "@elyra/canvas": "^13.0.0"
   ```
   Note: only the latest major receives security patches per `SECURITY.md`. If the old major needs a security backport, it must be explicitly agreed.
3. **Resolution:** Ship a patch release on the new major once the regression is fixed. Restore `@latest` using the same `npm dist-tag` command.
4. **Post-mortem:** Document what went wrong in <!-- TODO: Where do post-mortems live? e.g. a GitHub Discussion, Confluence page, etc. -->

---

## 7. Open Questions

These are items that need a decision from Craig, Matt, or the broader team before implementation can begin.

- Is there anything that needs to be added to the phase-by-phase checklist? (currently based on similar checklists from well-known libraries + knowledge of this codebase)
- Where do post-mortems and release retrospectives live?
- What lessons from previous major upgrades should be incorporated here?

---

## 8. Information Needed to Finalise This Document

The following items are marked as `<!-- TODO -->` in the document above. They must be resolved before this guide can be used as an active execution plan.

### Info Needed from Team

- [ ] **Deferred items** — What was considered and explicitly left out of this major?
- [ ] **History from original repo** — Are there lessons from previous major version upgrades (either in this repo or its predecessor) that should be incorporated into this process?
- [ ] **Post-mortem location** — Where do release retrospectives or post-mortems get documented? (GitHub Discussion, Confluence, etc.)
- [ ] **`@next` publish automation** — Does the existing `publish.yml` workflow need changes to support publishing the `@next` pre-release tag from the `next` branch, or will this be handled manually?

---

*This document is part of the [`@elyra/canvas`](https://github.com/elyra-ai/canvas) project.*
