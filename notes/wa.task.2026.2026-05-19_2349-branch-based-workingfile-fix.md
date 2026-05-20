---
id: 0rel5ok0mf2onxigtybqjb1
title: 2026 05 19_2349 Branch Based Mesh Floating Source Fix
desc: ''
updated: 1779259750960
created: 1779259750960
---

## Goals

- Make branch-published meshes portable when their current working payload files live on a different branch, checkout, or worktree than the publication mesh root.
- Stop serializing host- or topology-dependent source checkout paths such as `/home/...`, `../../home/...`, or `../sflo-source/...` into durable mesh files for branch-based mesh floating sources.
- Represent branch-based mesh floating sources with durable repository identity plus repository-relative path, then resolve that intent to local filesystem paths from the command profile or local operational config at command time.
- Keep the model repository-specific for now. A repository is a concrete durable thing to hang the floating source on; a generic filesystem root or URL root does not add enough over local path policy or direct URL resolution to earn a separate first-class locator.
- Preserve the useful distinction between floating working sources and pinned release/import sources: floating working files should not acquire branch/ref, commit, or digest constraints merely because they are repository-described.
- Keep ordinary mesh-local working files simple: files authored inside the mesh root can continue to use mesh-relative `sflo:hasWorkingLocatedFile` / `sflo:workingLocalRelativePath`.
- Update the SFLO scratch gh-pages replay documentation once the portable source locator behavior exists, rather than documenting a sibling `/tmp/sflo-source` workaround as the durable approach.

## Summary

While attempting to regenerate the SFLO `gh-pages` mesh from scratch, we found that `weave integrate` currently records external working sources as paths relative to the publication mesh root. Integrating SFLO source files from the normal source checkout into `/tmp/sflo` first produced host-local paths like `../../home/djradon/...`. Moving the source checkout to `/tmp/sflo-source` changed those values to `../sflo-source/...`, but that only made the output less ugly; it did not make the mesh portable.

For branch-published meshes, the durable model should not depend on where a human or CI job materializes the source checkout, or which branch happens to be checked out there. A `gh-pages` mesh should be able to say that `ontology` has a floating working source at `semantic-flow-core-ontology.ttl` in the SFLO repository. The runtime command profile should map that repository identity to a local checkout path when `weave`, `extract`, `validate`, or `generate` needs to read bytes.

The fix should treat branch-based mesh floating source resolution as a first-class resolution mode, not as a path-layout convention. Repository metadata can express the portable locator, but this is not the same as pinning a release to a branch ref, exact commit, or digest. Branch/ref, commit, content digest, and `expectsContentDigest` evidence should remain deliberate non-working or release/import choices.

Terminology: "branch-based mesh floating source" means a mutable working source used by a branch-published mesh. It does not mean the source binding records a durable branch name. The durable source identity is repository identity plus repository-relative path; the active branch is whatever the resolved checkout currently has.

## Discussion

### Current Behavior

`executeIntegrate` resolves a source path to an absolute path, computes `workingLocalRelativePath = relative(meshRoot, absoluteSourcePath)`, and passes that value into `planIntegrate`. The core planner renders that value onto the payload artifact as `sflo:workingLocalRelativePath` when the path is outside the mesh root, and it renders the same local path into the source registry as `sflo:targetLocalRelativePath`.

That behavior works tolerably for a sidecar source directory whose location is intentionally part of a single local workspace, but it is wrong for branch-published meshes. A `gh-pages` publication branch and a source branch are often checked out as separate worktrees, and the relative path between those worktrees is an accident of the operator's machine or CI workspace.

The current repository-backed source binding path is also too pinned for this use case. Supplying repository metadata requires a repository ref and causes the runtime to compute and serialize digest evidence. That is useful for deterministic import/release flows, but it should not be forced for a branch-based mesh floating source.

### Desired Model

A branch-published mesh needs a durable source locator like:

- repository identity, such as canonical remote URL or a configured repository id.
- repository-relative source path, such as `semantic-flow-core-ontology.ttl`.
- artifact resolution mode `working` or equivalent floating-current semantics.
- no branch/ref, commit, or digest evidence for the default working-file case.

