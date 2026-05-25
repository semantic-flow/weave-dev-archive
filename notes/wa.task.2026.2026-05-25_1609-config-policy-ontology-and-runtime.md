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
- Add built-in ResourcePage presentation profiles needed for durable mesh-wide layout choices, especially all generated panels and no generated panels.
- Avoid a throwaway implementation that first honors mesh config through the old direct-default vocabulary and then immediately replaces it.
- Keep the implementation fail-closed: malformed, unsupported, unsafe, or ambiguous config must stop the operation before affected behavior is applied.

## Summary

The earlier [[wa.task.2026.2026-05-24_2304-honor-mesh-config]] task identified a real implementation gap: Weave writes mesh-owned config but most existing-mesh commands still load only built-in defaults plus command-scoped overrides. During review, that task also exposed broader config-model issues around direct policy assertions on `Config`, all-artifact selectors, policy precedence, ResourcePage presentation, and whether reusable config is a layer.

The current behavior spec, [[sf.spec.2026-05-25-config-behavior]], is now the conceptual source of truth for this pass. It separates scoped settings from layered policies, treats reuse as a property of config content rather than a precedence layer, defines attachment points as the source of effective scope/layer, requires explicit policy targets, and uses layer precedence, selector specificity, optional priority, and fail-closed conflict handling for policy resolution.

This task should update the config ontology and Weave runtime together enough that the ontology is not merely aspirational. The first runnable slice should compile application defaults, mesh-local `_mesh/_config/config.ttl`, and command overrides into one effective runtime structure. The runtime should be designed around the full layer model, but arbitrary referenced config retrieval and full inherited Knop projection can be deferred if they would make this slice sprawl.

The review notes [[wa.review.2026-05-25_1541-config-behavior-spec-claude]] and [[wa.review.2026-05-25_1542-honor-mesh-config-claude]] should be treated as background. Most of their spec corrections have already been incorporated. One important non-adopted suggestion: do not require `sfcfg:policyPriority` for ordinary same-layer role-specific exceptions to any-artifact baselines. Structural selector specificity should handle that; priority is for conflicts at the same layer and same effective selector specificity.

## Discussion

### Current implementation shape

- `defaults/application.ttl` currently uses direct terms such as `sfcfg:hasDefaultHistoryTrackingPolicy`, `sfcfg:hasHistoryTrackingDefault`, `sfcfg:hasDefaultResourcePageGenerationPolicy`, and `sfcfg:hasDefaultResourcePagePresentationConfig`.
- `src/runtime/config/effective_config.ts` parses those direct defaults into `EffectiveConfig`.
- `src/runtime/weave/execution_config.ts` loads only Weave defaults, then optionally overlays an all-role command-scoped history override.
- Mesh config already exists for publication profile and sidecar workspace layout, but existing-mesh commands do not yet use `_mesh/_config/config.ttl` as a behavioral input for history, ResourcePage generation, ResourcePage presentation, or related policy.
- ResourcePage rendering already has a presentation-profile object and panel-selection model, but only the default profile is recognized and Semantic Flow metadata still relies on a boolean command opt-in.

### Conceptual model for this task

Config source location is not authority. Config authority comes from the attachment point that uses the config. A config artifact stored under a Knop can participate at mesh scope only if mesh-level config explicitly references or embeds it. Knop-local config cannot promote itself to mesh-wide behavior.

All authored config is potentially reusable, but "reusable config" is not a separate layer. Referenced config participates at the scope and layer of the attachment point that referenced it. The ontology may still need terms for source/provenance roles, but the resolver must not treat referenced config as one universal global layer.

Policy targets are explicit. Missing target fields never mean "any". The broad artifact selector is "any governed artifact in this config scope," not "all artifact roles" as a role value. A governed artifact is a Semantic Flow-managed `sflo:DigitalArtifact` governed by the current config scope through mesh or Knop support, inventory, payload, reference, page-definition, or config structure. Mesh support artifacts are governed by their mesh. Knop support and payload artifacts are governed by their Knop and the containing mesh. A merely referenced external artifact is not governed by a scope just because config in that scope points to it.

Under the default resolver profile, policy resolution is per queried target:

