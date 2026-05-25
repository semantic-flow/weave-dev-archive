---
id: d2kyk9aqoevufphsrfwdamn
title: 2026 05 24_2304 Honor Mesh Config
desc: ''
updated: 1779745999478
created: 1779689090915
---

## Goals

- refine our conceptual model of configuration, and revise the semantic-flow-config-ontology.ttl and 
- Make existing-mesh commands honor mesh-local config from `_mesh/_config/config.ttl`, not only built-in defaults and command-line overrides.
- Let a mesh persist durable history-tracking policy, especially "history on everywhere" for branch-published SFLO.
- Let a mesh persist ResourcePage presentation preferences that currently require command flags, especially inclusion of the Semantic Flow metadata panel.
- Let a mesh persist a universal ResourcePage layout/profile so generated panels such as references, properties, blank nodes, raw source, and history can be selected mesh-wide instead of being repeated in each authored `ResourcePageDefinition`.
- Keep command-line options as explicit command overrides with clear precedence over mesh config.
- Keep `weave mesh create` small, but allow it to seed the mesh config needed for common publication profiles.
- Preserve fail-closed config behavior: malformed, unsupported, or contradictory mesh config should stop execution rather than silently falling back to defaults.

## Summary

Weave has a config ontology and a built-in default config, and `weave mesh create` already writes a mesh-owned `_mesh/_config/config.ttl` when needed for `workspaceRootRelativeToMeshRoot` or a persisted publication profile. However, the main execution path still calls `loadEffectiveConfigForExecution`, which loads only the built-in defaults and then applies command-scoped history overrides. That means mesh-local policy terms such as `sfcfg:hasDefaultHistoryTrackingPolicy`, `sfcfg:hasHistoryTrackingDefault`, `sfcfg:hasDefaultResourcePageGenerationPolicy`, `sfcfg:hasDefaultResourcePagePresentationConfig`, and naming-policy terms are not yet honored as durable mesh behavior. The history-policy terms also need review because direct policy assignments on `Config` are context-free and ambiguous. The current preferred direction is to disallow direct policy assignments on `Config` and instead use named policy definitions, named policy contexts, and named policy bindings.

The immediate pressure is the SFLO `gh-pages` replay. We want the SFLO mesh to have history tracking turned on everywhere, and we want generated pages to include the Semantic Flow metadata panel without having to remember a final `weave generate --include-semantic-flow-metadata` command. Those should be mesh policy choices, not fragile runbook details.

There is also pressure from the Alice Bio ResourcePage fixture ladder. We want to change generated-page layout policy universally across the mesh, including enabling the references panel and eventually all ordinary data-gated generated panels, without hand-editing every page definition. Page-local `sfcfg:hasGeneratedResourcePagePanelSelection` is still useful for authored page definitions that intentionally opt into generated panels, but mesh-wide presentation defaults should be controlled by `_mesh/_config/config.ttl`.

This task should make mesh config a real input to the effective execution config. It should also decide which `mesh create` options are worth adding now versus which policies should be edited into `_mesh/_config/config.ttl` manually until the UI/CLI surface is clearer.

## Discussion

Current behavior:

- `defaults/application.ttl` sets the built-in application history fallback to `currentOnly`, with role policies for `payload` and `config` as `versioned`, and `meshInventory`, `knopInventory`, and `runtimeMeta` as `currentOnly`.
- `defaults/config-resolution.ttl` already names the intended config layer order, including `meshLocal`, `meshInheritable`, `knopInherited`, `reusableConfig`, `knopLocal`, `knopInheritable`, and `commandOverride`.
- `weave mesh create --publication-profile github-pages` persists `sfcfg:hasPublicationProfile sfcfg:publicationProfile_githubPages` and creates `.nojekyll`.
- `weave mesh create` does not currently accept history, page-generation, ResourcePage presentation, or naming-policy options.
- `loadEffectiveConfigForExecution` currently loads only Weave defaults. If `--history-tracking-policy` is passed to `weave`, `weave version`, or `weave generate`, it constructs an in-memory all-role override. That override is useful, but it is not a mesh-owned policy.
- `weave generate --include-semantic-flow-metadata` includes the Semantic Flow metadata panel. Top-level `weave` does not expose that flag, and the preference is not persisted in mesh config.
- The built-in ResourcePage presentation config includes a `semantic-flow-metadata` panel selection, but it has the data requirement `sfcfg:panelDataRequirement_semanticFlowMetadataOptIn`, so rendering still needs an explicit opt-in flag.
- The config ontology already models `sfcfg:ResourcePagePresentationConfig`, `sfcfg:ResourcePagePanelSelection`, `sfcfg:hasDefaultResourcePagePresentationConfig`, and panel data requirements. The current Weave parser recognizes the built-in Semantic Site presentation profile and its panel selections, but it does not yet let `_mesh/_config/config.ttl` supply a mesh-local or built-in-variant layout that changes the selected panels for every generated ResourcePage.