The local command profile then resolves that portable locator into an absolute path. On a developer machine this might map SFLO to `/home/djradon/hub/semantic-flow/weave/dependencies/github.com/semantic-flow/sflo`. In CI it might map the same intent to `$GITHUB_WORKSPACE/source` or an action-created worktree. The durable mesh should be identical across both environments, even when those checkouts have different active branches.

The likely ontology shape is a new locator such as `sflo:RepositorySourceFloatingLocator`, separate from exact or branch-constrained repository source locators. It should express "use this repository's currently materialized working checkout and this repository-relative file path." It should not persist the repository checkout path; that path belongs to local command profile or host-local operational config.

### Branch Identity Is Runtime State

For a true `workingFile`, the currently checked-out branch is ambient runtime state. Persisting it into the mesh would make the source less working-like, because the mesh would now constrain which mutable line of development is acceptable. That may be useful for a later "floating named branch" mode, but it is not the default branch-based mesh floating source contract.

Validation and logging should still report the observed checkout branch and commit after local resolution. That helps humans understand what was read without converting the observation into durable source evidence.

### Branch-Published Meshes Motivate The Fix

This task is broader than "repository-backed sources" as currently implemented. The motivating pattern is a branch-published mesh where the published branch is not the source branch. Repository fields are likely the right durable vocabulary, but the feature is really "portable branch-based mesh floating source resolution."

That means the repository-relative path should work for the SFLO `gh-pages` publication branch without requiring humans to arrange sibling worktree names. It should also remain compatible with future CI that validates or regenerates `gh-pages` from whatever source branch the workflow checked out.

### Relationship To Local Path Policy

Local path policy should still guard filesystem reads, but it should not be the durable source locator for branch-based mesh floating sources. The mesh can carry portable source intent; local config grants and command profiles decide which materialized checkout is allowed to satisfy that intent on this machine.

Operationally, this probably means resolution happens in two phases:

- resolve the portable repository/path locator through profile/local config to an absolute candidate path.
- apply local path policy to that candidate path before reading it.

This keeps fail-closed behavior while removing host-specific strings from published artifacts.

### Suggested CLI Shape

Prefer an explicit `integrate` flag that says "record this local source as a floating repository source":

```sh
weave integrate "$SFLO_SRC/semantic-flow-core-ontology.ttl" ontology \
  --mesh-root "$SFLO_PUB" \
  --source-repository-current
```

`--source-repository-current` would inspect the source file's containing Git checkout, derive the repository identity from the configured remote, derive a path from the repository root such as `sflo:sourceRepositoryPathFromRoot`, and persist a floating repository locator with no branch/ref, commit, or digest evidence.

Optional override flags can cover non-standard local checkouts without making the common case noisy:

- `--source-repository-url <url>`: override or confirm the canonical durable repository identity when remote inference is ambiguous.
- `--source-repository-remote <name>`: optional helper if the checkout has multiple remotes and `origin` is not the durable identity.

Do not add a path helper in the first pass. The source file argument already gives `integrate` enough information to derive the repository-relative source path from the containing checkout. If that cannot be derived, failing closed is clearer than asking the operator to manually restate a second path that may diverge from the local source file they passed.

Do not combine `--source-repository-current` with `--source-repository-ref`, `--source-repository-commit`, or source digest flags. Those are exact or constrained repository source semantics, not branch-based mesh floating source semantics.

A profile is still useful for later resolution, but not primarily to reduce `integrate` ceremony. After the mesh stores repository identity plus repository-relative path, later `weave`, `extract`, `generate`, and validation commands need a local mapping from that repository identity to a checkout root. For a single-repository workflow that mapping can be supplied by command flags or local config; reusable profiles become more useful when several sources or CI jobs share the same checkout mapping.

The first use case is the repository that also owns the branch-published mesh, such as SFLO source plus SFLO `gh-pages`. There is still a plausible cross-repository use case: a publication mesh can aggregate or curate source payloads from sibling ontology, spec, package, or data repositories. The durable locator model should not assume the source repository and publication repository are the same; `--source-repository-current` should infer identity from the source file's checkout, not from the mesh root checkout.

