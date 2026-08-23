---
id: 2aff38eb-1f21-4755-886b-ec5b76c7e6ff
title: Stagecraft Phase 1 Claude Review
desc: 'Read-only Claude Opus max-effort review of the completed knop.create append-planner/indexed-read-model Phase 1 working tree'
created: 1787443907000
---

## Scope

Read-only Claude Opus max-effort review of Phase 1 in [[wa.plan.2026.2026-08-22_1550-stagecraft-iri-initialization]] against its brief and receipts in [[wa.task.2026.2026-05-17-append-onlyish-inventory]]. The review inspected the current uncommitted Weave/archive diff, append planner, create inventory index, compact rendering, unit/integration/e2e coverage, scale probe, CSV rows, and before/after claims. No files were modified by the reviewer.

## Reviewer Validation

- Focused `deno check`, `deno lint`, `deno fmt --check`, and both repositories' `git diff --check` passed.
- Append-planner/index/core-create tests: 16 passed.
- Integration create and scale-probe tests: 6 passed.
- E2E create tests: 4 passed.
- Two adversarial carried-inventory inputs were exercised through real `planKnopCreate` and exposed the blocker below.
- Full `deno task ci` was not rerun by the reviewer; the implementation receipt's 847-pass CI claim remains independently unverified by this review.

## Required Evidence

| Evidence | Review result |
| --- | --- |
| Reusable probe plus N=552 before receipt | Recorded, but chronology is self-reported because both rows share one HEAD and no intermediate commit exists. |
| Unknown predicate/comment byte preservation | Proven by core and E2E fail-on-old tests. |
| Semantic duplicate no-op | Proven at helper boundary; public create still correctly refuses an already registered Knop. |
| Single-valued conflict | Proven; `hasWorkingKnopInventoryFile` matches the SHACL maximum-one contract. |
| No legacy block helpers in migrated path | Proven by current source absence, not an executable guard. |
| Indexed membership | Partly proven: index consumes its iterable once, but the call site is not instrumented and planner grouping still adds full passes. |
| Existing create/root/fixture behavior | Proven by 26 focused/integration/E2E tests run by reviewer. |
| Like-for-like after receipt | Recorded honestly, but one sample per arm gives no variance. |
| Appended bytes equal planner-approved RDF | Disproven by the blocker. |

## BLOCKING

### B1. Compact suffix rendering can disagree with the append plan

`renderCompactKnopCreateInventoryAppend` in `src/core/knop/create.ts` discards `plan.appendTurtle` and manually reconstructs a suffix with `sflo:` prefixed names and mesh-relative IRIs. It does not prove that the current document declares `sflo:` or that its effective `@base` equals `meshBase`.

The reviewer reproduced two fail-open cases through real `planKnopCreate`:

- A valid carried inventory using full SFLO IRIs and no `sflo:` declaration produces output that fails Turtle parsing with an undefined prefix.
- A valid carried inventory with `@base <https://elsewhere.example/>` produces parseable output whose appended facts denote `elsewhere.example` resources rather than the planner-approved mesh IRIs.

The old block renderer failed closed on the second shape because it could not find the expected mesh-relative subject block. The new path therefore regresses fail-closed behavior.

The same hand renderer matches known facts by formatted strings. A later sixth or literal-valued requested fact could be planner-approved yet silently omitted from the output.

**Required fix:** parse the rendered output using `meshBase` and prove its quad set equals `currentInventoryQuads ∪ plan.missing`; on mismatch use `plan.outputTurtle`'s absolute-IRI suffix or fail. Add the two reproduced cases plus coverage-drift protection as fail-on-current tests.

## MAJOR

### M1. The first-Knop renderer still deletes settled facts

The legacy/first-Knop path still replaces the entire MeshInventory with a fixed template. Fixture forensics show the Alice `a.03` mesh config support graph disappears at `a.04` and never returns. This pre-existing path is allowed by the Phase 1 brief, but it is the path every fresh mesh uses for its first Knop and is a larger deletion than the migrated later path.

Record it as an owned residual and scope Gate G1 wording explicitly to the later/current create path; do not claim all of `knop.create` is append-onlyish.

### M2. E2E RDF equality was weakened to containment

The E2E test moved MeshInventory from isomorphism to containment to allow preservation of a carried config fact absent from stale expected fixtures. Containment cannot detect spurious appended facts—the main new-writer failure mode. Assert exact equality against `expected ∪ deliberately carried facts`, and record fixture drift as owed work rather than weakening the acceptance oracle indefinitely.

### M3. Pre-parsed append-planner input lacks invariant and tests

The new optional `currentInventoryQuads` skips parsing `currentInventoryTurtle`, although output bytes still derive from the string. A disagreement can yield wrong append/no-op/conflict decisions. The production caller currently supplies matching values, but the invariant is undocumented and the planner's new arm has no direct tests.

Document the equality/provenance invariant and test matching and mismatching inputs, or narrow the API so the caller cannot supply inconsistent representations.

### M4. A second bespoke compactor duplicated the known residual

The prior append-onlyish receipt said to extract shared compact suffix handling at the second/third consumer. The ReferenceCatalog consumer post-processes `plan.appendTurtle`, while create regenerates facts from scratch and caused B1. Fix through one shared, fact-preserving compaction path or use absolute planner output; record any deferred serializer work explicitly.

## ADVISORY

- Deduping named-node objects silently relaxes duplicate-singleton refusal while literal duplicates still refuse. Rule/test that asymmetry.
- The create index ignores graph while the planner keys by graph; reject non-default graph input or document/test the retained behavior.
- One N=552 run per arm supports only “no demonstrated speedup,” not precise percentage regression claims.
- The new path performs index, quad-key, and subject-predicate grouping passes; reusing the index inside planner comparison may be the next optimization before drawing causal conclusions.
- Add a runtime test proving a single-valued conflict creates zero files and leaves MeshInventory byte-identical.
- Runtime still writes updated files unconditionally even when planner output is unchanged.
- The trailing-newline residual was touched but neither fixed nor re-boarded.
- Update `wd.todo` and `wd.codebase-overview`; the former still calls `knop.create` wholly unmigrated and the latter omits the planner/index.
- No executable guard prevents future block-helper reintroduction.

## NIT

- Scale-probe test may leak its temp directory on early failure and bypasses the shared temp harness.
- `createElapsedMs` includes the final `Deno.stat`.
- Probe lacks `--help`.
- `@internal` on an exported planner helper is not mechanically enforced.
- `indexedQuadCount` exists only for its test.
- Empty carried input can yield prefix-less output, currently unreachable but unguarded.

## Scope Discipline

Clean. Production/test changes stay within the Phase 1 slice; no page, reference, extract/integrate, payload/history, progression, remote, or fixture-topology implementation was added. Deleted private block helpers are genuine dead-weight removal.

## Verdict

**GO WITH CHANGES.** B1 must be fixed before merge. M1–M4 must be fixed or recorded as named residuals with owners before Gate G1 is marked passed.
