---
id: epycvde4vhh8pyzbpi6yex2
title: 2026 05 24_0754 Framework Hygiene
desc: ''
updated: 1779635953970
created: 1779634865209
---

## Goals

- Clean up stale fixture refs and fixture documentation that make current conformance lanes harder to understand.
- Treat `a.*` fixture refs as the canonical active scenario branches for the current Accord lanes, while preserving `main` and real publication branches such as `gh-pages`.
- After `weave import` lands, use a regenerated `b.*` lane as a comparison target for fixture-helper generalization, rather than overwriting or immediately deleting the `a.*` baseline.
- Remove or clearly quarantine old unprefixed fixture branches after confirming no active scenario index, conformance runner, documentation, or downstream test still resolves them.
- Align the Semantic Flow Framework notes with the current import boundary from [[wa.completed.2026.2026-05-21_0907-import]] without turning this task into the Weave implementation task.

## Summary

The current fixture ecosystem carries two different kinds of names:

- manifest step names such as `20-bob-page-imported-source`
- actual active fixture branch refs such as `a.20-bob-page-imported-source`

The current `scenario-index.jsonld` files declare `branchPrefix: "a."` and point their lane bindings at `a.*` refs. Those are the active carried fixture branches. The unprefixed remote branches still present in some fixture repos appear to be stale duplicate ladders from before the `a.*` lane split. They now create confusion, especially around the Bob imported-source steps, and some historical refs carry older namespace vocabulary or older import-shaped ideas that should not guide new implementation work.

This task should prune stale duplicate fixture refs where they are truly unused and clarify fixture docs so logical step names are not mistaken for branch refs. After `weave import` lands, a `b.*` regeneration lane can compare new fixture-helper output against the current `a.*` baseline. Bob `b.20/b.21` can then become a real remote-origin import fixture while `a.20/a.21` remains available as the comparison case for the old half-state. Carol can become a later three-version import/weave ladder for `--replace-working` and ResourcePage panel work.

## Discussion

Observed fixture repo state:

- `mesh-alice-bio` has both unprefixed remote branches `00-blank-slate` through `25-root-page-customized-woven` and canonical `a.00-blank-slate` through `a.25-root-page-customized-woven`.
- `mesh-sidecar-fantasy-rules` has both unprefixed remote branches for part of the ladder and canonical `a.*` branches. The unprefixed ladder is also incomplete relative to the current scenario index, which makes it especially poor as a reference surface.
- `mesh-branch-fantasy-rules` is already closer to the desired shape: it carries canonical `a.*` refs plus `main` and `gh-pages`. Those should be preserved.
- `semantic-flow-framework/examples/*/conformance/scenario-index.jsonld` uses `branchPrefix: "a."` and lane bindings with explicit `a.*` refs. That makes the active branch prefix clear to machines, but humans still see unprefixed `fromRef` and `toRef` in individual manifests and can mistake those values for branch names.

The Bob import-boundary pair is a good example of the confusion. `20-bob-page-imported-source.jsonld` says the transition imports Markdown from a raw GitHub URL, but the active `a.*` fixture assertions prove a narrower page-source boundary: local `bob-page-main.md`, a `ResourcePageDefinition` source with `sflo:targetLocalRelativePath "bob-page-main.md"`, and no direct `targetAccessUrl` on the page source. It does not prove a productized `import` operation, does not create a `bob/page-main` payload artifact, and does not record raw URL provenance.

That split is probably the wrong end state. A "local import" fixture is weak in a whole-repo mesh because the source bytes could simply be authored in place or integrated after an explicit local copy. If Bob `20/21` keeps the imported-source name in a regenerated `b.*` lane, the regenerated fixture should actually carry remote-origin provenance: the copied local working bytes, the outside source URL recorded through the source registry, and an observed digest. Bob should intentionally omit a provided expected digest, so this fixture proves the floating-source record-keeping path rather than the deterministic expected-digest path. The page definition should still render from governed local bytes rather than fetching the remote URL during `weave`.

