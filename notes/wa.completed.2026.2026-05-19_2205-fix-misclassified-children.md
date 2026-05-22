---
id: 4882lo8gvwz6i3wyk1jnewd
title: 2026 05 19_2205 Fix Misclassified Children
desc: ''
updated: 1779253603359
created: 1779253603359
---

## Goals

- Fix ResourcePage child row classification so targeted generation cannot move known typed children, such as SHACL `sh:NodeShape`, `sh:PropertyShape`, and `sh:Shape` terms, back into `Individuals`.
- Preserve the intended separation between versioning scope and display refresh scope: a targeted weave should still version only requested designators, while any pages it refreshes should render child groupings from complete type evidence for their displayed children.
- Add regression coverage using the branch-published Fantasy Rules failure mode: refreshing `ontology/index.html` while targeting only representative extracted term pages must not misclassify `AbilityScoreShape` and the `PrimaryAbilityChoice` shapes, and focused coverage must keep `sh:PropertyShape` and generic `sh:Shape` children in their dedicated rows.
- Keep the fix in generation/model assembly if possible, not in HTML scraping or name-based heuristics.
- Avoid adding durable duplicate RDF-type metadata unless there is a stronger reason than this display bug.

## Summary

The live symptom is on `mesh-branch-fantasy-rules/ontology/`: `AbilityScoreShape` and most other SHACL NodeShapes appear under `Individuals`, while `CharacterShape` appears under `Node Shapes`.

The RDF data is not the problem. The individual pages for `AbilityScoreShape`, `CharacterPrimaryAbilityByClassShape`, and the class-specific `PrimaryAbilityChoice` shapes correctly show `a sh:NodeShape`. The same failure mode applies to `sh:PropertyShape` and generic `sh:Shape` children when their type triples come from a sibling or supporting source artifact rather than the parent page's own raw panel. The bug is that the refreshed parent page does not always have complete child RDF type evidence when it rebuilds its `Children` table.

This is a targeted page-refresh problem, not a semantic change in the source RDF. A previous branch state had the released SHACL shapes correctly grouped under `Node Shapes`. A later targeted reference weave refreshed only a few term pages and their display ancestors. During that refresh, the model had type hints for the targeted `ontology/CharacterShape` child but not for the other sibling shape children. Since the renderer falls back to `Individuals` for children with no recognized type, the refreshed `ontology/index.html` overwrote previously-correct grouping with partial grouping.

## Discussion

### What Exists Today

We do not currently preserve a durable child RDF type hint cache between generations.

The generated HTML is not read back as input, and the mesh inventory does not record `rdf:type` for arbitrary designator children. The current behavior only looks like it preserves some hints because some hints are recomputed in the current generation run and others are not.

There are two current sources of child classification evidence:

- `ResourcePageChildIdentifierModel.rdfTypes`, which is assembled in `src/runtime/weave/weave.ts` from the selected designator contexts for the current generate/weave request.
- Raw RDF panels available to the parent page renderer, which `src/runtime/weave/pages.ts` parses to find child `rdf:type` triples in the same source panel as the parent page.

That means a parent page sourced from `ontology/fantasy-rules-ontology.ttl` can classify ontology classes and properties from its own raw panel. But children whose defining triples live in a separate source artifact, such as SHACL shapes defined in `shacl/fantasy-rules-shacl.ttl` while living under the `ontology/` namespace, need type hints from their own extracted designator/source context. If those child contexts are not part of the current targeted generation set, their `rdfTypes` can be missing even though their individual pages are correct and even though previous generated HTML had the right row. The existing renderer already has separate rows for `Node Shapes`, `Property Shapes`, and `Shapes`; the model assembly needs to feed all three asserted SHACL shape types reliably.

### Why Targeted Weave Exposes It

Targeted weave intentionally expands the generate phase to include ancestor pages. This was added because child lists, breadcrumbs, and similar navigation/display data live on ancestors even when versioning remains scoped to the requested child target.

