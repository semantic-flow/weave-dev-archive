---
id: ztx9pxp171t4yf5wyyfzfke
title: 2026 05 25_1609 Config Policy Ontology and Runtime
desc: ''
updated: 1779750606550
created: 1779750606550
---

## Goals

- Establish the Semantic Flow config policy-binding ontology vocabulary needed by [[sf.spec.2026-05-25-config-behavior]].
- Replace context-free history/page-generation/page-presentation defaults with explicit policy definitions, bindings, and targets where the behavior is target-selective.
- Preserve direct scoped settings for simple scope facts such as publication profile and `workspaceRootRelativeToMeshRoot`.
- Build a Weave internal config runtime that compiles RDF config into queryable scoped settings, a policy index, and resolver/meta-policy state.
- Make existing-mesh Weave commands honor mesh-local config from `_mesh/_config/config.ttl` after application defaults and before command overrides.
- Keep command overrides explicit, volatile, and stronger than portable mesh config.
- Add built-in ResourcePage presentation policies needed for durable mesh-wide layout choices, especially all generated panels and no generated panels.
- Avoid a throwaway implementation that first honors mesh config through the old direct-default vocabulary and then immediately replaces it.
- Keep the implementation fail-closed: malformed, unsupported, unsafe, or ambiguous config must stop the operation before affected behavior is applied.

## Summary

The earlier [[wa.task.2026.2026-05-24_2304-honor-mesh-config]] task identified a real implementation gap: Weave writes mesh-owned config but most existing-mesh commands still load only built-in defaults plus command-scoped overrides. During review, that task also exposed broader config-model issues around direct policy assertions on `Config`, all-artifact selectors, policy precedence, ResourcePage presentation, and whether reusable config is a layer.

The current behavior spec, [[sf.spec.2026-05-25-config-behavior]], is the conceptual source of truth for this pass, but it currently has one known drift point: it still mentions `sfcfg:hasDefaultResourcePagePresentationConfig` as transitional policy-slot language. This task supersedes that line until the spec is updated. The implementation pass should update the spec first or in the same first commit before changing ontology/runtime behavior.

This task should update the config ontology and Weave runtime together enough that the ontology is not merely aspirational. The first runnable slice should compile application defaults, mesh-local `_mesh/_config/config.ttl`, and command overrides into one effective runtime structure. The runtime should be designed around the full layer model, but durable CLI config-edit commands, full Knop discovery, mesh-create policy authoring, arbitrary referenced config retrieval, and full inherited Knop projection can be deferred if they would make this slice sprawl.

The review notes [[wa.review.2026-05-25_1541-config-behavior-spec-claude]] and [[wa.review.2026-05-25_1542-honor-mesh-config-claude]] should be treated as background. Most of their spec corrections have already been incorporated. One important non-adopted suggestion: do not require `sfcfg:policyPriority` for ordinary same-layer role-specific exceptions to any-artifact baselines. Structural selector specificity should handle that; priority is for conflicts at the same layer and same effective selector specificity.

## Discussion

### Current implementation shape

- `defaults/application.ttl` currently uses direct terms such as `sfcfg:hasDefaultHistoryTrackingPolicy`, `sfcfg:hasHistoryTrackingDefault`, `sfcfg:hasDefaultResourcePageGenerationPolicy`, and `sfcfg:hasDefaultResourcePagePresentationConfig`.
- `src/runtime/config/effective_config.ts` parses those direct defaults into `EffectiveConfig`.
- `src/runtime/weave/execution_config.ts` loads only Weave defaults, then optionally overlays an all-role command-scoped history override.
- Mesh config already exists for publication profile and sidecar workspace layout, but existing-mesh commands do not yet use `_mesh/_config/config.ttl` as a behavioral input for history, ResourcePage generation, ResourcePage presentation, or related policy.
- ResourcePage rendering already has a presentation-profile object and panel-selection model, but only the default profile is recognized and Semantic Flow metadata still relies on a boolean command opt-in.
- `defaults/config-resolution.ttl` still orders `sfcfg:configLayerRole_reusableConfig` as a layer even though reuse is now defined to participate at the referencing attachment point.