1. collect bindings whose selectors cover the queried target
2. prefer higher-precedence layers
3. within the winning layer, prefer structurally more specific selectors
4. within the same layer/family/specificity, prefer higher `sfcfg:policyPriority`
5. treat omitted priority as `0`
6. fail closed on unresolved incompatible ties

Selector specificity is structural. For artifacts, exact artifact is more specific than artifact role, and artifact role is more specific than any governed artifact. Priority must not be required for that ordinary specificity relationship.

### Policy families and scoped settings

Use policy bindings for target-selective behavior:

- history tracking
- ResourcePage generation
- ResourcePage presentation defaults

Do not force every config concern into the policy-binding model. Keep these as direct scoped settings unless/until they become target-selective:

- publication profile
- `workspaceRootRelativeToMeshRoot`
- history/state/manifestation naming defaults
- resolver/meta-policy values, with the bootstrap limits described in the spec

ResourcePage generation is not just a boolean. It answers what the runtime should do about ResourcePage materialization for a target: generate, suppress, defer, or generate on request. The first Weave implementation may only exercise generate/suppress, but the vocabulary should preserve the richer policy shape.

ResourcePage presentation is separate from publication profile. A GitHub Pages publication profile may create `.nojekyll`, but it must not imply a ResourcePage layout/profile, all-panels rendering, or Semantic Flow metadata inclusion.

### ResourcePage profiles

Add or support at least these built-in presentation profiles:

- `semantic-site-default`: current default Semantic Site behavior.
- `semantic-site-all-panels`: selects all ordinary data-gated generated panels and removes the Semantic Flow metadata opt-in gate so metadata can render when data exists.
- `semantic-site-no-panels`: keeps the generated page shell/chrome available, including canonical/title/breadcrumb/template structure as applicable, but suppresses generated data panels.

The CLI flag currently called `--include-semantic-flow-metadata` may remain as an operation convenience, but internally it should compile to a command-override presentation/profile or panel-selection policy. It should not become a durable boolean config concept.

### Resolver config

`sfcfg:ConfigResolutionConfig` is meta-config for the resolver. Portable config may request stricter resolver behavior but must not loosen the active application/operational resolver policy or expand trust. If Weave does not implement portable resolver narrowing in this slice, it should reject or ignore such declarations according to the active resolver and unknown-term policy rather than partially applying them.

## Open Issues

- Decide exact ontology names for the binding model. Current preferred direction is target/selector terminology, not context terminology: `sfcfg:PolicyBinding`, `sfcfg:PolicyDefinition`, `sfcfg:PolicyTarget`, `sfcfg:ArtifactPolicyTarget`, candidate target subclasses such as any-governed-artifact, artifact-role, and exact-artifact targets, plus `sfcfg:bindsPolicy`, `sfcfg:appliesToPolicyTarget`, `sfcfg:hasPolicyFamily`, and `sfcfg:policyPriority`.
- Decide whether policy-family values should be resources such as `sfcfg:policyFamily_historyTracking`, `sfcfg:policyFamily_resourcePageGeneration`, and `sfcfg:policyFamily_resourcePagePresentation`, or whether family is implicit from the policy-definition class.
- Decide whether old direct history and ResourcePage generation default terms are removed, deprecated, or retained only as non-authoring convenience vocabulary. Before v1.0, prefer migrating Weave defaults and parser behavior to the new model instead of adding long-term compatibility shims.
- Decide whether `sfcfg:hasDefaultResourcePagePresentationConfig` remains a direct scoped setting for simple defaults or becomes a policy-binding value at config scopes. Page-local `sfcfg:hasResourcePagePresentationConfig` on `sflo:ResourcePageDefinition` should remain direct presentation adjacency.
- Decide exact built-in IRI names for `semantic-site-all-panels` and `semantic-site-no-panels`.
- Decide CLI/API names for durable config edits after creation. Candidate shapes include `weave mesh config set-history-tracking-policy` and `weave mesh config set-resource-page-presentation`, but this task can land the runtime before the final UX if necessary.
- Decide whether Knop-local and Knop-inherited source discovery is implemented in this task or only supported in the compiled config API. The runtime data structure should support those layers even if source discovery is deferred.
- Decide where audit/explanation output lands in the first implementation: audit log, runtime metadata, structured debug output, or some combination.

