---
id: waprogversionapi20260721
title: 'Programmatic version API — pass-bytes-and-designator, in-process'
desc: 'Cut 2026-07-21 13:22 by the Stagecraft flagship PM seat (cross-shop lane, Dave-authorized). The 2026-07-06 deferred pass-bytes programmatic `version` API; wake fired at the Stagecraft press panel. Deliverable: the API itself, spec''d and landed in weave.'
---

## Goals

- A stable, named, in-process Weave library API that lets a consumer say: **"here are payload bytes and a designator path (or payload IRI); record them"** — working-surface update and/or history versioning — with exactly the guarantees the CLI path gives (single-writer, fail-closed validation, snapshot verification, retry-safe no-op semantics), and **no subprocess spawn, no temp-file requirement on the caller**.
- The product-path law this serves (ruled, Stagecraft 2026-07-21): **consumer product paths call the API, never exec the CLI.** The standing counterexample: `weave` was absent from the consuming host's PATH until 2026-07-21 morning — CLI-spawn is a deployment coupling, not an API.

## Summary

Weave already versions payloads two-step: `weave payload update <file> <designatorPath>` replaces working bytes from a FILE source; `weave version --target designatorPath=...` records targeted working state into history via the version-plan machinery (`src/core/payload/update.ts`, `src/core/payload/version_intent.ts`, `src/core/weave/version_plan.ts`), with the CLI (`src/cli/run.ts`) owning invocation. `src/mod.ts` re-exports the core wholesale, so internals are *reachable* as a library today — but nothing is a *named, stable, documented* programmatic contract, and the file-source assumption in `payload update` forces callers to stage temp files.

This task was deferred on 2026-07-06 (Stagecraft disc "semantic-flow-in-the-platform" §6: no new API surface until a real trigger) and the wake fired 2026-07-21: the Stagecraft checkpoint press is the natural weave-version moment (publication cadence == version cadence; an unwoven pressed checkpoint is a second-class mesh state against SF-serialization doctrine), and the srd-rules publication estate now execs the CLI from its pipeline.

**Consumers, in ruled order:** (1) the `stagecraft/srd-rules` mesh estate's extraction/publication pipeline; (2) the Stagecraft checkpoint press — whose consumption is **explicitly not this task**: it folds into the Stagecraft press-completeness lane later, via the dependency-PIN idiom (full-commit pin + out-of-gate build + contract check). The deliverable here is the API itself, landed in weave.

## Discussion

- **Pass-bytes semantics vs the working-file capture contract.** The CLI's multi-target batch hashes the *working payload files* before batch capture and verifies after; a changed file refuses the whole batch. A bytes-in API changes what "the working surface" means at capture time: the API presumably (a) writes the passed bytes to the working payload file and (b) versions from that state — but whether (a)+(b) are one atomic composed call, two composable calls, or both-offered is the central spec question. The refusal/verification semantics must be restated precisely for the bytes-in path — nothing weakens.
- **The mesh stays file-backed.** Pass-bytes means the CALLER passes bytes; the mesh on disk remains the truth the API mutates. This is not an in-memory mesh.
- **Single-writer enforcement in-process.** The CLI's single-writer guarantee is process-shaped today. A library caller (long-lived daemon like the Stagecraft session authority, or a pipeline script) needs a stated concurrency contract: what serializes, what refuses, what happens on concurrent CLI use of the same mesh.
- **Return contract.** Callers need machine-usable results: the applied version-plan (created/updated files, new state segment, the payload artifact IRI, the current-history path), and typed refusals. `VersionPlan`/`PayloadVersionIntentPlan` are the natural seeds.
- **Behavior-spec law.** This is externally visible behavior spanning subsystems — per `wd.general-guidance`, an `sf.spec.*` note in semantic-flow-framework should ride the landing (proposed; spec review confirms scope).
- **Known 0.3.0 operational defects are SEPARATE work** (boarded on the Stagecraft side after the srd-rules phase-A run): `extract --all-terms` file-URL crash; extracted-term Knops missing `sflo:hasResourcePage` claims; whole-mesh `validate` heap exhaustion at ~6,900 files. Not in scope here unless the spec round proves one trivially adjacent — say so explicitly if so.

## Open Issues

*(for the spec review to attack; proposed leans in bold)*

