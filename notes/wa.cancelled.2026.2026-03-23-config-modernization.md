---
id: 0cm22gs24b5ufl01qdtw1fg
title: 2026 03 23 Config Modernization
desc: ''
updated: 1774411153356
created: 1774330976647
---

## Context

The old config work in `old/sflo-config-ontology.jsonld` was designed around ontology structures that no longer exist in the active model.

## Status

Superseded by [[wd.task.2026.2026-05-06-grand-config-synthesis]]. Keep this note as historical rationale for the config modernization line; do not treat its "Current Direction", "Current Decisions", or "Suggested Plan" sections as active implementation guidance.

The useful ideas carried forward were:

- `Config` stays distinct from `DigitalArtifact`, while reusable persisted configs may also be `ConfigArtifact`, `DigitalArtifact`, and `RdfDocument`
- config should have explicit mesh/Knop attachment relations rather than relying only on generic `hasConfig`
- reusable named config artifacts are valid
- computed effective config is non-authoritative runtime/debug output
- the old Flow/State/Distribution framing and `AbstractArtifact` vocabulary should not be carried forward
- ResourcePage template/style terms belong in the config line while ResourcePage content composition belongs in core

The active synthesis changed or rejected several early ideas from this note:

- do not add `ConfigFragment`; use inline ordinary `Config` data for embedded fragments and `ConfigArtifact` for independently identified reusable configs
- do not introduce `KnopConfigDefaults` as the main model; use mesh/Knop inheritable config attachments, config layer roles, and resolution context
- do not replace Knop-local/inheritable config with mesh-scoped defaults only; Knops need both local config and inheritable config surfaces
- do not revive boolean switches such as `generateResourcePages` or `createHistoricalStatesOnWeave`; use policy-valued config
- do not make regex-heavy template mapping the first presentation-selection contract
- keep automated integration support deferred unless there is concrete pressure for source-file claiming/matching/mapping

The main mismatches are:

- `sflo:Mesh` should now be `sflo:SemanticMesh`
- `sflo:AbstractArtifact` should now be `sflo:DigitalArtifact`
- old Flow / State / Distribution framing should not be carried forward directly
- old template-mapping naming should be revisited, but likely in a more ResourcePage-specific direction than the broad `OutputTemplateMapping`

The primary current use case is figuring out a `Knop`'s config.

Secondarily, the same modernization should probably support mesh-level config, especially where a mesh provides both mesh-surface behavior and defaults for contained `Knop`s.

## Working Goal

Define a modern config model that fits the current core ontology and can answer:

- what config applies to this `Knop`?
- what config applies to this `SemanticMesh`?
- what explicitly authored config artifacts exist?
- what effective config was computed for debugging or runtime inspection?

## Current Direction

The modernized config model should be grounded in current active classes:

- `sflo:SemanticMesh`
- `sflo:Knop`
- `sflo:DigitalArtifact`
- `sflo:RdfDocument`

The likely shape is:

- `Config` remains an RDF configuration resource and should probably be a first-class class distinct from `DigitalArtifact`
- `ConfigFragment` may be useful as a smaller reusable unit for shared config pieces that are not themselves full `MeshConfig` or `KnopConfig` resources
- named reusable config may also be modeled as a `DigitalArtifact`, probably also an `RdfDocument`
- config should attach directly to a `SemanticMesh` and `Knop` via explicit mesh/knop config relations rather than a generic attachment property
- a `SemanticMesh` should probably be able to carry both `MeshConfig` and `KnopConfigDefaults`, so mesh-surface behavior and defaults for contained `Knop`s do not collapse into one bucket
- if config needs its own persistent identity, history, or reuse, the config resource itself may be modeled as a `DigitalArtifact`
- computed effective config remains explicitly non-authoritative runtime/debugging data

It may be useful to distinguish at least two major config families:

- presentation or publication-oriented config, especially for ResourcePage generation
- integration-oriented config, which can live in the same ontology for now even though the richer integration-support modeling is deferred

This means the old “config as Flow/Distribution” framing should be dropped rather than translated literally. The [current core decision](ont.decision-log.md) is that `DigitalArtifact` is the governing artifact-level resource and that the old named classes `AbstractArtifact`, `ArtifactFlow`, `ArtifactState`, `WorkingState`, and `CurrentState` are not part of the active core as such. Their concerns are now handled more explicitly through the current structure: `ArtifactState` is roughly replaced by `HistoricalState`, `CurrentState` is approximated operationally by `latestHistoricalState`, and `WorkingState` is replaced by `hasWorkingLocatedFile`.

## Current Decisions