For example, a targeted weave of `ontology/CharacterShape` can also refresh `/` and `ontology` so their child lists stay current. That is the right display behavior, but it means ancestor pages are now written in runs where only a subset of their children are selected as generation contexts.

The failure appears when these conditions line up:

- a parent page is refreshed because it is an ancestor/display page, not because its source artifact changed.
- the parent has children whose RDF types are defined outside the parent page's own raw RDF panel.
- only some of those children are included in the selected designator contexts for this generation run.
- the renderer treats missing type evidence as `Individuals`.

This is not about the child RDF data changing. Source RDF data changes should still change grouping when the relevant page is regenerated from the changed source. The bug is that unchanged sibling children can be recategorized merely because a targeted refresh had incomplete evidence.

### Do We Need An In-Memory Graph?

We probably want a small generation-time RDF type index, but not a general persisted graph store.

The useful shape is closer to:

- identify the pages Weave is going to write.
- identify the immediate child resource paths those pages will display.
- collect `rdf:type` triples for those child resource paths from the relevant current raw source panels and extraction/reference source panels.
- pass the resulting `resourcePath -> rdf:type[]` hints into child identifier models before rendering.

That is "graph-ish" because it parses RDF and answers a small lookup query, but it does not need full reasoning, SHACL execution, SPARQL, HTML scraping, or long-lived cached metadata. Keeping it generation-local avoids making `rdf:type` another duplicated durable fact that can drift from source RDF.

### Scope Of Recompute

We do not need to recompute every RDF fact for every mesh page on every targeted weave.

The invariant should be narrower:

Any ResourcePage that Weave chooses to write and that renders a `Children` section must classify all displayed immediate children using the best current type evidence available for those children, not only type evidence for explicitly requested targets.

In practice this includes refreshed ancestor pages, because they are the common pages added during targeted generation. But the rule is not "all ancestors forever"; it is "every page being written must have complete child type hints for the children it is about to display."

### Why Not Preserve Old Page Rows

Reading the old generated HTML and preserving prior child rows would hide the immediate regression but would be the wrong source of truth. It would make display state depend on previous renderer output, and it would fail when source RDF legitimately changes. The source graph should drive the table, with generated HTML treated as disposable output.

## Open Issues

None currently.

The earlier design questions are resolved as follows:

- Source set: build the type index for immediate child resource paths displayed by pages Weave is about to write. Do not load every mesh designator merely because one page is refreshed.
- Loader shape: prefer a slimmer source-panel/type extraction path over reusing full `loadGenerateDesignatorContexts` for all children. It can share low-level helpers, but it should avoid creating full page-generation contexts unless the narrow path becomes brittle.
- Missing sibling source panels: keep generation best-effort for non-target siblings. If a non-requested sibling's source cannot be read, leave that child's type unknown and let the renderer fallback apply. Do not fail the requested weave unless the missing panel belongs to a requested target or another artifact already required by the operation.
- Ancestor refresh: keep the current targeted weave behavior that refreshes ancestor/display pages. This task fixes incomplete evidence during refresh; it does not narrow when ancestor pages are refreshed.
- Type semantics: use asserted `rdf:type` only. Cover the recognized direct SHACL shape types `sh:NodeShape`, `sh:PropertyShape`, and `sh:Shape`, plus the existing OWL/RDFS/RDF categories. Do not add inference for subclasses of `sh:Shape` in this task.

## Decisions

- Treat generated ResourcePages as disposable output, not as a cache to preserve or mine for child classifications.
- Keep child classification RDF-driven; do not revive name suffix heuristics like "ends with Shape".
- Keep the targeted weave ancestor-refresh behavior. The bug is incomplete type evidence during refresh, not the existence of ancestor refresh.
- Prefer a generation-local child RDF type index over durable duplication in mesh inventory.
- Keep missing type evidence as `Individuals` only when type evidence is genuinely unavailable, not when it was skipped because a sibling was outside the explicit target set.
- Build the child type index for displayed child paths of pages being written, not for the entire mesh by default.
- Use asserted direct RDF types only; specifically preserve `Node Shapes`, `Property Shapes`, and `Shapes` row behavior for `sh:NodeShape`, `sh:PropertyShape`, and `sh:Shape`.
- Keep missing non-target sibling source panels non-fatal.

