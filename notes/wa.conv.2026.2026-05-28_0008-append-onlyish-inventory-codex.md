---
id: append-onlyish-inventory-7jmiua
title: 'append-onlyish-inventory'
desc: ''
created: 1779952090587
updated: 1779952090587
participants: [codex.gpt-5.5]
conversationEventKinds: [message.assistant, message.user, tool.call, tool.result]
---

# User_2026-05-27_2325_48

append-onlyish-inventory

Weave task: start [[wa.task.2026.2026-05-17-append-onlyish-inventory]] with the smallest safe foundation slice.

Goal:
Add a shared RDF-aware inventory append planner with focused tests. Do not refactor production inventory writers yet.

Scope:
- Create a small helper under src/core/weave or src/core/rdf that compares existing Turtle inventory facts with requested settled facts.
- It should classify requested facts as:
  - already present: no-op
  - missing: append needed
  - conflicting with a single-valued settled predicate: fail closed
- Preserve unknown existing triples byte-for-byte in output.
- Use RDF parsing for semantic comparison; do not rely only on string search.
- Add unit tests for duplicate/no-op, append, conflict, and unknown triple preservation.

Non-goals:
- Do not migrate MeshInventory or KnopInventory writers yet.
- Do not move current/latest/next progression facts yet.
- Do not regenerate fixtures.
- Do not introduce repair/retraction UX.

After implementation:
- Run focused tests, `deno task check`, and `deno task lint`; do not run the whole test suite.
- Report the next production writer that should consume the planner.

# gpt-5.5_2026-05-27_2326_03

I’ll take the narrow foundation-slice path: read the repo guidance and the task note first, then trace the existing inventory code and RDF utilities before touching files. After that I’ll add the helper plus focused tests and run only the requested checks.

The task note confirms this slice should only introduce the planner and its unit tests, leaving writer migration for later. I’m also checking the ontology summary now because the local guidance calls that out as part of touching inventory behavior.

I’ve found the existing inventory renderers still have block-rewrite helpers, but there are already RDF parser utilities in `src/core/weave`. For this slice I’m going to keep the new planner separate from those writers so future migrations can adopt it deliberately instead of changing behavior today.

I’ve got the planner shape: parse current and requested Turtle under the same base IRI, compare normalized RDF quads, return a conflict plan with no output when a caller marks a predicate single-valued, otherwise append only the missing requested triples. The existing inventory bytes stay as the prefix of the output, so unfamiliar triples and comments survive untouched.

I’m making the scoped edits now: one new planner module, one focused test file, and a small export so later production writers can consume it without reaching into a private path.

The files are in place. I’m formatting just the touched TypeScript now, then I’ll run the new test file before the broader type and lint gates.

The first test run caught only TypeScript narrowing in the tests, not planner behavior. I’m tightening the assertions so the discriminated union narrows before checking append output.

The focused planner tests are green. I’m moving on to the requested repo gates now: `deno task check` first, then `deno task lint`.

`deno task check` and `deno task lint` both passed. I’m doing a quick diff/status pass now so I can give you an accurate change summary and avoid mistaking any pre-existing work for mine.

Implemented the foundation slice.

Added [inventory_append_planner.ts](../../../../../src/core/weave/inventory_append_planner.ts:1), exported it from [mod.ts](../../../../../src/core/weave/mod.ts:1), and covered it with focused tests in [inventory_append_planner_test.ts](../../../../../src/core/weave/inventory_append_planner_test.ts:1). The planner parses current/requested Turtle with `n3`, classifies requested facts as present, missing, or conflicting, appends only missing facts, and returns no output on single-valued predicate conflicts. Existing inventory bytes remain the exact prefix of appended output, so unknown triples/comments are preserved.

Verification passed:

```bash
deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env src/core/weave/inventory_append_planner_test.ts
deno task check
deno task lint
```

I did not refactor production inventory writers, move progression facts, regenerate fixtures, or add repair UX.

