---
id: watask20260721extractordefects
title: 'Extractor defect pair — all-terms file-URL crash + missing hasResourcePage claims'
desc: 'Fix the two Weave 0.3.0 extraction defects surfaced by the srd-rules big-mesh publication run (2026-07-21): (a) extract --all-terms enumerates checked-in file URLs as terms then dies (Not a directory); (b) extracted-term Knops get no sflo:hasResourcePage claims, forcing consumers to patch inventories. Boarded with a wake trigger that fired at the version-API lane; cut 2026-07-21 16:03 by the stagecraft flagship seat under the we-own-the-substrate law.'
---

## Goals

Make `weave extract` correct on big real meshes so downstream pipelines stop working around it: `--all-terms` must enumerate only genuine terms (never checked-in file URLs), and extracted-term Knops must carry their governed `sflo:hasResourcePage` claims without consumer-side patching.

## Summary

The srd-rules publication run (the first big-mesh consumer, ~6,900 files) surfaced both defects with exact evidence:

- **(a) all-terms filtering:** `weave extract --all-terms` includes checked-in file URLs such as `conditions.csvw.json` in the term set, then attempts `conditions.csvw.json/_knop/...` and refuses with `Not a directory`. The pipeline works around via supported single-term extraction over a governed non-file census.
- **(b) page-claim emission:** extracted term Knops receive no `sflo:hasResourcePage` claims, so the consumer's `prepare-srd-publication.mjs` deterministically adds the missing governed claims to Weave-created inventories before `weave generate` renders HTML.

Both workarounds live in the stagecraft repo's `tools/srd-extraction/` and are cited in the srd-rules cutover task's phase-A receipts (stagecraft-lab note `flag.task.2026-07-21_1116-srd-rules-package-repo`, "Named upstream candidate" entries).

## Decisions

- Fix BOTH defects in one lane off weave `main` (`b7cbf89`); they share the extraction surface and one consumer proof.
- The whole-mesh `weave validate` 4 GiB heap exhaustion and any parse-once restructuring are OUT (they stay on wd.todo — architecture work, not a defect lane).
- Consumer-side workaround removal (the stagecraft pipeline) is OUT — boarded stagecraft-side, fires after this lands.
- Branch `fix/extractor-defect-pair`; path-scoped commits; no push until the runner lands it per weave convention.

## Contract Changes

None to the public API/CLI surfaces beyond corrected behavior: `--all-terms` output narrows to genuine terms (the file-URL entries were never valid); extracted inventories gain claims they were always supposed to carry. If either correction requires a documented behavior-contract edit (wd notes), it rides the same lane.

## Testing

- Fail-on-old repro per defect: (a) a fixture mesh containing a checked-in file URL among term candidates — old code dies `Not a directory`, new code enumerates cleanly without the file URL; (b) an extraction fixture asserting the extracted Knop inventory carries the governed `sflo:hasResourcePage` claims — red on old.
- Focused suites for the touched extraction families, then one full `deno task ci` green at lane end.

## Non-Goals

Heap/scale work, parse-once, whole-mesh validation green, npm/release work, consumer pipeline edits.

## Implementation Plan

1. Locate the term-enumeration path for `--all-terms`; add the genuine-term filter at the enumeration source (not a post-filter on the crash site); fail-on-old.
2. Locate extracted-Knop inventory emission; emit the governed `sflo:hasResourcePage` claims where the mesh's page-generation contract expects them (mirror what `weave generate`/validate consider governed); fail-on-old.
3. Full ci; receipts to this note.

## Spec review

