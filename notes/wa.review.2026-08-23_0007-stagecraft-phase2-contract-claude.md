---
id: b9b6dca0-690f-41d1-b1fc-831446ab1214
title: Stagecraft Phase 2 Contract Claude Review
desc: 'Read-only SFLO and Semantic Flow Framework review before FoundingReferentData runtime implementation'
created: 1787468820000
---

## Scope

Read-only Claude Opus review of committed SFLO `cf7e79a5` and the Semantic Flow Framework contract slice that became `e6d8bdd`, against [[wa.completed-plan.2026.2026-08-22_1550-stagecraft-iri-initialization]] and [[wa.completed.2026.2026-08-22_1112-founding-referent-data]]. No reviewer file, ref, or repository mutation was allowed.

The review checked ontology/SHACL structure, the bounded portable profile, initialization/settlement/correction lifecycle, payload-only `versionPayloads` boundary, prohibited scope, and the founding-created/founding-versioned/founding-corrected Accord sequence.

## Evidence

- SFLO `FoundingReferentData` is a `DigitalArtifact` plus `RdfDocument`, not a `SemanticFlowResource`.
- `hasFoundingReferentData` is an optional Knop slot; no Stagecraft predicate, generic `ReferentMetadata`, page contract, root support, remote fetch, batch work, or ontology version-metadata change entered SFLO.
- The portable spec carries the exact flat-document limits, lexical subject rule, forbidden vocabulary rules, binary preservation, no-network/no-page boundary, initial settlement, later-state correction, validation findings, and preservation requirements.
- `versionFoundingReferentData({ meshRoot, designatorPath, bytes? })` is separate and `versionPayloads` remains payload-only.
- All three Accord manifests are registered in one `27-carol-woven` → created → versioned → corrected sequence and preserve state 1 by byte expectation and digest.
- Framework manifests and the scenario index passed Accord SHACL validation after disposition edits.

## Findings And Disposition

### Blocker

- **B1 accepted as the first Weave implementation bite.** Accord's valid `inlineSource`/`inlineValue` provenance is not materialized by the current Weave fixture-ladder runner, which incorrectly falls through to an asset lookup. The manifests retain self-contained digest-declared inline bytes to avoid an unruled fifth-repository assets-branch mutation. Weave must add exact inline materialization before replaying the three rungs; FoundingReferentData runtime code does not precede that fix.

### Major

- **M1 fixed.** The created transition's directory-shaped history absence expectation now targets the exact state-1 snapshot path, so it cannot pass vacuously against Git's blob map.
- **M2 fixed.** The create and correction transitions explicitly ignore temporary `.accord/**` admission staging paths so tree completeness covers only the intended fixture surface.

### Minor

- **N1 fixed.** All three manifests explicitly assert that the mutable working `data.ttl` has no standing digest.
- **N2 no change, ruled.** The reviewer noted that the current CLI does not yet implement positional `weave version <D> --artifact-role founding-referent-data`. Dave's R3 ruling and the authoritative Phase 2 handoff already settle that exact new surface; runtime implementation is the task, not an open contract decision.
- **N3 no change, ruled.** Warning severity for the optional founding slot matches the task's explicit SHACL requirement and sibling optional Knop slots.
- **N4 no change.** Founding settlement/correction advances KnopInventory, not MeshInventory, so the versioned/corrected manifests correctly omit a MeshInventory change expectation.

### Advisory

- The two declared source digests were independently recomputed from the exact inline bytes and match the state assertions.
- Negative profile, unsettled-working, and digest-mismatch behavior remains TypeScript/integration coverage rather than additional Accord rungs in this ruled three-transition slice.
- Document-local content constraints deliberately remain operation-level rather than union-graph SHACL.

## Verdict

**GO WITH CHANGES.** The semantic contract is sound. The required manifest corrections are folded, and the sole remaining blocker is explicitly owned as the first Weave/harness implementation change before the three rungs are replayed.
