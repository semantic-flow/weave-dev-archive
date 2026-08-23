---
id: 75fe9ae9-b1a4-4186-89b8-8ad3e0f12416
title: Stagecraft Phase 1 Final Claude Review
desc: 'Final read-only Claude Opus review of the review-safe knop.create append migration, including repeated N=552 performance attribution'
created: 1787455935000
---

## Scope

Final read-only Claude Opus max-effort review of Weave `9a2ad7f` and archive receipt `0d0a4d6` for Phase 1 of [[wa.completed-plan.2026.2026-08-22_1550-stagecraft-iri-initialization]]. The review verified closure of [[wa.review.2026-08-22_1711-stagecraft-phase1-claude]], independently reran focused/full gates and repeated scale measurements, probed adversarial RDF shapes, and evaluated whether the measured slowdown should trigger batching. No repository files were modified by the reviewer.

## Reviewer Validation

- Format, lint, type-check, both repositories' `git diff --check`: green.
- Full test suite: 857 passed, 0 failed.
- Focused Phase 1: 35 passed; create E2E: 4 passed.
- N=552 repeated three times for legacy and review-safe trees; receipts reproduced.
- N=1104 comparison confirms quadratic aggregate growth in legacy and migrated singular paths.
- Temporary suffix-proof and directive experiments ran outside the working tree.
- End-to-end `deno task ci` was not run only because LCOV writes into the repository; its format/lint/check/test components were independently confirmed.

## Prior Review Closure

Every B1/M1–M4 finding from r0 is closed on its own terms:

- The exact-union renderer derives from planner output, is self-contained, and fails closed on semantic mismatch.
- First and later Knop creation share the append path; the fixed first-Knop template is gone and carried mesh config survives.
- E2E uses exact RDF equality with an explicitly carried config union; stale fixture regeneration is named.
- `PreparedCurrentInventory` makes string/quads consistency structural and has append/no-op/conflict/named-graph tests.
- Create and ReferenceCatalog share `renderInventoryAppendPlan`.
- Prior duplicate, graph, zero-write conflict, probe, newline, export, and documentation findings are fixed or explicitly owned.

## New Major Findings

### N1. Carried blank nodes cause a false semantic mismatch

`renderedAppendMatchesPlan` reparses the complete current-plus-append document and compares quad keys with the independently parsed prepared current inventory. N3 allocates fresh blank-node labels per parse, so isomorphic carried blank nodes receive different internal identifiers and every append fails closed.

This is no corruption, but it rejects previously valid unknown carried RDF and conflicts with Phase 2's user-RDF direction.

**Fix:** use the suffix-only proof below or compare complete graphs by blank-node isomorphism. Add the reproduced carried blank-node case.

### N2. Absolute fallback and terminal failure branches are untested

The two branches that provide the B1 safety net lack direct coverage. Use known edge IRIs such as `<meshBase>//evil.example/thing` and `<meshBase>sub/../up` to prove compact diversion to absolute fallback and terminal refusal behavior.

### N3. E2E parser reuse can leak base state

The exact E2E oracle reuses one N3 `Parser` across actual, expected, and carried documents. N3 retains `@base` between parses, so a rogue base in actual output can reinterpret expected RDF and partially mask the original B1 class.

**Fix:** instantiate one parser per document, as the core test already does.

### N4. Governing plan/review/founding notes are absent from committed archive history

Archive receipt `0d0a4d6` links the active plan, first Claude review, and FoundingReferentData task, but those notes remain untracked. Gate G1 cannot pass on committed governance that points to nonexistent committed notes.

**Fix:** the planning seat commits the plan, reviews, founding task, schema/template convention, and affected guidance together; do not leave the receipt ahead of its owners.

### N5. The 1.9× regression is largely recoverable and must be removed before Phase 3 measurement

Repeated like-for-like measurements:

| Arm | Wall time | Mean create loop | Final MeshInventory |
| --- | --- | ---: | ---: |
| Legacy | 1.65 / 1.64 / 1.70 s | 1,515 ms | 150,713 B |
| Review-safe | 3.35 / 3.25 / 3.23 s | 3,130 ms | 220,817 B |
| Suffix-only proof | 2.16 / 2.03 / 2.02 s | 1,918 ms | 220,817 B |
| Suffix proof plus safely amortized directives upper-bound experiment | 1.84 / 1.80 / 1.74 s | 1,639 ms | 154,577 B |

The full-candidate reparse costs roughly 1.21 seconds. Repeated self-contained directives cost roughly 0.28 seconds and 66,240 bytes. Together they account for about 92% of the observed regression.

This is not evidence for batch creation: all singular arms remain quadratic in aggregate, and the absolute N=552 cost is small. The removable verification/serialization overhead must be addressed before the founding-data Phase 3 receipt so that receipt measures founding cost rather than Phase 1 safety overhead.

## Suffix-Only Proof Ruling

Parsing only the self-contained append chunk and proving its quads equal `plan.missing` is sound for the current construction because:

- prepared current input is already a parsed complete document and its bytes remain an exact prefix
- Turtle directives are forward-only and cannot reinterpret the prefix
- the append chunk is self-contained
- requested append facts forbid blank nodes, so blank-node scope cannot cross the join
- `appendToCurrentTurtle` prevents token merging
- RDF set semantics give `parse(prefix + suffix) = preparedCurrent ∪ parsedSuffix`

Keep complete-output reparse on the cold fallback path and in tests as defense in depth. This suffix proof fixes N1 and recovers the dominant parse cost.

Do not remove repeated directives naively: that fails the no-prefix/different-base tests. If directive overhead is optimized, `prepareCurrentInventory` must capture/prove the effective trailing base/prefix state before emitting a directive-free compact suffix. Otherwise retain self-contained directives and accept the remaining cost.

## Advisory

- Mid-document directives are harmless in current tests but are a trap for remaining string-based writers.
- `RenderInventoryAppendPlanInput` still pairs a prepared inventory and plan without a construction-time brand; mismatch fails closed, so this is latent.
- Reusing index structures for remaining linear passes is lower priority than removing the full reparse.
- `Object.freeze(quads)` does not freeze individual Quad objects.

## Gate And Batch Disposition

- The slowdown is an owned residual, not a merge blocker by itself.
- N1–N4 must be fixed or explicitly owned before G1 passes.
- Suffix-only proof and any safe directive amortization must land before the Phase 3 founding-data receipt, but do not block SFLO/framework or Founding runtime implementation strictly.
- Do not carve a batch task from this evidence. The legacy path is also quadratic, and Stagecraft's operational budget remains the G3 decision input.

## Verdict

**GO WITH CHANGES.** Gate G1 may pass after N1–N4. Phase 3 measurement must not use the inflated review-safe path; land the suffix-proof optimization first and decide directive amortization through an evidence-backed, base-safe design.