Next production writer I’d point at this planner: [knop_support_renderers.ts](../../../../../src/core/weave/knop_support_renderers.ts:42), especially `renderKnopInventoryWithPreservedSupportArtifacts`. It is already trying to preserve source-registry/reference-catalog facts while rebuilding KnopInventory blocks, so it is the most direct consumer for reducing accidental inventory churn and unknown-fact loss.

Suggested commit message for the `weave` repo:

```text
weave: add RDF-aware inventory append planner

- introduce planInventoryAppend for append/no-op/conflict inventory planning
- compare current and requested Turtle semantically using RDF parsing
- append only missing settled facts while preserving existing inventory bytes
- fail closed on caller-declared single-valued predicate conflicts
- add focused unit coverage for duplicate no-op, append, conflict, and unknown triple preservation
```

Note: the working tree also has pre-existing modified docs under `documentation/notes/` that I did not touch.

# User_2026-05-27_2337_43

Weave next slice: replace block-based carried Knop support preservation with the inventory append planner.

Context:
- Main task note: `dependencies/github.com/semantic-flow/weave-dev-archive/notes/wa.task.2026.2026-05-17-append-onlyish-inventory.md`.
- The shared planner exists in `src/core/weave/inventory_append_planner.ts`.
- Target file: `src/core/weave/knop_support_renderers.ts`, especially `renderKnopInventoryWithPreservedSupportArtifacts`.

Goal:
- Reduce accidental KnopInventory churn and unknown-fact loss around carried support artifacts.
- Replace as much of the subject-block surgery as is safe with RDF-aware append/no-op/conflict planning.
- Keep this slice scoped to preserving/appending support-artifact facts already handled by `renderKnopInventoryWithPreservedSupportArtifacts`: source registry, extraction source, reference catalog, and their located-file/artifact blocks.
- Do not attempt the full KnopInventory progression split yet.

Important caution:
- Current KnopInventory renderers may still write mutable progression facts. Do not blindly treat the entire freshly rendered KnopInventory as settled facts if that would duplicate or conflict with current/latest/next progression.
- Add tests first for the support-preservation behavior we actually want:
  - semantically equivalent carried support facts are no-op
  - missing carried support facts are appended
  - unrelated unknown current facts are preserved byte-for-byte where this helper is responsible for preservation
  - source-registry preservation does not drop reference-catalog facts, and vice versa
  - malformed/conflicting carried support shape fails closed with a useful error

Validation:
- Run the focused `src/core/weave` tests touched by this change.
- Run `deno task fmt`.
- Run broader checks only if exports/types shift.
- Report remaining block-based KnopInventory rewrite paths as follow-up candidates.

# gpt-5.5_2026-05-27_2338_02

I’ll take this as the next narrow migration: first I’ll pin down the existing preservation behavior with focused tests around `knop_support_renderers`, then swap the fragile block insertions for planner-backed append where it fits. I’ll keep progression facts out of the requested append graph.

The key design wrinkle is byte preservation: the planner can classify semantically, but if we want carried unknown support facts to survive byte-for-byte, the append payload needs to come from the existing support blocks instead of a fresh Turtle rendering. I’m going to use the planner for no-op/conflict/missing decisions, then append the original carried support blocks only when the rendered inventory is missing facts from them.

I’m adding the tests before the implementation change. They’ll target the helper directly rather than driving a full weave operation, which keeps this slice sharp and makes failures easy to read.

# User_2026-05-27_2341_07

Yes, use the planner for semantic no-op/conflict/missing decisions, and preserve original carried support blocks byte-for-byte for absent support subjects/subtrees. But don’t wholesale append the current Knop subject block; for the Knop subject, append only the specific missing support-link facts. Also don’t append a carried block if the planner found a single-valued conflict for any fact in that block.

# gpt-5.5_2026-05-27_2341_20

Yes, agreed. I’ll make the Knop subject a small fact-level append surface, while support artifact subjects can be carried as original blocks only when they’re absent and non-conflicting. I’m also filtering mutable progression predicates out of the carried support graph so this slice doesn’t smuggle current/latest/next along.

