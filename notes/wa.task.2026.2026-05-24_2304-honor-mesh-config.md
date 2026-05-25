---
id: d2kyk9aqoevufphsrfwdamn
title: 2026 05 24_2304 Honor Mesh Config
desc: ''
updated: 1779689090915
created: 1779689090915
---

## Goals

- Make existing-mesh commands honor mesh-local config from `_mesh/_config/config.ttl`, not only built-in defaults and command-line overrides.
- Let a mesh persist durable history-tracking policy, especially "history on everywhere" for branch-published SFLO.
- Let a mesh persist ResourcePage presentation preferences that currently require command flags, especially inclusion of the Semantic Flow metadata panel.
- Let a mesh persist a universal ResourcePage layout/profile so generated panels such as references, properties, blank nodes, raw source, and history can be selected mesh-wide instead of being repeated in each authored `ResourcePageDefinition`.
- Keep command-line options as explicit command overrides with clear precedence over mesh config.
- Keep `weave mesh create` small, but allow it to seed the mesh config needed for common publication profiles.
- Preserve fail-closed config behavior: malformed, unsupported, or contradictory mesh config should stop execution rather than silently falling back to defaults.

## Summary

Weave has a config ontology and a built-in default config, and `weave mesh create` already writes a mesh-owned `_mesh/_config/config.ttl` when needed for `workspaceRootRelativeToMeshRoot` or a persisted publication profile. However, the main execution path still calls `loadEffectiveConfigForExecution`, which loads only the built-in defaults and then applies command-scoped history overrides. That means mesh-local policy terms such as `sfcfg:hasDefaultHistoryTrackingPolicy`, `sfcfg:hasHistoryTrackingDefault`, `sfcfg:hasDefaultResourcePageGenerationPolicy`, `sfcfg:hasDefaultResourcePagePresentationConfig`, and naming-policy terms are not yet honored as durable mesh behavior.

The immediate pressure is the SFLO `gh-pages` replay. We want the SFLO mesh to have history tracking turned on everywhere, and we want generated pages to include the Semantic Flow metadata panel without having to remember a final `weave generate --include-semantic-flow-metadata` command. Those should be mesh policy choices, not fragile runbook details.

There is also pressure from the Alice Bio ResourcePage fixture ladder. We want to change generated-page layout policy universally across the mesh, including enabling the references panel and eventually all ordinary data-gated generated panels, without hand-editing every page definition. Page-local `sfcfg:hasGeneratedResourcePagePanelSelection` is still useful for authored page definitions that intentionally opt into generated panels, but mesh-wide presentation defaults should be controlled by `_mesh/_config/config.ttl`.

This task should make mesh config a real input to the effective execution config. It should also decide which `mesh create` options are worth adding now versus which policies should be edited into `_mesh/_config/config.ttl` manually until the UI/CLI surface is clearer.

## Discussion

Current behavior:

- `defaults/application.ttl` sets the built-in default history policy to `currentOnly`, with role overrides for `payload` and `config` as `versioned`, and `meshInventory`, `knopInventory`, and `runtimeMeta` as `currentOnly`.
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
- Mesh-local config should be able to override default history tracking policy and per-role history policies.
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

<> a sfcfg:MeshConfig ;
  sfcfg:hasPublicationProfile sfcfg:publicationProfile_githubPages ;
  sfcfg:hasDefaultHistoryTrackingPolicy sfcfg:historyTrackingPolicy_versioned ;
  sfcfg:hasDefaultResourcePageGenerationPolicy sfcfg:resourcePageGenerationPolicy_generate ;
  sfcfg:hasDefaultResourcePagePresentationConfig <https://semantic-flow.github.io/weave/defaults/resource-page-presentation/semantic-site-publication> .
```

This should be enough for "history on everywhere" because a higher-layer default history tracking policy should reset lower-layer role defaults for the same policy family. The built-in defaults currently have role-specific history policies for `payload`, `config`, `meshInventory`, `knopInventory`, and `runtimeMeta`; those are application-default details, not mesh-local exceptions. When a mesh declares `sfcfg:hasDefaultHistoryTrackingPolicy sfcfg:historyTrackingPolicy_versioned`, the effective config should treat that as the mesh's baseline for every artifact role unless the same mesh config also declares role-specific exceptions through `sfcfg:hasHistoryTrackingDefault`.

Example exception:

```ttl
<> a sfcfg:MeshConfig ;
  sfcfg:hasDefaultHistoryTrackingPolicy sfcfg:historyTrackingPolicy_versioned ;
  sfcfg:hasHistoryTrackingDefault [
    a sfcfg:ArtifactRolePolicy ;
    sfcfg:hasArtifactRole sfcfg:artifactRole_runtimeMeta ;
    sfcfg:hasHistoryTrackingPolicy sfcfg:historyTrackingPolicy_currentOnly
  ] .