- `Config` should probably remain a first-class class distinct from `DigitalArtifact`, even though some named reusable configs may also be modeled as `DigitalArtifact`s.
- `ConfigFragment` seems worth keeping as a reusable building block for shared config pieces.
- Mesh config and knop config should probably use explicit attachment relations such as `hasMeshConfig` / `meshConfigFor` and `hasKnopConfig` / `knopConfigFor`, rather than relying on a generic `hasConfig` / `configFor` pair as the main public pattern.
- `MeshConfig` and `KnopConfig` should probably be the main concrete subclasses immediately rather than postponing that distinction to attachment properties alone.
- `MeshConfig` and `KnopConfig` should probably stay separate, at least conventionally, even if some shared fragments can be reused across them.
- The mesh should be the inheritance boundary for now. If you want different config inheritance semantics, declare a new mesh.
- No direct artifact-directed policy attachment point seems necessary right now. The main exception is integration-oriented policy living on the mesh and providing guardrails for how candidate `DigitalArtifact`s get integrated into `Knop`s.
- Output-template mapping should stay under config for now.
- Template mapping should stay explicitly ResourcePage-oriented for now, and possibly permanently unless a second concrete output family emerges.
- Named reusable config artifacts should probably be typed directly as config subclasses, while also being typed as `DigitalArtifact` / `RdfDocument` when they are persisted as authored artifacts.
- Integration-oriented config can live in the same ontology as presentation config for now.
- Inner/outer template terminology should probably stay, with names like `InnerResourcePageTemplate` and `OuterResourcePageTemplate`, rather than switching to body/shell terminology.
- A `SemanticMesh` should probably be able to carry both `MeshConfig` and `KnopConfigDefaults`.
- `MeshConfig` should describe mesh-surface behavior, such as whether to generate a ResourcePage for the mesh itself (for example `M/_mesh/index.html`).
- `KnopConfigDefaults` should describe default behavior for contained `Knop`s, such as `createHistoricalStatesOnWeave` or whether to generate ResourcePages for those `Knop`s.
- The old `LocalConfig` / `InheritableConfig` split should probably be replaced by mesh-scoped defaults plus explicit knop overrides.

## Likely Term Mapping

- `sflo:Mesh` -> `sflo:SemanticMesh`
- `sflo:AbstractArtifact` -> `sflo:DigitalArtifact`
- `hasAbstractArtifactConfig` -> probably drop rather than rename directly
- `TemplateMappingSet` -> probably `ResourcePageTemplateMappingSet`
- `TemplateMapping` -> probably `ResourcePageTemplateMapping`
- `InnerTemplate` -> probably `InnerResourcePageTemplate`
- `OuterTemplate` -> probably `OuterResourcePageTemplate`
- `mappingTargetSlugRegex` -> probably something closer to `mappingTargetDesignatorPathRegex`, or a broader selector term if regex-only targeting feels too narrow

## ResourcePage vs Browser Layer

[Recent ResourcePage discussion](../../sflo-dendron-notes/sflo.conv.2026.2026-03-13_2159-chrome-on-static-sites-codex-2.md) suggests a useful boundary for the config work:

- woven `index.html` pages should remain real standalone HTML with meaningful content
- shared chrome like global nav/search/header can be emitted separately and loaded by the browser
- Turbo is promising as a browser-layer enhancement for smoother navigation, not as the thing that defines page identity or page structure

That suggests template vocabulary should primarily describe woven ResourcePage composition, not the separately loaded browser shell.

So for now, the “inner/outer template” lineage still seems useful, but it should probably be renamed in a more explicit ResourcePage direction rather than generalized too quickly into “output templates” if the actual semantics are still page-specific.

## Scope For This Task

This task should focus first on modernizing the configuration ontology for current active concepts.

That likely includes:

- config attached to `SemanticMesh`
- knop-default config attached to `SemanticMesh`
- config attached to `Knop`
- template-mapping vocabulary, likely still ResourcePage-focused in this pass
- effective-config vocabulary

It also should not automatically conflate:

- ResourcePage composition rules
- browser-layer chrome/navigation behavior
- detailed integration-support modeling for mapping arbitrary artifacts into `Knop`s

Detailed integration-support modeling is now deferred to [ont.task.2026-03-24-integration-support.md](./ont.task.2026-03-24-integration-support.md).

## Open Questions

- Should a single persisted config artifact ever intentionally carry both `MeshConfig` and `KnopConfigDefaults` types, or should that be treated as a smell and split into separate artifacts?
- Should `KnopConfigDefaults` be modeled as its own class, or as a constrained mesh-attached use of `KnopConfig`?
- Which properties belong on `MeshConfig` versus `KnopConfigDefaults` as the config surface grows beyond `generateResourcePages` and similar first examples?
- How much inheritance/merging semantics should be stated in ontology comments versus left entirely to application logic and examples?

## Candidate Deliverables

- a modern `semantic-flow-config-ontology.ttl`
- a migration note from old config terms to new terms
- one worked example showing mesh config, knop config defaults, and explicit knop override

## Suggested Plan

1. Audit old config terms and mark each as keep, rename, replace, or drop.
2. Define `Config`, `ConfigFragment`, `MeshConfig`, `KnopConfigDefaults`, and `KnopConfig` as the main config classes, with explicit attachment relations.
3. Define the minimal modern config ontology for `SemanticMesh` and `Knop`, while deciding how named reusable config artifacts relate to `DigitalArtifact` and how mesh-level defaults relate to explicit knop overrides.
4. Keep template vocabulary ResourcePage-specific, and rename `InnerTemplate` / `OuterTemplate` to `InnerResourcePageTemplate` / `OuterResourcePageTemplate`.
5. Keep integration-oriented config in the same ontology for now, but defer detailed integration-support modeling to [ont.task.2026-03-24-integration-support.md](./ont.task.2026-03-24-integration-support.md).