The initial tests show the current helper already no-ops some equivalent facts, but it still loses unknown source-registry facts, misses single-valued conflicts, and carries mutable progression out of reference-catalog blocks. I’m replacing that center section now with planner-backed requested fact selection and append construction.

The refactor is in. One thing I’m checking now is the exact emitted Turtle shape: because byte-carrying unknown blocks may depend on prefixes that only existed in the current inventory, I added missing prefix declarations only when a byte-preserved block is appended.

The new support-preservation tests are green. I’m running the touched weave unit files together now, then the requested full formatter pass. Since I changed only internal helper code and added tests, I’ll decide on broader type checking after those results.

Formatter passed across the repo. I’m doing a focused lint on just the touched files to satisfy the repo’s “lint after significant edits” rule without widening into the full validation matrix.

Focused lint is clean too. Because this helper feeds `planWeave`, I’m going to include `src/core/weave/weave_test.ts` in the focused pass as the nearby caller coverage, without jumping to the whole suite.

The helper tests, planner tests, and nearby `weave_test.ts` caller coverage all passed. I’m doing a final status/diff scan and then I’ll summarize the remaining block-rewrite paths that look like the next good bites.

Implemented the next slice.

Changed [knop_support_renderers.ts](../../../../../src/core/weave/knop_support_renderers.ts:1) so `renderKnopInventoryWithPreservedSupportArtifacts` now uses `planInventoryAppend` for semantic no-op/missing/conflict decisions. It appends only missing Knop support-link facts for the Knop subject, preserves absent carried support blocks byte-for-byte when safe, fails closed on single-valued conflicts, and filters mutable progression predicates from carried support facts.

Added focused coverage in [knop_support_renderers_test.ts](../../../../../src/core/weave/knop_support_renderers_test.ts:1) for semantic no-op, missing append, byte preservation, source/reference coexistence, conflict failure, and not carrying progression facts.

Validation run:

```bash
deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env src/core/weave/knop_support_renderers_test.ts src/core/weave/inventory_append_planner_test.ts src/core/weave/weave_test.ts
deno task fmt
deno lint src/core/weave/knop_support_renderers.ts src/core/weave/knop_support_renderers_test.ts
```

I did not run `deno task check` because this slice did not shift exports or public types.

Remaining block-based KnopInventory follow-up candidates: [knop_inventory_renderers.ts](../../../../../src/core/weave/knop_inventory_renderers.ts:312) current-only reference/page-definition updates, [src/core/knop/add_reference.ts](../../../../../src/core/knop/add_reference.ts:331) copied source-fact preservation, and [src/core/import/import.ts](../../../../../src/core/import/import.ts:666) source-registry append logic.

Suggested commit message:

```text
weave: use append planner for carried Knop support facts

- route carried source-registry and reference-catalog preservation through the RDF append planner
- append only missing Knop support-link facts instead of rewriting the Knop block
- preserve absent carried support blocks byte-for-byte when they do not contain mutable progression
- fail closed on single-valued support fact conflicts
- add focused unit coverage for semantic no-op, append, conflict, coexistence, and byte preservation
```

# User_2026-05-27_2352_41

Weave next slice: reuse the planner-backed Knop support preservation path in `knop add-reference`.

Context:
- Main task note: `dependencies/github.com/semantic-flow/weave-dev-archive/notes/wa.task.2026.2026-05-17-append-onlyish-inventory.md`.
- The shared append planner exists in `src/core/weave/inventory_append_planner.ts`.
- `src/core/weave/knop_support_renderers.ts` now has planner-backed `renderKnopInventoryWithPreservedSupportArtifacts`.
- `src/core/knop/add_reference.ts` still has a local block-based `renderKnopInventoryWithPreservedSourceFacts` helper.