Config gaps to close:

- Existing-mesh commands should load built-in defaults plus the current mesh config, then apply command overrides. The command override should remain the final layer.
- Mesh config should be parsed with the same strictness as built-in config: unsupported terms, duplicate singleton values, unsupported policy IRIs, cycles, or unsafe config references should be errors.
- The effective config loader needs a path-aware entry point such as `loadEffectiveConfigForExecution({ meshRoot, historyTrackingPolicyOverride, includeSemanticFlowMetadataOverride })` rather than only accepting a history override.
- Mesh-local config should be able to set governed-artifact history tracking policy and per-role history policies through named policy bindings rather than direct policy assertions on the config resource.
- Mesh-local config should be able to override default ResourcePage generation policy and per-role generation policies.
- Mesh-local config should be able to choose the default ResourcePage presentation config when the referenced config is known and supported.
- Mesh-local config should be able to choose a ResourcePage presentation profile that applies to all generated ResourcePages in the mesh, including a profile that selects all ordinary data-gated panels and therefore makes the references panel appear automatically wherever reference data exists.
- Mesh-local ResourcePage layout policy should not require adding `sfcfg:hasGeneratedResourcePagePanelSelection` triples to every authored page definition. Page-local selections should remain an override or composition hook for authored pages, not the only way to affect generated panels mesh-wide.
- Mesh-local config should be able to express whether the Semantic Flow metadata panel is included by default by selecting a supported ResourcePage presentation variant whose metadata panel no longer has the opt-in data requirement.
- Mesh-local config should be able to override naming defaults for history, state, and manifestation segments where the ontology already supports those policies.
- Mesh-local config should continue to carry publication profile and workspace-root relationship without conflating publication host policy with execution policy.

SFLO desired mesh config:

```ttl
@prefix sfcfg: <https://semantic-flow.github.io/sflo/config/> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .

<> a sfcfg:MeshConfig ;
  sfcfg:hasPublicationProfile sfcfg:publicationProfile_githubPages ;
  sfcfg:hasPolicyBinding <#history-versioned-for-governed-artifacts> ;
  sfcfg:hasDefaultResourcePageGenerationPolicy sfcfg:resourcePageGenerationPolicy_generate ;
  sfcfg:hasDefaultResourcePagePresentationConfig <https://semantic-flow.github.io/weave/defaults/resource-page-presentation/semantic-site-all-panels> .

<#history-versioned-for-governed-artifacts> a sfcfg:PolicyBinding ;
  sfcfg:bindsPolicy <#history-versioned-policy> ;
  sfcfg:appliesToPolicyContext <#governed-artifacts> .

<#history-versioned-policy> a sfcfg:TrackingPolicyDefinition ;
  sfcfg:hasTrackingPolicyKind sfcfg:trackingPolicyKind_history ;
  sfcfg:hasHistoryTrackingPolicy sfcfg:historyTrackingPolicy_versioned .

<#governed-artifacts> a sfcfg:GovernedArtifactPolicyContext .
```