### First runnable slice

The first runnable slice is accepted when these pieces work together:

- [[sf.spec.2026-05-25-config-behavior]] is aligned with the decisions in this task, especially ResourcePage presentation policy naming and the removal of direct page-definition config adjacency.
- SFLO ontology and SHACL define the policy-binding vocabulary, policy targets, value predicates, ResourcePage presentation policy terms, zero-panel presentation validity, exact artifact target validation, and removed/retired legacy terms.
- Weave defaults use the new policy-binding model for application-level history, ResourcePage generation, and ResourcePage presentation.
- Weave compiles application defaults, mesh-local `_mesh/_config/config.ttl`, and command overrides into one runtime config structure.
- Weave policy resolution is target-object based, including exact governed-artifact validation from mesh inventory and support-artifact knowledge.
- Built-in ResourcePage presentation policies exist for `semantic-site-default`, `semantic-site-all-panels`, and `semantic-site-no-panels`.
- Runtime diagnostics include structured logger output plus an in-memory resolution trace, without persisted `sfcfg:ResolvedConfig` or `sfcfg:ConfigResolutionRecord`.
- Focused ontology, Weave, and framework tests/docs are updated enough that the slice is coherent and runnable.

Deferred follow-through includes durable config-edit CLI commands, full Knop-local/inherited source discovery, arbitrary referenced config retrieval, mesh-create policy authoring, persisted resolved-config records, and any Weave-specific ontology for host/workspace access rules.

### Conceptual model for this task

Config source location is not authority. Config authority comes from the attachment point that uses the config. A config artifact stored under a Knop can participate at mesh scope only if mesh-level config explicitly references or embeds it. Knop-local config cannot promote itself to mesh-wide behavior.

All authored config is potentially reusable, but "reusable config" is not a separate layer. Referenced config participates at the scope and layer of the attachment point that referenced it. The ontology may still need terms for source/provenance roles, but the resolver must not treat referenced config as one universal global layer.

Policy targets are explicit. Missing target fields never mean "any". The broad artifact selector is "any governed artifact in this config scope," not "all artifact roles" as a role value. A governed artifact is a Semantic Flow-managed `sflo:DigitalArtifact` governed by the current config scope through mesh or Knop support, inventory, payload, reference, page-definition, or config structure. Mesh support artifacts are governed by their mesh. Knop support and payload artifacts are governed by their Knop and the containing mesh. A merely referenced external artifact is not governed by a scope just because config in that scope points to it.

Under the default resolver profile, policy resolution is per queried target:

1. collect bindings whose selectors cover the queried target
2. prefer higher-precedence layers
3. within the winning layer, prefer structurally more specific selectors
4. within the same layer/policy-slot/specificity, prefer higher `sfcfg:policyPriority`
5. treat omitted priority as `0`
6. fail closed on unresolved incompatible ties

Selector specificity is structural. For artifacts, exact artifact is more specific than artifact role, and artifact role is more specific than any governed artifact. Priority must not be required for that ordinary specificity relationship.

### Policy slots and scoped settings

Use policy bindings for target-selective behavior:

- history tracking
- ResourcePage generation
- ResourcePage presentation

Do not force every config concern into the policy-binding model. Keep these as direct scoped settings unless/until they become target-selective:

- publication profile
- `workspaceRootRelativeToMeshRoot`
- history/state/manifestation naming defaults
- resolver/meta-policy values, with the bootstrap limits described in the spec

`sfcfg:hasResourcePageRegenerationConfigPolicy` remains an open decision. It is currently shaped like a direct singleton setting, but this task must decide whether it stays a scoped setting, becomes a policy binding, or is deferred/rejected in the first runtime compiler.

ResourcePage generation is not just a boolean. It answers what the runtime should do about ResourcePage materialization for a target: generate, suppress, defer, or generate on request. The first Weave implementation may only exercise generate/suppress, but the vocabulary should preserve the richer policy shape.

ResourcePage presentation is separate from publication profile. A GitHub Pages publication profile may create `.nojekyll`, but it must not imply a ResourcePage layout/profile, all-panels rendering, or Semantic Flow metadata inclusion.