### Suggested SFLO Names

Current naming proposal:

- `sflo:RepositorySourceFloatingLocator`: class for mutable current-checkout repository source locators.
- `sflo:hasRepositorySourceFloatingLocator`: object property from an `sflo:ArtifactResolutionTarget` or source binding to a floating repository locator.
- Reuse or lightly adapt `sflo:sourceRepositoryUrl` for the durable repository identity.
- Prefer a path property such as `sflo:sourceRepositoryPathFromRoot` for the repository-root-based source path, rather than reusing `sflo:sourceRepositoryPath` from the exact repository locator shape. `sflo:sourceRepositoryBasedPath` is another possible name, but `PathFromRoot` better says how to resolve the value.

The repository-relative source path should be persisted in the mesh. Operational config can map a repository identity to a checkout root, but it should not be the only place that says which file under that repository is the source. Otherwise two machines could resolve the same mesh to different source files without changing mesh state. Operational config may supply defaults or aliases for operator convenience, but the durable locator needs either a repository-relative path or an equivalent repository-internal source identifier.

### SFLO Replay Implication

The current draft update to [[wu.cli-reference.examples.sflo]] uses `/tmp/sflo-source` to avoid `/home/...` publication leakage. That should be treated as a diagnostic workaround, not as final documentation. Once this task lands, the SFLO replay should describe the source repository/path and let the command profile resolve the local checkout.

## Open Issues

- Should `--source-repository-current` derive repository identity from `origin` by default, require `--source-repository-url`, or warn when multiple remotes exist?
- What is the matching local checkout-resolution CLI/config shape for post-integrate commands: a command flag, host-local config entry, named profile, or all three?

## Decisions

- Treat `/tmp/sflo-source` adjacency as a temporary diagnostic workaround, not the durable solution.
- Do not persist branch/ref, commit, or digest pinning for ordinary branch-based mesh floating sources.
- Do not persist repository checkout paths into the mesh.
- Keep exact commit/digest import and release replay as a separate, deliberate path.
- Keep local path policy fail-closed after profile resolution.
- Prefer durable repository-relative paths over paths relative to the publication worktree.
- Prefer a distinct floating repository source locator shape over magic branch values such as `@current`.
- Keep the durable locator repository-specific for this task. A repository is concrete enough to provide durable source identity and root discovery; a generic filesystem root does not add a useful portable contract.
- Add a new SFLO shape, tentatively `sflo:RepositorySourceFloatingLocator`, rather than loosening the existing exact/repository source locator shape.
- Prefer `sflo:hasRepositorySourceFloatingLocator` as the new predicate from the current payload/source binding model to the floating repository locator, rather than relying on `sflo:workingLocalRelativePath`, `sflo:targetLocalRelativePath`, or `sflo:observedSourceLocalRelativePath` for branch-based mesh floating source resolution.
- Use `sflo:sourceRepositoryPathFromRoot` for the repository-root-based source path.
- Treat same-repository publication/source worktrees as a local checkout-resolution concern. The source is the configured source checkout, not the publication branch, and the durable locator does not need to encode that both worktrees share a remote.
- Put the portable behavior in the Semantic Flow spec/ontology layer, then reference it from Weave runtime and CLI documentation.
- Do not implement floating named branch mode in this task. A future enhancement may warn or fail if the resolved checkout is not on a named branch, but ordinary working-file behavior should not persist or enforce branch identity.
- Prefer `--source-repository-current` as the initial integrate flag for creating branch-based mesh floating sources, with optional URL/remote overrides only when inference needs help.
- Do not add a manual repository-relative path override flag in the first pass. Persist the derived repository-relative path in the mesh, and let operational config resolve repository identity to checkout root rather than replace the durable source path.

## Contract Changes

