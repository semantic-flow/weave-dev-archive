---
id: d96bwf77pbbj90brlwmcstx
title: 2026 05 17_2206 Prepare Symmetry
desc: ''
updated: 1779080820388
created: 1779080820388
---

## Goals

- Record the resolved symmetry decision: branch-published meshes should use the same mesh/source/publication primitives as sidecar and whole-repository meshes.
- Preserve the useful live-local versus materialized-copy analysis without keeping `prepare gh-pages` as an operation boundary.
- Point remaining follow-up work to the newer remove-prepare and integrate-manifest tasks.

## Summary

This task is now resolved by [[wa.completed.2026.2026-05-18_0627-remove-prepare]].

The earlier question was whether `prepare gh-pages` contained a genuine branch-only responsibility. The answer is no. Branch publication still has operational concerns, but they are not a separate Semantic Flow operation:

- `mesh create` bootstraps the mesh support surface, including detached publication roots.
- publication-host presets own host files such as GitHub Pages `.nojekyll`.
- `integrate` binds source-lane payload bytes to designators while leaving those bytes where they are.
- `import` is the explicit future copy-into-mesh boundary when a source file should become governed local working content.
- `weave`, `version`, `generate`, and `validate` operate over the governed mesh state and should not hide source fetch/copy/import, host setup, git commits, or pushes.

The useful conceptual split from this note remains: source topology is not the semantic boundary. The important distinction is source-resolution/materialization mode.

- A sidecar mesh may use a live local working locator such as `../ontology/example.ttl` under explicit local-path policy.
- A detached publication root may also use a live local locator when host-local policy grants a separate checkout, but public mesh facts must not expose absolute checkout paths.
- If the workflow needs a portable copy inside the publication tree, that should be an explicit `import`, not hidden preparation.
- Repository URL/ref/path/commit/digest facts are provenance and reproducibility evidence, not a reason to invent a branch-specific command.

The old `prepare gh-pages` command has therefore been removed, not deprecated. The remaining work is either already handled by generic primitives or intentionally deferred:

- first-pass publication profiles and validation are implemented in Weave and specified in [[sf.spec.2026-05-18-publication-source-binding]].
- repository-backed working source binding is implemented through `integrate`.
- manifest-driven integration is a future planning convenience in [[wa.task.2026.2026-05-18_1846-integrate-manifest]].
- first-class `weave import`, source refresh/update, optional git output handling, and CI rerun idempotency tests remain future work.

## Discussion

### What changed from the original analysis

The original note treated `prepare gh-pages` as a transitional wrapper that might disappear later. That design has moved on. The command surface is removed immediately, because keeping a wrapper would preserve the wrong mental model.

The former `prepare` responsibilities are now mapped this way:

- mesh bootstrap: `mesh create`.
- GitHub Pages `.nojekyll`: publication profile/preset.
- source binding without copying: `integrate`.
- copying bytes into the mesh/publication tree: future `weave import`.
- source refresh/update after import or existing binding: future explicit update/refresh work.
- publication readiness checks: `weave validate publication` and retained publication checks inside `weave validate mesh`.
- dirty worktree checks: future optional commit/output policy only, not general mesh validity.
- generated-output freshness: not a validation rule for now.
- source/publication boundary checking: deferred until operations expose planned read/write sets.

### Current flows

In-place sidecar flow:

- `weave mesh create --workspace . --mesh-root docs --publication-profile github-pages`
- `weave integrate ./ontology/example.ttl ontology --mesh-root docs --grant-source-directory ontology`
- `weave --mesh-root docs --target 'designatorPath=ontology,historySegment=releases,stateSegment=v0.1.0,manifestationSegment=ttl'`

Detached branch-published flow:

- create or check out the publication root separately.
- run `weave mesh create` in that publication root, with an explicit or auto-resolved publication profile.
- run `weave integrate` for each source payload, using repository-backed source metadata and local-path policy as needed.
- run `weave version`, `weave generate`, `weave validate mesh`, or `weave validate publication` according to the release phase.

That detached flow is not a special branch operation. It is the same mesh/source/publication model applied to a distinct publication worktree.

### The live-local/materialized-copy distinction