This should be enough for "history on everywhere" because a higher-layer governed-artifact policy binding should reset lower-layer role policies for the same policy family. The built-in application config should provide the fallback that catches all artifacts when no later config layer has anything to say. A mesh-local governed-artifact policy binding is not a "default" in the same sense; it is a scope-wide policy override for artifacts governed by that mesh config. When a mesh binds a versioned history policy to `sfcfg:GovernedArtifactPolicyContext`, the effective config should treat that as the mesh's baseline for every artifact role unless the same mesh config also declares a more specific or higher-priority applicable binding. Config artifacts governed by that mesh are included in `sfcfg:GovernedArtifactPolicyContext`; use an artifact-role context for `sfcfg:artifactRole_config` if config artifacts need a different policy from the rest of the mesh.

Do not use `sfcfg:hasHistoryTrackingPolicy` directly on a `sfcfg:Config` for this scope-wide meaning. A config resource may also be a `sfcfg:ConfigArtifact` and `sflo:DigitalArtifact`; a bare `sfcfg:hasHistoryTrackingPolicy` on that resource lacks enough context to say whether it is an artifact policy, a governed-artifact policy, or something else. Use policy definitions plus policy bindings and explicit contexts instead. A separate context for the config artifact itself is only needed when a policy must target that exact artifact rather than all governed artifacts or all `artifactRole_config` artifacts.

Policy definitions should be reusable. Prefer one `sfcfg:TrackingPolicyDefinition` class over a separate definition class for every policy family unless implementation pressure proves the generic shape too vague. A tracking policy definition can identify its included dimensions, such as `sfcfg:trackingPolicyKind_history`, and then carry the corresponding policy value. A named tracking policy definition can be bound to multiple contexts in the same config or from reusable config, and a binding can carry resolution metadata that should not be part of the reusable policy itself.

Example exception:

```ttl
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .

<> a sfcfg:MeshConfig ;
  sfcfg:hasPolicyBinding
    <#history-versioned-for-governed-artifacts>,
    <#runtime-meta-current-only-binding> .

<#history-versioned-for-governed-artifacts> a sfcfg:PolicyBinding ;
  sfcfg:bindsPolicy <#history-versioned-policy> ;
  sfcfg:appliesToPolicyContext <#governed-artifacts> .

<#history-versioned-policy> a sfcfg:TrackingPolicyDefinition ;
  sfcfg:hasTrackingPolicyKind sfcfg:trackingPolicyKind_history ;
  sfcfg:hasHistoryTrackingPolicy sfcfg:historyTrackingPolicy_versioned .

<#governed-artifacts> a sfcfg:GovernedArtifactPolicyContext .

<#runtime-meta-current-only-binding> a sfcfg:PolicyBinding ;
  sfcfg:bindsPolicy <#history-current-only-policy> ;
  sfcfg:appliesToPolicyContext <#runtime-meta-artifacts> ;
  sfcfg:policyPriority "10"^^xsd:integer .

<#history-current-only-policy> a sfcfg:TrackingPolicyDefinition ;
  sfcfg:hasTrackingPolicyKind sfcfg:trackingPolicyKind_history ;
  sfcfg:hasHistoryTrackingPolicy sfcfg:historyTrackingPolicy_currentOnly .

<#runtime-meta-artifacts> a sfcfg:ArtifactRolePolicyContext ;
  sfcfg:hasArtifactRole sfcfg:artifactRole_runtimeMeta .
```

Do not add an "all artifact roles" vocabulary term for this slice. `sfcfg:GovernedArtifactPolicyContext` already has that job once merge semantics are clear. Do not have `mesh create --history-tracking-policy versioned` write explicit artifact-role bindings for every known artifact role; that would make config noisy, stale as roles evolve, and harder to override locally.

Policy conflict resolution should be deterministic and fail closed. Config layer precedence comes first and is resolved per layer: command overrides beat Knop-local config, Knop-local config beats inherited/mesh config, and mesh-local config beats application defaults. For this slice, a higher layer's applicable policy wins even when a lower layer has a more explicit context. Within one layer and policy family, higher `sfcfg:policyPriority` wins. If `sfcfg:policyPriority` is omitted, treat it as integer `0`. If two applicable bindings in the same layer and policy family have the same effective priority and produce different policy values, reject the config as ambiguous rather than relying on RDF serialization order. More specific contexts can either be compiled to a higher priority by convention or explicitly carry a higher `sfcfg:policyPriority`; for this slice, explicit priority is less surprising.

