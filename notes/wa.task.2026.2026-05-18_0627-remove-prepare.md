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
- Keep GitHub Pages behavior modular as a publication-host preset rather than baking `.nojekyll` or `CNAME` into `mesh.create`.
- Make import/source synchronization explicit and provenance-bearing instead of a hidden side effect of `weave`.
- Refine API documentation so [[sf.api]] is an overview that links to behavior specs instead of repeating their contracts.
- Preserve CI/CD release ergonomics: rerunning over unchanged source bytes, unchanged config, and unchanged target designators should be a no-op or reproduce the same semantic current surface.

## Summary

The current `prepare gh-pages` shape bundles too many responsibilities: detached/root bootstrap, publication-root checks, GitHub Pages control files, import/source synchronization, provenance updates, stale-output checks, local path leakage validation, and optional commits.

Those responsibilities should be dissolved into reusable pieces:

- `mesh.create` bootstraps the mesh support surface.
- `import` or source synchronization makes governed local copies when source bytes need to move into the mesh or publication tree.
- `integrate` binds available bytes to target designators and payload artifacts.
- `weave`, `version`, `validate`, and `generate` record and render the governed mesh state.
- publication-host presets apply host-specific controls such as GitHub Pages `.nojekyll` and `CNAME`.
- publication validation checks publishability without caring whether the mesh is sidecar, whole-repository, or branch-published.
- optional git output handling commits, tags, pushes, or otherwise hands the result to CI/CD.

The important asymmetry to remove is not "branch publication is bad"; it is that branch publication currently has a bespoke command while sidecar and whole-repository publication rely on the ordinary mesh operation set. The source topology should affect policy and provenance, not the Semantic Flow model.

## Discussion

### `prepare gh-pages` should become a wrapper at most

A transitional Weave CLI may keep `prepare gh-pages` while the smaller pieces are being implemented. If it remains, it should call the same primitives that a sidecar or whole-repository publication path would call. It should not become the durable API name.

### `mesh.create` owns mesh bootstrap only

`mesh.create` should create `SemanticMesh`, `MeshMetadata`, `MeshInventory`, and any mesh config needed to describe the workspace/mesh-root relationship. It should not hide GitHub Pages behavior inside core mesh bootstrap.

GitHub Pages files are publication-host preset output. GitLab Pages, Vercel, Netlify, plain static hosting, and local preview can have their own presets or no preset. A CLI or API request may still apply a selected publication profile at create time, for example `mesh.create` plus a GitHub Pages preset in one user-facing operation. The point is explicit composition, not forcing users to run a second command for ordinary publishing setup.

A default `auto` publication profile is reasonable if it is conservative and visible. For example, a `meshBase` under `github.io` can strongly suggest the GitHub Pages preset; a custom domain alone should not, because the hosting provider is not encoded in the URL. When `auto` chooses a profile, the resolved profile should be reported in the plan/result and should be overrideable with `none` or an explicit profile.

The mesh should also remember the resolved concrete profile in `MeshConfig`, using a config ontology property such as `sfcfg:hasPublicationProfile`. Persist `sfcfg:publicationProfile_githubPages` or `sfcfg:publicationProfile_none`, not request-time `auto`, so later runs do not need to re-infer host behavior from `meshBase`.

### `integrate` is the right semantic boundary for source-lane payloads

When a branch-published ontology integrates Turtle files from a source checkout into a publication mesh, those files are payload sources being associated with Semantic Flow designators. That is still `integrate`, even if the source checkout is a separate repository.

The missing piece is source availability:

- if the source bytes are already mesh-local, `integrate` can bind them directly.
- if the source bytes are policy-approved live local bytes, `integrate` can record/follow the locator under policy while leaving the bytes where they are.
- if the source bytes must be copied into the publication tree, that copy step is `import` or source synchronization, and it must record source provenance.

Same-repository and separate-repository cases should use the same model. The durable facts are repository/ref/path/digest/provenance plus either the approved current-byte locator or the resulting governed local copy, not the local checkout path.

### `weave` should not be a fetcher

`weave` should follow governed current-byte locators only under explicit operational policy. It should not fetch source repositories, copy source files into a publication tree, update source provenance, apply host presets, or commit/push changes.

That keeps reruns explainable. If source bytes, config, and target designators have not changed, a release action should either do nothing or reproduce the same semantic current surface.

### Publication validation is generic

Stale generated-output checks, local path leakage validation, dirty worktree warnings, source-root/publication-root checks, and host preset validation should be usable across sidecar, whole-repository, and branch-published meshes.

Dirty publication worktrees should usually warn rather than fail by default. A human may intentionally add static files such as a favicon before committing.

## Open Issues

