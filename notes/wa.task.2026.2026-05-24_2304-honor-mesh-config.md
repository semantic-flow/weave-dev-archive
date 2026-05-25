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
- Keep command-line options as explicit command overrides with clear precedence over mesh config.
- Keep `weave mesh create` small, but allow it to seed the mesh config needed for common publication profiles.
- Preserve fail-closed config behavior: malformed, unsupported, or contradictory mesh config should stop execution rather than silently falling back to defaults.

## Summary

Weave has a config ontology and a built-in default config, and `weave mesh create` already writes a mesh-owned `_mesh/_config/config.ttl` when needed for `workspaceRootRelativeToMeshRoot` or a persisted publication profile. However, the main execution path still calls `loadEffectiveConfigForExecution`, which loads only the built-in defaults and then applies command-scoped history overrides. That means mesh-local policy terms such as `sfcfg:hasDefaultHistoryTrackingPolicy`, `sfcfg:hasHistoryTrackingDefault`, `sfcfg:hasDefaultResourcePageGenerationPolicy`, `sfcfg:hasDefaultResourcePagePresentationConfig`, and naming-policy terms are not yet honored as durable mesh behavior.

The immediate pressure is the SFLO `gh-pages` replay. We want the SFLO mesh to have history tracking turned on everywhere, and we want generated pages to include the Semantic Flow metadata panel without having to remember a final `weave generate --include-semantic-flow-metadata` command. Those should be mesh policy choices, not fragile runbook details.

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

Config gaps to close:

- Existing-mesh commands should load built-in defaults plus the current mesh config, then apply command overrides. The command override should remain the final layer.
- Mesh config should be parsed with the same strictness as built-in config: unsupported terms, duplicate singleton values, unsupported policy IRIs, cycles, or unsafe config references should be errors.
- The effective config loader needs a path-aware entry point such as `loadEffectiveConfigForExecution({ meshRoot, historyTrackingPolicyOverride, includeSemanticFlowMetadataOverride })` rather than only accepting a history override.
- Mesh-local config should be able to override default history tracking policy and per-role history policies.
- Mesh-local config should be able to override default ResourcePage generation policy and per-role generation policies.
- Mesh-local config should be able to choose the default ResourcePage presentation config when the referenced config is known and supported.
- Mesh-local config should be able to express whether the Semantic Flow metadata panel is included by default. The current ontology may need a small term for this, unless we instead model a presentation variant whose metadata panel no longer has the opt-in data requirement.
- Mesh-local config should be able to override naming defaults for history, state, and manifestation segments where the ontology already supports those policies.
- Mesh-local config should continue to carry publication profile and workspace-root relationship without conflating publication host policy with execution policy.

SFLO desired mesh config:

```ttl
@prefix sfcfg: <https://semantic-flow.github.io/sflo/config/> .

<> a sfcfg:MeshConfig ;
  sfcfg:hasPublicationProfile sfcfg:publicationProfile_githubPages ;
  sfcfg:hasDefaultHistoryTrackingPolicy sfcfg:historyTrackingPolicy_versioned ;
  sfcfg:hasDefaultResourcePageGenerationPolicy sfcfg:resourcePageGenerationPolicy_generate ;
  sfcfg:hasDefaultResourcePagePresentationConfig <https://semantic-flow.github.io/weave/defaults/resource-page-presentation/semantic-site-default> .
```

That may also not be enough by itself for "history on everywhere" if the config merge keeps lower-layer per-role overrides from `defaults/application.ttl`. The built-in defaults currently have role-specific history policies for `payload`, `config`, `meshInventory`, `knopInventory`, and `runtimeMeta`; a simple upper-layer default would leave those role entries in place. The implementation needs to choose one of these semantics:

- Treat a mesh-local default history policy as replacing lower-layer role defaults for that policy family.
- Have `mesh create --history-tracking-policy versioned` write explicit `sfcfg:hasHistoryTrackingDefault` entries for every known artifact role.
- Add a clearer config term for "all artifact roles use this policy" and compile it into effective per-role policy.

The second option is the least surprising for the immediate SFLO replay because it mirrors the current command override behavior, which expands the requested policy across all known artifact roles.

The desired config is also not enough by itself for the Semantic Flow metadata panel until the opt-in behavior has a durable config hook. For the immediate SFLO replay, the runbook can still use `weave generate --include-semantic-flow-metadata`; this task should make that an implementation gap rather than a permanent habit.

One caution: "history on everywhere" will produce more generated support history. That is correct for SFLO because the publication mesh itself is a public artifact we want to inspect. It is not necessarily the right default for small sidecar meshes or quick fixtures, so this should be mesh-local policy, not a global Weave default.

## Open Issues

- Should `weave mesh create` expose a single `--history-tracking-policy versioned` option that persists `sfcfg:hasDefaultHistoryTrackingPolicy`, or should initial policy editing remain a manual config-file step until more policies are implemented?
- If `--history-tracking-policy versioned` is meant to mean "everywhere", should it write explicit per-role history policy entries for all known artifact roles?
- Should `weave mesh create` expose `--include-semantic-flow-metadata`, or should that belong to a more explicit ResourcePage presentation option?
- Do we need a new SFLO config term for "include Semantic Flow metadata by default", or should this be represented by a built-in ResourcePage presentation variant?
- Should mesh-local config be limited to `_mesh/_config/config.ttl` in this slice, or should the full config source vocabulary already allow reusable config attachments?
- Should workspace and machine-local operational config be loaded in the same implementation pass, or should this task focus only on mesh-local config plus command overrides?
- Should command-line `--history-tracking-policy` continue to override all artifact roles, or should it only override the default policy while preserving per-role mesh config?
- How should generated logs and audit records describe the config sources and policy values that were applied?
- How much of the config-resolution ontology should be implemented now versus left as a declared future shape?

