---
id: d96bwf77pbbj90brlwmcstx
title: 2026 05 17_2206 Prepare Symmetry
desc: ''
updated: 1779080820388
created: 1779080820388
---

## Goals

- Clarify the relationship between primitive mesh commands (`mesh create`, `integrate`, `weave`) and `prepare gh-pages`.
- Make the current `prepare gh-pages` idempotence contract visible enough for release automation.
- Reframe future release/publication flows around source location and materialization policy, not whether source bytes happen to live in the same Git repository as the mesh.
- Aim to dissolve `prepare gh-pages` into reusable mesh/source/publication primitives unless a real branch-only responsibility remains.

## Summary

`prepare gh-pages` is currently an orchestration command for detached publication roots, not a general synonym for `mesh create + integrate + weave`. Its mesh-bootstrap part is not a different semantic operation: it already delegates to `mesh create` when the publication root has no mesh. The rest exists because branch-published worktrees need publication-root checks and controls: read bytes from a separate source checkout, write generated mesh output to a publication root, avoid local path leakage, preserve publication host files such as `.nojekyll` / `CNAME`, validate the publication worktree, and record repository source provenance in `_knop/_sources/sources.ttl`.

Today it accepts at most one `--source-path` per invocation. A detached publication-root release that needs to materialize or refresh several source payloads therefore uses one `prepare gh-pages --source-path ...` run per source payload. That does not mean every later release operation needs to prepare every already-integrated payload again: once a payload is already integrated in the publication root and its working bytes are current, ordinary mesh commands such as `weave`, `weave version`, or `weave generate` should be able to operate from the publication mesh state. `prepare gh-pages` remains the source-sync/provenance operation for copying bytes from `--source-root`, detecting source-byte changes, and updating `_knop/_sources`.

The deeper asymmetry is not only "sidecar command versus branch-published command"; it is live locator versus materialized copy. In an in-place sidecar mesh, `integrate` can record an extra-mesh `sflo:workingLocalRelativePath` such as `../ontology/example.ttl` when mesh config grants that path. Later `weave` can read the current bytes from that live locator under local-path policy. In a detached publication root, `prepare gh-pages` copies source bytes into the publication root and the integrated payload's current working file is the copied, publication-local file; the repository binding in `_knop/_sources` is provenance and source-sync input, not something ordinary `weave` currently dereferences.

That asymmetry is sometimes appropriate: a public `gh-pages` branch should not contain host-local path grants or require the source checkout to exist beside the publication checkout. But it should be a named modality, not a surprising difference between two unrelated command families. It also should not be determined by same-repository versus separate-repository source layout. An intra-repo source can still require materialization for portable publication; an inter-repo source can still be a live local checkout under policy during local work. The likely next design is shared source-sync/materialized-integrate support that targets either an in-place sidecar mesh or a detached publication root and explicitly chooses source resolution/materialization mode.

So "bootstrap detached root" should split in two. The SemanticMesh bootstrap belongs to `mesh create` regardless of whether the root is detached. Publication-root preparation is a wrapper concern: root distinctness, clean-worktree policy, stale-output checks, Pages control files, local-path-leak validation, optional commit, and source materialization.

Even those wrapper concerns are mostly not branch-only:

- Pages controls such as `.nojekyll` and `CNAME` are publication-host preset controls and can matter for sidecar `docs/` publication as much as branch publication. They should belong to a modular GitHub Pages preset, not to a branch-published mesh primitive.
- "source root and publication root are distinct" is a special case of source/output boundary validation, similar to the existing workspace-root / mesh-root relationship.
- Dirty worktree checks are git safety policy, not mesh semantics. They may be useful before an automated commit, but ordinary local runs should be allowed to work with a dirty mesh root and report or warn clearly.
- Stale generated-output checks should protect sidecar, whole-repo, and branch-published outputs. They are about not mixing old generated shapes with current output.
- Local path leakage validation is a portability/privacy check: generated public RDF should not contain absolute host paths such as `/home/...`, local checkout roots, or parent-directory traversal. That is relevant for any public mesh output.
- Optional local commit is generic git-output behavior. It could apply to any mesh root that is a git worktree, not just a publication branch.
- Source materialization and provenance updates belong to source-sync/integrate behavior, regardless of repository topology.