Runtime policy APIs should query by target object, not only by artifact role. Role-only helpers may remain as compatibility/convenience wrappers inside Weave while call sites migrate, but the core compiled-config API should accept a target descriptor that can carry exact artifact IRI, artifact roles, governing scope, ResourcePage kind, and other selector inputs. Otherwise exact-artifact targets and ResourcePage selectors will be awkward immediately.

Policy-slot identification in the first pass should be registry-driven and derived from supported value predicates. Do not add an explicit policy-slot marker unless implementation pressure proves it is needed. The runtime should maintain a central policy-slot registry; reject policy definitions with no supported slot; and if one policy definition carries multiple supported slots, trace and resolve each slot independently.

### ResourcePage presentation policies

Add or support at least these built-in presentation policies:

- `semantic-site-default`: current default Semantic Site behavior.
- `semantic-site-all-panels`: selects all ordinary data-gated generated panels and gives the Semantic Flow metadata panel a real policy-level path, not a hidden `includeSemanticFlowMetadata` boolean gate.
- `semantic-site-no-panels`: keeps the generated page shell/chrome available, including canonical/title/breadcrumb/template structure as applicable, but has zero generated panel selections.

`semantic-site-no-panels` is a valid presentation policy. The parser and SHACL must allow a `ResourcePagePresentationPolicy` with no `sfcfg:hasResourcePagePanelSelection` values. This is useful for intermediary pages for IRIs surfaced indirectly by extraction, where title, breadcrumbs, class/type hints, and shell chrome are useful but generated data panels would be noise.

`sfcfg:hasGeneratedResourcePagePanelSelection` on `sflo:ResourcePageDefinition` remains local authored page composition over the resolved presentation policy. It is not a config attachment and does not pick the presentation policy itself. Runtime should render authored page regions first, then generated panels selected by resolved policy and local authored panel selections according to the page-composition rules.

The CLI flag currently called `--include-semantic-flow-metadata` may remain as an operation convenience, but internally it should compile to a command-override presentation or panel-selection policy. It should not become a durable boolean config concept.

### Resolver config

`sfcfg:ConfigResolutionConfig` is meta-config for the resolver. Portable config may request stricter resolver behavior but must not loosen the active application/runtime resolver policy or expand trust. If Weave does not implement portable resolver narrowing in this slice, it should reject or ignore such declarations according to the active resolver and unknown-term policy rather than partially applying them. Weave-specific service or host trust settings should live outside the portable Semantic Flow config ontology.

## Open Issues

- Decide exact ontology names for the binding model. Current preferred direction is target/selector terminology, not context terminology: `sfcfg:PolicyBinding`, `sfcfg:PolicyDefinition`, `sfcfg:PolicyTarget`, `sfcfg:ArtifactPolicyTarget`, candidate target subclasses such as any-governed-artifact, artifact-role, and exact-artifact targets, plus `sfcfg:hasPolicyBinding`, `sfcfg:bindsPolicy`, `sfcfg:appliesToPolicyTarget`, and `sfcfg:policyPriority`.
- Decide exact built-in IRI names for `semantic-site-all-panels` and `semantic-site-no-panels`.
- Decide whether `sfcfg:hasResourcePageRegenerationConfigPolicy` remains a scoped singleton setting, becomes a policy binding, or is deferred/rejected in the first compiled-config runtime.
- Decide CLI/API names for durable config edits after creation. Candidate shapes include `weave mesh config set-history-tracking-policy` and `weave mesh config set-resource-page-presentation`, but this task can land the runtime before the final UX if necessary.
- Decide whether Knop-local and Knop-inherited source discovery is implemented in this task or only supported in the compiled config API. The runtime data structure should support those layers even if source discovery is deferred.

## Decisions

