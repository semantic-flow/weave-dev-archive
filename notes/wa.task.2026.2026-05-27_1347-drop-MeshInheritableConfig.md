---
id: ck1a1hs1jhm32utu5qy0jgb
title: 2026-05-27_1347-drop-MeshInheritableConfig
desc: Simplify config attachment roles by keeping mesh config singular, dropping KnopConfig, and using generic Knop inheritable attachments.
updated: 1779918284724
created: 1779914883772
---

## Goals

- Retire the mesh-specific inheritable config concept/vocabulary before config-source discovery builds more runtime surface around it.
- Keep `sfcfg:MeshConfig` as the portable mesh-owned config class for mesh concerns such as publication profile, workspace relationship, mesh workspace path rules, resolver policy, and mesh support behavior.
- Drop `sfcfg:KnopConfig`; Knop-authored config can be ordinary `sfcfg:Config` / `sfcfg:ConfigArtifact` attached to a Knop.
- Collapse config attachment roles toward four generic properties: `sfcfg:hasConfig`, `sfcfg:hasConfigSource`, `sfcfg:hasInheritableConfig`, and `sfcfg:hasInheritableConfigSource`.
- Make `sfcfg:hasConfig` / `sfcfg:hasConfigSource` mean local config for recognized config-bearing scopes, and make `sfcfg:hasInheritableConfig` / `sfcfg:hasInheritableConfigSource` a Knop subtree-default attachment.
- Preserve fail-closed behavior for role ambiguity: generic config attachment properties should be meaningful only on recognized config-bearing scopes such as `sflo:SemanticMesh` and `sflo:Knop`.
- Settle conventional filesystem discovery on a single `_config/config.ttl` location for meshes and Knops.
- Update the config ontology, framework specs, Weave runtime task notes, and tests before implementing [[wa.task.2026.2026-05-27_1246-config-source-discovery-and-resolution]].

## Summary

The current config ontology has role-specific attachment properties such as `sfcfg:hasMeshConfig`, `sfcfg:hasMeshInheritableConfig`, `sfcfg:hasKnopLocalConfig`, `sfcfg:hasKnopInheritableConfig`, plus source variants. That made sense while the layer model was still forming, but it is now doing too much vocabulary work. Mesh config should be singular mesh-owned config, while Knop config can be ordinary config attached to a Knop. The property only needs to distinguish local participation from Knop subtree-default participation.

This task should retire the mesh-specific inheritable vocabulary and move to a simpler attachment model:

- `sfcfg:hasConfig`: local config attached to the subject's scope
- `sfcfg:hasConfigSource`: source-resolved local config attached to the subject's scope
- `sfcfg:hasInheritableConfig`: config authored at a Knop and offered to descendant Knop scopes
- `sfcfg:hasInheritableConfigSource`: source-resolved config authored at a Knop and offered to descendant Knop scopes

Under this model:

- `sflo:SemanticMesh sfcfg:hasConfig ?config` means mesh-local config.
- `sflo:Knop sfcfg:hasConfig ?config` means Knop-local config.
- `sflo:Knop sfcfg:hasInheritableConfig ?config` means Knop-authored defaults projected into descendant Knops.

Mesh-level defaults for Knops should live in the singular mesh config as ordinary mesh-level policy whose selectors cover Knop-governed targets. They should not require a separate mesh-inheritable authoring surface. `MeshConfig` remains because the mesh still has real portable configuration concerns that are not naturally Knop config. `KnopConfig` can disappear because a Knop can attach ordinary `Config` / `ConfigArtifact` values.

## Discussion

### Why keep MeshConfig

`sfcfg:MeshConfig` is still useful and should not be retired in this task. It is the right class for portable mesh-owned configuration, including:

- `sfcfg:hasPublicationProfile`
- `sfcfg:workspaceRootRelativeToMeshRoot`
- `sfcfg:hasMeshWorkspacePathRule`
- mesh-level resolver policy or unknown-term policy
- mesh support-artifact behavior, including `_mesh/_meta`, `_mesh/_inventory`, and `_mesh/_config`
- mesh-local policy bindings for artifacts governed directly by the mesh

Those are not facts about a root resource or a top-level Knop. They are facts about the mesh container, publication surface, support artifacts, and runtime interpretation of the mesh.

### Why retire mesh-specific inheritable attachment vocabulary

The previous direction made mesh-inheritable config a peer to Knop-inheritable config. Conceptually, though, mesh-authored Knop defaults are just mesh-level policy whose targets include Knop-governed resources. They do not need their own filesystem surface, authored property family, or runtime layer role.

This keeps the split simpler:

- mesh subject plus `sfcfg:hasConfig`: mesh config, including mesh-wide policy defaults when their targets cover Knops
- Knop subject plus `sfcfg:hasConfig`: Knop-local config
- Knop subject plus `sfcfg:hasInheritableConfig`: subtree-authored defaults for descendant Knops

The resolver should not add an internal mesh-authored-inherited layer just to preserve the old concept. Mesh config remains the mesh layer. Knop-inheritable config is projected into descendant Knop scopes and can be traced as Knop-inherited config when consumed.

### Conventional filesystem locations

We already have a conventional mesh config location:

- `_mesh/_config/config.ttl`

Past designs used `_config-local/` and `_config-inheritable/`. Those names are humane because they put local and inheritable config in visibly different places, but they also create an awkward question: can the local file contain inheritable attachment properties, or vice versa? If yes, the path distinction is not authoritative; if no, the path becomes another validation axis that duplicates RDF semantics.

Use a single conventional config file per scope:

- `_mesh/_config/config.ttl`: mesh config
- `<knop>/_knop/_config/config.ttl`: Knop config

Within a Knop config file, RDF attachment properties can distinguish local Knop config from inheritable subtree defaults when both are needed. Within mesh config, there is no separate mesh-inheritable modality: mesh-level policies apply according to their targets and scope.

### Relationship to config-source discovery

[[wa.task.2026.2026-05-27_1246-config-source-discovery-and-resolution]] currently references the role-specific source properties. This task should update that follow-up to use the generic four-property model before implementation starts. Otherwise config-source discovery will harden the vocabulary we are trying to simplify.

## Open Issues

- None currently. Current direction is to remove the mesh-authored-inherited layer role, keep mesh-local, Knop-inherited, Knop-local, Knop-inheritable offer, application/default, and command override roles, and fail closed for generic attachment properties on unrecognized subjects.

## Decisions

- Keep `sfcfg:MeshConfig`.
- Remove `sfcfg:KnopConfig`; use ordinary `sfcfg:Config` / `sfcfg:ConfigArtifact` attached to Knops.
- Retire the authored concept of a distinct mesh-specific inheritable config property family.
- Treat inheritable as a Knop attachment role expressed by `sfcfg:hasInheritableConfig` / `sfcfg:hasInheritableConfigSource`.
- Use a single conventional config file per scope: `_mesh/_config/config.ttl` for meshes and `<knop>/_knop/_config/config.ttl` for Knops.
- Treat those conventional filesystem locations as discovery bootstrap for the corresponding scope. The RDF attachment subject and property still determine mesh versus Knop scope and local versus inheritable participation inside a config graph.
- Keep `sfcfg:hasConfig` as the generic local attachment property; do not rename it to `hasLocalConfig`.
- Do not make `sfcfg:hasInheritableConfig` a subproperty of `sfcfg:hasConfig`, and do not make `sfcfg:hasInheritableConfigSource` a subproperty of `sfcfg:hasConfigSource`; inheritable attachments are sibling role properties, not local attachments under RDFS entailment.
- Keep derived runtime config links such as `sfcfg:hasEffectiveConfig` separate from authored local `sfcfg:hasConfig`.
- Remove old role-specific attachment properties from the live pre-v1 ontology rather than retaining deprecated aliases. Unknown retired terms should fail closed under existing unknown-term handling; do not add bespoke guardrail tests unless the repo already has a general retired-term guardrail list.
- Mesh config is the mesh layer. Mesh-level policy can cover Knops when its selectors and scope say so; do not model that as a separate mesh-authored-inherited layer.
- Use the attachment subject to determine mesh versus Knop scope. `sflo:SemanticMesh` plus `hasConfig` is mesh config; `sflo:Knop` plus `hasConfig` is Knop-local config; `sflo:Knop` plus `hasInheritableConfig` is Knop-authored descendant defaults.
- Require recognized config-bearing subjects for generic attachment properties. `hasConfig` or `hasInheritableConfig` on an unknown subject must not silently affect runtime config.
- Keep config-source location/provenance separate from authority. A referenced config source participates at the scope and local/inheritable role of the attachment that selected it.

## Contract Changes

### SFLO Config Ontology