The fixture regeneration should wait until real remote-origin import has landed. If that does not happen before a broader fixture cleanup is needed, the honest fallback is to rename or reword Bob `20/21` as local page-source coverage and leave remote-origin import to a later fixture. Keeping the current wording and current assertions together is the confusing middle.

Using `b.*` for the post-import full regeneration lets the project keep `a.*` as a known-good comparison lane while proving that the generalized fixture helper can reproduce or intentionally evolve the ladder. After the comparison is accepted, `b.*` can become the new canonical lane and old duplicate refs can be pruned in a controlled follow-up.

The framework notes already describe the conceptual import/integrate boundary in several places, including [[sf.api]], [[sf.api.examples]], [[sf.spec.2026-04-11-identifier-page-customization-and-root-lifecycle]], and [[sf.spec.2026-05-18-publication-source-binding]]. Those notes are broadly pointing in the right direction, but this cleanup should make sure they distinguish portable behavior/spec intent from a shipped Weave CLI command. The Weave implementation details remain in [[wa.completed.2026.2026-05-21_0907-import]].

## Open Issues

- Which unprefixed fixture refs are still referenced by any active runner, CI job, documentation, or external consumer? Recommendation: assume none are canonical, but audit before deleting remote branches.
- Should individual conformance manifests keep unprefixed `fromRef` and `toRef` as logical scenario state names, or should the manifest vocabulary grow an explicit distinction between logical step ids and resolved lane refs? Recommendation: keep existing manifests stable for now, but improve README/spec wording and rely on `scenario-index.jsonld` as the resolved branch source of truth.
- Should the next full fixture regen write to `b.*` refs for comparison against `a.*`? Recommendation: yes, but after `weave import` lands.
- Should Bob `b.20/b.21` become the real remote-origin import fixture during the next full regen, or should it be renamed/reworded as local page-source coverage? Recommendation: if it keeps the import name, regenerate it as real remote-origin import from the floating raw `main` URL with source-registry URL and observed-digest provenance; otherwise stop calling it import.
- Should Carol become a six-step fixture sequence? Recommendation: yes, as three import/weave pairs over three page versions, using `--replace-working` on the second and third imports.

## Decisions

- `a.*` refs are the current canonical fixture branch refs for Alice Bio, sidecar Fantasy Rules, and branch-published Fantasy Rules conformance lanes.
- A full fixture regeneration can use a `b.*` branch series first, so `b.*` can be compared against `a.*` before any canonical-lane switch or pruning. That regeneration is deferred until after `weave import` lands.
- Preserve `main`, active publication branches such as `gh-pages`, and all `a.*` refs that appear in a current scenario index.
- Treat unprefixed fixture branches in `mesh-alice-bio` and `mesh-sidecar-fantasy-rules` as stale candidates for pruning unless the audit finds an active dependency.
- Do not prune by rewriting history or deleting local user work. Use explicit remote branch deletion only after the audit and a final human check.
- Framework docs should call unprefixed manifest `fromRef` and `toRef` values logical scenario state names unless a scenario index resolves them to a concrete branch ref.
- Bob `20/21` should not remain in the current half-state in the regenerated lane. Either make `b.20/b.21` a real remote-origin import fixture that records origin URL and observed-digest provenance from a floating source, or reword/rename it as page-source regression coverage that does not claim import.
- Carol should be planned as three import/weave pairs over one governed `carol/page-main` working file, with later imports replacing working bytes explicitly.

## Contract Changes