The desired config is also not enough by itself for the Semantic Flow metadata panel until the opt-in behavior has a durable config hook. Prefer representing this as a supported ResourcePage presentation profile variant rather than as a boolean config switch. The useful built-in extremes are `semantic-site-all-panels`, which selects all data-gated generated panels including Semantic Flow metadata, and `semantic-site-no-panels`, which suppresses generated panels for sparse or highly authored pages. For the immediate SFLO replay, the runbook can still use `weave generate --include-semantic-flow-metadata`; this task should make that an implementation gap rather than a permanent habit.

One caution: "history on everywhere" will produce more generated support history. That is correct for SFLO because the publication mesh itself is a public artifact we want to inspect. It is not necessarily the right default for small sidecar meshes or quick fixtures, so this should be mesh-local policy, not a global Weave default.

## Resolved Questions

- Add `weave mesh create --history-tracking-policy <policy>` once persisted mesh policy is honored at runtime. It should write a named policy definition, a `sfcfg:GovernedArtifactPolicyContext`, and a binding from the config to that policy/context, not a bundle of per-role policies.
- Treat a mesh-local governed-artifact history tracking binding as a policy-family reset over lower-layer role policies. Mesh-local artifact-role context bindings then mean exceptions to the mesh baseline.
- Resolve policy precedence per config layer before considering binding priority or context specificity. A higher layer's applicable policy wins over any lower-layer policy for this slice, even if the lower-layer policy has a more explicit context.
- Keep command-scoped `--history-tracking-policy` as an all-role per-run override. It should remain stronger than mesh config and should not be reinterpreted as default-only.
- Do not add an "all artifact roles" or "all artifact types" vocabulary term in this slice. If future policy needs class-, shape-, or artifact-specific matching, introduce that as a broader selector model rather than a one-off history shortcut.
- Do not add `weave mesh create --include-semantic-flow-metadata` as a standalone persisted option. Use a presentation/profile option instead, because the metadata panel is one layout concern among references, raw source, history, blank nodes, and other generated panels.
- Represent Semantic Flow metadata inclusion through a supported ResourcePage presentation variant, not a new SFLO boolean term.
- Add built-in ResourcePage presentation profiles for `semantic-site-all-panels` and `semantic-site-no-panels`. `semantic-site-all-panels` should select all data-gated generated panels, including Semantic Flow metadata; `semantic-site-no-panels` should suppress generated panels while keeping the page shell available.
- In this slice, accept only known supported ResourcePage presentation profile IRIs from mesh config. Defer arbitrary mesh-owned `sfcfg:ResourcePagePresentationConfig` profiles until asset resolution, trust boundaries, template compatibility, and validation are more complete.
- `weave mesh create` should expose a persisted ResourcePage presentation/profile option once the profile is honored at runtime.
- Add API and CLI surfaces for changing a mesh's durable history profile and ResourcePage presentation profile after mesh creation.
- Limit portable mesh config loading to `_mesh/_config/config.ttl` for this task. Leave reusable config attachments and full `hasConfigSource` resolution as declared future shape.
- Do not load workspace or machine-local operational config in this implementation pass. Focus on built-in defaults, mesh-local config, and command overrides.
- Audit logs should report participating config sources, command overrides, and resolved high-level policy values by family. They do not need to serialize the entire effective config in this slice.
- Implement only the config-resolution semantics needed for this path now: built-in defaults, mesh-local config, command override precedence, strict singleton/policy validation, and fail-closed handling. Leave the rest of the config-resolution ontology as a forward contract.

## Remaining Open Issues

- Decide exact term names for the reusable policy model. Prefer a small class set if it stays clear: `sfcfg:PolicyBinding`, `sfcfg:TrackingPolicyDefinition`, `sfcfg:GovernedArtifactPolicyContext`, `sfcfg:ArtifactRolePolicyContext`, and `sfcfg:ArtifactPolicyContext`. Candidate properties/values include `sfcfg:hasPolicyBinding`, `sfcfg:bindsPolicy`, `sfcfg:appliesToPolicyContext`, `sfcfg:policyPriority`, `sfcfg:hasTrackingPolicyKind`, and `sfcfg:trackingPolicyKind_history`.
- Decide the exact API and CLI command names for changing durable history profile and ResourcePage presentation profile after mesh creation.

