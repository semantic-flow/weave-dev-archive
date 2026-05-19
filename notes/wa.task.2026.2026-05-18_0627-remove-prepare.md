---
id: k0dlgpu89rtrd5kr80kbvbg
title: 2026 05 18_0627 Remove Prepare
desc: ''
updated: 1779110867573
created: 1779110867573
---

## Goals

- Remove `prepare gh-pages` as a distinct Semantic Flow concept.
- Factor the useful parts of `prepare gh-pages` into generic operations that work for whole-repository, sidecar, branch-published, same-repository, and separate-repository topologies.
- Keep GitHub Pages behavior modular as a publication-host preset rather than baking `.nojekyll` into `mesh.create`.
- Make source binding, explicit import, and later update/refresh provenance-bearing instead of hidden side effects of `weave`.
- Clarify artifact-resolution vocabulary so source bindings can distinguish mutable working bytes from settled latest-state bytes.
- Refine API documentation so [[sf.api]] is an overview that links to behavior specs instead of repeating their contracts.
- Preserve CI/CD release ergonomics: rerunning over unchanged source bytes, unchanged config, and unchanged target designators should be a no-op or reproduce the same semantic current surface.

## Summary

The current `prepare gh-pages` shape bundles too many responsibilities: detached/root bootstrap, publication-root checks, GitHub Pages control files, source binding, provenance updates, stale-output checks, local path leakage validation, optional source copying, and optional commits.

Those responsibilities should be dissolved into reusable pieces:

- `mesh.create` bootstraps the mesh support surface.
- `integrate` binds available source bytes to target designators and payload artifacts without moving those bytes.
- `import` copies a working file into the mesh or publication tree when the copy itself should become governed local working content.
- `weave`, `version`, `validate`, and `generate` record and render the governed mesh state.
- publication-host presets apply host-specific controls such as GitHub Pages `.nojekyll`.
- publication validation checks publishability without caring whether the mesh is sidecar, whole-repository, or branch-published.
- optional git output handling commits, tags, pushes, or otherwise hands the result to CI/CD.

The important asymmetry to remove is not "branch publication is bad"; it is that branch publication currently has a bespoke command while sidecar and whole-repository publication rely on the ordinary mesh operation set. The source topology should affect policy and provenance, not the Semantic Flow model.

## Discussion

### `prepare gh-pages` should become a wrapper at most

A transitional Weave CLI may keep `prepare gh-pages` while the smaller pieces are being implemented. If it remains, it should call the same primitives that a sidecar or whole-repository publication path would call. It should not become the durable API name.

### `mesh.create` owns mesh bootstrap only

`mesh.create` should create `SemanticMesh`, `MeshMetadata`, `MeshInventory`, and any mesh config needed to describe the workspace/mesh-root relationship. It should not hide GitHub Pages behavior inside core mesh bootstrap.

GitHub Pages files are publication-host preset output. For now the GitHub Pages preset only creates or validates `.nojekyll`. GitLab Pages, Vercel, Netlify, plain static hosting, and local preview can have their own presets or no preset. A CLI or API request may still apply a selected publication profile at create time, for example `mesh.create` plus a GitHub Pages preset in one user-facing operation. The point is explicit composition, not forcing users to run a second command for ordinary publishing setup.

A default `auto` publication profile is reasonable if it is conservative and visible. For now, only a `meshBase` under `github.io` is strong enough to auto-select the GitHub Pages preset. A custom domain alone should not, because the hosting provider is not encoded in the URL; people publishing GitHub Pages behind a custom domain should select the GitHub Pages profile explicitly. CI metadata and repository remotes should not participate in `auto` inference yet. When `auto` chooses a profile, the resolved profile should be reported in the plan/result and should be overrideable with `none` or an explicit profile.

The mesh should also remember the resolved concrete profile in `MeshConfig`, using a config ontology property such as `sfcfg:hasPublicationProfile`. Persist `sfcfg:publicationProfile_githubPages` or `sfcfg:publicationProfile_none`, not request-time `auto`, so later runs do not need to re-infer host behavior from `meshBase`.

### `integrate` is the right semantic boundary for source-lane payloads

When a sidecar or branch-published ontology uses Turtle files from a source checkout, those files are payload sources being associated with Semantic Flow designators. That is `integrate`, even if the source checkout is a separate repository or a different branch from the publication mesh.

The missing piece is source availability:

- if the source bytes are already mesh-local, `integrate` can bind them directly.
- if the source bytes are policy-approved live local bytes, `integrate` can record/follow the locator under working-source policy while leaving the bytes where they are.
- if the source bytes are repository-backed, `integrate` can record/follow repository/ref/path facts with either working-source policy or exact commit/digest/state evidence.
- if the source bytes must be copied into the publication tree, that copy step is `import`, and it must record source provenance.