Goal:
- Replace or retire the local block-surgery preservation helper in `src/core/knop/add_reference.ts`.
- Use the shared planner-backed support preservation path when adding a reference catalog to a KnopInventory that may already carry a source registry or extraction source.
- Keep this slice scoped to `knop add-reference`; do not refactor all KnopInventory renderers.

Expected behavior:
- Adding `_references` must not drop existing `_sources` or `hasExtractionSource` facts.
- Existing source-registry/support subject blocks with unknown facts should survive byte-for-byte where preservation is responsible.
- Semantically equivalent carried facts should no-op.
- Conflicting carried support facts should fail closed.
- Existing `knop add-reference` behavior and tests should keep passing.

Likely files:
- `src/core/knop/add_reference.ts`
- `src/core/knop/add_reference_test.ts`
- possibly `src/core/weave/knop_support_renderers.ts` only if a small API adjustment is needed

Validation:
- Run focused `knop add-reference` and `knop_support_renderers` tests.
- Run `deno task fmt`.
- Report remaining block-based inventory mutation paths as follow-ups, especially `knop/create` and MeshInventory import/integrate paths, but do not broaden this slice.

# gpt-5.5_2026-05-27_2352_54

I’ll keep this one tight around `knop add-reference`: inspect the local helper and current tests, add a regression for byte-preserved `_sources`, then route the add-reference path through the shared support preservation helper.

The existing add-reference tests cover that `_sources` links survive, but not byte-preservation of unknown source-registry facts. I’m going to add that sharper regression, then replace the local helper with the shared `renderKnopInventoryWithPreservedSupportArtifacts` call and trim the now-dead block utilities.

The local block helper is now removed from `add_reference.ts`; add-reference delegates preservation to the shared support renderer and wraps its errors in `KnopAddReferenceInputError`. I added regressions for byte-preserved unknown `_sources` facts and duplicate source-registry conflicts.

The first run showed the old tests were checking for support links embedded inside the Knop subject block. The new contract appends those support-link triples as facts instead, so I’m converting those assertions to semantic RDF checks while leaving the support-block byte assertions intact.

The focused add-reference and shared support-renderer tests are green now. I’m running the requested full formatter pass and a focused lint on the touched files to catch any cleanup from retiring the local helper.

Formatter and focused lint both passed. I’m doing a final diff/status pass so the closeout distinguishes this slice from the prior planner/support changes already in the worktree.

Implemented the `knop add-reference` slice.

`planKnopAddReference` now reuses the shared planner-backed support preservation path via [knop_support_renderers.ts](../../../../../src/core/weave/knop_support_renderers.ts:1), and the local block-surgery helper in [add_reference.ts](../../../../../src/core/knop/add_reference.ts:296) is gone. Shared preservation errors are wrapped back into `KnopAddReferenceInputError`.

Added/updated focused tests in [add_reference_test.ts](../../../../../src/core/knop/add_reference_test.ts:306) for semantic `_sources` preservation, byte-for-byte unknown source-registry facts, and fail-closed duplicate source-registry facts.

Validation passed:

```bash
deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env src/core/knop/add_reference_test.ts src/core/weave/knop_support_renderers_test.ts
deno task fmt
deno lint src/core/knop/add_reference.ts src/core/knop/add_reference_test.ts src/core/weave/knop_support_renderers.ts src/core/weave/knop_support_renderers_test.ts
```

I did not run broader checks because exports/types did not shift.

Follow-up block-based inventory candidates: [knop/create.ts](../../../../../src/core/knop/create.ts:780), [mesh_inventory_renderers.ts](../../../../../src/core/weave/mesh_inventory_renderers.ts:28), [integrate.ts](../../../../../src/core/integrate/integrate.ts:786), and [import.ts](../../../../../src/core/import/import.ts:666).

Suggested commit message:

```text
weave: reuse support append planner in add-reference

- replace add-reference source preservation block surgery with the shared support preservation helper
- preserve carried source-registry blocks and unknown facts through planner-backed append decisions
- keep Knop support links as appended RDF facts instead of rewriting the Knop subject block
- add focused regressions for source preservation and conflicting source registry facts
```

