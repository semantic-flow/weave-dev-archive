---
id: 4c8df82e-c955-49aa-b205-3f376e532e82
title: SFLO v0.5.0 Release
desc: 'Publish the FoundingReferentData ontology, SHACL contract, and Pages surface'
created: 1787503800000
---

## Parent Plan

[[wa.completed-plan.2026.2026-08-23_0950-founding-capability-releases]]

## Status

Completed 2026-08-23. SFLO v0.5.0 source/tag and Pages were published and verified before Weave v0.9.0.

## Goals

- Publish SFLO v0.5.0 from reviewed FoundingReferentData contract commit `cf7e79a5` plus release preparation.
- Make the new class and optional Knop slot available through an immutable source tag and dereferenceable Pages resources before Weave v0.9.0 publishes.
- Preserve the agreeing 14-case PySHACL/public-JavaScript/Jena receipts and the already-reviewed no-Stagecraft-vocabulary boundary.

## Summary

SFLO v0.4.0 is the current source/Pages release. Commit `cf7e79a5` adds `FoundingReferentData`, `hasFoundingReferentData`, bounded SHACL warnings, decision/reference guidance, and 14-case cross-engine fixtures; it also repairs the pre-existing `KnopMetadata` Turtle typo that blocked parsing at the reviewed baseline. This additive public vocabulary and validation behavior warrants v0.5.0.

## Discussion

The release script updates version metadata across all active ontology files as one version line. Release notes must distinguish the project-Page-published core/config/SHACL artifacts from source-only job/prov topology exactly as [[ont.dev.release-runbook]] requires.

Pages publication must add the v0.5.0 payload states and generated resource pages, not merely regenerate old current state. Use the established explicit source binding/version/generate flow and verify source-tag/Pages payload byte identity.

## Open Issues

- Confirm the local `sflo-gh-pages` worktree is clean and at canonical `gh-pages` before mutation.

## Decisions

- Version is 0.5.0, issued 2026-08-23.
- No Stagecraft-specific predicate or generic `ReferentMetadata` enters SFLO.
- `FoundingReferentData` remains `DigitalArtifact` plus `RdfDocument`, not `SemanticFlowResource`; no page-generation contract is implied by class membership.

## Contract Changes

- Add `sflo:FoundingReferentData` and optional `sflo:hasFoundingReferentData` on Knop.
- Add SHACL cardinality/type structure and working-file RDF-document warning coverage.
- Document the bounded exception to the earlier generic referent-metadata decision.
- Publish positive, duplicate-slot warning, and non-RDF-working-file warning fixtures within the 14-case cross-engine corpus.

## Testing

- `deno task fmt`, `deno task ci`, and `deno task conformance:jena` followed by `deno task conformance:compare`.
- `deno task release:validate -- --version 0.5.0` before tag and `--require-tag` after tag.
- Riot/Turtle syntax for all five active ontology files.
- Immutable raw-tag download and byte comparison for all five source files.
- Pages mesh/publication validation, payload byte identity, current/release ResourcePage links, and direct dereference checks for the two new terms.

## Non-Goals

- Weave runtime publication.
- Page semantics for FoundingReferentData.
- Batch initialization, remote fetching, adoption, or retraction vocabulary.
- Any unrelated ontology change.

## Implementation Plan

- [x] Push reviewed `cf7e79a5` and require source CI green.
- [x] Inventory the exact v0.4.0..candidate delta and write `ont.release-notes.v0.5.0`.
- [x] Run `release:set-version` for 0.5.0 / 2026-08-23 and review every metadata diff.
- [x] Run full source, SHACL, cross-engine, syntax, release, and whitespace gates.
- [x] Commit, tag, push, and verify immutable source bytes.
- [x] Publish the v0.5.0 Pages states/pages and verify live byte identity/dereferenceability.
- [x] Record source/tag/Pages/CI receipts and close the task.

## Implementation Receipt

- Source/tag: `cf10917ee759901f40226e65a3c24b6824459b25`; annotated tag object `b783461ac93c627deced5abaff356ca28eb6d1e3`.
- Source CI: `https://github.com/semantic-flow/sflo/actions/runs/32654106737`, success.
- Full source gate: 33/33 Deno tests, 14 PySHACL fixtures, 14 public `shacl-engine` fixtures, 14 Apache Jena SHACL fixtures, release validation, and Riot syntax green.
- Final normalized receipts agreed across all 14 cases; bundle digests are recorded in [[ont.report.2026-08-23-v0.5.0-release]].
- Pages: `cc416147f61c6ead9cd9110cf4a34fb9b75e40f8`; deployment `https://github.com/semantic-flow/sflo/actions/runs/32654532757`, success.
- Pages census: 374 Knops, 1,506 Turtle files, 3,402 files; mesh/publication zero findings; same-timestamp generate 0 created / 0 updated.
- Live core/config/SHACL payloads are byte-identical to the tag, and the FoundingReferentData class/property/shape pages return 200 with canonical links.