Same-repository and separate-repository cases should use the same model. The durable facts are repository/ref/path/digest/provenance plus either the approved working-source locator, the latest settled state target, the exact state/manifestation/commit/digest target, or the resulting governed local copy. Local checkout paths may be used as approved operational locators, but absolute host paths should not become public mesh facts.

### Source registry vocabulary is already mostly present

The minimal source-registry vocabulary does not need a new mini-model. The existing core shape is:

- `KnopSourceRegistry` owns source bindings with `hasSourceBinding`.
- each source binding is an `ArtifactResolutionTarget`.
- repository-backed bindings use `hasTargetRepositorySource` to a `RepositorySourceLocator`.
- `RepositorySourceLocator` already carries `sourceRepositoryUrl`, `sourceRepositoryRef`, optional `sourceRepositoryCommit`, and `sourceRepositoryPath`.
- `ArtifactResolutionTarget` already carries `targetLocalRelativePath`, `hasTargetArtifact`, `hasTargetLocatedFile`, `hasTargetDistribution`, `expectsContentDigest`, `hasRequestedTargetHistory`, `hasRequestedTargetState`, `hasArtifactResolutionMode`, and `hasArtifactResolutionFallbackPolicy`.

The gaps are in precision and validation:

- repository-backed source bindings should warn when no resolution mode is declared unless their target coordinates already make the default obvious.
- local working source bindings need their own SHACL shape, distinct from repository-backed bindings.
- mutable repository refs should warn unless paired with `sourceRepositoryCommit`, `expectsContentDigest`, or other durable byte evidence.
- `observedAt` should mean "these source or target bytes were observed during resolution at this time"; it should not double as the normal way to say that bytes were copied/imported into the mesh.
- true import/copy provenance should be modeled as an explicit provenance activity or a later import-specific term only if `observedAt` plus existing artifact provenance is not enough.

### Resolution modes should separate working and settled surfaces

The current `artifactResolutionMode_current` term is doing too much. It can mean mutable working bytes, or it can be read as "latest known settled bytes." Those are different release semantics.

The desired distinction is:

- `Working`: resolve mutable working bytes, such as `workingLocalRelativePath`, `hasWorkingLocatedFile`, `workingAccessUrl`, `targetLocalRelativePath`, or another approved live local source.
- `LatestState`: resolve the latest settled `HistoricalState` for the target artifact across histories.
- requested `HistoricalState` or manifestation: exact settled byte identity by default.
- requested `ArtifactHistory`: latest state in that history by default.

`Pinned` is therefore less central than earlier notes implied. Targeting an exact state, manifestation, commit, or digest already pins identity. Targeting a history is bounded but not pinned; by default it asks for the latest state in that requested history. `artifactResolutionMode_current` should be replaced or retired in favor of less ambiguous mode terms before the API hardens.

### `weave` should not be a fetcher

`weave` should follow governed working-byte locators only under explicit operational policy. It should not fetch source repositories, copy source files into a publication tree, update source provenance, apply host presets, or commit/push changes.

That keeps reruns explainable. If source bytes, config, and target designators have not changed, a release action should either do nothing or reproduce the same semantic current surface.

### Publication validation is generic

Stale generated-output checks, local path leakage validation, dirty worktree warnings, source-root/publication-root checks, and host preset validation should be usable across sidecar, whole-repository, and branch-published meshes.

Dirty publication worktrees should usually warn rather than fail by default. A human may intentionally add static files such as a favicon before committing.

## Open Issues

- Should publication validation be an explicit command, an option on `weave`, or both?
- How should CI distinguish "no semantic changes" from "semantic changes were expected but not produced"?
- When should the transitional `prepare gh-pages` wrapper be removed rather than deprecated?

## Decisions