## Decisions

- Keep Weave's built-in defaults conservative; do not globally change `defaults/application.ttl` to history-on-everywhere.
- For SFLO, prefer mesh-local durable policy over repeating `--history-tracking-policy versioned` in every command.
- Make "history on everywhere" explicit through merge semantics: a mesh-local binding from a history policy to `sfcfg:GovernedArtifactPolicyContext` resets lower-layer role policies for that policy family.
- Keep precedence per config layer. All Knop-local applicable policy beats mesh-level applicable policy; all mesh-level applicable policy beats application-level applicable policy.
- Do not write explicit all-role artifact-role policy bindings for `weave mesh create --history-tracking-policy versioned`.
- Use `sfcfg:policyPriority` on policy bindings for same-layer conflict resolution. Missing priority means integer `0`; equal-priority conflicting values fail closed.
- Disallow direct tracking policy assignments on `Config`; configs should carry contextual policy bindings instead.
- Keep command overrides last in precedence. A command flag should win over mesh config because it is explicit operator intent for that run.
- Treat unsupported mesh config as an error. Silent fallback would make publication replays hard to trust.
- Keep `publicationProfile` persistence separate from history/page-generation/presentation policy even though all live in `_mesh/_config/config.ttl`.
- Do not block the SFLO manual replay on this task; the current recipe can use command overrides until mesh config is honored.
- Do the universal ResourcePage layout work through mesh config before adding Alice Bio fixture rungs that rely on the references panel or all ordinary generated panels being selected mesh-wide.
- Prefer built-in ResourcePage presentation profiles over boolean mesh config terms for durable page-layout choices such as Semantic Flow metadata inclusion.
- Add built-in ResourcePage presentation profiles for all generated panels and no generated panels.
- Expose durable history and ResourcePage presentation profile changes through both API and CLI surfaces.

## Contract Changes

- Add or expose an execution-config entry point that accepts `meshRoot` and loads the current mesh config before command overrides.
- Merge mesh-local config over Weave defaults for the policy families already parsed from `ApplicationConfig`/`MeshConfig`: history tracking, ResourcePage generation, ResourcePage presentation, ResourcePage regeneration config policy, and naming policies. For history tracking, an upper-layer applicable policy binding resets lower-layer applicable bindings for that same policy family. Within one layer and policy family, choose the highest-priority applicable binding; reject equal-priority conflicting values. Keep the ResourcePage generation merge semantics aligned after the naming review for that family.
- Extend ResourcePage presentation resolution so a mesh can select a supported default profile that changes panel selections for every generated ResourcePage. The first target profiles should be `semantic-site-all-panels` and `semantic-site-no-panels`.
- Extend `weave mesh create` and `planMeshCreate` with `--history-tracking-policy <policy>` once mesh-local history config is honored at runtime. Persist it as a named policy definition plus a governed-artifact policy binding.
- Extend `weave mesh create` and `planMeshCreate` with a persisted ResourcePage presentation/profile option once mesh-local presentation config is honored at runtime.
- Add API and CLI surfaces for changing durable history profile and ResourcePage presentation profile after mesh creation.
- Keep `--history-tracking-policy` on `weave`, `weave version`, and `weave generate` as an all-role per-run override.
- Add a durable config path for including the Semantic Flow metadata panel by default through a supported presentation profile variant.
- Update CLI docs so `mesh create`, `weave`, and `generate` explain which options persist mesh policy and which options are per-run overrides.
- Update the SFLO replay recipe in [[wu.cli-reference.examples.sflo]] once mesh-local policy can replace repeated command flags.

## Testing