## Decisions

- Keep Weave's built-in defaults conservative; do not globally change `defaults/application.ttl` to history-on-everywhere.
- For SFLO, prefer mesh-local durable policy over repeating `--history-tracking-policy versioned` in every command.
- Make "history on everywhere" explicit in the effective config. Do not rely on a default-only setting if lower-layer per-role defaults would survive the merge.
- Keep command overrides last in precedence. A command flag should win over mesh config because it is explicit operator intent for that run.
- Treat unsupported mesh config as an error. Silent fallback would make publication replays hard to trust.
- Keep `publicationProfile` persistence separate from history/page-generation/presentation policy even though all live in `_mesh/_config/config.ttl`.
- Do not block the SFLO manual replay on this task; the current recipe can use command overrides until mesh config is honored.

## Contract Changes

- Add or expose an execution-config entry point that accepts `meshRoot` and loads the current mesh config before command overrides.
- Merge mesh-local config over Weave defaults for the policy families already parsed from `ApplicationConfig`/`MeshConfig`: history tracking, ResourcePage generation, ResourcePage presentation, ResourcePage regeneration config policy, and naming policies.
- Extend `weave mesh create` and `planMeshCreate` only for options we are ready to honor at runtime. Likely first option: `--history-tracking-policy <policy>`, with explicit all-role output if that option is documented as "everywhere".
- Keep `--history-tracking-policy` on `weave`, `weave version`, and `weave generate` as an override. Update behavior/docs if its semantics shift from "all roles" to "default only".
- Add a durable config path for including the Semantic Flow metadata panel by default, either through a new config term or a supported presentation variant.
- Update CLI docs so `mesh create`, `weave`, and `generate` explain which options persist mesh policy and which options are per-run overrides.
- Update the SFLO replay recipe in [[wu.cli-reference.examples.sflo]] once mesh-local policy can replace repeated command flags.

## Testing

- Unit-test the chosen "history on everywhere" representation and verify all artifact roles resolve to `versioned`.
- Unit-test a mesh-local config that sets only `sfcfg:hasDefaultHistoryTrackingPolicy sfcfg:historyTrackingPolicy_versioned` so the intended interaction with lower-layer per-role defaults is locked down.
- Unit-test per-role mesh-local history overrides through `sfcfg:hasHistoryTrackingDefault`.
- Unit-test mesh-local ResourcePage generation policy overrides through `sfcfg:hasDefaultResourcePageGenerationPolicy` and `sfcfg:hasResourcePageGenerationDefault`.
- Unit-test mesh-local naming-policy overrides for history, state, and manifestation naming.
- Unit-test unsupported mesh-local policy values and duplicate singleton values fail closed.
- Integration-test `weave mesh create --history-tracking-policy versioned` writes the expected `_mesh/_config/config.ttl`.
- Integration-test a generated mesh with history-on-everywhere creates historical support states without requiring command-scoped `--history-tracking-policy`.
- Integration-test top-level `weave` and `weave generate` both honor the mesh's Semantic Flow metadata panel preference once that config hook exists.
- Regression-test that command-scoped `--history-tracking-policy currentOnly` can still override a mesh-local versioned default for a single run.
- Update fixture ladder coverage only after the core mesh-config behavior is stable enough that fixture commands can stop carrying redundant overrides.

## Non-Goals

- Do not redesign the entire config ontology.
- Do not change Weave's global built-in defaults to history-on-everywhere.
- Do not make `weave mesh create` an interactive policy wizard in this slice.
- Do not implement remote config retrieval or unpinned external config references.
- Do not make `publicationProfile` imply history or ResourcePage presentation behavior.
- Do not require the first SFLO `/tmp/sflo` manual replay to wait for this task.

## Implementation Plan

- [ ] Audit current effective-config loading and identify every command path still calling built-in defaults directly.
- [ ] Add a mesh-root-aware effective-config loader that reads `_mesh/_config/config.ttl` when present.
- [ ] Parse mesh-local policy terms by reusing or generalizing the existing `ApplicationConfig` parser where the shapes are shared with `MeshConfig`.
- [ ] Preserve command override precedence and update audit logging to report command overrides separately from mesh-local config.
- [ ] Add `MeshConfig` tests for default history tracking policy, per-role history policy, ResourcePage generation policy, presentation config, naming policies, unsupported values, and duplicate values.
- [ ] Add `weave mesh create --history-tracking-policy <policy>` if we decide that option is worth persisting immediately.
- [ ] Decide the durable config representation for Semantic Flow metadata panel inclusion.
- [ ] Implement the Semantic Flow metadata panel config hook for both top-level `weave` generation and explicit `weave generate`.
- [ ] Update [[wu.cli-reference.mesh.create]], [[wu.cli-reference.weave]], [[wu.cli-reference.generate]], and [[wu.cli-reference.examples.sflo]].
- [ ] Run focused integration tests for mesh config plus `deno task check`; run `deno task ci` before merge.