- Core `mesh.create` must not create `.nojekyll` implicitly or by guessing from `meshBase`.
- A user-facing create operation may apply an explicitly selected publication profile at create time.
- A user-facing create operation may default to a conservative `auto` publication profile if the resolved concrete profile is visible and overrideable.
- For now, `publicationProfile=auto` may infer GitHub Pages only from `*.github.io` mesh bases. Custom domains, CI metadata, and repository remotes are not auto-inference signals.
- The resolved concrete publication profile should be persisted in `MeshConfig`; `auto` is request-time behavior, not a persisted mesh profile.
- GitHub Pages behavior belongs to a modular publication-host preset, and that preset only creates or validates `.nojekyll` for now.
- Custom-domain host files are human-owned for now. Weave should not create, validate, or derive them from mesh base.
- `prepare gh-pages` is not a durable Semantic Flow API operation.
- `integrate` remains the ordinary semantic operation for binding source-lane payload bytes to target designators, regardless of whether the source is intra-repo or inter-repo.
- Bringing a working file into the mesh/publication tree by copying it is `import`, not `integrate`.
- Sidecar and branch-published ontology source files should normally use `integrate` with working, latest-state, or exact source policy, not `import`.
- `weave import` is the possible CLI spelling for the explicit copy-into-mesh boundary, even though the general CLI surface is not implemented yet and it is not the desired path for sidecar or branch-published ontology release sources.
- Manifest-driven or batch import should be deferred until repeated real workflows need it.
- Manifest-driven integrate is a separate later feature for explicitly discovering and binding new source files that match configured directories, extensions, or globs.
- After an artifact has been imported, later source refresh is an explicit update/refresh concern rather than another first import by default.
- Source copying must be explicit, policy-visible, and provenance-bearing.
- Source registries should use the existing `KnopSourceRegistry` -> `hasSourceBinding` -> `ArtifactResolutionTarget` model with `RepositorySourceLocator` for repository-backed source evidence; no new minimal source-registry vocabulary is needed for URL/ref/commit/path/digest.
- `observedAt` records when source or target bytes were observed during resolution. It is not the default import/copy timestamp.
- A true copy into the mesh should be represented by explicit import/copy provenance if `observedAt` and artifact provenance are not specific enough.
- `artifactResolutionMode_current` should be split or replaced with explicit working-vs-settled terms.
- `artifactResolutionMode_working` should mean "resolve mutable working/source bytes under the active operational policy."
- `artifactResolutionMode_latestState` should mean "resolve the latest settled `HistoricalState` for the target artifact across histories."
- A requested `HistoricalState`, manifestation, commit, or digest should imply exact byte identity even without `artifactResolutionMode_pinned`.
- A requested `ArtifactHistory` should imply latest state in that history unless an explicit exact state is also supplied.
- SHACL should distinguish `Violation`, `Warning`, and `Info`: invalid graph structure is a violation; reproducibility risk is a warning; omitted but inferable defaults are informational.
- `weave` must not hide source fetch/copy/import or host publication setup.
- `sf.api` should stay an overview and link to behavior specs for detailed contracts.

## Contract Changes

- Add a new Semantic Flow behavior spec for publication source binding and host presets: [[sf.spec.2026-05-18-publication-source-binding]].
- Add config ontology vocabulary for persisted publication profiles, including `sfcfg:hasPublicationProfile`, `sfcfg:publicationProfile_none`, and `sfcfg:publicationProfile_githubPages`.
- Add or revise core ontology artifact-resolution mode vocabulary for working-source resolution and latest-settled-state resolution.
- Update or retire `artifactResolutionMode_current` before the API hardens so it no longer conflates working bytes with latest settled state.
- Clarify the role of `artifactResolutionMode_pinned`: exact target coordinates pin identity by default; the mode should not be required to make a requested state, manifestation, commit, or digest exact.
- Add SHACL guidance for resolution-mode severity:
  - missing mode on local or repository working-source bindings is a warning when it leaves operational behavior ambiguous.
  - missing mode on exact state/manifestation bindings can be informational because the coordinate already fixes identity.
  - mutable repository refs without commit/digest evidence should warn.
  - contradictory combinations such as working mode plus an exact requested state should warn or fail according to the final mode contract.
- Add an explicit local working source-binding SHACL shape for `KnopSourceRegistry` bindings that use `targetLocalRelativePath` without `hasTargetRepositorySource`.
- Update repository-backed source-binding SHACL so mode/digest/commit expectations reflect working-source vs exact-source semantics.
- Update [[sf.spec.2026-04-03-mesh-create]] so static host files are outside implicit core `mesh.create` behavior but may be applied by an explicitly selected create-time publication profile.
- Update [[sf.spec.2026-04-04-integrate-behavior]] so integration leaves source bytes in place, supports working, latest-state, and exact source policy, and treats copied source bytes as prior import.
- Update [[sf.spec.2026-04-03-weave-behavior]] so `weave` explicitly does not perform import, source refresh, host preset setup, or git publication.
- Update [[sf.spec.2026-04-05-extract-behavior]] so extraction assumes governed source artifacts are already available.
- Update [[sf.glossary]] to describe `ArtifactResolutionTarget` defaults for working bytes, latest state across histories, latest state in a requested history, and exact target coordinates.
- Update [[sf.api]] so it is an API overview with links to behavior specs.
- Update Weave user docs after implementation so `prepare gh-pages` examples are replaced by the composed operation sequence.

## Testing

- Add tests for the publication-host preset abstraction:
  - GitHub Pages preset creates/validates `.nojekyll`.
  - core `mesh create` does not create host-specific files without a selected preset.
  - `mesh create --publication-profile github-pages` or equivalent creates/validates the selected preset output in the same user-facing operation.
  - `mesh create --publication-profile auto` persists the resolved concrete profile in `_mesh/_config/config.ttl`.
  - `mesh create --publication-profile auto` selects GitHub Pages for `*.github.io` mesh bases and does not infer a profile from custom domains, CI metadata, or repository remotes.
  - later publication runs read the persisted profile before attempting host inference.
