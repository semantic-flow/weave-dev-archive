---
id: ztjerlzsapa0vglf6ycnqin
title: 2026 03 29 Conformance for Mesh Alice Bio
desc: ''
updated: 1775225097669
created: 1774804332190
---

## Goals

- Define machine-readable Accord manifests for the `mesh-alice-bio` transition ladder.
- Keep the manifests in the framework repo under `examples/alice-bio/conformance`, not in the fixture repo branches.
- Use the manifests as the first concrete acceptance-spec layer for Semantic Flow operations.
- Seed the work with the earliest completed transitions, then extend it incrementally as the fixture grows.

## Summary

This task captures conformance manifests for the `mesh-alice-bio` fixture ladder using the new Accord ontology and SHACL. The intended unit is one manifest per transition, even if that effectively maps one-to-one with the destination branch in the current linear ladder.

The manifests should live in `semantic-flow-framework/examples/alice-bio/conformance/` and describe expected transitions between fixture refs in the separate `mesh-alice-bio` repository. They should not be spread across the fixture branches themselves.

The first wave should cover the earliest completed transitions through `13-bob-extracted-woven`:

- `00-blank-slate` -> `01-source-only`
- `01-source-only` -> `02-mesh-created`
- `02-mesh-created` -> `03-mesh-created-woven`
- `03-mesh-created-woven` -> `04-alice-knop-created`
- `04-alice-knop-created` -> `05-alice-knop-created-woven`
- `05-alice-knop-created-woven` -> `06-alice-bio-integrated`
- `06-alice-bio-integrated` -> `07-alice-bio-integrated-woven`
- `07-alice-bio-integrated-woven` -> `08-alice-bio-referenced`
- `08-alice-bio-referenced` -> `09-alice-bio-referenced-woven`
- `09-alice-bio-referenced-woven` -> `10-alice-bio-updated`
- `10-alice-bio-updated` -> `11-alice-bio-v2-woven`
- `11-alice-bio-v2-woven` -> `12-bob-extracted`
- `12-bob-extracted` -> `13-bob-extracted-woven`

Subsequent manifests should be added as each later transition is modeled and reviewed.

In the active model, `08-alice-bio-referenced` introduces a dedicated `ReferenceCatalog` artifact for `alice` at `alice/_knop/_references`; the manifest should not treat `ReferenceLink`s as Knop-inventory-local data, and the `ReferenceLink` identities themselves should be stable fragment IRIs rooted at the catalog resource.

In the active Bob extraction model, `12-bob-extracted` should create a minimal `bob` Knop whose inventory points to `sfc:ExtractionSource <bob/_knop/_sources#extraction-source>` while the source details live in `bob/_knop/_sources/sources.ttl`. The payload file `alice-bio.ttl` should remain unchanged in that branch, and page generation should remain deferred to the woven `13` step.

For `13-bob-extracted-woven`, the active expectation is to version `bob/_knop/_meta` and `bob/_knop/_inventory`, generate `bob/index.html` and the Bob Knop-facing pages, and advance `_mesh/_inventory` because Bob's public pages now become part of the current mesh surface.

## Discussion

This work is not the public OpenAPI contract. It is a separate behavior/conformance layer that says what a Semantic Flow operation is expected to do to fixture state.

The current ladder is linear, so "one manifest per transition" and "one manifest per destination branch" are effectively equivalent for now. The naming should still reflect that these are transitions, not branch metadata.

Recommended initial naming:

- `01-source-only.jsonld`
- `02-mesh-created.jsonld`
- `03-mesh-created-woven.jsonld`
- `04-alice-knop-created.jsonld`
- `05-alice-knop-created-woven.jsonld`
- `06-alice-bio-integrated.jsonld`
- `07-alice-bio-integrated-woven.jsonld`
- `08-alice-bio-referenced.jsonld`
- `09-alice-bio-referenced-woven.jsonld`
- `10-alice-bio-updated.jsonld`
- `11-alice-bio-v2-woven.jsonld`
- `12-bob-extracted.jsonld`
- `13-bob-extracted-woven.jsonld`

Those files should each carry:

- fixture repository
- from/to refs
- operation identifier
- optional target designator path where applicable
- file expectations
- RDF expectations for RDF-bearing file expectations

The manifests should be authored before any runner or pseudo-runner implementation. That keeps the conformance layer normative instead of reverse-engineered from a validator after the fact.

The practical sequence should be:

1. Write manifests for already settled transitions.
2. Validate the manifests themselves with Accord SHACL.
3. Only then write pseudo-implementation or runner logic that checks fixture refs against those manifests.

That ordering matters. If the runner comes first, the manifests become documentation of the runner's behavior instead of the other way around.

## Open Issues

