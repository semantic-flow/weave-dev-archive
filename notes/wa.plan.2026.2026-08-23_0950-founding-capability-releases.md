---
id: 1fd7b84b-2a95-4562-98d2-87226dc78c38
title: Founding Capability Releases
desc: 'Coordinate SFLO v0.5.0, the portable contract push, and Weave/weave-lib v0.9.0 publication'
created: 1787503800000
---

## Status

Active coordination plan. Dave instructed immediate release on 2026-08-23. The child tasks execute directly under that instruction and do not enter [[wd.queues]].

## Goals

- Publish the FoundingReferentData vocabulary and validation contract as SFLO v0.5.0 before releasing a Weave implementation that emits those terms.
- Push the matching Semantic Flow Framework behavior spec and executable Accord transitions.
- Publish the complete Stagecraft IRI-initialization capability as Weave v0.9.0 across native CLI packages, archives, and `@semantic-flow/weave-lib`.
- Prove the packed npm library exports and executes `versionFoundingReferentData` byte-equivalently to the source API under off-tree Node, rather than inferring parity from the source barrel.
- Preserve exact release receipts and leave every repository clean.

## Summary

The capability is committed and passed Gate G2 under [[wa.plan.2026.2026-08-22_1550-stagecraft-iri-initialization]]. Dave's instruction to release the measured singular implementation rules Gate G3 as singular accepted; no batch task is owed for this release.

Release ordering is contractual, not cosmetic. Weave now emits `sflo:FoundingReferentData` and `sflo:hasFoundingReferentData`, while published SFLO v0.4.0 and the current Pages mesh do not define or dereference those terms. SFLO v0.5.0 source and Pages publication therefore gate Weave v0.9.0. The Framework repository has no package release line, but its contract commits must reach canonical `main` before publication.

`@semantic-flow/weave-lib` is built from `src/api/mod.ts`, which already exports `versionFoundingReferentData`; however the existing packed off-tree Node smoke covers only `versionPayloads` and `validateMesh`. The Weave child task adds a real founding settlement/correction parity receipt before rehearsing or publishing.

## Child Tasks

| Order | Owning artifact | Role |
| --- | --- | --- |
| 1 | [[ont.task.2026.2026-08-23_0950-sflo-v0.5.0-release]] | Prepare, validate, tag, and publish the additive FoundingReferentData ontology/SHACL release and Pages surface. |
| 2 | Semantic Flow Framework `e6d8bdd`/`f391813` | Push the reviewed portable spec and executable Accord transitions to canonical `main`; no independent package release is defined. |
| 3 | [[wa.task.2026.2026-08-23_0950-weave-v0.9.0-release]] | Prove library parity, prepare the minor release, rehearse all artifacts, publish, and verify consumers. |

## Sequence

1. Record the G3 singular-accepted ruling and establish these release artifacts.
2. Push the already-reviewed SFLO, Framework, Weave, and archive source state; require remote CI green before release-specific version commits.
3. Execute and publish SFLO v0.5.0, including immutable source-tag and live Pages receipts.
4. Add the missing packed-library founding parity smoke, prepare Weave v0.9.0 notes/version, and run local full release gates.
5. Push the Weave release commit and require canonical CI green.
6. Run Release Manual with npm dry-run and draft GitHub Release; inspect every artifact and ensure registries/tags remain unchanged.
7. Run Release Manual publication from the identical commit, then verify npm, archives/checksums, Git tag, GitHub Release, installed CLI, installed library, and the relevant Stagecraft flow.
8. Record receipts, close child tasks and this plan under [[wd.plans-and-tasks]], and leave clean worktrees.

## Gates

### R0 — Release Scope

Passes when current registry/tag state is verified, v0.5.0/v0.9.0 are confirmed as the next minor versions, and the vocabulary-before-runtime ordering is recorded.

Current status: **PASSED.** npm and GitHub expose Weave/`weave-lib` v0.8.0 as latest; SFLO source and Pages are v0.4.0; local reviewed commits are ahead of canonical `main`.