- [[sf.spec.2026-05-25-config-behavior]] is the behavior contract for this task.
- Use "policy target" / "selector" terminology rather than "policy context" for new ontology terms.
- Do not treat referenced/reusable config as a separate precedence layer. It participates at its attachment point.
- Knop-inheritable config is an outbound offer to descendants and does not apply to the declaring Knop unless the same config is also attached as Knop-local.
- Omitted selector fields do not mean "any"; wildcard behavior must be explicit.
- Structural selector specificity beats broad selectors within the same layer. `sfcfg:policyPriority` is only for same-layer, same-policy-slot, same-effective-specificity conflicts.
- Do not include `PolicyFamily` in the normative policy-binding model. Conflict detection and merge semantics are driven by the policy slot/value predicate set by a `PolicyDefinition`, not by broad grouping labels.
- The broad artifact target is any governed artifact in the current config scope, not an all-artifact-role value.
- Policy slots are identified by supported value predicates in the first pass. Keep a central runtime slot registry, reject policy definitions with no supported slot, and resolve each supported slot independently when one policy definition carries multiple supported slots.
- Publication profile and workspace-root relationship remain scoped settings, not policy bindings.
- Naming defaults may remain direct layered scoped settings unless/until a target-selective naming use case appears.
- Use `sfcfg:ResourcePagePresentationPolicy`, not `sfcfg:ResourcePagePresentationConfig`, for ResourcePage presentation values. Do not keep a separate `sfcfg:hasDefaultResourcePagePresentationConfig` property; defaultness comes from attachment point, layer, and policy resolution.
- Do not keep `OperationalConfig`, `WorkspaceOperationalConfig`, `HostLocalOperationalConfig`, or host/workspace access-rule vocabulary in the portable Semantic Flow config ontology.
- Treat host/workspace access rules as internal Weave runtime structures only in this slice. Defer any Weave-specific ontology until there is stronger pressure from service/runtime use cases.
- Remove `sfcfg:configLayerRole_reusableConfig` from default precedence and, unless a concrete non-layer validation use appears, from the portable `ConfigLayerRole` vocabulary. Reuse is not a layer.
- Remove live Weave authoring/parser support for old direct default terms such as `sfcfg:hasDefaultHistoryTrackingPolicy`, `sfcfg:hasHistoryTrackingDefault`, and `sfcfg:hasDefaultResourcePageGenerationPolicy`. Released ontology history can remain historical, but current pre-v1 authoring should use policy bindings.
- Treat config-scope ResourcePage presentation defaults as policy bindings. A `sflo:ResourcePageDefinition` should not carry direct config adjacency; page-specific presentation should be expressed through a policy binding with an exact page/resource target or an appropriate Knop-local scope.
- Preserve `sfcfg:hasGeneratedResourcePagePanelSelection` on `sflo:ResourcePageDefinition` as local authored composition over the resolved presentation policy. It is not config adjacency and does not select the presentation policy.
- Replace or retire `sfcfg:panelDataRequirement_semanticFlowMetadataOptIn`. Metadata inclusion selected by policy must use actual metadata availability or equivalent policy selection semantics, not an opt-in flag.
- Allow exact artifact policy targets in mesh config in this slice, using mesh inventory and governed support-artifact knowledge as the governance authority. Exact artifact targets identify governed `sflo:DigitalArtifact` IRIs only. `sflo:ResourcePageDefinition` can be targeted when it is the governed definition artifact. Generated `sflo:ResourcePage` files and public referent IRIs need ResourcePage/resource-target selector shapes, not exact artifact targets.
- ResourcePage generation policy targets the governed resource/artifact for which a ResourcePage would be generated, not the generated page file itself.
- Exact-target validation is not pure Turtle parsing. The compiler needs scope/governance context from mesh inventory and support-artifact knowledge; non-governed exact IRIs, malformed inventories, or unavailable governance facts must fail closed.
- First-slice diagnostics should be structured logger output plus an in-memory resolution trace. Do not persist `sfcfg:ResolvedConfig` or `sfcfg:ConfigResolutionRecord` in this slice.
- Command overrides remain the highest ordinary layer and must not be persisted unless the user explicitly performs a config-editing operation.
- Do not implement a short-lived old-vocabulary mesh-config merge just to unblock the SFLO replay unless the user explicitly reprioritizes that over the clean policy model.

## Contract Changes

### Config ontology