- Add `integrate` tests for publication source binding:
  - same-repository sidecar source path integrates without copying bytes into the mesh.
  - separate-repository source path integrates with repository/ref/path/digest provenance.
  - working source policy follows the approved local or repository locator only under explicit operational policy.
  - exact source policy records a stable state, manifestation, commit, digest, or equivalent evidence and does not drift on rerun.
  - latest-state source policy resolves the latest settled state rather than the working file.
- Add artifact-resolution mode tests:
  - `Working` resolves mutable working bytes and does not silently fall back to settled history.
  - `LatestState` resolves the latest settled state across histories without requiring the caller to know the exact state IRI.
  - a requested `ArtifactHistory` without an exact state resolves latest-in-that-history by default.
  - a requested `HistoricalState`, manifestation, commit, or digest is exact by default.
  - ambiguous omitted mode on local/repository working bindings produces the intended SHACL warning.
  - malformed or contradictory mode combinations produce the intended SHACL severity.
- Add `weave import` tests only for the explicit copy-into-mesh boundary:
  - same-repository source path copies exact bytes with provenance.
  - separate-repository source path copies exact bytes with provenance.
  - re-importing the same target fails closed or reports an explicit no-op instead of silently overwriting the governed copy.
- Add later update/refresh tests:
  - unchanged source/config/designators produce a no-op or identical output.
  - changed source digest produces a new governed current surface only through an explicit update/refresh path.
- Add publication validation tests:
  - absolute local paths such as `/home/...` and `file:///...` do not leak into public RDF/HTML.
  - stale generated output is detected generically.
  - dirty publication worktree warns by default.
  - source root and publication/mesh root boundary checks work for sidecar and branch-published layouts.
- Add CLI/integration tests replacing `prepare gh-pages` with the composed command sequence once the primitives exist.

## Non-Goals

- Do not make `weave` perform live network fetches by default.
- Do not turn publication-host presets into ontology-specific semantics.
- Do not require immediate removal of a transitional `prepare gh-pages` wrapper before equivalent primitives exist.
- Do not design a general CMS, broad remote-content pipeline, or arbitrary graph-refactoring system as part of this task.
- Do not make sidecar, whole-repository, and branch-published meshes use different artifact models.

## Implementation Plan

- [x] Create/update Semantic Flow behavior specs for publication source binding, mesh create, integrate, weave, extract, and page source relationships.
- [x] Refactor [[sf.api]] into an overview that links to specs instead of restating each operation contract.
- [x] Add config ontology vocabulary for persisted publication profiles.
- [ ] Update the core ontology with working/latest-state resolution mode vocabulary and clarified default exact/history semantics.
- [ ] Update core SHACL to add local working source-binding validation, repository-backed source-binding mode guidance, mutable-ref warnings, and warning/info severity distinctions.
- [ ] Update existing examples and conformance fixtures that currently use `artifactResolutionMode_current` or overstate `artifactResolutionMode_pinned`.
- [ ] Introduce a publication-host preset abstraction in Weave, starting with a GitHub Pages preset for `.nojekyll`.
- [ ] Persist the resolved publication profile in mesh config with `sfcfg:hasPublicationProfile`.
- [ ] Remove implicit GitHub-specific static-file creation from core `mesh create`; allow explicit create-time publication profiles to call the preset where needed.
- [ ] Extend `integrate` so branch-published and separate-repository ontology sources can bind repository-backed working files with working, latest-state, or exact source policy without copying them into the publication mesh.
- [ ] Implement or finish the general `weave import` CLI surface only for one-target copy acquisition into the mesh/publication tree, with repository/ref/path/digest provenance where available.
- [ ] Defer manifest-driven or batch import until a workflow proves it is useful.
- [ ] Defer manifest-driven integrate to [[wa.task.2026.2026-05-19_1846-integrate-manifest]].
- [ ] Define the later update/refresh surface for refreshing already-imported or source-bound artifacts.
- [ ] Ensure `integrate` can bind mesh-local, policy-approved live local, and repository-backed working files without copying them.
- [ ] Add generic publication validation for stale output, local path leakage, root-boundary checks, and dirty worktree warnings.
- [ ] Make `prepare gh-pages` a thin transitional wrapper over the generic primitives, or deprecate it once the composed command sequence is ergonomic.
- [ ] Update [[wu.cli-reference]], [[wu.repository-options]], and release-runbook examples to show the composed release sequence instead of durable `prepare gh-pages` usage.
- [ ] Add automated release rerun tests covering unchanged source/config/designator no-op behavior.