If all of these move to generic primitives or presets, `prepare gh-pages` can become a thin compatibility alias or disappear.

## Discussion

- Primitive in-place flow:
  - `weave mesh create --workspace . --mesh-root docs`
  - `weave integrate ./ontology/example.ttl ontology --mesh-root docs --grant-source-directory ontology`
  - `weave --mesh-root docs --target 'designatorPath=ontology,historySegment=releases,stateSegment=v0.1.0,manifestationSegment=ttl'`
- Detached publication-root flow:
  - `weave prepare gh-pages --source-root . --publish-root ../project-gh-pages ... --source-path ontology/example.ttl`
- The detached flow is not fundamentally about the branch name `gh-pages`; it is about separate source and publication roots.
- The `_mesh` bootstrap for a detached root can be ordinary `mesh create --workspace <publishRoot> --mesh-base ...`. `prepare gh-pages` should not need a parallel mesh-bootstrap concept; it should only call or require that primitive.
- Publication preparation is about host/publication policy, not mesh ontology: clean publication worktree checks, distinct source/publication roots, stale generated-output checks, host preset files, local-path-leak validation, and optional local commit.
- Those publication checks should not be limited to branch-published roots. A sidecar mesh under `docs/` can also be a GitHub Pages publication root, can also need `.nojekyll`, can also leak local paths, and can also contain stale generated output.
- In-place `integrate` does not necessarily copy bytes into the mesh root. If the source is outside `--mesh-root` but inside the configured workspace boundary, `integrate --grant-source-directory ...` records a `workingLocalRelativePath`, and later `weave` reads the current bytes from that path when local-path policy permits it.
- That means `weave` does "fetch" current sidecar bytes in the narrow local sense: it resolves the current working locator from inventory and policy, then reads that local file. It does not fetch arbitrary repository bytes or remote URLs.
- Pinning changes the read target. Current-tracking operations read the current working locator; pinned operations such as `extract --source-state` read an already materialized historical state snapshot. A future source-sync operation should make current-tracking versus pinned source resolution explicit for both sidecar and detached publication roots.
- The source being intra-repo or inter-repo should be metadata and provenance, not a behavioral fork. The behavior should follow the selected locator mode: mesh-local located file, granted local working path, materialized copy from repository path/ref/commit, pinned historical state, or future remote access URL.
- `prepare gh-pages --source-path` is needed when the release action wants Weave to materialize a new source payload, compare source bytes from the source checkout against the publication root, update an existing materialized payload, or record/update repository source provenance.
- After a payload exists in the publication root, a plain `weave --mesh-root <publishRoot>` or targeted `weave version --mesh-root <publishRoot>` should not need `prepare` in order to read that payload's current mesh-local working file. If that is not true, the runtime reader is incomplete.
- `prepare gh-pages` already runs the underlying weave work needed for the payload it materializes. A separate unconditional top-level `weave` after each prepare run is not currently a safe no-op because generated ResourcePages can be rewritten even when no new histories are created.
- A later standalone `weave --mesh-root <publishRoot>` is still appropriate after operations that create new un-woven candidates, such as term extraction.

## Open Issues