- Add policy-binding vocabulary for attaching bindings to config, reusable policy definitions, explicit bindings, explicit targets/selectors, policy slots/value predicates, and optional priority.
- Add explicit artifact policy target shapes for any governed artifact, artifact-role targets, and exact-artifact targets.
- Add or clarify ResourcePage policy target vocabulary, including page-kind and selector specificity rules where possible.
- Do not add controlled `PolicyFamily` terms for history tracking, ResourcePage generation, or ResourcePage presentation. If grouping metadata is useful later, add it as non-normative UI/logging/docs metadata rather than resolution input.
- Replace `sfcfg:ResourcePagePresentationConfig` with `sfcfg:ResourcePagePresentationPolicy`; replace `sfcfg:hasResourcePagePresentationConfig` with `sfcfg:hasResourcePagePresentationPolicy`; remove `sfcfg:hasDefaultResourcePagePresentationConfig`.
- Use `sfcfg:hasResourcePagePresentationPolicy` as a policy-slot/value predicate on policy definitions, not as direct config adjacency on `sflo:ResourcePageDefinition`.
- Keep `sfcfg:hasGeneratedResourcePagePanelSelection` as local authored ResourcePage composition over the resolved presentation policy, not as config attachment.
- Replace or retire `sfcfg:panelDataRequirement_semanticFlowMetadataOptIn`; metadata panel selection should depend on actual available metadata or explicit policy selection semantics.
- Remove portable operational config classes and portable host/workspace local/remote access-rule vocabulary from the Semantic Flow config ontology.
- Remove `sfcfg:configLayerRole_reusableConfig` from layer-precedence vocabulary/defaults. If reusable config provenance needs a term later, model it as config source/provenance metadata rather than a precedence layer.
- Clarify or revise `ConfigLayerRole`/attachment-role vocabulary so referenced config and Knop-inheritable config are not misread as ordinary consumed layers at the declaring scope.
- Clarify direct history and ResourcePage generation default terms in light of the binding model.
- Decide and encode the first-pass treatment of `sfcfg:hasResourcePageRegenerationConfigPolicy`; if not implemented, the compiler should reject it rather than silently applying an unclear direct singleton.
- Keep publication profile, `workspaceRootRelativeToMeshRoot`, and naming defaults as direct scoped settings unless a later ontology pass intentionally changes them.

### Weave runtime

- Replace the current defaults-only effective config loader with a compiled config loader that can read application defaults, optional mesh-local config, and command overrides.
- Introduce internal runtime types for `CompiledConfig`, scoped settings, policy bindings, policy targets, policy index resolution, and resolution explanations.
- Make the primary policy-resolution API target-object based, for example by accepting a policy slot plus a target descriptor with exact artifact IRI, artifact roles, governing scope, and ResourcePage selector inputs. Keep role-only helper methods only as wrappers over the target-object API.
- Implement a central runtime policy-slot registry derived from supported value predicates; reject definitions with no supported slot; resolve multiple supported slots independently when present on one definition.
- Compile application defaults from `defaults/application.ttl` using the new policy-binding model.
- Update `defaults/application.ttl` to use `sfcfg:ResourcePagePresentationPolicy` and `sfcfg:hasResourcePagePresentationPolicy`, with no default-specific ResourcePage presentation property.
- Update `defaults/config-resolution.ttl` to remove `sfcfg:configLayerRole_reusableConfig` from the default precedence profile.
- Read `_mesh/_config/config.ttl` for existing-mesh commands when present and treat it as mesh-local config.
- Preserve fail-closed behavior for unsupported terms, duplicate singletons, malformed selectors, unknown policy values, and unresolved conflicts.
- Preserve command-scoped history overrides by compiling them into command-override policy bindings for any governed artifact or the relevant operation target.
- Replace boolean-shaped Semantic Flow metadata command internals with a command-override ResourcePage presentation or panel-selection policy. `semantic-site-all-panels` must include metadata through policy data requirements or equivalent selection semantics, not through a hidden runtime boolean.
- Add supported built-in ResourcePage presentation policies for default, all-panels, and no-panels behavior.
- Update effective-config parsing to read `sfcfg:hasResourcePagePresentationPolicy` through policy definitions, reject removed ResourcePage presentation config terms, and use ResourcePage presentation policy naming internally where practical.
- Remove direct `ResourcePageDefinition` presentation-config parsing; page-specific presentation should come from resolved policy bindings.
- Preserve direct `ResourcePageDefinition` generated panel selections as local authored composition over resolved policy.
- Relax ResourcePage presentation parsing and SHACL so a presentation policy may declare zero generated panel selections.
- Implement exact artifact policy target parsing and matching for mesh-governed `sflo:DigitalArtifact` resources in this slice. This requires mesh inventory/support-artifact governance context and must fail closed when context is unavailable or the target is not governed by the active scope.
- Keep ResourcePage generation policy queries centered on the governed resource/artifact that would receive a page. Do not treat the generated page file as the default target of generation policy.
- Remove or relocate runtime support for removed `machineLocalOperational`/`workspaceOperational` config-layer roles and portable local/remote path access-rule terms.
- Keep host/workspace access rules internal to Weave runtime for this slice; do not introduce a Weave-specific persisted ontology yet.
- Update structured diagnostics and in-memory resolution traces to include participating config sources, command overrides, selected high-level policy values by slot, and conflict/exclusion reasons.