```

Do not add an "all artifact roles" vocabulary term for this slice. The config-level default already has that job once merge semantics are clear. Do not have `mesh create --history-tracking-policy versioned` write explicit `sfcfg:hasHistoryTrackingDefault` entries for every known artifact role; that would make config noisy, stale as roles evolve, and harder to override locally.

The desired config is also not enough by itself for the Semantic Flow metadata panel until the opt-in behavior has a durable config hook. Prefer representing this as a supported ResourcePage presentation profile variant rather than as a boolean config switch. A built-in publication-oriented profile can select all ordinary data-gated generated panels and make the Semantic Flow metadata panel available by default without requiring `--include-semantic-flow-metadata`. For the immediate SFLO replay, the runbook can still use `weave generate --include-semantic-flow-metadata`; this task should make that an implementation gap rather than a permanent habit.

One caution: "history on everywhere" will produce more generated support history. That is correct for SFLO because the publication mesh itself is a public artifact we want to inspect. It is not necessarily the right default for small sidecar meshes or quick fixtures, so this should be mesh-local policy, not a global Weave default.

## Open Issues

- Should `weave mesh create` expose a single `--history-tracking-policy versioned` option that persists `sfcfg:hasDefaultHistoryTrackingPolicy`, or should initial policy editing remain a manual config-file step until more policies are implemented?
- If `--history-tracking-policy versioned` is meant to mean "everywhere", should it write explicit per-role history policy entries for all known artifact roles?
- Should `weave mesh create` expose `--include-semantic-flow-metadata`, or should that belong to a more explicit ResourcePage presentation option?
- Do we need a new SFLO config term for "include Semantic Flow metadata by default", or should this be represented by a built-in ResourcePage presentation variant?
- Should the "all ordinary generated panels" layout be represented as a second built-in ResourcePage presentation profile, as mesh-local `sfcfg:ResourcePagePresentationConfig` triples, or as an overlay/patch on the built-in Semantic Site profile?
- If mesh-local ResourcePage presentation config repeats built-in templates/stylesheets but changes panel selections, should Weave accept those mesh-owned profile IRIs directly or require them to refer to known built-in variant IRIs?
- Should mesh-local config be limited to `_mesh/_config/config.ttl` in this slice, or should the full config source vocabulary already allow reusable config attachments?
- Should workspace and machine-local operational config be loaded in the same implementation pass, or should this task focus only on mesh-local config plus command overrides?
- Should command-line `--history-tracking-policy` continue to override all artifact roles, or should it only override the default policy while preserving per-role mesh config?
- How should generated logs and audit records describe the config sources and policy values that were applied?
- How much of the config-resolution ontology should be implemented now versus left as a declared future shape?

## Suggested Resolutions

- Add `weave mesh create --history-tracking-policy <policy>` now, but only for persisted mesh policy values that the runtime actually honors. It should write `sfcfg:hasDefaultHistoryTrackingPolicy`, not a bundle of per-role defaults.
- Treat a mesh-local default history tracking policy as a policy-family reset over lower-layer role defaults. Mesh-local `sfcfg:hasHistoryTrackingDefault` entries then mean exceptions to the mesh baseline.
- Keep command-scoped `--history-tracking-policy` as an all-role per-run override. It should remain stronger than mesh config and should not be reinterpreted as default-only.
- Do not add an "all artifact roles" or "all artifact types" vocabulary term in this slice. If future policy needs class-, shape-, or artifact-specific matching, introduce that as a broader selector model rather than a one-off history shortcut.
- Do not add `weave mesh create --include-semantic-flow-metadata` as a standalone persisted option. Use a presentation/profile option instead, because the metadata panel is one layout concern among references, raw source, history, blank nodes, and other generated panels.
- Represent Semantic Flow metadata inclusion through a supported ResourcePage presentation variant, not a new SFLO boolean term. The first useful publication-oriented variant could be `semantic-site-publication`, selecting the ordinary data-gated generated panels plus the Semantic Flow metadata panel without the current opt-in requirement.
- Represent the "all ordinary generated panels" layout as a built-in ResourcePage presentation profile rather than arbitrary mesh-local presentation triples or an overlay patch. A candidate profile such as `semantic-site-all-panels` can stay metadata-opt-in, while `semantic-site-publication` can include Semantic Flow metadata by default. Built-in profile IRIs are easier to validate fail-closed and easier to document.
- In this slice, accept only known supported ResourcePage presentation profile IRIs from mesh config. Defer arbitrary mesh-owned `sfcfg:ResourcePagePresentationConfig` profiles until asset resolution, trust boundaries, template compatibility, and validation are more complete.
- Limit portable mesh config loading to `_mesh/_config/config.ttl` for this task. Leave reusable config attachments and full `hasConfigSource` resolution as declared future shape.
- Do not load workspace or machine-local operational config in this implementation pass. Focus on built-in defaults, mesh-local config, and command overrides.
- Audit logs should report the config sources that participated, the command overrides applied, and the resolved high-level policy values by family. They do not need to serialize the entire effective config in this slice.
- Implement only the config-resolution semantics needed for this path now: built-in defaults, mesh-local config, command override precedence, strict singleton/policy validation, and fail-closed handling. Leave the rest of the config-resolution ontology as a forward contract.
- Include a small ontology review pass before wiring runtime behavior. Clarify or rename `sfcfg:hasHistoryTrackingDefault` because it is too easy to confuse with `sfcfg:hasDefaultHistoryTrackingPolicy`; candidate replacement names include `sfcfg:hasArtifactRoleHistoryTrackingDefault` or `sfcfg:hasArtifactRoleHistoryTrackingPolicy`. Also clarify whether direct `sfcfg:hasHistoryTrackingPolicy` on a `Config` is supported; if not, parser validation should reject it until a precise artifact-level policy shape exists.

## Decisions

- Keep Weave's built-in defaults conservative; do not globally change `defaults/application.ttl` to history-on-everywhere.
- For SFLO, prefer mesh-local durable policy over repeating `--history-tracking-policy versioned` in every command.
- Make "history on everywhere" explicit through merge semantics: a mesh-local default history tracking policy resets lower-layer role defaults for that policy family.
- Do not write explicit all-role `sfcfg:hasHistoryTrackingDefault` entries for `weave mesh create --history-tracking-policy versioned`.
- Keep command overrides last in precedence. A command flag should win over mesh config because it is explicit operator intent for that run.
- Treat unsupported mesh config as an error. Silent fallback would make publication replays hard to trust.
- Keep `publicationProfile` persistence separate from history/page-generation/presentation policy even though all live in `_mesh/_config/config.ttl`.
- Do not block the SFLO manual replay on this task; the current recipe can use command overrides until mesh config is honored.
- Do the universal ResourcePage layout work through mesh config before adding Alice Bio fixture rungs that rely on the references panel or all ordinary generated panels being selected mesh-wide.
- Prefer built-in ResourcePage presentation profile variants over boolean mesh config terms for durable page-layout choices such as Semantic Flow metadata inclusion.

## Contract Changes

- Add or expose an execution-config entry point that accepts `meshRoot` and loads the current mesh config before command overrides.
- Merge mesh-local config over Weave defaults for the policy families already parsed from `ApplicationConfig`/`MeshConfig`: history tracking, ResourcePage generation, ResourcePage presentation, ResourcePage regeneration config policy, and naming policies. For history tracking and ResourcePage generation, an upper-layer default resets lower-layer role defaults for that same policy family; same-layer role defaults remain more specific than that layer's default.
- Extend ResourcePage presentation resolution so a mesh can select a supported default profile that changes panel selections for every generated ResourcePage. The first target profile should include all ordinary data-gated panels, including references, and should have a publication variant that includes Semantic Flow metadata without forcing empty panels to render.
- Extend `weave mesh create` and `planMeshCreate` with `--history-tracking-policy <policy>` once mesh-local history config is honored at runtime. Persist it as `sfcfg:hasDefaultHistoryTrackingPolicy`.
- Keep `--history-tracking-policy` on `weave`, `weave version`, and `weave generate` as an all-role per-run override.
- Add a durable config path for including the Semantic Flow metadata panel by default through a supported presentation profile variant.
- Update CLI docs so `mesh create`, `weave`, and `generate` explain which options persist mesh policy and which options are per-run overrides.
- Update the SFLO replay recipe in [[wu.cli-reference.examples.sflo]] once mesh-local policy can replace repeated command flags.

## Testing

- Unit-test the chosen "history on everywhere" representation and verify all artifact roles resolve to `versioned`.
- Unit-test a mesh-local config that sets only `sfcfg:hasDefaultHistoryTrackingPolicy sfcfg:historyTrackingPolicy_versioned` and verify it resets lower-layer role defaults.
- Unit-test per-role mesh-local history overrides through `sfcfg:hasHistoryTrackingDefault` as same-layer exceptions to the mesh baseline.
- Unit-test that direct `sfcfg:hasHistoryTrackingPolicy` on `sfcfg:MeshConfig` is either rejected or explicitly mapped only after the ontology review defines that shape.
- Unit-test mesh-local ResourcePage generation policy overrides through `sfcfg:hasDefaultResourcePageGenerationPolicy` and `sfcfg:hasResourcePageGenerationDefault`.
- Unit-test mesh-local ResourcePage presentation/profile selection, including a profile that selects the references panel and all ordinary data-gated generated panels while still omitting panels with missing data.
- Integration-test a mesh whose `_mesh/_config/config.ttl` selects the all-panels ResourcePage presentation and verify `weave`/`weave generate` apply it across generated identifier, Knop, simple, and ReferenceCatalog pages as appropriate.
- Unit-test mesh-local naming-policy overrides for history, state, and manifestation naming.
- Unit-test unsupported mesh-local policy values and duplicate singleton values fail closed.
- Integration-test `weave mesh create --history-tracking-policy versioned` writes the expected `_mesh/_config/config.ttl` with `sfcfg:hasDefaultHistoryTrackingPolicy` and no generated all-role `sfcfg:hasHistoryTrackingDefault` block.
- Integration-test a generated mesh with history-on-everywhere creates historical support states without requiring command-scoped `--history-tracking-policy`.
- Integration-test top-level `weave` and `weave generate` both honor the mesh's Semantic Flow metadata panel preference once that config hook exists.
- Regression-test that command-scoped `--history-tracking-policy currentOnly` can still override a mesh-local versioned default for a single run.
- Update fixture ladder coverage only after the core mesh-config behavior is stable enough that fixture commands can stop carrying redundant overrides.

## Non-Goals

- Do not redesign the entire config ontology.
- Do not add an all-artifact-role selector term in this slice.
- Do not change Weave's global built-in defaults to history-on-everywhere.
- Do not make `weave mesh create` an interactive policy wizard in this slice.
- Do not implement remote config retrieval or unpinned external config references.
- Do not make `publicationProfile` imply history or ResourcePage presentation behavior.
- Do not require the first SFLO `/tmp/sflo` manual replay to wait for this task.

## Implementation Plan

- [ ] Audit current effective-config loading and identify every command path still calling built-in defaults directly.
- [ ] Review and clarify the history-policy vocabulary before runtime wiring, especially `sfcfg:hasHistoryTrackingDefault` versus `sfcfg:hasDefaultHistoryTrackingPolicy`.
- [ ] Add a mesh-root-aware effective-config loader that reads `_mesh/_config/config.ttl` when present.
- [ ] Parse mesh-local policy terms by reusing or generalizing the existing `ApplicationConfig` parser where the shapes are shared with `MeshConfig`, including policy-family reset semantics for upper-layer defaults.
- [ ] Add ResourcePage presentation/profile support for mesh-local universal layout policy, including built-in all-ordinary-panels and publication profiles that select references but remain data-gated.
- [ ] Preserve command override precedence and update audit logging to report command overrides separately from mesh-local config.
- [ ] Add `MeshConfig` tests for default history tracking policy, per-role history policy, ResourcePage generation policy, presentation config, naming policies, unsupported values, and duplicate values.
- [ ] Add `weave mesh create --history-tracking-policy <policy>` once the persisted mesh policy is honored at runtime.
- [ ] Implement the durable ResourcePage presentation profile variant for Semantic Flow metadata panel inclusion.
- [ ] Implement the Semantic Flow metadata panel config hook for both top-level `weave` generation and explicit `weave generate`.
- [ ] Update [[wu.cli-reference.mesh.create]], [[wu.cli-reference.weave]], [[wu.cli-reference.generate]], and [[wu.cli-reference.examples.sflo]].
- [ ] Run focused integration tests for mesh config plus `deno task check`; run `deno task ci` before merge.