*(appended by a fresh review seat with PROPOSED verdicts/severities; execution blocked until the stagecraft PM's sign-off)*

### r1 — adversarial spec review (2026-07-21)

#### PROPOSED BLOCKER — F1: defect (a) is real, but "genuine term" has no landed predicate

The actual enumeration source is `discoverAllTermDesignatorPaths` in `src/runtime/extract/extract.ts`. Both preview and execution call it. It parses the selected payload, passes every subject, predicate, and named-node object through `listQuadNamedNodeIris`, retains every mesh-scoped IRI, and excludes only paths found in `GENERATED_RESOURCE_CLASS_IRIS` typing or containing a reserved `_mesh`/`_knop` segment. `normalizeSafeDesignatorPath` deliberately permits dots, so a graph edge such as `dcat:downloadURL <.../conditions.csvw.json>` becomes the valid-looking designator path `srd-5-2-1/conditions/conditions.csvw.json`. `planExtract` then plans `<file>/_knop/...`; `stagePlanMutation` reaches `ensureDirectoryExists` in the same runtime file, finds that the `<file>` prefix is an existing non-directory, and throws `Workspace path is not a directory`. This matches the reported failure (the installed binary's surface may shorten the underlying condition to `Not a directory`).

The landed docs describe "named mesh-scoped RDF terms" and exclusions for generated support/file artifacts, but neither designator syntax nor SFLO typing defines a general positive "genuine term" test. An extension heuristic is unsafe because dots are valid designator characters. Requiring `rdf:type` on the candidate subject would be a new rule and would exclude currently documented predicate/object discovery; requiring `sflo:LocatedFile` to exclude a path does not catch the reported `dcat:downloadURL`, which is untyped in the defining payload. The code has no reusable predicate that settles this.

Required resolution: rule the positive/negative enumeration predicate in the note before build, including how subjects, predicates, object-only references, untyped named nodes, file-like IRIs, and explicitly typed SFLO resources are treated. Name `discoverAllTermDesignatorPaths` as the enforcement point and require the fail-on-old test to prove the ruled boundary, not merely a `.json` suffix special case.

#### PROPOSED BLOCKER — F2: defect (b) contradicts the landed extract/weave lifecycle and does not rule which page claims extraction owns

Non-woven inventories are emitted by `renderExtractKnopInventoryTurtle` in `src/core/extract/extract.ts`. Its omission of page claims is intentional under landed `sf.spec.2026-04-05-extract-behavior`: extraction creates no pages or `hasResourcePage` claims; the following weave owns page materialization. The current code implements that contract. `planFirstExtractedKnopWeave` in `src/core/weave/weave.ts` uses `renderFirstExtractedKnopWovenKnopInventoryTurtle` in `src/core/weave/knop_inventory_renderers.ts` for Knop/support-page claims and `renderGenericFirstExtractedKnopWovenMeshInventoryTurtle` (or the current-only equivalent) in `src/core/weave/mesh_inventory_renderers.ts` for the public designator and Knop claims. `assertCurrentMeshInventoryShapeForFirstExtractedKnopWeave` rejects public designator/Knop claims in the pre-weave mesh inventory, while `detectPendingWeaveSlice` uses current Knop/inventory page claims as evidence that the Knop is already woven. Adding claims during extraction can therefore suppress or invalidate the later first-extracted-Knop weave unless the lifecycle is changed coherently.

`weave generate` genuinely consumes claims: `listGeneratedResourcePagePaths` in `src/core/weave/resource_page_policy.ts` enumerates only `sflo:hasResourcePage` objects, and runtime page-context assembly calls it for mesh and per-Knop inventories. `weave validate mesh` does not independently decide a governed page set; `executeValidate` in `src/runtime/weave/weave.ts` dry-runs version planning, validates planned RDF, and runs publication-preset checks. Thus "mirror what generate/validate consider governed" is not executable guidance. The consumer workaround adds only `<D> sflo:hasResourcePage <D/index.html>` plus the page declaration to both mesh and local inventories, whereas a normal first extracted weave governs additional Knop, metadata, inventory, history, state, and manifestation pages. "Governed claims" does not choose between canonical public pages only and the whole rendered support-page set.

Required resolution: choose and state one lifecycle. Either preserve non-woven extraction and locate the real failure preventing the supported extract→weave→generate sequence, or intentionally make extraction publication-bearing. For the latter, enumerate the exact subject/page claim matrix and which inventory owns each fact; rule declarations, history-policy variants, and later-weave behavior; update the cross-repo `[[sf.spec.2026-04-05-extract-behavior]]` contract plus Weave user docs; and include classifier/shape-assertion compatibility tests. The present "no public contract change" statement must be withdrawn or justified against that landed spec. Heap/parse-once work need not be included, but it cannot be used as an unstated reason to replace the supported lifecycle.

#### PROPOSED HIGH — F3: the shared all-terms discovery surface silently broadens defect (a)'s CLI scope

`discoverAllTermDesignatorPaths` is also used by `previewSetExtractionSourceAllTerms` / `executeSetExtractionSourceAllTerms`, not only by `extract --all-terms`. A filter at the required enumeration source will therefore change the census and preview of `weave set extraction-source --all-terms`. The task claims only corrected `extract` output and does not say whether the maintenance command must share the new predicate.

Required resolution: explicitly rule that both all-terms commands share the same census and require focused coverage for both, or split the discovery policies and explain why their term domains differ. Reflect the chosen command scope in Contract Changes.

#### PROPOSED MEDIUM — F4: the fail-on-old fixture instructions are not located, although landed fixtures are sufficient

No new external fixture mesh is necessary. `tests/e2e/extract_cli_test.ts` already materializes `mesh-alice-bio` ref `11-alice-bio-v2-woven` into an isolated `createTestTmpDir()` workspace and overwrites `alice-data.ttl` inline for the all-terms case; that test can add an untyped mesh-local file IRI plus the checked-in file and reproduce old staging failure without a new fixture rung. `src/core/extract/extract_test.ts` already owns the non-woven inventory contract, while `src/core/weave/weave_test.ts` owns first-extracted-Knop claims and lifecycle compatibility. If F2 changes public generation behavior, an e2e extract→generate or extract→weave→generate assertion is also needed. The current note only says "a fixture mesh" and "an extraction fixture", leaving the builder to choose whether to mutate the cross-repo ladder.

Required resolution: name these test files and the temp-workspace strategy in Testing. State that no fixture-repo refs are to be minted unless the ruled F2 behavior requires an Accord transition; if it does, require the `[[wd.testing.fixture-ladder-regeneration]]` workflow and the relevant behavior-spec/conformance update.

#### PROPOSED MEDIUM — F5: receipts ownership is ambiguous across repositories

`deno task ci` is the correct landed pre-merge gate, and the proposed branch/base and heap/parse-once fences are coherent. The term filter is local to already-parsed quads, and page-contract work does not force heap or parse-once restructuring. No CLI flag spelling needs to change. However, the implementation branch exists in Weave while this task/receipt note is tracked in the separate `weave-dev-archive` repository; F2 may additionally require a `semantic-flow-framework` spec edit. "One lane off weave main", "path-scoped commits", and "receipts to this note" do not assign those cross-repo writes or their commits.

Required resolution: state whether the build seat may edit/commit the archive note and framework spec, or whether Kato/PM owns those updates out of band. Require final receipts to name the focused test commands, `deno task fmt`, `deno task ci`, per-repo status/commit identities, and any ruled documentation changes. No full fixture regeneration is needed merely for this code lane.

### r1 proposed verdict

**BLOCKED-ON-SPEC** — the note must rule the genuine-term predicate and the page-claim lifecycle/exact claim set before a builder can implement either fix without inventing product semantics.

### r1 adjudication + RESOLVED SPEC (amendment r1 — stagecraft PM, flagship seat `52b05338`, 2026-07-21 16:12; GOVERNS wherever it conflicts with the body)

All five findings CONCUR. F2 corrected the task's premise and is owned as a PM reversal: the body's defect (b) framing ("extraction should emit hasResourcePage claims") CONTRADICTS the landed `sf.spec.2026-04-05-extract-behavior` lifecycle — extraction is deliberately non-publication-bearing; the following weave owns page materialization. The body's Contract Changes sentence ("inventories gain claims they were always supposed to carry") is WITHDRAWN.

**R1. The enumeration predicate (F1 — ruled):** `discoverAllTermDesignatorPaths` is the enforcement point. NEW EXCLUSION, positive rule: a candidate designator path that exists in the workspace as a NON-DIRECTORY FILE is a file artifact, never a term home — excluded at enumeration with the existing `GENERATED_RESOURCE_CLASS_IRIS` and reserved-segment exclusions unchanged. Rationale: a term's Knop lives under `<designator>/_knop/`, which is impossible where `<designator>` is a file; the existence check grounds the boundary in workspace reality rather than extension heuristics or new typing rules, and preserves documented subject/predicate/object discovery for genuine terms. The fail-on-old test proves the ruled boundary with the reported shape (an untyped `dcat:downloadURL` object naming a checked-in file), not a `.json` suffix special case.

**R2. Defect (b) RECAST as diagnosis, lifecycle PRESERVED (F2 — ruled arm: keep non-woven extraction):** this lane makes NO page-claim change anywhere. Deliverable instead: a read-only ROOT-CAUSE DIAGNOSIS of why the supported extract→weave→generate sequence was not viable for the srd-rules mesh (~1,700 terms) — suspects to check: per-term first-extracted-Knop weave cost at that cardinality, the 4 GiB heap ceiling, or a real defect in `planFirstExtractedKnopWeave` at scale — reported as a receipts section with evidence and boarded to wd.todo as a NAMED follow-up. The consumer workaround stays in place until the supported sequence is proven viable; no framework-spec edit occurs in this lane.

**R3. Command scope (F3 — ruled):** `extract --all-terms` AND `set extraction-source --all-terms` SHARE the ruled census predicate (one discovery surface, one boundary); focused coverage for both commands. Contract Changes now reads: the all-terms census narrows identically for both commands; no flag spellings change.

**R4. Test strategy (F4 — ruled):** no new fixture repos, no fixture-ladder refs. Extend `tests/e2e/extract_cli_test.ts` (the landed `mesh-alice-bio` temp-workspace pattern) with the untyped file-IRI case; unit coverage in `src/core/extract/extract_test.ts` / the discovery function's home as fits. No Accord transition.

**R5. Receipts ownership (F5 — ruled):** the build seat appends receipts to THIS note and leaves the archive repo UNCOMMITTED (the runner sweeps); the build commits only to the weave repo branch `fix/extractor-defect-pair` (base `b7cbf89`), path-scoped. Receipts name the focused test commands, `deno task fmt`/`ci` results, and per-repo status/commit identities.

r2 light re-check fires on this amendment; build after r2-clean + PM GO.