- Decide whether expectation nodes should remain freely reusable or whether Accord should eventually require explicit case ownership.
- Decide whether the current self-contained per-file JSON-LD contexts should later be replaced by a shared published Accord context.
- Decide whether `fixture.seedSourceOnly` should remain the long-term name for the `00` -> `01` seed transition, or whether fixture-preparation operation IDs should be normalized further.
- Decide whether later woven manifests should keep using `text` comparison for generated HTML ResourcePages or move toward a stricter HTML-specific comparison mode.

## Decisions

- Use one manifest per transition.
- Store all manifests in `semantic-flow-framework/examples/alice-bio/conformance/`.
- Do not store manifests in `mesh-alice-bio` branches.
- Start with the completed transitions through `13-bob-extracted-woven`, then add later manifests as later steps are completed.
- Author manifests first, then implement pseudo-runner or execution logic against them.
- Patch Accord when real authoring work exposes concrete gaps, rather than freezing weak shapes and working around them in the manifests.
- Treat the destination-branch filename convention as a convenience for the current linear ladder, not as the underlying semantic unit.

## Contract Changes

No OpenAPI changes are in scope here.

This task adds a separate conformance/acceptance layer for framework behavior. The relevant contract surface is the Accord manifest vocabulary and the conventions used by the `examples/alice-bio/conformance/` manifests.

## Testing

- Validate each manifest against Accord SHACL.
- Keep at least one human-readable review pass for every manifest to make sure the transition intent matches the corresponding fixture refs.
- Once pseudo-runner logic exists, run each manifest against the referenced fixture transition and confirm the result matches the manifest's expected pass/fail state.
- Add at least one negative-path check once the first runner exists, so the conformance layer is not only exercised on happy paths.

## Non-Goals

- Replacing the public API spec in `semantic-flow-api-spec.yaml`
- Moving manifests into the fixture repository
- Defining every future Semantic Flow operation before there are concrete examples
- Building a full conformance runner in the same first pass as the manifests

## Implementation Plan

- [x] Create `examples/alice-bio/conformance/README.md` with naming conventions and a short explanation of the transition-per-manifest approach.
- [x] Create `examples/alice-bio/conformance/01-source-only.jsonld` for `00-blank-slate` -> `01-source-only`.
- [x] Create `examples/alice-bio/conformance/02-mesh-created.jsonld` for `01-source-only` -> `02-mesh-created`.
- [x] Create `examples/alice-bio/conformance/03-mesh-created-woven.jsonld` for `02-mesh-created` -> `03-mesh-created-woven`.
- [x] Create `examples/alice-bio/conformance/04-alice-knop-created.jsonld` for `03-mesh-created-woven` -> `04-alice-knop-created`.
- [x] Create `examples/alice-bio/conformance/05-alice-knop-created-woven.jsonld` for `04-alice-knop-created` -> `05-alice-knop-created-woven`.
- [x] Create `examples/alice-bio/conformance/06-alice-bio-integrated.jsonld` for `05-alice-knop-created-woven` -> `06-alice-bio-integrated`.
- [x] Create `examples/alice-bio/conformance/07-alice-bio-integrated-woven.jsonld` for `06-alice-bio-integrated` -> `07-alice-bio-integrated-woven`.
- [x] Create `examples/alice-bio/conformance/08-alice-bio-referenced.jsonld` for `07-alice-bio-integrated-woven` -> `08-alice-bio-referenced`.
- [x] Create `examples/alice-bio/conformance/09-alice-bio-referenced-woven.jsonld` for `08-alice-bio-referenced` -> `09-alice-bio-referenced-woven`.
- [x] Create `examples/alice-bio/conformance/10-alice-bio-updated.jsonld` for `09-alice-bio-referenced-woven` -> `10-alice-bio-updated`.
- [x] Create `examples/alice-bio/conformance/11-alice-bio-v2-woven.jsonld` for `10-alice-bio-updated` -> `11-alice-bio-v2-woven`.
- [x] Create `examples/alice-bio/conformance/12-bob-extracted.jsonld` for `11-alice-bio-v2-woven` -> `12-bob-extracted`.
- [x] Create `examples/alice-bio/conformance/13-bob-extracted-woven.jsonld` for `12-bob-extracted` -> `13-bob-extracted-woven`.
- [x] Validate the first thirteen manifests against Accord SHACL.
- [x] Patch Accord to add `unchanged`, tighten `compareMode`, enforce same-case RDF targeting, and prevent duplicate file expectations per path.
- [x] After the first eight manifests are stable, create the next manifest only when the corresponding fixture transition is settled.
- [x] After several manifests exist, design a minimal pseudo-runner that can compare refs and report Accord-level pass/fail results.