### Documentation and tasks

- Update Weave CLI/user docs when durable config options are exposed.
- Update ResourcePage docs and examples to use ResourcePage presentation policy terminology.
- Update the config behavior spec to remove transitional `hasDefaultResourcePagePresentationConfig` language.
- Update the config behavior spec to state that direct ResourcePageDefinition config adjacency is not the model for presentation defaults; use scoped/exact-target policy bindings instead.
- Update the config behavior spec to preserve the distinction between page-local generated panel selections as authored composition and ResourcePage presentation policy as config.
- Update [[wa.task.2026.2026-05-24_2304-honor-mesh-config]] or leave a note there that this task supersedes its implementation model.
- Update the SFLO replay recipe only after mesh-local policy config is honored at runtime.

## Required Follow-Up From Initial Ontology Cleanup

- [ ] Update `defaults/application.ttl`: replace `sfcfg:hasDefaultResourcePagePresentationConfig` with `sfcfg:hasResourcePagePresentationPolicy`, and replace `sfcfg:ResourcePagePresentationConfig` with `sfcfg:ResourcePagePresentationPolicy`.
- [ ] Update `src/runtime/config/effective_config.ts`: replace removed ResourcePage presentation config/default constants, parse `sfcfg:hasResourcePagePresentationPolicy`, and stop treating ResourcePage presentation as a default-only property.
- [ ] Update `src/runtime/weave/page_definition.ts`: remove direct `sfcfg:hasResourcePagePresentationConfig` page-definition parsing and route page-specific presentation through resolved policy bindings instead.
- [ ] Preserve `sfcfg:hasGeneratedResourcePagePanelSelection` parsing as local authored page composition over the resolved presentation policy.
- [ ] Update runtime tests and TTL fixtures in `src/runtime/config/effective_config_test.ts` and `src/runtime/weave/page_definition_test.ts`.
- [ ] Update docs and specs that mention `ResourcePagePresentationConfig`, `hasResourcePagePresentationConfig`, or `hasDefaultResourcePagePresentationConfig`, especially `documentation/notes/wu.resource-pages.md` and [[sf.spec.2026-05-25-config-behavior]].
- [ ] Remove or relocate Weave runtime references to removed portable operational config vocabulary, including `configLayerRole_machineLocalOperational`, `configLayerRole_workspaceOperational`, local path access rules, remote access rules, and related tests/defaults.
- [ ] Remove `sfcfg:configLayerRole_reusableConfig` from `defaults/config-resolution.ttl`, runtime config-layer role parsing, and ontology layer-role vocabulary unless retained outside layer precedence as provenance/source metadata.
- [ ] Remove live parser support and default authoring for old direct policy-default terms such as `sfcfg:hasDefaultHistoryTrackingPolicy`, `sfcfg:hasHistoryTrackingDefault`, and `sfcfg:hasDefaultResourcePageGenerationPolicy`.
- [ ] Relax ResourcePage presentation parser/tests so `semantic-site-no-panels` can have zero generated panel selections.
- [ ] Replace or retire `sfcfg:panelDataRequirement_semanticFlowMetadataOptIn` and add an all-panels metadata path that does not depend on `includeSemanticFlowMetadata` / `semanticFlowMetadataOptIn` as a hidden boolean gate.
- [ ] Decide and implement/reject/defer `sfcfg:hasResourcePageRegenerationConfigPolicy` explicitly.
- [ ] Replace role-only config policy query call sites with target-object queries, keeping role helpers only as wrappers where still useful.
- [ ] Implement exact artifact policy targets for mesh-governed artifacts and reject exact targets outside the governing mesh/scope.
- [ ] Keep host/workspace access rules as internal Weave runtime structures for this slice and remove portable persisted vocabulary references.