`integrate` does not copy bytes. It records a governed payload surface and a working source locator. Later `weave` may read the working file under local-path policy, including host-local policy for a separate checkout.

`import` will be the copy boundary, but it does not exist as a supported CLI/runtime surface yet. When it exists, it should copy exact bytes into the mesh/publication tree with provenance. It should not be smuggled into `weave`, and it should not be confused with `integrate`.

### Release automation consequence

Automated release workflows should be explicit about their phase:

- regenerate ResourcePages: `weave generate`, optionally preceded/followed by validation.
- release/version payload state: `weave set history`, `weave set next-state`, and `weave version`, or equivalent target flags.
- release validation: `weave validate mesh` plus SFLO-specific release validation.

CI should not infer intent from a generic dirty-source heuristic. The needed idempotency checks are release-action-specific and are tracked in [[wa.completed.2026.2026-05-18_0627-remove-prepare]].

## Open Issues

No open issues remain in this task. The remaining work is tracked elsewhere:

- remove-prepare implementation and deferred work: [[wa.completed.2026.2026-05-18_0627-remove-prepare]]
- manifest-driven integrate: [[wa.task.2026.2026-05-18_1846-integrate-manifest]]
- latest-state page-source/runtime improvement: [[wa.task.2026.2026-05-19_0022-lateststate-improvement]]

## Decisions

- `prepare gh-pages` is removed, not deprecated.
- There is no branch-only mesh operation.
- Same-repository, separate-repository, sidecar, whole-repository, and branch-published layouts share the same semantic model.
- The selected source locator and materialization mode decide behavior, not repository topology.
- Detached publication-root bootstrap is ordinary `mesh create`.
- GitHub Pages support is a publication-host preset; for now it manages `.nojekyll` only.
- Clean/dirty git handling is an optional output/commit policy concern, not mesh validity.
- Local path leakage validation is generic publication validation.
- Stale generated-output checks are deferred; freshness is not a generic publication validation rule.
- Source/publication root boundary validation is deferred until it can be checked against actual planned read/write behavior.
- `integrate` binds source-lane bytes without copying them.
- `import` is the future explicit copy-into-mesh boundary; it is not implemented yet.
- Ordinary `weave` should not fetch repositories, copy source files, update source provenance, apply host presets, or commit/push.
- Release automation should compose explicit operations instead of relying on hidden preparation.

## Contract Changes

- Superseded by [[wa.completed.2026.2026-05-18_0627-remove-prepare]] and [[sf.spec.2026-05-18-publication-source-binding]].
- The CLI reference should describe composed mesh/source/publication flows, not `prepare gh-pages`.

## Testing

- Active coverage now lives with remove-prepare implementation tests:
  - `prepare gh-pages` is rejected as an unknown command.
  - publication profiles create/validate `.nojekyll`.
  - `weave validate mesh`, `weave validate publication`, `--validate-before`, and `--validate-after` are covered.
  - repository-backed `integrate` records source provenance while leaving bytes in place.
- Future CI idempotency tests are tracked in [[wa.completed.2026.2026-05-18_0627-remove-prepare]].

## Non-Goals

- Do not reintroduce a `prepare gh-pages` compatibility wrapper.
- Do not make `weave` push publication branches.
- Do not add a hidden automatic top-level `weave` after every prepare run.
- Do not force same-branch sidecar meshes to use detached publication-root semantics.
- Do not make branch-published meshes use a different artifact model.

## Implementation Plan

- [x] Resolve the branch/sidecar asymmetry analysis in favor of removing `prepare gh-pages`.
- [x] Move host controls to publication profiles.
- [x] Move publication checks to scoped validation.
- [x] Move source-lane binding to `integrate`.
- [x] Document that `import` is the future copy boundary, not a hidden prepare/weave behavior.
- [x] Defer manifest-driven integrate to [[wa.task.2026.2026-05-18_1846-integrate-manifest]].
- [x] Defer source refresh/update, explicit import, optional git output, boundary validation, and CI idempotency tests to [[wa.completed.2026.2026-05-18_0627-remove-prepare]].
- [c] Do not add a `prepare gh-pages` deprecation path.
- [d] Add top-level dry-run/planner behavior only if a future workflow needs it.