## Decisions

- [[sf.spec.2026-05-25-config-behavior]] is the behavior contract for this task.
- Use "policy target" / "selector" terminology rather than "policy context" for new ontology terms.
- Do not treat referenced/reusable config as a separate precedence layer. It participates at its attachment point.
- Knop-inheritable config is an outbound offer to descendants and does not apply to the declaring Knop unless the same config is also attached as Knop-local.
- Omitted selector fields do not mean "any"; wildcard behavior must be explicit.
- Structural selector specificity beats broad selectors within the same layer. `sfcfg:policyPriority` is only for same-layer, same-family, same-effective-specificity conflicts.
- The broad artifact target is any governed artifact in the current config scope, not an all-artifact-role value.
- Publication profile and workspace-root relationship remain scoped settings, not policy bindings.
- Naming defaults may remain direct layered scoped settings unless/until a target-selective naming use case appears.
- Command overrides remain the highest ordinary layer and must not be persisted unless the user explicitly performs a config-editing operation.
- Do not implement a short-lived old-vocabulary mesh-config merge just to unblock the SFLO replay unless the user explicitly reprioritizes that over the clean policy model.

## Contract Changes

### Config ontology

- Add policy-binding vocabulary for reusable policy definitions, explicit bindings, explicit targets/selectors, policy family, and optional priority.
- Add explicit artifact policy target shapes for any governed artifact, artifact-role targets, and exact-artifact targets.
- Add or clarify ResourcePage policy target vocabulary, including page-kind and selector specificity rules where possible.
- Add controlled policy-family terms for history tracking, ResourcePage generation, and ResourcePage presentation defaults if the ontology does not encode those families through subclasses.
- Clarify or revise `ConfigLayerRole`/attachment-role vocabulary so referenced config and Knop-inheritable config are not misread as ordinary consumed layers at the declaring scope.
- Clarify direct history and ResourcePage generation default terms in light of the binding model.
- Keep publication profile, `workspaceRootRelativeToMeshRoot`, and naming defaults as direct scoped settings unless a later ontology pass intentionally changes them.

### Weave runtime

- Replace the current defaults-only effective config loader with a compiled config loader that can read application defaults, optional mesh-local config, and command overrides.
- Introduce internal runtime types for `CompiledConfig`, scoped settings, policy bindings, policy targets, policy index resolution, and resolution explanations.
- Compile application defaults from `defaults/application.ttl` using the new policy-binding model.
- Read `_mesh/_config/config.ttl` for existing-mesh commands when present and treat it as mesh-local config.
- Preserve fail-closed behavior for unsupported terms, duplicate singletons, malformed selectors, unknown policy values, and unresolved conflicts.
- Preserve command-scoped history overrides by compiling them into command-override policy bindings for any governed artifact or the relevant operation target.
- Replace boolean-shaped Semantic Flow metadata command internals with a command-override ResourcePage presentation/profile or panel-selection policy.
- Add supported built-in ResourcePage presentation profiles for default, all-panels, and no-panels behavior.
- Update audit/explanation output to include participating config sources, command overrides, and selected high-level policy values by family.

### Documentation and tasks

- Update Weave CLI/user docs when durable config options are exposed.
- Update [[wa.task.2026.2026-05-24_2304-honor-mesh-config]] or leave a note there that this task supersedes its implementation model.
- Update the SFLO replay recipe only after mesh-local policy config is honored at runtime.

## Testing