- Add `sfcfg:hasInheritableConfig` as a sibling of `sfcfg:hasConfig`, with domain `sflo:Knop` and range `sfcfg:Config`.
- Add `sfcfg:hasInheritableConfigSource` as a sibling of `sfcfg:hasConfigSource`, with domain `sflo:Knop` and range `sfcfg:ConfigSource`.
- Clarify `sfcfg:hasConfig` and `sfcfg:hasConfigSource` comments so they mean local config for the subject when used on a recognized config-bearing scope.
- Remove `sfcfg:hasEffectiveConfig` as a subproperty of `sfcfg:hasConfig`; effective config is derived runtime/view state, not authored local config.
- Remove `sfcfg:KnopConfig`.
- Remove `sfcfg:hasMeshConfig`, `sfcfg:hasMeshInheritableConfig`, `sfcfg:hasKnopConfig`, `sfcfg:hasKnopLocalConfig`, `sfcfg:hasKnopInheritableConfig`, and their source variants from live authoring vocabulary.
- Remove `sfcfg:configLayerRole_meshInheritable`; keep mesh-local, Knop-inherited, Knop-local, Knop-inheritable offer, application/default, resolved-runtime, and command override roles.

### Semantic Flow Framework

- Update [[sf.config]] to explain the four-property attachment model and the chosen conventional filesystem locations.
- Update [[sf.spec.2026-05-25-config-behavior]] to say that local versus inheritable is expressed by generic attachment properties and mesh versus Knop is determined by the subject.
- Update any framework examples/conformance manifests that mention role-specific properties.

### Weave Runtime

- Update config parser constants and validation for the generic attachment properties before implementing config-source discovery.
- Remove any existing config layer role parsing/tracing for `configLayerRole_meshInheritable`.
- Keep conventional `_mesh/_config/config.ttl` behavior working for current mesh-local config.
- Add conventional discovery for `<knop>/_knop/_config/config.ttl` when that runtime slice is implemented.
- Update [[wa.task.2026.2026-05-27_1246-config-source-discovery-and-resolution]] so it no longer asks implementation to discover `hasMeshConfigSource`, `hasKnopLocalConfigSource`, or `hasKnopInheritableConfigSource`.

### Documentation

- Update user-facing docs only after Weave can actually discover the selected conventional locations.
- Update developer notes to distinguish mesh-owned config from root-Knop config.
- Make clear that mesh-level policy can cover Knops by selector/scope without a separate mesh-inheritable config modality.

## Testing

- SFLO ontology parse/guardrail tests for new `hasInheritableConfig` and `hasInheritableConfigSource`.
- Framework/spec examples use the generic four-property model.
- Weave config parser tests accept mesh-local `hasConfig` on a `SemanticMesh`.
- Weave config parser tests accept Knop-local `hasConfig` and Knop-authored `hasInheritableConfig` on a `Knop` once Knop config discovery is implemented.
- Weave config parser tests reject `hasInheritableConfig` on a `SemanticMesh`.
- Weave config parser tests reject `KnopConfig` if old class terms are treated as unknown/unsupported current authoring vocabulary.
- Runtime tests show generic attachment properties on unknown subjects fail closed or are ignored with explicit diagnostics according to the selected resolver policy.
- Config-source discovery tests are updated to use `hasConfigSource` and `hasInheritableConfigSource`.
- Ontology guardrail tests assert inheritable and effective config attachment properties are not subproperties of local attachment properties.

## Non-Goals

- Do not remove `sfcfg:MeshConfig`.
- Do not implement full Knop config inheritance in this task unless it is needed to validate the vocabulary change.
- Do not implement the config-source discovery runtime slice here; update its task and unblock it.
- Do not add a new root-Knop special case to replace mesh-authored inherited defaults.
- Do not make filesystem location the only authority for config role.
- Do not add long-lived compatibility aliases unless implementation sequencing proves they are necessary.
- Do not add bespoke retired-term guardrails for every removed pre-v1 attachment property if existing unknown-term fail-closed behavior already covers them.

## Implementation Plan

- [x] Decide the conventional filesystem discovery locations: use `_mesh/_config/config.ttl` for meshes and `<knop>/_knop/_config/config.ttl` for Knops.
- [x] Update `semantic-flow-config-ontology.ttl` with `sfcfg:hasInheritableConfig` and `sfcfg:hasInheritableConfigSource`.
- [x] Remove `sfcfg:KnopConfig` and role-specific attachment properties in the config ontology, and update comments for `hasConfig` / `hasConfigSource`.
- [x] Update SFLO SHACL and tests for the generic attachment model.
- [x] Update SFLO release notes and decision log with the attachment-role simplification.
- [x] Update [[sf.config]] and [[sf.spec.2026-05-25-config-behavior]].
- [x] Update Weave constants/parser/tests for retired role-specific properties if any live code references them.
- [x] Update [[wa.task.2026.2026-05-27_1246-config-source-discovery-and-resolution]] to use the simplified attachment vocabulary and selected conventional locations.
- [x] Run SFLO repo validation, including ontology/SHACL tests and release validation.
- [x] Run focused Weave checks/tests if runtime constants or tests change.
- [x] Provide commit messages per touched repo.
