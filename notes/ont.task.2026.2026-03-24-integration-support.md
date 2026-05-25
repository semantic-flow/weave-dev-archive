---
id: 1ybtxdxkle2ln2uh0r63j5w
title: 2026 03 24 Integration Support
desc: ''
updated: 1779246543364
created: 1774411739870
---

## Context

The config modernization work surfaced a distinct integration-support problem: how raw candidate artifacts are recognized, claimed, mapped into `Knop`s, and initialized during `weave integrate`.

This should be treated as related to config, but not collapsed into the presentation-oriented ResourcePage template work.

Integration-oriented config can still live in the same ontology as presentation config for now. What is deferred here is the richer modeling of integration support itself.

The [March 14 `weave integrate` discussion](../../sflo-dendron-notes/sflo.conv.2026.2026-03-14_0958-existing-solutions-that-could-be-extended-with-weave-codex.md) is the main immediate design anchor.

Integration should be able to target arbitrary tags/commits/refs to get a historical version mapped to a particular ArtifactHistory and HistoricalState, repeatedly if necessary, so that a mesh can be regenerated/recomposed piecemeal with particular committed versions of a DigitalArtifact mapped to histories and states. 

## Current Status

This is no longer an immediate implementation need. The newer branch-published fixture work settled two adjacent pieces that this note should not pull back into integration config:

- source provenance for carried bytes and extracted terms now belongs in the Knop-owned `_knop/_sources` support artifact, modeled as `KnopSourceRegistry` plus `ArtifactResolutionTarget` / `ExtractionSource` bindings
- operational access policy and reusable config-source resolution belong with the grand config synthesis work, especially [[wd.task.2026.2026-05-06-grand-config-synthesis]]

Automated integration remains a useful future capability, but it should wait until there is real pressure for automatic claiming/matching/mapping of candidate source files. The current `weave integrate`, `weave extract`, and branch-published deploy flows can be driven explicitly by commands and Accord manifests.

When this task resumes, keep the boundary crisp: integration support chooses how to recognize and initialize candidate artifacts; source registries record what source was actually used; operational config decides which local/remote sources the runtime is allowed to follow.

## Deferred Working Goal

Define the minimal ontology and concept model needed to support `weave integrate` without overcommitting to a heavyweight template/matcher system too early.

At minimum, the model should help answer:

- how a mesh declares integration guardrails
- how candidate artifacts are matched
- how designatorPaths are derived from source names
- how multiple integration rules can apply to the same mesh
- how integration behavior can start simple and grow toward richer selectors later

## Deferred Direction

- `IntegrationConfig` may still be a useful subclass of `Config`, but it should be introduced only when automated integration needs authored matching/mapping policy
- individual `IntegrationContext`s may be better modeled as relator-like things that can apply multiply to a `SemanticMesh`
- matching can start with filename-to-designatorPath rules; avoid committing to regex-first until real examples show that regex is the right authoring surface
- Dendron-style dotted hierarchy may be a useful additional convention for expressing hierarchy in source naming
- future matching/mapping may need to inspect filename parts, extensions, YAML frontmatter, embedded metadata, and RDF attributes, but that should not block the simpler filename-first start

For the first pass, just turning filenames into `designatorPath`s is probably adequate.

## Candidate Vocabulary

Likely areas of vocabulary include:

- `IntegrationConfig`
- `IntegrationContext`
- target mesh or integration scope
- candidate artifact selector
- designator-path mapping rule
- metadata/default-reference behavior
- history vs no-history behavior at integration time
- support for external working sources like paths outside the tree or URLs

Do not use this vocabulary to store repository source provenance, extraction provenance, or runtime path grants. Those are already covered by source registries and operational config.

## Open Questions

- How exactly should `IntegrationConfig` relate to individual `IntegrationContext` instances?
- Should `IntegrationContext` be a relator attached directly to `SemanticMesh`, or should it only be reached through `IntegrationConfig`?
- How far should selector language go in this pass: plain regex, dotted hierarchy conventions, or a broader selector model?
- When should attribute-based matching be introduced beyond plain filenames and filename-to-designatorPath mapping?
- Which integration behaviors belong in the ontology versus application logic in Weave?

## Candidate Deliverables

- a small integration-support section in the future config ontology, only if automated integration becomes an active feature
- a short note or example showing filename-to-designatorPath mapping
- a worked example showing enough `IntegrationConfig` / `IntegrationContext` to drive a realistic `weave integrate`
- follow-up notes if richer selector language is needed