## Testing

- Ontology parse/shape tests for the new policy-binding vocabulary.
- Unit-test policy resolution layer precedence: command override beats Knop-local, Knop-local beats inherited/mesh, mesh-local beats application defaults for targets covered by the upper layer.
- Unit-test structural specificity: exact artifact beats artifact role, artifact role beats any governed artifact, with no priority required.
- Unit-test `sfcfg:policyPriority`: omitted priority is `0`, higher priority wins at same specificity, and equal-priority incompatible values fail closed.
- Unit-test that missing selector fields do not imply "any" and malformed/incomplete targets are rejected.
- Unit-test governed-artifact matching for mesh support artifacts, Knop support artifacts, payload artifacts, and merely referenced external artifacts.
- Unit-test exact artifact policy targets for governed artifacts and fail-closed rejection for non-governed exact targets.
- Unit-test exact-target validation failure for malformed inventory or unavailable governance context.
- Unit-test policy definitions with no supported slot failing validation and policy definitions with multiple supported slots resolving/tracing each slot independently.
- Unit-test scoped settings validation for publication profile and `workspaceRootRelativeToMeshRoot`.
- Unit-test `sfcfg:hasResourcePageRegenerationConfigPolicy` according to the selected first-pass behavior.
- Unit-test that naming defaults remain layered scoped settings if they are not migrated to policy bindings.
- Unit-test mesh-local history tracking policy for any governed artifact and role-specific same-layer exceptions.
- Unit-test mesh-local ResourcePage generation policy values, especially generate versus suppress.
- Unit-test mesh-local ResourcePage presentation policy selection for default, all-panels, and no-panels.
- Unit-test `semantic-site-all-panels` includes the Semantic Flow metadata panel without the old metadata opt-in data requirement.
- Unit-test `semantic-site-no-panels` preserves page shell/chrome while suppressing generated data panels and allowing zero panel selections.
- Unit-test page-specific presentation policy through exact target or Knop-local policy binding rather than direct ResourcePageDefinition config.
- Unit-test direct `sfcfg:hasGeneratedResourcePagePanelSelection` on `sflo:ResourcePageDefinition` remains available as local authored composition over the resolved presentation policy.
- Integration-test `weave`, `weave version`, and `weave generate` honoring `_mesh/_config/config.ttl` without repeated command flags.
- Regression-test command-scoped history and ResourcePage presentation overrides beating mesh-local config for one run.
- Integration-test unsupported mesh-local policy values, duplicate singleton settings, and conflicting policy bindings failing closed.
- Run focused `deno task test`/`deno task check` for Weave runtime changes; run `deno task lint` after broader structural changes; run repo-appropriate ontology/framework checks if ontology/spec files change.

## Validation Matrix

- SFLO repo: parse the edited Turtle ontologies, run SHACL/ontology validation tests for config policy vocabulary and ResourcePage presentation policy shapes, and include a semantic-commit-style message summarizing ontology and SHACL changes.
- Weave repo: run focused `deno task test` for touched config/runtime/page-generation tests, run `deno task check` for API/type changes, run `deno task lint` after broader structural edits, and include a semantic-commit-style message summarizing runtime/defaults/test/doc changes.
- Semantic Flow Framework repo: update [[sf.spec.2026-05-25-config-behavior]] before or with the implementation changes, run repo-appropriate spec/example/conformance checks when manifests change, and include a semantic-commit-style message summarizing spec/conformance changes.
- Weave dev archive: keep this task current as decisions change, do not rename it to completed unless explicitly requested, and include a note-style commit message if committed separately.