# User_2026-05-27_2359_30

Kim: before we merge PR #20, please do a narrow review-fix pass.

Fix only still-valid PR readiness issues:
- `tests/support/fixture_snapshot.ts`: use `resolved.commitSha` instead of `resolved.gitRef` for `git ls-tree` and `git show`, so snapshot cache reads match the cache key.
- `src/runtime/artifact_resolution/resolver.ts`: latest-state resolution should validate that the resolved `latestHistoricalState` node is typed `sflo:HistoricalState`, matching exact-state behavior.
- `src/runtime/weave/page_generation.ts`: verify/fix `ownerDesignatorPathForPage` so page presentation config uses the page model owner when available, not reconstructed page paths.

For `src/runtime/weave/page_definition.ts` targetLocatedFile/direct targetLocalRelativePath delegation: assess whether this is a quick safe fix or should be filed as a follow-up. Do not broaden the PR if it starts dragging.

in `@src/runtime/weave/page_generation.ts`:
- Around line 186-225: The function ownerDesignatorPathForPage is reconstructing
a designator from page.path (handling _knop and _history) which causes wrong
results for simple pages; instead, use the owning designator already threaded
into the page model: for non-mesh pages return page.ownerDesignatorPath (or
page.designatorPath for kinds that already use it like
identifier/customIdentifier/knop) and stop inferring from page.path in
ownerDesignatorPathForPage; leave the existing special-case branches only if
page.ownerDesignatorPath is undefined as a strict fallback, and ensure
resourcePagePresentationForTarget consumes the ownerDesignatorPath value rather
than any path-derived value.
Validation:
- focused tests for touched files
- `deno task fmt`
- `deno task lint && deno task check` if signatures move
- report whether `codecov/patch` is still the only failing status

# gpt-5.5_2026-05-27_2359_49

I’ll keep this as a narrow review-fix pass: first confirm the repo guidance and local diff, then patch only the three named readiness issues, assess `page_definition.ts` separately, and check PR status at the end. I’m using the GitHub workflow only for the PR/status bit; the code fixes will stay local and focused.

The tree is clean on `next/v0.2.2`, so I’m not stepping on the prior slices. I’m reading the three target files plus nearby tests now, and I’ll add only the narrow regression coverage needed for these fixes.

I’ll keep this pass tight around the three readiness fixes, then do the page-definition assessment separately so it doesn’t balloon. I’ve already confirmed the branch is clean on `next/v0.2.2`; next I’m reading the exact resolver/page model shapes so the tests land in the right layer.

The resolver gap is exactly where expected: latest-state resolves the `latestHistoricalState` IRI and then skips the type assertion that exact-state already performs. I’m checking the page-model union now so the owner-path fix can be tested without accidentally depending on full page generation.

I’m going to make the minimal code edits now: pin fixture snapshot reads to the resolved commit SHA, add the missing HistoricalState assertion for latest-state, and thread an optional owner path onto simple resource page models so presentation lookup can avoid path inference when the model already knows the owner.

The page-generation change needed a tiny model-level addition: simple pages can now carry `ownerDesignatorPath` when assembly already knows the designator context. The resolver and fixture snapshot fixes are both small and have focused regression tests.

Diff looks scoped. I’m running the focused tests now; after that I’ll format, then run lint/check because the page model/interface and exported helper changed.

Focused tests are green: 21 passed across the touched fixture, resolver, and page-generation tests. Formatting is next; then I’ll let lint/check shake out any interface fallout.

Format changed only the resolver line wrapping. Since I exported a helper and extended the page model interface, I’m running `deno task lint && deno task check` now.

Lint and check are clean. I’m rerunning the focused tests once after formatting so the validation chain is tidy.

The page-definition path is still untouched: no diff there. My read is that delegation is not a quick safe fix because the current direct `targetLocalRelativePath` branch intentionally blocks fallback/policy fields, while the shared artifact resolver would accept fallback semantics; that needs a follow-up with explicit behavior tests. I’m checking PR #20 statuses now.

