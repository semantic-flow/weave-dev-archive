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

The current `prepare gh-pages` shape bundles too many responsibilities: detached/root bootstrap, publication-root checks, GitHub Pages control files, source binding, provenance updates, local path leakage validation, optional source copying, and optional commits.

Those responsibilities should be dissolved into reusable pieces:

- `mesh.create` bootstraps the mesh support surface.
- `integrate` binds available source bytes to target designators and payload artifacts without moving those bytes.
- `import` copies a working file into the mesh or publication tree when the copy itself should become governed local working content.
- `weave` is the default orchestration surface for already governed targets: by default it runs the configured versioning, validation, and generation phases. Its versioning phase may append historical states; generation can also render governed artifacts that are intentionally unversioned.
- `version` is the narrower surface for explicitly appending versioned payload states.
- `validate` is the narrower surface for reporting mesh or publication problems without recording new state.
- `generate` is the narrower surface for rendering ResourcePages and other generated surfaces from the current mesh state.
- publication-host presets apply host-specific controls such as GitHub Pages `.nojekyll`.
- publication validation checks publishability without caring whether the mesh is sidecar, whole-repository, or branch-published.
- optional git output handling commits, tags, pushes, or otherwise hands the result to CI/CD.

The important asymmetry to remove is not "branch publication is bad"; it is that branch publication currently has a bespoke command while sidecar and whole-repository publication rely on the ordinary mesh operation set. The source topology should affect policy and provenance, not the Semantic Flow model.

## Discussion

### `prepare gh-pages` should be removed immediately

There should be no deprecation period or long-lived compatibility wrapper for `prepare gh-pages`. Remove the command surface and replace its useful behavior with explicit generic primitives: mesh creation, source integration, host publication profiles, publication validation, generation, and optional git output handling.

The old command shape is actively misleading because it makes branch-published meshes look semantically special. If someone tries to follow old documentation, the answer should be updated docs and examples, not a wrapper that keeps the old boundary alive.

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

The old ambiguous artifact-resolution term did too much. It could mean mutable working bytes, or it could be read as "latest known settled bytes." Those are different release semantics.

The desired distinction is:

- `Working`: resolve mutable working bytes, such as `workingLocalRelativePath`, `hasWorkingLocatedFile`, `workingAccessUrl`, `targetLocalRelativePath`, or another approved live local source.
- `LatestState`: resolve the latest settled `HistoricalState` for the target artifact across histories.
- requested `HistoricalState` or manifestation: exact settled byte identity by default.
- requested `ArtifactHistory`: latest state in that history by default.

The old explicit pinned mode is therefore unnecessary. Targeting an exact state, manifestation, commit, or digest already fixes identity. Targeting a history is bounded but not exact; by default it asks for the latest state in that requested history. The old ambiguous mode should not remain as a deprecated alias; use explicit working/latest-state/exact semantics instead.

### Versioning intent should be explicit

Supplying an `ArtifactHistory` or changing the selected history should not by itself append a new state. History selection and state creation are different acts:

- `weave set history <payload-designator-path> <history-segment>` selects the payload artifact history that the next explicit versioning operation should use.
- `weave set next-state <payload-designator-path> <state-segment>` selects the next payload state segment, such as a release segment.
- `weave version <payload-designator-path>` is the operation that actually creates a new versioned state and consumes the selected history/next-state intent.

This user-facing surface should apply to payload DigitalArtifact Knops. Supporting artifacts such as inventories, page definitions, extracted term catalogs, generated ResourcePages, manifests, and internal metadata have their own lifecycle rules; there is no good reason to let users steer those with release-oriented `set history` or `set next-state` commands.

Unversioned payloads can still be woven without creating versioned states. The `set history` and `set next-state` controls only matter when a payload is being explicitly versioned.

`currentArtifactHistory` remains the persisted default lane for payload versioning. Changing it should be a metadata/configuration update, not a hidden weave. Setting a `next-state` hint should likewise be inert until `weave version` is invoked. If a caller wants to mint a release state, the CLI/API should say that directly instead of relying on "a different ArtifactHistory was supplied" as an implicit trigger.

### `weave` should not be a fetcher

`weave` should follow governed working-byte locators only under explicit operational policy. It should not fetch source repositories, copy source files into a publication tree, update source provenance, apply host presets, or commit/push changes.