- Ontology parse/shape tests for the new policy-binding vocabulary.
- Unit-test policy resolution layer precedence: command override beats Knop-local, Knop-local beats inherited/mesh, mesh-local beats application defaults for targets covered by the upper layer.
- Unit-test structural specificity: exact artifact beats artifact role, artifact role beats any governed artifact, with no priority required.
- Unit-test `sfcfg:policyPriority`: omitted priority is `0`, higher priority wins at same specificity, and equal-priority incompatible values fail closed.
- Unit-test that missing selector fields do not imply "any" and malformed/incomplete targets are rejected.
- Unit-test governed-artifact matching for mesh support artifacts, Knop support artifacts, payload artifacts, and merely referenced external artifacts.
- Unit-test scoped settings validation for publication profile and `workspaceRootRelativeToMeshRoot`.
- Unit-test that naming defaults remain layered scoped settings if they are not migrated to policy bindings.
- Unit-test mesh-local history tracking policy for any governed artifact and role-specific same-layer exceptions.
- Unit-test mesh-local ResourcePage generation policy values, especially generate versus suppress.
- Unit-test mesh-local ResourcePage presentation profile selection for default, all-panels, and no-panels.
- Unit-test `semantic-site-all-panels` includes the Semantic Flow metadata panel without the old metadata opt-in data requirement.
- Unit-test `semantic-site-no-panels` preserves page shell/chrome while suppressing generated data panels.
- Integration-test `weave`, `weave version`, and `weave generate` honoring `_mesh/_config/config.ttl` without repeated command flags.
- Regression-test command-scoped history and ResourcePage presentation overrides beating mesh-local config for one run.
- Integration-test unsupported mesh-local policy values, duplicate singleton settings, and conflicting policy bindings failing closed.
- Run focused `deno task test`/`deno task check` for Weave runtime changes; run `deno task lint` after broader structural changes; run repo-appropriate ontology/framework checks if ontology/spec files change.

## Non-Goals

- Do not make publication profile imply history, ResourcePage generation, or ResourcePage presentation behavior.
- Do not add an all-artifact-role enum value. Use explicit target/selector shapes for broad matching.
- Do not treat referenced config as a global layer.
- Do not let portable config expand host trust, local path access, remote retrieval, or resolver authority.
- Do not implement unpinned remote config retrieval in this slice.
- Do not persist `sfcfg:ResolvedConfig` or full `sfcfg:ConfigResolutionRecord` output by default.
- Do not require a compatibility layer for pre-v1.0 direct policy-default vocabulary unless the implementation needs a very short migration bridge.
- Do not make `weave mesh create` an interactive config wizard.
- Do not block the first SFLO manual replay if command overrides remain the temporary runbook path.

## Implementation Plan

- [ ] Finalize ontology term names for policy bindings, policy definitions, policy families, policy targets, target properties, and priority.
- [ ] Update `semantic-flow-config-ontology.ttl` with the chosen policy-binding and target vocabulary.
- [ ] Clarify or revise config layer/attachment-role vocabulary around referenced config and Knop-inheritable config.
- [ ] Update Weave default config TTL to express history tracking, ResourcePage generation, and ResourcePage presentation defaults using the chosen binding model.
- [ ] Add or revise built-in ResourcePage presentation profiles for `semantic-site-default`, `semantic-site-all-panels`, and `semantic-site-no-panels`.
- [ ] Design the Weave internal `CompiledConfig` / `PolicyIndex` runtime types and policy-resolution trace structure.
- [ ] Implement policy-target parsing and validation, including explicit any-governed-artifact, artifact-role, and exact-artifact targets.
- [ ] Implement policy resolution by queried target, layer, structural specificity, priority, and fail-closed conflict detection.
- [ ] Implement scoped-setting parsing/validation for publication profile, `workspaceRootRelativeToMeshRoot`, naming defaults, and resolver/meta-policy fields that remain in scope.
- [ ] Replace `loadWeaveDefaultEffectiveConfig`/`loadEffectiveConfigForExecution` call sites with a mesh-root-aware loader that can compile defaults, mesh-local config, and command overrides.
- [ ] Update command override handling so history overrides and metadata/presentation overrides compile into the command-override layer.
- [ ] Wire generated ResourcePage rendering to selected ResourcePage presentation policy rather than a durable boolean metadata flag.
- [ ] Add unit tests for ontology parsing, policy-target parsing, policy resolution, scoped settings, and ResourcePage profile behavior.
- [ ] Add integration tests for mesh-local `_mesh/_config/config.ttl` changing history, ResourcePage generation, and ResourcePage presentation behavior.
- [ ] Update CLI docs and the SFLO replay recipe when durable mesh-local config can replace repeated command overrides.
- [ ] Run focused tests/checks during development, then run `deno task lint` for broader Weave structural changes and repo-appropriate checks for ontology/framework changes.