One small polish before I close this: the owner lookup in page-model assembly can be precomputed instead of doing a per-page scan. Same behavior, less needless work.

That cleanup is in, with the root-designator empty string handled explicitly. I’ll rerun fmt and the same validation set.

# gpt-5.5_2026-05-28_0008_18

Implemented the narrow PR readiness fixes.

- [fixture_snapshot.ts](../../../../../tests/support/fixture_snapshot.ts:7): renamed resolved commit to `commitSha` and now uses it for cache keys, `git ls-tree`, and `git show`.
- [resolver.ts](../../../../../src/runtime/artifact_resolution/resolver.ts:910): latest-state resolution now asserts the `latestHistoricalState` target is typed `sflo:HistoricalState`.
- [page_generation.ts](../../../../../src/runtime/weave/page_generation.ts:175) and [page_model_assembly.ts](../../../../../src/runtime/weave/page_model_assembly.ts:131): simple pages can carry `ownerDesignatorPath`, and presentation config lookup now prefers model ownership before falling back to path inference.

Added focused regressions in:
- [fixture_snapshot_test.ts](../../../../../tests/support/fixture_snapshot_test.ts:57)
- [resolver_test.ts](../../../../../src/runtime/artifact_resolution/resolver_test.ts:179)
- [page_generation_test.ts](../../../../../src/runtime/weave/page_generation_test.ts:1)

Validation passed:
- `deno task fmt`
- `deno test -A tests/support/fixture_snapshot_test.ts src/runtime/artifact_resolution/resolver_test.ts src/runtime/weave/page_generation_test.ts`
- `deno task lint && deno task check`

`page_definition.ts` assessment: I left it as a follow-up. Delegating direct `targetLocalRelativePath` through the shared resolver would change currently tested fallback behavior, and `targetLocatedFile` support should land with explicit page-definition behavior tests.

PR #20 status: `codecov/patch` is still the only failing check; Analyze, CodeQL, CodeRabbit, and CI are passing.

Heads-up: `documentation/notes/wd.testing.md` has an unrelated uncommitted diff in the worktree; I left it alone.

Suggested commit message:
```text
Fix PR readiness issues for fixture snapshots and page config

- pin fixture snapshot extraction to resolved commit SHAs
- validate latestHistoricalState nodes as HistoricalState
- thread owner designator paths through simple page models
- add focused regression coverage
```

# User_2026-05-28_0040_24

Please verify and address these CodeRabbit comments against current code. Fix only still-valid issues, keep the diff minimal, and validate.

Context from Jimbo:
- The planner conflict finding is valid, but only for predicates declared single-valued. Do not treat all repeated subject/predicate requested facts as conflicts, because multi-valued predicates like `rdf:type` and `sflo:hasHistoricalState` are legitimate.
- The progression-fact test comment is valid.
- The `add_reference.ts` catch/remap comment is valid but lower priority; fix if cheap.

Tasks:

1. In `src/core/weave/inventory_append_planner.ts`, detect divergent requested facts for the same subject/predicate slot when that predicate is in `singleValuedSettledPredicates`.
   - Track previously seen requested facts keyed by `toSubjectPredicateKey(requestedQuad)`.
   - If the slot was already requested with a different object, push a conflict using the existing `InventoryFactConflict` shape and `formatConflictMessage`.
   - Continue to allow duplicate identical requested facts to dedupe/no-op as today.
   - Continue to allow multi-valued predicates to request multiple objects.

2. Add a regression in `src/core/weave/inventory_append_planner_test.ts`.
   - Current inventory can be empty or unrelated.
   - Requested Turtle should contain two different objects for the same subject and single-valued predicate, e.g. `sflo:hasPayloadArtifact`.
   - Assert `plan.kind === "conflict"` and the message mentions both requested objects.
   - Also add or preserve coverage showing multi-valued repeated predicates still append/behave normally if needed.