- No Semantic Flow behavior contract should change as part of branch pruning.
- The framework conformance documentation should clarify that scenario indexes and their lane bindings are authoritative for actual fixture refs.
- The Bob `20/21` manifest prose and assertions should agree. If the fixture keeps the imported-source contract, assert remote-origin provenance and observed digest in the Knop source registry; if it remains local page-source coverage, remove raw-URL import claims from the prose.
- The regenerated `b.*` scenario index should make its branch prefix explicit, so fixture consumers can compare `a.*` and `b.*` without guessing which lane is active.
- API/spec notes that mention import should remain conceptual unless they explicitly link to the future import behavior spec or Weave task.

## Testing

- Run a conformance reference audit that resolves every `scenario-index.jsonld` lane binding and confirms every referenced `a.*` ref exists in its fixture repo.
- For the regenerated lane, resolve every `b.*` lane binding and compare expected file/RDF outcomes against the existing `a.*` baseline except where the regen intentionally changes behavior, such as Bob import provenance.
- Search the Semantic Flow Framework notes, examples, Weave docs, and task archive for references to unprefixed fixture branch names before pruning.
- After pruning stale remote branches, re-run the conformance tests that fetch or checkout fixture refs.
- Confirm `mesh-alice-bio`, `mesh-sidecar-fantasy-rules`, and `mesh-branch-fantasy-rules` still expose all refs named in their scenario indexes.
- Confirm Bob `20/21` prose and assertions agree with each other after regen or wording updates, including URL and observed-digest provenance if the fixture keeps the import role.
- Confirm Carol replacement rungs show the same governed working file being replaced and then woven into distinct historical states.

## Non-Goals

- Do not implement `weave import` here.
- Do not create the new Carol import/weave ladder here unless this task is explicitly expanded.
- Do not require the `b.*` regeneration to land before `weave import`; the regen is follow-on acceptance/regression coverage.
- Do not delete `main`, `gh-pages`, or any `a.*` branch referenced by a scenario index.
- Do not promote `b.*` to canonical or delete `a.*` as part of the first regeneration pass; comparison comes first.
- Do not rename existing manifest files unless the conformance runner is updated in the same change.
- Do not redesign the Accord manifest vocabulary unless the audit proves wording alone cannot resolve the confusion.
- Do not update ontology/content-kind modeling here. If imported non-RDF typing needs framework-level vocabulary later, track that as a separate spec task.

## Implementation Plan

- [ ] Audit all current scenario indexes under `semantic-flow-framework/examples/*/conformance/` and record the complete set of concrete fixture refs they require.
- [ ] Audit `mesh-alice-bio`, `mesh-sidecar-fantasy-rules`, and `mesh-branch-fantasy-rules` local and remote refs; classify refs as canonical active, publication branch, default branch, or stale duplicate fixture branch.
- [ ] Search framework notes, examples, Weave docs, and task notes for unprefixed branch refs and decide whether each reference means a logical scenario state or a concrete git ref.
- [ ] Update conformance READMEs and relevant spec notes to say that `scenario-index.jsonld` lane bindings resolve logical scenario names to concrete `a.*` refs.
- [d] After `weave import` lands, regenerate the next full fixture lane as `b.*` so the generalized fixture helper can be compared against the current `a.*` lane.
- [d] During the `b.*` regen, either make Bob `b.20-bob-page-imported-source` and `b.21-bob-page-imported-source-woven` a real remote-origin import pair with floating-source URL and observed-digest provenance, or rename/reword them as local page-source coverage.
- [d] Add the Carol six-step replacement ladder when the import fixture scope expands: import/weave v1, replace-working import/weave v2, replace-working import/weave v3.
- [ ] If the audit is clean, prune stale unprefixed fixture branches from `mesh-alice-bio` and `mesh-sidecar-fantasy-rules` while preserving `main`, publication branches, and all active `a.*` refs.
- [ ] Re-run conformance or fixture tests that checkout the fixture refs.
- [ ] Update [[sf.api]], [[sf.api.examples]], [[sf.spec.2026-04-11-identifier-page-customization-and-root-lifecycle]], and [[sf.spec.2026-05-18-publication-source-binding]] only where they blur conceptual import with shipped command behavior.