- Should `integrate` or a shared source-sync layer grow first-class separate-root / separate-repository materialization support, with `prepare gh-pages` becoming a preset wrapper for publication-root bootstrap, Pages controls, validation, and optional commit?
- Should a manifest/batch input target that shared layer rather than `prepare` specifically, listing payloads, target designators, release segments, source locators, materialization modes, and provenance metadata?
- Should source synchronization be a first-class operation shared by sidecar and detached publication roots, with explicit modes such as live-local-current, materialized-current, pinned-state, and repository-ref/commit?
- Should repository source bindings in `_knop/_sources` ever be dereferenced by ordinary `weave`, or should dereferencing external source bindings always stay in `prepare`/source-sync so `weave` remains a mesh-local operation?
- Is there any responsibility left that is genuinely branch-only? If not, deprecate `prepare gh-pages` after equivalent mesh/source/publication primitives exist.
- Should GitHub Pages controls such as `.nojekyll` and `CNAME` move to a host/publication-control preset so they are available for both sidecar and branch-published meshes, while `mesh create` remains host-neutral except when explicitly asked?
- What does the preset/plugin boundary look like for other publication targets such as GitLab Pages, Vercel, Netlify, or plain static hosting?
- Should clean/dirty git worktree handling become a generic git-output policy, with "warn" as the normal local default and "fail if dirty" only for automated commit/publication modes?
- Should local commit support become a generic mesh-root operation for any git worktree, rather than a `prepare gh-pages` option?
- Should local path leakage validation and stale-output checks become generic `weave validate-publication` or `weave validate --publication` behavior?
- Should top-level `weave --dry-run` exist as a planner contract that reports created/updated paths without writing, including generated ResourcePage changes?
- Should ResourcePage generation become stable no-op output by default, for example by removing or controlling volatile generated timestamps?

## Decisions

- Current release docs should say that multiple `prepare gh-pages --source-path` invocations are required when multiple source payloads must be materialized or refreshed from a detached source checkout.
- Current release docs should not imply that already-integrated, current publication-root payloads need `prepare` before ordinary mesh-local `weave` operations.
- Current release docs should not recommend an unconditional top-level `weave` after `prepare gh-pages`.
- The current asymmetry is acceptable only as an explicit source-resolution/materialization mode split: sidecar meshes may keep live local current locators under policy, while detached publication roots materialize portable current files and keep external repository details as provenance/source-sync data.
- Same-repository versus separate-repository source layout should not decide the semantic behavior. It may influence defaults and safety checks, but the operation contract should be driven by the requested source locator and materialization mode.
- The SemanticMesh bootstrap for detached roots belongs to `mesh create`; `prepare gh-pages` should wrap it with publication-root validation and Pages/publication controls rather than own a separate bootstrap model.
- Treat `prepare gh-pages` as a transitional wrapper over lower-level primitives. The target design should remove or thin it unless a genuine branch-only responsibility remains.

## Contract Changes

- `prepare gh-pages --dry-run` reports "No publication file changes would be made." when the simulated created and updated path sets are both empty.
- The CLI reference now describes the in-place sidecar and detached publication-root workflows side by side.

## Testing

- Added/updated deploy tests for settled `prepare gh-pages` dry-run no-op behavior.
- Existing targeted deploy CLI/runtime tests pass.

## Non-Goals

- Do not make `prepare gh-pages` push publication branches.
- Do not add a hidden automatic top-level `weave` after every prepare run.
- Do not force same-branch sidecar meshes to use detached publication-root semantics.

## Implementation Plan

- [x] Document workflow-shape differences in [[wu.cli-reference]].
- [x] Document why the SFLO example uses both primitive commands and `prepare gh-pages` in [[wu.cli-reference.examples.sflo]].
- [x] Make no-op `prepare gh-pages --dry-run` output explicit.
- [ ] Design separate-root / separate-repository materialized integrate or source-sync support that can be reused by `prepare gh-pages`.
- [ ] Design a manifest/batch input over the shared source-sync/integrate layer for multi-payload releases.
- [ ] Factor publication host controls (`.nojekyll`, `CNAME`) into a modular GitHub Pages preset usable by sidecar, whole-repo, and branch-published meshes.
- [ ] Sketch publication-host preset interfaces for GitHub Pages, GitLab Pages, Vercel, Netlify, and plain static output without committing to all implementations.
- [ ] Factor stale-output and local-path-leak validation into generic publication validation.
- [ ] Factor optional git commit support into a generic mesh-root git operation or output policy.
- [ ] Decide whether any remaining behavior justifies keeping `prepare gh-pages`; otherwise plan a replacement/deprecation path.
- [ ] Add or defer top-level `weave --dry-run`.
