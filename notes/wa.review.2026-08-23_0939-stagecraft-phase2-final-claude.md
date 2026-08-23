---
id: 5d7da904-0f8d-4da5-a39e-0f9d12c845a4
title: Stagecraft Phase 2 Final Claude Review
desc: 'Recovered final implementation review, hardening disposition, and committed-state G2 verification'
created: 1787503185000
---

## Scope

Final read-only Claude Opus review of the complete FoundingReferentData implementation for Gate G2 in [[wa.completed-plan.2026.2026-08-22_1550-stagecraft-iri-initialization]], followed by a bounded committed-state verification after hardening. The reviewed contract/runtime state is SFLO `cf7e79a5`, Semantic Flow Framework `e6d8bdd`/`f391813`, and Weave `1b2f080`/`54f7f7c` plus hardening `06da19c`. Reviewers were not allowed to modify files, refs, configuration, or repository state.

The initial final review completed in the recovered Codex session after archive commit `ec3e5e2`. It returned GO with no blocker or major, found six non-landing minors, and ruled G2 eligible to pass. The interrupted implementation session had announced that it would fix minors 1–5 before recording closure but ended before any edit began. The resumed session implemented those five fixes as `06da19c`, ran full CI, and obtained a second read-only verification against the exact commit.

## Initial Findings And Disposition

- **M1 fixed:** publication FoundingReferentData validation no longer swallows missing or unparseable registered KnopInventory files; it emits `missing-artifact` or `malformed-inventory`.
- **M2 fixed:** `versionFoundingReferentData` maps residual planner failures into the stable `WeaveApiError` base with `malformed-mesh` rather than leaking internal error types.
- **M3 fixed:** atomic rollback tracks and removes only operation-created empty directories, preserving pre-existing empty ancestors.
- **M4 fixed:** `executeKnopCreateForTesting` is withheld from the documented public barrel chain.
- **M5 fixed:** both founding source readers are covered for a ruled repo-adjacent workspace path and for workspace-escape refusal through the real local-path policy boundary.
- **M6 accepted residual:** `advanceHistoryProgression` still uses a fail-closed compact-`sflo:` regex rewrite. Representation-independent progression hardening is boarded in [[wd.todo]] and does not reopen this capability gate.

## Verification Evidence

- Weave hardening commit: `06da19c` over reviewed parent `54f7f7c`; all repositories were clean before verification.
- Focused regression gate: 19/19 tests green.
- Full Weave `deno task ci`: format, lint, repository-wide type checks, 887/887 coverage tests, and LCOV generation green.
- Fail-closed validation findings flow into both publication and ordinary validation result assembly, and registered Knop paths originate from MeshInventory membership.
- The public error catch closes the production leak; no core/runtime module throws `WeaveApiError`, so no existing public error is double-wrapped.
- Rollback records missing parents before writes, removes completed created files first, and then removes only its recorded directories deepest-first. Update parents already exist by preflight.
- The public barrel chain reaches `src/runtime/knop/mod.ts`, which explicitly exports every intended create symbol except the testing seam.
- Source-policy coverage reaches the actual `LocalPathAccessError` branch rather than failing incidentally at a different guard.

## Follow-Up Advisories

- The named-graph API regression proves the public typed malformed-inventory mapping but does not uniquely force the new catch-all branch; the production mapping is still correct.
- The export regression asserts the leaf public barrel rather than the repository root barrel; the current barrel chain was independently inspected and is correct.
- Both source readers delegate to the same private policy function, so their looped coverage proves both exported surfaces over one implementation path.
- Directory recording has the expected no-concurrent-writer TOCTOU boundary already excluded from the atomicity claim; external non-empty contents are preserved.

These are test-precision or already-declared concurrency advisories, not shipped-behavior defects. They do not reopen G2.

## Gate Disposition

Gate G2 passes. All original contract-review dispositions remain closed, the complete capability satisfies the task exit criteria, and the final implementation/hardening state has no blocker or major finding.

Gate G3 remains open. No Stagecraft wall-clock or peak-memory budget was supplied, and the descriptive N=552 receipt does not itself justify a batch task. Dave must rule singular accepted or batch required against an explicit consumer budget.

## Verdict

**GO.**