3. In `src/core/weave/knop_support_renderers_test.ts`, change the mutable progression fact absence test to parse output Turtle and assert semantically:
   - no quad predicate equals `${SFLO_NAMESPACE}currentArtifactHistory`
   - no quad predicate equals `${SFLO_NAMESPACE}nextHistoryOrdinal`
   This replaces string checks like `output.includes("sflo:currentArtifactHistory")`.

4. In `src/core/knop/add_reference.ts`, narrow the catch around `renderKnopInventoryWithPreservedSupportArtifacts`.
   - Import `WeaveInputError` from `../weave/errors.ts`.
   - Remap only `WeaveInputError` to `KnopAddReferenceInputError`.
   - Rethrow all other errors unchanged so renderer bugs preserve original type/stack.

Validation:
- Run `deno test -A src/core/weave/inventory_append_planner_test.ts src/core/weave/knop_support_renderers_test.ts src/core/knop/add_reference_test.ts`.
- Run `deno fmt --check` on touched files.
- Run `deno task lint && deno task check` if the changes touch imports/types.

# gpt-5.5_2026-05-28_0040_36

I’ll verify the comments against the current tree first, then only patch the ones that still apply. Since the worktree already has prior PR-readiness edits, I’ll keep this new pass tightly scoped and avoid touching unrelated files.

# gpt-5.5_2026-05-28_0041_05

I confirmed the planner currently only checks requested facts against existing inventory, so the “two divergent requested objects for a single-valued slot” case is still open. The progression test is still string-based, and the add-reference remap is catching all errors, so all three comments are still valid.

# gpt-5.5_2026-05-28_0041_29

I’ll patch this in three small spots: planner slot tracking, semantic predicate assertions in the Knop support test, and the catch narrowing in `knop add-reference`. The planner change will only activate for predicates the caller declares single-valued.

# gpt-5.5_2026-05-28_0043_04

The patches are in. I’m going to let Deno format/check the touched files, then run the requested focused tests; because `add_reference.ts` gained an import, I’ll run lint/check after the focused tests.

# gpt-5.5_2026-05-28_0043_16

Focused tests are green: 26 passed. Since the import/type surface changed a little, I’m running lint and check now.

# gpt-5.5_2026-05-28_0043_47

Implemented the still-valid CodeRabbit fixes.

- [inventory_append_planner.ts](../../../../../src/core/weave/inventory_append_planner.ts:78): now detects divergent requested facts for the same subject/predicate only when the predicate is declared single-valued.
- [inventory_append_planner_test.ts](../../../../../src/core/weave/inventory_append_planner_test.ts:90): added regression coverage for divergent `sflo:hasPayloadArtifact`, plus multi-valued `sflo:hasHistoricalState` still appending normally.
- [knop_support_renderers_test.ts](../../../../../src/core/weave/knop_support_renderers_test.ts:184): progression absence test now parses Turtle and checks predicates semantically.
- [add_reference.ts](../../../../../src/core/knop/add_reference.ts:10): narrowed renderer error remapping to `WeaveInputError`; other errors rethrow unchanged.

Validation passed:
- `deno fmt --check src/core/weave/inventory_append_planner.ts src/core/weave/inventory_append_planner_test.ts src/core/weave/knop_support_renderers_test.ts src/core/knop/add_reference.ts`
- `deno test -A src/core/weave/inventory_append_planner_test.ts src/core/weave/knop_support_renderers_test.ts src/core/knop/add_reference_test.ts`
- `deno task lint && deno task check`

Unrelated dirty docs remain untouched: `documentation/notes/wd.todo.md` and untracked `documentation/notes/wd.vision.md`.

Suggested commit message:
```text
Fix append planner requested-fact conflicts

- detect divergent requested objects for declared single-valued inventory slots
- preserve multi-valued requested predicate behavior
- assert Knop support progression omissions semantically
- narrow knop add-reference renderer error remapping
```