## Non-Goals

- Do not make publication profile imply history, ResourcePage generation, or ResourcePage presentation behavior.
- Do not add an all-artifact-role enum value. Use explicit target/selector shapes for broad matching.
- Do not treat referenced config as a global layer.
- Do not let portable config expand host trust, local path access, remote retrieval, or resolver authority.
- Do not introduce persisted Weave-specific host/workspace access-rule ontology in this slice.
- Do not implement unpinned remote config retrieval in this slice.
- Do not persist `sfcfg:ResolvedConfig` or full `sfcfg:ConfigResolutionRecord` output by default.
- Do not require a compatibility layer for pre-v1.0 direct policy-default vocabulary unless the implementation needs a very short migration bridge.
- Do not keep page-local direct config adjacency on `sflo:ResourcePageDefinition` as the durable presentation override model.
- Do not make `weave mesh create` an interactive config wizard.
- Do not block the first SFLO manual replay if command overrides remain the temporary runbook path.

## Implementation Plan

- [ ] Update [[sf.spec.2026-05-25-config-behavior]] first, or in the same first implementation commit, to remove `sfcfg:hasDefaultResourcePagePresentationConfig` drift and align page-local generated panel selection semantics.
- [ ] Finalize ontology term names for policy bindings, policy definitions, policy slots/value predicates, policy targets, target properties, and priority.
- [ ] Update `semantic-flow-config-ontology.ttl` with the chosen policy-binding and target vocabulary.
- [ ] Clarify or revise config layer/attachment-role vocabulary around referenced config and Knop-inheritable config.
- [ ] Remove `sfcfg:configLayerRole_reusableConfig` from default precedence and layer-role handling.
- [ ] Decide and encode the first-pass treatment of `sfcfg:hasResourcePageRegenerationConfigPolicy`.
- [ ] Replace or retire `sfcfg:panelDataRequirement_semanticFlowMetadataOptIn` in favor of actual metadata availability or explicit policy selection semantics.
- [ ] Update Weave default config TTL to express history tracking, ResourcePage generation, and ResourcePage presentation defaults using the chosen binding model.
- [ ] Add or revise built-in ResourcePage presentation policies for `semantic-site-default`, `semantic-site-all-panels`, and `semantic-site-no-panels`.
- [ ] Design the Weave internal `CompiledConfig` / `PolicyIndex` runtime types and policy-resolution trace structure.
- [ ] Implement the runtime policy-slot registry based on supported value predicates.
- [ ] Implement policy-target parsing and validation, including explicit any-governed-artifact, artifact-role, and exact-artifact targets.
- [ ] Implement policy resolution by queried target object, layer, structural specificity, priority, and fail-closed conflict detection.
- [ ] Implement scoped-setting parsing/validation for publication profile, `workspaceRootRelativeToMeshRoot`, naming defaults, and resolver/meta-policy fields that remain in scope.
- [ ] Replace `loadWeaveDefaultEffectiveConfig`/`loadEffectiveConfigForExecution` call sites with a mesh-root-aware loader that can compile defaults, mesh-local config, and command overrides.
- [ ] Update command override handling so history overrides and metadata/presentation overrides compile into the command-override layer.
- [ ] Wire generated ResourcePage rendering to selected ResourcePage presentation policy rather than a durable boolean metadata flag.
- [ ] Apply the required follow-up updates from the initial ontology cleanup, including Weave defaults, parser constants, test fixtures, docs, and removed operational/access-rule vocabulary references.
- [ ] Add unit tests for ontology parsing, policy-target parsing, policy resolution, scoped settings, and ResourcePage presentation policy behavior.
- [ ] Add integration tests for mesh-local `_mesh/_config/config.ttl` changing history, ResourcePage generation, and ResourcePage presentation behavior.
- [ ] Update CLI docs and the SFLO replay recipe when durable mesh-local config can replace repeated command overrides.
- [ ] Run focused tests/checks during development, then run `deno task lint` for broader Weave structural changes and repo-appropriate checks for ontology/framework changes.