- Unit-test the chosen "history on everywhere" representation and verify all artifact roles resolve to `versioned`.
- Unit-test a mesh-local config that binds only the governed-artifact history policy to `sfcfg:historyTrackingPolicy_versioned` and verify it resets lower-layer role policies.
- Unit-test per-role mesh-local history overrides through named artifact-role policy context bindings as same-layer exceptions to the mesh baseline.
- Unit-test `sfcfg:policyPriority`: missing priority behaves as `0`, higher same-layer priority wins, and equal-priority conflicting values fail closed.
- Unit-test layer precedence: Knop-local governed-artifact policy beats mesh role-specific policy, and mesh governed-artifact policy beats application role-specific policy.
- Unit-test that direct `sfcfg:hasHistoryTrackingPolicy` on `sfcfg:MeshConfig` is rejected unless the ontology review defines an unambiguous artifact-level policy shape.
- Unit-test mesh-local ResourcePage generation policy overrides through `sfcfg:hasDefaultResourcePageGenerationPolicy` and `sfcfg:hasResourcePageGenerationDefault`.
- Unit-test mesh-local ResourcePage presentation/profile selection, including `semantic-site-all-panels` and `semantic-site-no-panels`.
- Integration-test a mesh whose `_mesh/_config/config.ttl` selects the all-panels ResourcePage presentation and verify `weave`/`weave generate` apply it across generated identifier, Knop, simple, and ReferenceCatalog pages as appropriate.
- Integration-test `weave mesh create` writes persisted ResourcePage presentation/profile config when requested.
- Unit-test mesh-local naming-policy overrides for history, state, and manifestation naming.
- Unit-test unsupported mesh-local policy values and duplicate singleton values fail closed.
- Integration-test `weave mesh create --history-tracking-policy versioned` writes the expected `_mesh/_config/config.ttl` with named policy/context/binding resources and no generated all-role artifact-role policy block.
- Integration-test a generated mesh with history-on-everywhere creates historical support states without requiring command-scoped `--history-tracking-policy`.
- Integration-test top-level `weave` and `weave generate` both honor the mesh's Semantic Flow metadata panel preference once that config hook exists.
- Regression-test that command-scoped `--history-tracking-policy currentOnly` can still override a mesh-local versioned governed-artifact policy for a single run.
- Update fixture ladder coverage only after the core mesh-config behavior is stable enough that fixture commands can stop carrying redundant overrides.

## Non-Goals

- Do not change Weave's global built-in defaults to history-on-everywhere.
- Do not make `weave mesh create` an interactive policy wizard in this slice.
- Do not implement remote config retrieval or unpinned external config references.
- Do not make `publicationProfile` imply history or ResourcePage presentation behavior.
- Do not require the first SFLO `/tmp/sflo` manual replay to wait for this task.

## Implementation Plan

- [ ] Audit current effective-config loading and identify every command path still calling built-in defaults directly.
- [ ] Review and clarify the history-policy vocabulary before runtime wiring, especially direct config policy assertions versus contextual policy bindings, and decide exact policy definition/binding/context terms.
- [ ] Add a mesh-root-aware effective-config loader that reads `_mesh/_config/config.ttl` when present.
- [ ] Parse mesh-local policy terms by reusing or generalizing the existing `ApplicationConfig` parser where the shapes are shared with `MeshConfig`, including policy-family reset semantics for upper-layer governed-artifact history bindings and priority-based same-layer conflict resolution.
- [ ] Add ResourcePage presentation/profile support for mesh-local universal layout policy, including built-in all-panels and no-panels profiles.
- [ ] Preserve command override precedence and update audit logging to report command overrides separately from mesh-local config.
- [ ] Add `MeshConfig` tests for governed-artifact history tracking policy, per-role history policy, ResourcePage generation policy, presentation config, naming policies, unsupported values, and duplicate values.
- [ ] Add `weave mesh create --history-tracking-policy <policy>` once the persisted mesh policy is honored at runtime.
- [ ] Add mesh-create, API, and CLI surfaces for persisted ResourcePage presentation/profile selection once honored at runtime.
- [ ] Implement `semantic-site-all-panels` and `semantic-site-no-panels` ResourcePage presentation profiles.
- [ ] Implement the Semantic Flow metadata panel config hook for both top-level `weave` generation and explicit `weave generate`.
- [ ] Update [[wu.cli-reference.mesh.create]], [[wu.cli-reference.weave]], [[wu.cli-reference.generate]], and [[wu.cli-reference.examples.sflo]].
- [ ] Run focused integration tests for mesh config plus `deno task check`; run `deno task ci` before merge.