### R1 — Canonical Source Green

Passes when reviewed pre-release source/spec/archive commits are pushed and their normal remote CI gates are green.

### R2 — SFLO v0.5.0 Published

Passes when the source tag, immutable raw bytes, cross-engine 14-case conformance, live Pages release payloads/current pages, and publication branch receipts are green.

### R3 — Weave Candidate And Library Parity

Passes when full CI, package builds, npm dry-runs, native smoke, the exact `versionFoundingReferentData` named export, and off-tree Node/source byte-equivalent founding settlement/correction all pass from the versioned candidate.

### R4 — All-Platform Rehearsal

Passes when Release Manual dry-runs every npm package, builds/smokes every native archive, builds/smokes `weave-lib`, and creates an inspected draft GitHub Release without publishing a tag or registry version.

### R5 — Publication Verified

Passes when the identical rehearsed commit is published and every tag, release, registry package, archive checksum, installed version receipt, library founding smoke, and Stagecraft consumer receipt is verified.

## Decisions

- Dave's 2026-08-23 instruction to release now selects Gate G3's **singular accepted** branch for the measured 552-entry implementation. No batch task is cut.
- SFLO v0.5.0 precedes Weave v0.9.0 because the runtime must not ship ahead of the vocabulary it emits.
- Both releases are minor: SFLO adds public vocabulary/SHACL behavior; Weave adds new CLI and programmatic capability.
- `@semantic-flow/weave` and `@semantic-flow/weave-lib` remain one version line and publish together at 0.9.0.
- A source-barrel export is not sufficient parity evidence; the packed off-tree Node artifact must exercise the new API.
- Rehearsal and publication use the same Weave release commit. Any source change after rehearsal invalidates it.

## Open Issues

- Identify the closest available Stagecraft consumer path for the new founding initialization/version surface. If Stagecraft has not yet wired that API, retain the direct real CLI/API and packed-library receipts without fabricating consumer coverage.

## Testing And Receipts

- SFLO full CI, PySHACL, public JavaScript engine, Jena, normalized receipt comparison, Riot syntax, and release validation with required tag.
- Framework JSON/Accord validation and executable created/versioned/corrected sequence receipts already recorded under the founding task; canonical push CI must remain green.
- Weave full CI, build/package/dry-run tasks, native install smoke, npm library build/off-tree smoke, exact package contents, and version metadata.
- Release Manual rehearsal and publication run identifiers plus job summaries.
- Post-publish npm dist-tags, package tarballs, Git tag/release assets, checksum verification, installed CLI JSON, installed library named exports and founding lifecycle.
- SFLO raw tag and Pages payload byte identity plus dereferenceable `FoundingReferentData`/`hasFoundingReferentData` pages.

## Non-Goals

- Adding batch Knop creation in this release.
- Reopening the ruled founding profile, correction surface, or no-page boundary.
- Publishing a Framework package or inventing a version line for that repository.
- Folding unrelated backlog work into either release.
- Weakening gates merely to publish.

## Exit Criteria

- SFLO v0.5.0 source and Pages are publicly verified.
- Framework founding contract is on canonical `main`.
- Weave v0.9.0, all platform packages, archives, and `weave-lib` are publicly verified.
- Packed and installed `weave-lib` execute the founding API with source-equivalent results and bytes.
- Release notes/receipts name all consumer-visible behavior and retained residuals.
- Child tasks, backlog, roadmap, decision log, wikilinks, and monthly maintenance reflect closure.
- Every affected repository is clean with no accidental user changes.

## Plan Checklist

- [x] Rule G3 singular accepted and establish release ordering/version scope.
- [ ] Complete [[ont.task.2026.2026-08-23_0950-sflo-v0.5.0-release]].
- [ ] Push and verify the Framework contract commits.
- [ ] Complete [[wa.task.2026.2026-08-23_0950-weave-v0.9.0-release]].
- [ ] Record cross-release receipts and close the plan.