- [ ] API shape: one composed `version(bytes, designator, opts)` vs composable `updatePayload(bytes, designator)` + `version(targets, opts)` pair. **Lean: the composable pair, mirroring the CLI's proven two-step, plus a thin composed convenience.**
- [ ] Batch surface: does the v1 API carry multi-target batches with the same all-or-nothing refusal, or single-target only? **Lean: mirror the CLI batch contract — the press's checkpoint moment is inherently multi-target.**
- [ ] Naming/segments: how `historySegment`/`stateSegment`/`manifestationSegment` and mesh-config defaults surface in the API signature. **Lean: one options object mirroring the CLI's precedence rules exactly.**
- [ ] Error taxonomy: typed refusals vs thrown errors; what the CLI's diagnostics become programmatically.
- [ ] The stable-surface boundary: what gets NAMED in `mod.ts` as the public API vs what stays incidental export. **Lean: a dedicated `src/api/` (or `core/api/`) module with an explicit export list; the wholesale re-export is not a contract.**
- [ ] Binary payloads: the press moment includes non-text artifacts (`PlannedBinaryFile` exists) — bytes-in must state its encoding contract (Uint8Array; no string-only API).
- [ ] `sf.spec.*` note: which repo lands it and whether it gates this task's close or rides as a fast-follow.
- [ ] Version/release: does this ship as 0.4.0? (`deno task bump:version`; release-note convention per `wd.release-runbook`.)

## Decisions

- **2026-07-21 (ruled at cut, Dave-concurred relay):** the task lives in the weave world under weave's own conventions; the Stagecraft lab keeps only a pointer. The portable seat-separated lifecycle runs here exactly as in the Stagecraft shop (detached spec review → adjudication on this note → build seat in a weave working clone → close review).
- **2026-07-21 (product-path law, standing):** consumers call the API; the CLI is for humans and scripts, never a product path's mechanism.
- **2026-07-21 (consumer order):** srd-rules estate first, Stagecraft press later via its own lane; press-side consumption is out of scope here.

## Contract Changes

- New named public programmatic API surface (module + explicit exports), documented in `wd.*` (developer) and referenced from the CLI reference notes where the CLI and API share semantics.
- No CLI behavior change; no mesh-format change; no daemon/HTTP surface.
- Docs: `wd.codebase-overview` updated before merge (standing law); release note entry.

## Testing

- Deno-native (`deno task test` / `check` / `lint`; `deno task fmt` + `deno task ci` before merge) per [[wd.testing]].
- **Equivalence proof (the load-bearing family):** on a fixture mesh, the API path and the CLI path produce byte-identical version output for the same inputs (single and batch), including the no-op re-run and the refusal families (malformed target, changed-under-capture, malformed batch member refusing the whole plan).
- Fail-on-old probes: each preserved invariant gets a test that demonstrably fails without the guard.
- Binary payload round-trip.
- No consumer-side (Stagecraft) tests in this task.

## Non-Goals

- Press-side consumption wiring (Stagecraft press-completeness lane owns it, later, via the PIN idiom).
- Node packaging / consumer build machinery / JSR publication decisions beyond what landing the API in this repo requires.
- CLI retirement or change; daemon/HTTP API; in-memory mesh.
- The boarded 0.3.0 defect trio (all-terms crash, missing page-claims, whole-mesh validate heap) — separately tracked.

## Implementation Plan

*(draft — the build seat refines after the spec round adjudicates)*

- [ ] Spec review r1 (detached seat, adversarial) → adjudication on this note → amend until converged.
- [ ] Carve the named API module + explicit exports; restate capture/refusal semantics for bytes-in.
- [ ] Implement composable calls (+ composed convenience if ruled); typed results/refusals.
- [ ] Equivalence + refusal + no-op + binary test families; fail-on-old probes.
- [ ] Docs (`wd.codebase-overview`, API doc note, CLI-reference cross-links); `sf.spec.*` per the ruled scope.
- [ ] `deno task fmt` + `deno task ci`; release-note entry; close review (fresh seat) → adjudication → land.

## Spec review

*(r1 appends here — PROPOSED verdicts/severities; execution blocked until PM adjudication signs off.)*

## Receipts

*(build seat appends; append-only; anything skipped is named.)*