- `integrate` should be able to create a branch-based mesh floating source binding without adding `sflo:sourceRepositoryRef`, `sflo:expectsContentDigest`, `sflo:hasContentDigest`, or `sflo:sourceRepositoryCommit`.
- Branch-published meshes should not need to serialize `../some-local-worktree/...` source paths to read current working payload bytes.
- Add or adjust SFLO terms/SHACL for the floating repository locator shape and its linking predicate before relying on the behavior in published meshes.
- Branch-based mesh floating repository locators should not use `sflo:workingLocalRelativePath`, `sflo:targetLocalRelativePath`, or `sflo:observedSourceLocalRelativePath` as their durable source locator. Those predicates remain valid for mesh-local or intentionally local path-based workflows.
- Validation should distinguish portable source intent from host-local operational resolution. Durable publication files should be portable; machine-local path grants belong in local config or command profile state.
- User-facing SFLO CLI examples should change after implementation so they describe repository checkout/source resolution rather than sibling worktree layout.

## Testing

- Add focused `integrate` tests proving a branch-based mesh floating source binding records repository/path intent without branch/ref, commit, or digest evidence.
- Add runtime tests proving repository checkout/source profile resolution maps a durable repository-relative locator to an allowed absolute local file path before reading.
- Add negative tests for unresolved repository checkout profiles and denied resolved paths.
- Add publication validation coverage that catches host-local and topology-local path leakage in generated publication files.
- Add validation coverage proving branch-based mesh floating repository locators do not require or emit local path locator predicates for source resolution.
- Add or update SFLO scratch replay coverage/manual validation so regenerating `/tmp/sflo` from the selected SFLO source checkout no longer writes `/home/...`, `../../home/...`, or `../sflo-source/...` into `_mesh`, Knop inventories, source registries, extracted source registries, or generated HTML.
- Run focused tests around `src/runtime/integrate`, `src/core/integrate`, source resolution, extraction source registries, and publication validation.
- Run `deno task check` and `deno task lint` after implementation.

## Non-Goals

- Do not implement remote cloning or network fetching inside default `weave`; materialization of source checkouts remains a command-profile/CI/operator concern.
- Do not make all repository-described sources pinned by default.
- Do not hide the issue by documenting a required sibling worktree name.
- Do not copy source payload files into the publication mesh merely to avoid path resolution.
- Do not add a generic filesystem-root or URL-root floating locator in this task.
- Do not broaden this into the future task for importing historical commits into particular `HistoricalState`s.
- Do not make generic dirty-source heuristics part of default `weave`.

## Implementation Plan

- [x] Capture the current failure with a focused test or fixture: branch-published mesh root plus source checkout outside the mesh currently serializes a mesh-root-relative external path.
- [x] Specify the durable RDF shape for branch-based mesh floating source locators, likely `RepositorySourceFloatingLocator`, with repository identity plus repository-relative path only.
- [x] Specify the new predicate that connects payload/source binding state to a floating repository locator, currently proposed as `hasRepositorySourceFloatingLocator`.
- [x] Update `integrate` planning so branch-based mesh floating source bindings use the floating repository locator instead of `targetLocalRelativePath`.
- [x] Update `executeIntegrate` and CLI option/profile handling so `--source-repository-current` records repository/path intent and later commands resolve it through local checkout mapping.
- [x] Update current payload source resolution for `weave` and `extract` so floating repository locators resolve through allowed local checkouts instead of durable local paths.
- [x] Preserve existing mesh-local and sidecar-local behavior where `sflo:hasWorkingLocatedFile` or `sflo:workingLocalRelativePath` is appropriate.
- [ ] Add validation that reports durable publication path leakage clearly, with all failing files listed.
- [ ] Update [[wd.runtime]] if the runtime/config contract needs durable developer documentation beyond the task note.
- [x] Replace the `/tmp/sflo-source` workaround in [[wu.cli-reference.examples.sflo]] with the final branch-based mesh floating source workflow.
- [ ] Regenerate the scratch SFLO `gh-pages` mesh from the selected SFLO source checkout and verify no host-local or sibling-worktree paths appear.
- [x] Run focused tests, `deno task check`, and `deno task lint`.
- [ ] Provide separate commit messages for Weave code/docs and any regenerated SFLO `gh-pages` publication changes.