That keeps reruns explainable. If source bytes, config, and target designators have not changed, a release action should either do nothing or reproduce the same semantic current surface.

### Publication validation is generic

Use the `weave validate` namespace for validation, with explicit scopes:

- `weave validate mesh` is whole-mesh validation. It should grow over time into the comprehensive mesh integrity check, and should include retained publication-readiness checks when the mesh has a publication surface or profile.
- `weave validate publication` is publication-readiness validation only. It can stay as a narrower standalone convenience for release/page-regeneration workflows, but the main `weave` command does not need a publication-only validation option.

Publication validation checks should be justified individually rather than inherited from the old `prepare gh-pages` bundle. First-pass candidates are local path leakage, host preset validation, and commit-time dirty worktree warnings. Some may belong in `weave validate mesh`, some may belong in `weave validate publication`, and some may not be valuable enough to keep.

Dirty publication worktrees should not be a general validation concern. They should warn only when an operation is about to make an optional local commit, because that is the moment when generated output could be mixed with unrelated human changes.

Candidate check assessment:

- Local path leakage has high value. Public RDF/HTML should not accidentally expose `/home/...`, `file:///...`, or other machine-local paths. This is cheap and should probably be part of both mesh and publication validation once public surfaces exist.
- Host preset validation has high value when a publication profile is configured. For GitHub Pages that currently means checking `.nojekyll` behavior. It is cheap and directly tied to whether published identifiers dereference as expected.
- Dirty publication worktree warnings have conditional value. They are useful only when an operation requests an optional local commit; then a warning helps avoid committing unrelated human edits with generated output. They are git-specific and not semantic validity.
- Stale generated-output validation is out of scope for now. "Stale" depends on generation policy, current renderer/config, and what the caller expected. If needed later, it should be designed as generation check behavior rather than a default publication validation rule.
- Source-root/publication-root boundary validation is deferred. It is valuable as a path-resolution safety check, but it needs explicit planned read/write sets or strong operation-level path policy to avoid guessing. When revisited, it should confirm that source reads stay inside approved source/workspace roots, publication writes stay inside the publication/mesh root, and source discovery does not accidentally ingest generated publication output. Sidecar and whole-repository layouts can be valid; branch-published and separate-repository layouts need stronger guardrails because source and publication roots are often distinct worktrees.

For ordinary weaving, use `--validate-before` and `--validate-after`. Those options should run `weave validate mesh`; they should not introduce a publication-only validation mode on the `weave` command.

## Open Issues

No conceptual open issues currently block this task. Remaining choices are implementation details.

## Decisions

- Core `mesh.create` must not create `.nojekyll` implicitly or by guessing from `meshBase`.
- A user-facing create operation may apply an explicitly selected publication profile at create time.
- A user-facing create operation may default to a conservative `auto` publication profile if the resolved concrete profile is visible and overrideable.
- For now, `publicationProfile=auto` may infer GitHub Pages only from `*.github.io` mesh bases. Custom domains, CI metadata, and repository remotes are not auto-inference signals.
- The resolved concrete publication profile should be persisted in `MeshConfig`; `auto` is request-time behavior, not a persisted mesh profile.
- GitHub Pages behavior belongs to a modular publication-host preset, and that preset only creates or validates `.nojekyll` for now.
- Custom-domain host files are human-owned for now. Weave should not create, validate, or derive them from mesh base.
- `prepare gh-pages` is not a durable Semantic Flow API operation.
- `prepare gh-pages` should be removed immediately rather than deprecated or kept as a transitional wrapper.
- Validation should use scoped commands, starting with `weave validate mesh` and `weave validate publication`.
- `weave validate mesh` should include retained publication-readiness checks when a publication surface or profile exists.
- `weave validate publication` may remain as a narrower standalone convenience, but ordinary `weave` does not need a publication-only validation option.
- `weave --validate-before` and `weave --validate-after` should invoke whole-mesh validation before and after ordinary weaving.
- Stale generated-output checks are not part of publication validation for now.
- Dirty publication worktree checks should warn only when an optional local commit is requested.
- Source-root/publication-root boundary checking is deferred until operations expose clear planned read/write sets or equivalent path-policy hooks.
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
- The old ambiguous artifact-resolution mode is removed rather than retained as a legacy alias.
- `artifactResolutionMode_working` should mean "resolve mutable working/source bytes under the active operational policy."
- `artifactResolutionMode_latestState` should mean "resolve the latest settled `HistoricalState` for the target artifact across histories."
- A requested `HistoricalState`, manifestation, commit, or digest should imply exact byte identity without an additional resolution mode.
- A requested `ArtifactHistory` should imply latest state in that history unless an explicit exact state is also supplied.
- Supplying or selecting a different `ArtifactHistory` should not implicitly append a new state.
- `weave set history` should change the selected `currentArtifactHistory` for a payload artifact without versioning it.
- `weave set next-state` should store explicit next-state intent for a payload artifact without versioning it.
- `weave version` should remain the operation that actually appends a payload state and consumes any selected history/next-state intent.
- The `set history` and `set next-state` surface is for payload DigitalArtifact Knops only, not supporting artifacts.
- Unversioned payload weaving remains valid; versioning-intent controls do not decide whether an unversioned payload may be woven.
- Next-state intent belongs with current/progression metadata, not in immutable historical snapshots. Weave already has `sfcfg:hasNextStateSegmentHint` as a consumed hint; the ontology/runtime contract should be aligned before hardening the CLI.
- CI should express the intended workflow explicitly, such as regenerate, validate, release-validate, or version, instead of depending on a generic dirty-source heuristic to infer whether a semantic change was expected.
- SHACL should distinguish `Violation`, `Warning`, and `Info`: invalid graph structure is a violation; reproducibility risk is a warning; omitted but inferable defaults are informational.
- `weave` must not hide source fetch/copy/import or host publication setup.
- `sf.api` should stay an overview and link to behavior specs for detailed contracts.

