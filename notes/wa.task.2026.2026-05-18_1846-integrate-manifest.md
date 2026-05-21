---
id: 5b9rmj647tzgloxp9jyrz86
title: 2026 05 18_1846 Integrate Manifest
desc: ''
updated: 1779246652822
created: 1779155218792
---

## Goals

- Sketch a later manifest-driven `integrate` workflow for discovering and binding source files in bulk and/or binding old versions to ArtifactHistory and HistoricalStates
- Keep manifest-driven integration distinct from `import`; matching files stay where they are unless a separate explicit import copies them.
- Support release workflows where new ontology, SHACL, or example files should be integrated automatically when they match configured directories, extensions, or globs.
- Preserve additive mesh behavior: unchanged bindings are no-ops, new matching files add new governed surfaces, and existing settled states are not rewritten.
- Keep core `weave` unsurprising by making any manifest-driven integrate step explicit in the workflow.

## Summary

Manifest-driven import is probably the wrong framing for the ontology release workflow. The ordinary sidecar and branch-published cases want to bind source files that remain in their source lane. That is `integrate`.

A future integrate manifest can describe source sets such as "all Turtle files under `ontology/`" or "all SHACL files under `shacl/`", derive target designators from path rules, and create source bindings for files that are not yet integrated. It is a convenience layer over repeated `integrate` calls, not a new source-copying mechanism.

This should be saved for later. The immediate remove-prepare work can proceed with explicit one-target integration and the ontology/source-binding cleanup in [[wa.completed.2026.2026-05-18_0627-remove-prepare]].

## Discussion

### Manifest-driven integrate, not import

The manifest should never imply that source bytes are copied into the mesh. It should produce an integration plan:

- discover source files from configured roots and include/exclude rules.
- derive a target designator and payload artifact for each discovered file.
- decide whether each discovered file is already integrated, newly integrable, conflicting, or excluded.
- record source bindings with the same `ArtifactResolutionTarget` and `RepositorySourceLocator` model used by one-off `integrate`.
- leave source bytes in place.

If a workflow truly needs to copy a file into the mesh, that remains `import` and should be invoked as a separate explicit operation.

### Relationship to `weave`

This can be part of a weave workflow without becoming a hidden behavior of core `weave`.

Acceptable shapes include:

- `weave integrate --manifest <path>` followed by `weave`.
- `weave --with-integrate-manifest <path>` as an explicit composed workflow.
- a CI runbook step that executes an integration plan, validates it, then weaves.

The operation result should show discovered additions, no-ops, conflicts, and skipped files before any write. A dry-run mode should be first-class.

### Manifest contents

A manifest probably needs:

- source root or roots, relative to the configured workspace or approved source boundary.
- include and exclude globs.
- accepted extensions or media types.
- target designator derivation rules, probably path-template based at first.
- payload artifact kind or default RDF document assumptions.
- source resolution policy: working source, exact commit/digest source, latest settled state, or another final mode term from the resolution-mode cleanup.
- repository metadata defaults when a source root maps to a repository checkout.
- conflict behavior for files whose derived target designator already exists with different source binding facts.
- whether new files are integrated automatically after planning or only reported.

### Additive behavior

The manifest should be append-onlyish by default:

- unchanged matched files are no-ops.
- new matched files can add new source bindings and payload surfaces.
- changed source bytes do not rewrite settled history implicitly; they flow through the normal update/version/weave path.
- missing files do not delete existing artifacts by default.
- renames are not guessed silently; they are either explicit retargeting or separate add/remove decisions.

## Open Issues

- What manifest syntax should be supported first: Turtle/RDF, YAML, JSON, or a small config-ontology shape?
- Should the manifest live in `_mesh/_config`, beside source files, or in CI-only configuration?
- How should target designator derivation handle case, spaces, extensions, nested directories, and collision avoidance?
- Should the first implementation support only one source set per run, or multiple named source sets such as ontology, shacl, and examples?
- What should the default conflict policy be when a matching file derives a target that already has a different source binding?
- How should deleted or renamed source files be reported without implying deletion of settled mesh state?

## Decisions

- This is a future feature, not part of the first remove-prepare replacement.
- The feature is manifest-driven `integrate`, not manifest-driven `import`.
- Core `weave` should not scan arbitrary directories for new source files unless the caller explicitly asks for a manifest-driven integrate workflow.
- The manifest is a planning convenience over ordinary `integrate` semantics.
- Source bindings produced by the manifest should use the same `KnopSourceRegistry`, `ArtifactResolutionTarget`, and optional `RepositorySourceLocator` model as explicit one-target integration.
- The default behavior should be additive: add new matched files, report conflicts, and leave missing/renamed/deleted files alone unless the user asks for a specific retarget/remove operation.

## Contract Changes

- Define a manifest-driven integrate operation or explicit weave workflow option after the one-target `integrate` contract is stable.
- Add source-set vocabulary or config shape if the manifest is represented in RDF.
- Document how manifest discovery maps source files to target designators, payload artifacts, and source bindings.
- Define dry-run output, conflict reporting, and no-op reporting.
- Keep import/copy semantics out of the manifest-driven integrate contract.

## Testing

- A manifest can discover new files by directory and extension and integrate them without copying bytes.
- Rerunning the same manifest over unchanged files reports no-ops.
- Adding a new matching file adds one new integration plan item and source binding.
- A file whose derived target conflicts with an existing different source binding is reported as a conflict.
- Missing files are reported without deleting settled mesh state.
- Dry-run reports the same plan without writing source bindings or payload support files.
- Repository-backed source roots can populate repository URL/ref/path and optional commit/digest evidence for each binding.

## Non-Goals

- Do not implement this as part of the immediate `prepare gh-pages` removal.
- Do not use this feature to copy source bytes into the mesh; that is import.
- Do not make core `weave` silently integrate new files from broad directory scans.
- Do not infer renames, deletes, or retargeting without an explicit user action.
- Do not design a general-purpose ETL pipeline.

## Implementation Plan

- [ ] Finish the one-target integrate/source-binding contract first.
- [ ] Decide the manifest syntax and storage location.
- [ ] Define source-set matching and target-designator derivation rules.
- [ ] Add dry-run planning output for discovered additions, no-ops, conflicts, and skipped files.
- [ ] Implement manifest-driven integrate as an explicit command or explicit weave workflow option.
- [ ] Add tests for additive reruns, new-file discovery, conflict reporting, missing-file reporting, and repository-backed source metadata.
