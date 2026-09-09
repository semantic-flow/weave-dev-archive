---
id: watask202609081709alltermsexcludeartifacthistorysubtrees
title: Enforce mutual exclusion between Knops and artifact-history coordinates
desc: Prevent source RDF infrastructure IRIs below a known ArtifactHistory path from being minted as Knops, and refuse history, state, or manifestation allocation when the coordinate already contains a Knop.
created: 1788912570000
---

## Parent Plan

URPX staging publication execution is coordinated externally in `urpx-fluxtailor/dev-docs/plans/plan.urpx.2026-09-08.semantic-flow-staging-pilot.md`.

## Goals

- Make `extract --all-terms` exclude every IRI equal to or below a known `sflo:ArtifactHistory` path.
- Preserve ordinary extraction of vocabulary identifiers beneath a payload designator such as `ontology/RatePlan`.
- Report excluded history descendants through `skippedSupportDesignatorPaths`.
- Refuse payload version planning when a proposed `ArtifactHistory`, `HistoricalState`, or `ArtifactManifestation` coordinate equals or contains an existing Knop designator.

## Summary

The URPX `v0.5.0` staging rehearsal exposed a creation-order defect. An untagged preview materialized `ontology/releases/preview-<commit>`, while the source ontology described `ontology/releases/v0.4.0`. All-terms extraction recognized the materialized preview state as infrastructure but minted Knops for the source-declared `v0.4.0` state, its `ttl` manifestation, and its located file because those exact resources were not yet present in mesh inventory.

An exact-tag rehearsal did not reproduce the bad Knops because `ontology/releases/v0.5.0` already existed and was typed `sflo:HistoricalState`. That success is order-dependent. The artifact history root `ontology/releases` already establishes the whole subtree as artifact infrastructure, so descendants must never become designators even before a particular state is materialized.

## Discussion

This is not a request to reserve the word `releases` globally. The exclusion follows an RDF fact in the active mesh: a particular path is an `sflo:ArtifactHistory`. Vocabulary paths elsewhere remain eligible.

Do not solve this with post-generation deletion. That would leave Knop and inventory claims for removed files and would allow invalid overlap during planning.

## Open Issues

None.

## Decisions

- ArtifactHistory descendant exclusion is structural and independent of state creation order.
- Exact history roots and every slash-delimited descendant are support paths for all-terms discovery.
- Knop/artifact-coordinate exclusion is mutual: extraction cannot mint into a history subtree, and versioning cannot allocate artifact containers over an existing Knop.
- Existing generated-resource and reserved-segment exclusions remain unchanged.

## Contract Changes

`extract --all-terms` adds a stronger exclusion invariant: a mesh-scoped named node below a known `sflo:ArtifactHistory` is reported as skipped support and is never minted as a Knop.

Payload version planning adds the inverse preflight: proposed history, state, and manifestation containers cannot equal or contain an existing Knop designator. The request fails with `plan-conflict` before writes.

Update the portable extract behavior specification because this is externally observable discovery behavior.

## Testing

- Regression: current payload history is `ontology/releases`, current state is `preview-<commit>`, and source RDF names `ontology/releases/v0.5.0`, its manifestation, and located file; all three are skipped support.
- Control: `ontology/RatePlan` is still extracted.
- Regression: named history or state allocation is refused when `releases` or `releases/<tag>` is already a Knop designator.
- Existing exact generated-resource and reserved-segment exclusions remain green.
- Run `deno task test`, `deno task check`, and `deno task lint`.
- Re-run the URPX exact-tag publication rehearsal and its explicit no-Knop-below-releases assertion.

Receipts, 2026-09-08:

- focused all-terms history-descendant regression: pass
- focused named history/state collision planner and CLI regressions: pass
- `deno task test`: 942 passed, 0 failed
- `deno task check`: pass
- `deno task lint`: pass
- URPX `v0.5.0` exact-tag rehearsal: both Weave validators pass; zero Knops below either release path

## Non-Goals

- Filtering vocabulary terms by OWL, SHACL, SKOS, or RDF class.
- Reserving common path words globally.
- Changing single-target extraction.
- Changing payload history naming or URPX workflow behavior.

## Implementation Plan

- [x] Collect ArtifactHistory paths from the active mesh and source Knop inventories used by all-terms discovery.
- [x] Exclude exact history paths and slash-delimited descendants before normalization and filesystem probing.
- [x] Check planned history, state, and manifestation containers against existing MeshInventory Knops before writes.
- [x] Add regression and control coverage.
- [x] Update the Semantic Flow extract/version behavior specifications and CLI documentation.
- [x] Run focused and repository validation.