## Contract Changes

- Add a new Semantic Flow behavior spec for publication source binding and host presets: [[sf.spec.2026-05-18-publication-source-binding]].
- Add config ontology vocabulary for persisted publication profiles, including `sfcfg:hasPublicationProfile`, `sfcfg:publicationProfile_none`, and `sfcfg:publicationProfile_githubPages`.
- Add or revise core ontology artifact-resolution mode vocabulary for working-source resolution and latest-settled-state resolution.
- Remove the old ambiguous artifact-resolution mode before the API hardens so it no longer conflates working bytes with latest settled state.
- Remove the old explicit pinned resolution mode: exact target coordinates fix identity by default, so no mode is required to make a requested state, manifestation, commit, or digest exact.
- Add or clarify payload-versioning vocabulary and API affordances for selected history and explicit next-state intent, distinct from source resolution modes.
- Keep user-directed `set history` / `set next-state` behavior scoped to payload DigitalArtifact Knops; support artifacts should reject those commands or remain unreachable through that surface.
- Remove the `prepare gh-pages` CLI/API surface and replace docs/tests with the composed operation sequence.
- Add scoped validation commands: `weave validate mesh` and `weave validate publication`.
- Add `weave --validate-before` and `weave --validate-after`, both invoking the `weave validate mesh` routine.
- Align the existing `sfcfg:hasNextStateSegmentHint` term with current/progression metadata usage, or add a dedicated core metadata term for payload next-state intent.
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
- Add payload versioning-intent tests:
  - `weave set history` updates the selected payload history without appending a state.
  - `weave set next-state` stores the next payload state segment without appending a state.
  - `weave version` appends the state and consumes the selected history/next-state intent.
  - supplying a different history to a non-versioning operation does not create a state.
  - `set history` and `set next-state` reject or ignore supporting artifacts rather than steering inventory, ResourcePage, or internal metadata histories.
- Add `weave import` tests only for the explicit copy-into-mesh boundary:
  - same-repository source path copies exact bytes with provenance.
  - separate-repository source path copies exact bytes with provenance.
  - re-importing the same target fails closed or reports an explicit no-op instead of silently overwriting the governed copy.
- Add later update/refresh tests:
  - unchanged source/config/designators produce a no-op or identical output.
  - changed source digest produces a new governed current surface only through an explicit update/refresh path.
- Add publication validation tests:
  - `weave validate publication` runs the retained publication-readiness checks.
  - `weave validate mesh` includes retained publication-readiness checks when publication is configured.
  - `weave --validate-before` runs whole-mesh validation before ordinary weaving.
  - `weave --validate-after` runs whole-mesh validation after ordinary weaving.
  - retained checks work across sidecar and branch-published layouts.
  - dirty publication worktree checks warn only when optional local commit is requested.