## Contract Changes

- No ontology vocabulary change is expected.
- No CLI surface change is expected.
- No Semantic Flow API contract change is expected.
- The externally visible behavior should become more stable: targeted generation should not alter child category rows for unchanged sibling children merely because the target set was narrow.
- If the branch fixture manifests assert generated HTML rows, update those expectations only to lock in the corrected stable behavior.

## Testing

- Add a focused runtime or integration regression that reproduces the branch-published Fantasy Rules case:
  - start from a publication state where `ontology/index.html` has many SHACL-sourced children.
  - perform a targeted weave/generate that refreshes `ontology/index.html` while selecting only a representative subset such as `ontology/CharacterShape`.
  - assert `AbilityScoreShape`, `CharacterPrimaryAbilityByClassShape`, and a class-specific `PrimaryAbilityChoice` child remain in the `Node Shapes` row.
  - assert ordinary individuals such as `barbarian` remain in `Individuals`.
- Add or extend unit coverage around child type hint collection so a parent page can classify children whose defining `rdf:type` triples come from child/source panels rather than the parent raw panel.
- Add focused coverage for all recognized SHACL shape rows:
  - `sh:NodeShape` -> `Node Shapes`
  - `sh:PropertyShape` -> `Property Shapes`
  - `sh:Shape` -> `Shapes`
- Keep existing renderer tests that prove unknown child types fall back to `Individuals`; the fix should not turn every `*Shape` name into a shape.
- Run focused tests after the code change:
  - `deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env src/runtime/weave/pages_test.ts tests/integration/weave_test.ts tests/integration/branch_fantasy_rules_fixture_test.ts`
  - `deno task check`
  - `deno task lint`
- After implementation, regenerate or repair the affected branch fixture output so `mesh-branch-fantasy-rules/ontology/index.html` shows the stable child rows again.

## Non-Goals

- Do not implement full RDF reasoning or SHACL-aware inference.
- Do not add a persistent graph database.
- Do not store copied child `rdf:type` facts in generated HTML, mesh inventory, or Knop inventory solely to fix this renderer issue.
- Do not remove targeted weave ancestor refresh.
- Do not classify children from labels or path/name suffixes.
- Do not broaden this task into latest-state source resolution or fixture-ladder renumbering.

## Implementation Plan

- [x] Write a failing regression for targeted refresh of a parent page whose children have type evidence from mixed source artifacts.
- [x] Trace the current `collectGeneratedPageFiles` path to identify exactly which pages will be written and which child identifiers each written page displays.
- [x] Add a generation-local child RDF type collection step that gathers type hints for displayed child resource paths, not just selected target contexts.
- [x] Prefer loading raw source panels only for relevant child designators; use broader context loading only if the narrower path becomes too brittle.
- [x] Keep `renderResourcePage`'s parent raw-panel parsing as a useful fallback, but make model-level `rdfTypes` complete enough for cross-source children.
- [x] Verify `sh:NodeShape`, `sh:PropertyShape`, and `sh:Shape` children remain in `Node Shapes`, `Property Shapes`, and `Shapes` respectively during targeted parent refresh.
- [x] Verify unknown children still render under `Individuals` when no asserted recognized type exists.
- [x] Run focused page/weave/branch-fixture tests.
- [x] Run `deno task check` and `deno task lint`.
- [x] Regenerate or repair the affected branch-published Fantasy Rules fixture output.
- [x] Provide a commit message for the Weave repo and, if fixture output changes in a separate repo, a separate fixture repo commit message.