- What CLI spelling should replace source copying inside `prepare gh-pages`? Candidates include `weave import`, `weave source sync`, or a manifest-driven import/source-sync command.
- Is `source sync` just an implementation-friendly import profile, or should it remain a separate operational wrapper around `import`?
- What is the minimal source registry vocabulary for repository URL, ref, commit, path, digest, and copy/import time?
- Which host signals are strong enough for `publicationProfile=auto` besides `*.github.io`, and should CI/repository metadata participate?
- Does `sfcfg:hasPublicationProfile` need companion properties for CNAME/domain behavior, or can CNAME be derived from mesh base plus the selected GitHub Pages profile?
- Should publication validation be an explicit command, an option on `weave`, or both?
- How should CI distinguish "no semantic changes" from "semantic changes were expected but not produced"?
- When should the transitional `prepare gh-pages` wrapper be removed rather than deprecated?

## Decisions

- Core `mesh.create` must not create `.nojekyll` or `CNAME` implicitly or by guessing from `meshBase`.
- A user-facing create operation may apply an explicitly selected publication profile at create time.
- A user-facing create operation may default to a conservative `auto` publication profile if the resolved concrete profile is visible and overrideable.
- The resolved concrete publication profile should be persisted in `MeshConfig`; `auto` is request-time behavior, not a persisted mesh profile.
- GitHub Pages behavior belongs to a modular publication-host preset.
- `prepare gh-pages` is not a durable Semantic Flow API operation.
- `integrate` remains the ordinary semantic operation for binding source-lane payload bytes to target designators, regardless of whether the source is intra-repo or inter-repo.
- Bringing a working file into the mesh/publication tree is `import` or source synchronization, not `integrate`.
- Source copying must be explicit, policy-visible, and provenance-bearing.
- `weave` must not hide source fetch/copy/import or host publication setup.
- `sf.api` should stay an overview and link to behavior specs for detailed contracts.

## Contract Changes

- Add a new Semantic Flow behavior spec for publication/source synchronization and host presets: [[sf.spec.2026-05-18-publication-source-sync]].
- Add config ontology vocabulary for persisted publication profiles, including `sfcfg:hasPublicationProfile`, `sfcfg:publicationProfile_none`, and `sfcfg:publicationProfile_githubPages`.
- Update [[sf.spec.2026-04-03-mesh-create]] so static host files are outside implicit core `mesh.create` behavior but may be applied by an explicitly selected create-time publication profile.
- Update [[sf.spec.2026-04-04-integrate-behavior]] so integration leaves source bytes in place, while copied source bytes go through import/source synchronization.
- Update [[sf.spec.2026-04-03-weave-behavior]] so `weave` explicitly does not perform source sync, host preset setup, or git publication.
- Update [[sf.spec.2026-04-05-extract-behavior]] so extraction assumes governed source artifacts are already available.
- Update [[sf.api]] so it is an API overview with links to behavior specs.
- Update Weave user docs after implementation so `prepare gh-pages` examples are replaced by the composed operation sequence.

## Testing

- Add tests for the publication-host preset abstraction:
  - GitHub Pages preset creates/validates `.nojekyll`.
  - GitHub Pages preset creates/validates `CNAME` only when configured.
  - core `mesh create` does not create host-specific files without a selected preset.
  - `mesh create --publication-profile github-pages` or equivalent creates/validates the selected preset output in the same user-facing operation.
  - `mesh create --publication-profile auto` persists the resolved concrete profile in `_mesh/_config/config.ttl`.
  - later publication runs read the persisted profile before attempting host inference.
- Add import/source synchronization tests:
  - same-repository source path copies exact bytes with provenance.
  - separate-repository source path copies exact bytes with provenance.
  - unchanged source/config/designators produce a no-op or identical output.
  - changed source digest produces a new governed current surface only through the explicit import/source-sync path.
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

- [x] Create/update Semantic Flow behavior specs for publication/source sync, mesh create, integrate, weave, extract, and page source relationships.
- [x] Refactor [[sf.api]] into an overview that links to specs instead of restating each operation contract.
- [x] Add config ontology vocabulary for persisted publication profiles.
- [ ] Introduce a publication-host preset abstraction in Weave, starting with a GitHub Pages preset for `.nojekyll` and optional `CNAME`.
- [ ] Persist the resolved publication profile in mesh config with `sfcfg:hasPublicationProfile`.
- [ ] Remove implicit GitHub-specific static-file creation from core `mesh create`; allow explicit create-time publication profiles to call the preset where needed.
- [ ] Add import/source-sync planning for source files that must be copied into the target mesh tree, with repository/ref/path/digest provenance.
- [ ] Ensure `integrate` can bind mesh-local and policy-approved external working files without copying them.
- [ ] Add generic publication validation for stale output, local path leakage, root-boundary checks, and dirty worktree warnings.
- [ ] Make `prepare gh-pages` a thin transitional wrapper over the generic primitives, or deprecate it once the composed command sequence is ergonomic.
- [ ] Update [[wu.cli-reference]], [[wu.repository-options]], and release-runbook examples to show the composed release sequence instead of durable `prepare gh-pages` usage.
- [ ] Add automated release rerun tests covering unchanged source/config/designator no-op behavior.