- Evaluate candidate publication checks before hardening them:
  - absolute local path leakage such as `/home/...` and `file:///...`.
  - host preset issues.
  - commit-time dirty publication worktree warnings.
- Add CLI/integration tests replacing `prepare gh-pages` with the composed command sequence once the primitives exist.
- Add CLI/integration tests proving `prepare gh-pages` is no longer accepted.

## Non-Goals

- Do not make `weave` perform live network fetches by default.
- Do not turn publication-host presets into ontology-specific semantics.
- Do not keep a deprecated `prepare gh-pages` compatibility wrapper.
- Do not design a general CMS, broad remote-content pipeline, or arbitrary graph-refactoring system as part of this task.
- Do not make sidecar, whole-repository, and branch-published meshes use different artifact models.

## Implementation Plan

- [x] Create/update Semantic Flow behavior specs for publication source binding, mesh create, integrate, weave, extract, and page source relationships.
- [x] Refactor [[sf.api]] into an overview that links to specs instead of restating each operation contract.
- [x] Add config ontology vocabulary for persisted publication profiles.
- [x] Update the core ontology with working/latest-state resolution mode vocabulary and clarified default exact/history semantics.
- [x] Define and implement the payload versioning-intent surface: `weave set history`, `weave set next-state`, and explicit `weave version` state creation.
- [x] Update core SHACL to add local working source-binding validation, repository-backed source-binding mode guidance, mutable-ref warnings, and warning/info severity distinctions.
- [x] Remove the old ambiguous artifact-resolution mode from ontology, SHACL, runtime support, examples, and conformance fixtures.
- [x] Omit resolution-mode triples from generated exact-state extraction-source fixtures now that exact target coordinates imply exact identity.
- [x] Introduce a publication-host preset abstraction in Weave, starting with a GitHub Pages preset for `.nojekyll`.
- [x] Persist the resolved publication profile in mesh config with `sfcfg:hasPublicationProfile`.
- [x] Remove implicit GitHub-specific static-file creation from core `mesh create`; allow explicit create-time publication profiles to call the preset where needed.
- [ ] Extend `integrate` so branch-published and separate-repository ontology sources can bind repository-backed working files with working, latest-state, or exact source policy without copying them into the publication mesh.
- [ ] Implement or finish the general `weave import` CLI surface only for one-target copy acquisition into the mesh/publication tree, with repository/ref/path/digest provenance where available.
- [ ] Defer manifest-driven or batch import until a workflow proves it is useful.
- [ ] Defer manifest-driven integrate to [[wa.task.2026.2026-05-19_1846-integrate-manifest]].
- [ ] Define the later update/refresh surface for refreshing already-imported or source-bound artifacts.
- [ ] Ensure `integrate` can bind mesh-local, policy-approved live local, and repository-backed working files without copying them.
- [x] Define the initial `weave validate publication` scope around concrete publication safety checks, not generated-output freshness.
- [ ] Defer source-root/publication-root boundary validation until path planning and policy hooks make it precise.
- [x] Add `weave validate mesh` for whole-mesh validation, with room to grow as mesh validation becomes more complete.
- [x] Add `weave validate publication` for publication-readiness validation.
- [x] Add `weave --validate-before` and `weave --validate-after`, both calling whole-mesh validation.
- [x] Align `sfcfg:hasNextStateSegmentHint` with current/progression metadata usage for payload ArtifactHistory version intent.
- [x] Remove the `prepare gh-pages` command surface.
- [x] Remove the legacy `src/runtime/deploy/gh_pages.ts` implementation island and its dedicated integration tests; retained behavior is now either covered by mesh create publication profiles or deferred to integrate/source-binding/import work.
- [x] Remove runnable `prepare gh-pages` examples from [[wu.cli-reference]], the SFLO CLI examples, and the current SFLO release runbook.
- [ ] Replace remaining branch-published conformance examples with the composed release sequence once the integrate/source-binding CLI surface exists.
- [ ] Add automated release rerun tests covering unchanged source/config/designator no-op behavior.

The branch-published conformance manifests still mention the old replay commands because the current `integrate` CLI can follow allowed local working bytes but cannot yet record repository/ref/commit source bindings without copying source files into the publication root. Rewrite those manifests with the composed sequence after that source-binding surface lands.
