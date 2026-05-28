---
id: uq31lflqb8xz50ap4hdvg41
title: 2026 05 27 2031 per Target Effective Config Resolution
desc: ''
updated: 1779939131981
created: 1779939131981
---

## Goals

- Broaden Knop-local and inherited config-source behavior beyond the safe single-target slice from [[wa.completed.2026.2026-05-27_1914-knop-config-source-discovery-and-inheritance]].
- Introduce a runtime effective-config provider/cache that can answer "what config applies to this designator path?" without pretending one `EffectiveConfig` object represents every target in a recursive or multi-target operation.
- Keep application defaults, mesh config, mesh-local config sources, and command/runtime overrides loaded once per operation where possible, while resolving Knop-local and inherited config per target scope.
- Make `version`, `generate`, and recursive `weave` planning/rendering ask for target-scoped config at the point they plan or render a target.
- Preserve fail-closed local path, digest, and unsupported remote-source behavior from mesh-local and Knop-local config-source resolution.
- Keep command overrides global and highest-priority across every target-specific effective config.
- Avoid broad config ontology churn; this is a runtime composition/cache task, not a new config vocabulary task.

## Summary

Per-target effective config resolution has not landed yet. What landed was the safe first slice: if `version` or `generate` resolves to exactly one target, Weave loads that target's Knop ancestor metadata and applies Knop-local and inherited config sources. If the operation has multiple targets, no explicit target, whole-mesh generation, or recursive target expansion, Weave currently falls back to a single effective config without Knop-scoped inputs.

That limitation is intentional. A single `EffectiveConfig` cannot honestly represent different inherited Knop config for `alice/data` and `bob/data` at the same time. The next implementation should therefore add a provider/cache keyed by target designator path or normalized Knop scope path. Planning and rendering should request config for the specific target being processed, while mesh-scoped support work continues to use mesh/global config.

The likely runtime shape is:

- load application defaults, mesh metadata, mesh inventory, mesh config, mesh-local config sources, local path policy, and command overrides once for the operation
- build an `EffectiveConfigProvider` that can resolve Knop ancestor scope metadata for a designator path
- cache Knop scope paths and compiled effective configs by scope path so whole-mesh operations do not repeatedly parse and compile the same inherited config stack
- make `version` planning compute policies for each candidate target immediately before calling the core planner for that target
- make page-generation model assembly/rendering request config per generated page target instead of passing one global config through the entire run
- keep command overrides applied globally and above every mesh/Knop config layer

## Discussion

### Current Runtime Limitation

`src/runtime/weave/page_generation.ts` and `src/runtime/weave/version_execution.ts` both carry the guardrail comment already: Knop config is target-scoped, so multi-target generation/versioning needs per-target effective config before single-target behavior can be safely broadened. The current code loads `knopConfigScopePath` only when there is exactly one selected/generated target or exactly one initial weaveable candidate.

This protects correctness, but it leaves useful behavior unavailable:

- `weave generate alice bob` cannot let Alice and Bob inherit different ResourcePage presentation or generation policy.
- `weave generate` across the whole mesh cannot apply each Knop's inherited page-generation config to its own page.
- recursive `weave version` planning cannot safely apply a different target config as it walks multiple candidates.
- command overrides still work globally, but they sit on top of a config stack that may omit target-scoped sources for multi-target runs.

### Provider Boundary

The provider should be runtime-side, probably near `src/runtime/weave/execution_config.ts`, because it needs local workspace reads, mesh state, local path policy, and command overrides. Core planners should not learn how to discover config sources; they should receive either already-derived policy objects for the current target or a small callback only if a deeper planner really needs target-specific decisions during iteration.

A first useful API could look roughly like:

```ts
const provider = await createEffectiveConfigProviderForExecution({
  meshRoot,
  meshState,
  localPathPolicy,
  historyTrackingPolicyOverride,
  includeSemanticFlowMetadata,
});

const effectiveConfig = await provider.configForTarget(designatorPath);
const meshEffectiveConfig = await provider.configForMeshScope();
```

The provider should expose full `EffectiveConfig` objects. Runtime call sites can still derive smaller policy bundles with existing helpers, but the provider boundary should not guess which parts of config a future caller may need.

Names are not important yet. The important boundary is that callers stop building one `EffectiveConfig` up front and instead ask the provider for the scope they are about to plan or render.

### Config Scope, Layer, And Policy Target

Keep config scope, config layer, and policy target distinct while wiring the provider.

- Config scope answers "from which mesh or Knop viewpoint is this effective config resolved?"
- Config layer answers "where did this policy input sit in precedence: application default, mesh-local, inherited Knop, Knop-local, or command override?"
- Policy target answers "what artifact or artifact role does this policy apply to?"

Mesh config is a global baseline layer visible to all Knop scopes, but it should not steer `_mesh` versus Knop artifacts except through normal policy targeting. Knop-local and inherited Knop config are target-scoped inputs, but an `EffectiveConfig` for a target can still answer broad role questions. Runtime call sites therefore need scope-correct policy derivation, not just "use the candidate target config for every question."

For planner inputs:

- mesh-owned roles such as `meshInventory`, `meshMetadata`, and `config` should be derived from `provider.configForMeshScope()`
- Knop-owned roles such as `knopInventory`, `knopMetadata`, `referenceCatalog`, and `resourcePageDefinition` should be derived from `provider.configForTarget(targetPath)`
- payload and target-owned ResourcePage generation decisions should use the relevant target scope
- ResourcePage rendering should use target config for target pages and mesh/global config for mesh support pages
- command/runtime overrides remain a global top layer in every provider result

Descendant Knop config must not affect ancestor Knop config. Knop-local exact-artifact policy targets should be constrained to artifacts inside the Knop's authority scope; mesh config can target mesh-governed artifacts broadly. Mesh config can turn off `_mesh` support history without turning off all Knop histories by targeting mesh-owned artifact roles rather than `AnyGovernedArtifactPolicyTarget`.

### Scope Keys

Use explicit provider/cache scope keys rather than overloading empty strings until nobody can tell whether a value means root Knop, mesh, or "not provided."

- root Knop provider/cache key: `/`
- next-level Knop scope keys: normal designator paths without a leading slash, such as `alice` or `alice/bio`
- mesh/global effective config key: a special internal value such as `MESH_CONFIG`

This should not require changing the lower-level normalized designator path representation in the same slice. The current ontology/comment and Weave runtime treat the empty string as the root designator path, while `/` is the CLI/display sentinel for that root. Existing helpers such as `toKnopPath("")` and Knop config-source discovery expect that shape. The provider can expose or cache the root as `/`, then normalize to `""` when it calls lower-level designator-path and `KnopConfigScopeInput.scopeKey` helpers.

The exact mesh/global constant name can be chosen during implementation. The important part is that root Knop config remains special and visible, while mesh/global config is an entirely separate case.

### Snapshot Semantics

Use a stable operation-start view of config authority unless there is a deliberate reason to do otherwise. Recursive `version` planning already uses an overlay for planned writes, but effective config should not accidentally change halfway through the command because an earlier planned target updates a config file or metadata file. That would make results order-dependent.

The provider should read from the starting `MeshState` and current workspace files. If the initial operation-start candidate loader already has current metadata Turtle for a target, the provider may accept a target-specific metadata override for that scope, matching the existing single-target behavior. That override must be tied to the exact scope and should not mutate the cache for unrelated scopes.

Recursive `version` planning reloads candidates through a staged overlay after planned writes have been applied. Those later staged candidate metadata reads must not feed config resolution, or effective config can become order-dependent. Seed provider target metadata overrides from the initial, operation-start candidate set only. If a later staged candidate has different metadata because an earlier planned write changed it, planning may use that staged metadata for weave shape decisions, but config resolution should still use the operation-start snapshot.

If we later want "config changes take effect during the same operation," that should be a separate explicit task with tests for ordering and invalidation.

### Cache Shape

The cache should avoid recompiling for every page in whole-mesh generation, but it should stay simple:

- cache ancestor scope metadata reads by scope key
- cache resolved Knop config-source inputs by normalized scope path
- cache compiled effective config by normalized target scope path plus the operation-wide command override signature
- keep mesh/global config source resolution separate from Knop-scope config resolution so mesh-local sources are not re-resolved per target

Do not over-optimize the first implementation. A clear provider with measurable timing fields is better than a complicated shared cache that hides stale reads.

### Version Planning

`prepareVersionExecution` already calls `planVersion` once per staged candidate during recursive planning. That is the right place to compose planner policy inputs from the correct scopes: mesh/global effective config for mesh-owned support artifacts, target effective config for payload and Knop-owned artifacts, and target or mesh config for ResourcePage work depending on which page or page fact is being planned.

The mesh-support-only branch, where there are no weaveable Knop candidates and no explicit targets, should use mesh/global effective config rather than an arbitrary Knop target config.

Be careful with support artifacts whose ownership is mesh-scoped rather than Knop-scoped. Target Knop config should affect target-owned payload/support behavior, not silently turn one Knop's local preference into mesh-wide inventory or metadata policy unless the config model already authorizes that target scope.

### Policy Compatibility

Versioning is per-Knop, so sibling targets should generally not have "incompatible" policies merely because their effective configs differ. Alice can require payload history while Bob stays current-only; each target's planner should apply its own effective config to its own payload and Knop-owned support files.

The real failure modes are narrower:

- a single target's effective config has unresolved same-layer conflicts or invalid values
- two planned targets produce the same concrete file path with different contents or incompatible create/update intent
- a target-scoped config attempts to control artifacts outside its authority scope, such as a descendant Knop config targeting an ancestor Knop artifact or a Knop-local exact-artifact policy targeting a mesh-owned artifact

So the implementation should not add a preflight "all target configs must be compatible" rule. Let the existing per-target compiler fail closed for intra-target config conflicts, and let planning fail closed on concrete file conflicts.

### Page Generation

`generatePreparedPages` currently resolves all selected designator paths, loads one config, collects all page models, and renders all pages. That should become per-page or per-target:

- whole-mesh generation should ask for each generated designator path's effective config
- explicit multi-target generation should do the same for each selected target
- root/mesh support pages need an explicit mesh/global config path rather than borrowing the first resource target
- page-path candidate selection should not prefilter every `hasResourcePage` fact through one global config before owner-scoped policy can run
- ResourcePage model assembly should receive the config for the model it is assembling, or receive a provider if it naturally walks targets internally
- rendering must also preserve per-page presentation config instead of flattening back to one render options object for the whole batch

Whole-mesh page selection is a hidden hotspot. `collectResourcePageModels` currently lists runtime-generated ResourcePage paths from the mesh inventory with one config before narrowing to selected/generated resource paths. With per-target ResourcePage generation policy, that list may differ by owner. The implementation likely needs to collect candidate page facts broadly, identify each page subject's owning designator/resource and artifact role, then filter using the effective config for that owner scope. Mesh support page facts should use mesh/global config.

Today `collectGeneratedPageFiles` passes one `effectiveConfig.resourcePagePresentation` into `renderResourcePages`, and `renderResourcePages` applies one options object to every page. Provider threading into model assembly is therefore not enough by itself. Either generated page models/render inputs must carry their selected presentation config, or the renderer must accept an `optionsForPage` / config-provider callback so each page can be rendered with the correct target or mesh scope.

Best-effort child context loading needs an explicit split. Selected/generated page targets and page-existence decisions must fail closed on config resolution errors, because suppress/generate/presentation policy affects the operation result. Best-effort-only child contexts, such as child type hints used to decorate an already selected parent page, may continue to skip optional context on ordinary best-effort read/path/runtime failures. If provider lookup for such a non-selected child fails because that child's config is malformed, the first implementation should prefer failing closed unless the code can prove the child config would only affect optional decoration and not page selection or rendering policy. This should be a deliberate call site decision rather than a blanket catch around provider errors.

This is the place where the user-visible effect will be easiest to prove: two sibling Knops can inherit different page presentation or page generation policy and `weave generate` should render each page accordingly, including final rendered panel selection/chrome rather than only model collection.

### Relationship To Later Resolver Work

This task should land before the next broader resolver cleanup consumer. Page-source exact/fallback semantics from [[wa.task.2026.2026-04-08_1545-resource-page-definition-and-sources]] are probably the best follow-on if we stay in source/resolver land, because page-source resolution has visible output and duplicated working/latest-state logic. If we shift to correctness debt instead, [[wa.task.2026.2026-05-17-append-onlyish-inventory]] is the stronger next slice. If the goal is reducing conceptual confusion, finish the ReferenceLink/source terminology work in [[wa.task.2026.2026-05-22_1128-referencelink-clarification]].

## Open Issues

- Should target metadata overrides from candidate loading be provider inputs up front, or passed on individual `configForTarget` calls? Recommendation: keep the first implementation simple and allow per-call overrides only where existing single-target code already needs them.
- Should the provider expose detailed config traces through normal timing fields, a debug logger, or only the returned `EffectiveConfig` trace? Recommendation: use existing `WEAVE_TIMING=1` output for aggregate provider/cache timing and keep detailed source traces on the config objects or debug-only surfaces.
- Should future chained config artifacts use `sfcfg:hasConfig` links, `sfcfg:hasConfigSource` links, or both? Recommendation: leave this outside the current task; metadata-inline `hasConfig` is dead for now, but config-artifact-to-config-artifact chaining is still desirable later.

## Decisions

- Per-target effective config is the correct next task before broadening Knop config-source inheritance to multi-target and recursive operations.
- Do not use one preloaded `EffectiveConfig` as the operation-wide truth when selected targets may have different Knop ancestry.
- The provider should return full `EffectiveConfig` values; callers may derive smaller policy bundles locally.
- Use `/` as the provider/cache key for the root Knop, normal slash-separated designator paths without a leading slash for descendant Knops, and a special internal mesh/global key such as `MESH_CONFIG` for mesh effective config. Normalize `/` back to the existing empty-string root designator path when calling lower-level `KnopConfigScopeInput` and designator-path helpers unless the implementation deliberately updates those lower layers and tests in the same slice.
- Command/runtime overrides remain global and highest-priority for every target.
- Mesh-local config remains operation-wide; Knop-local and inherited config are target-scoped.
- Effective config should be stable for the duration of an operation. Planned writes should not change later targets' effective config unless a later task explicitly changes that contract.
- Whole-mesh and multi-target operations should compute target-specific config lazily and cache results by normalized scope path.
- Mesh-scoped support work should use mesh/global config, not whichever target happens to be planned first.
- Runtime policy derivation must be scope-correct: mesh-owned roles come from mesh/global effective config, Knop-owned roles come from the relevant target effective config, and payload/page decisions use the relevant target or mesh scope for the artifact being planned or rendered.
- Descendant Knop config must not affect ancestor Knop config.
- Knop-local exact artifact targets should be constrained to artifacts inside the Knop's authority scope; mesh config can target mesh-governed artifacts broadly.
- Mesh config can turn off `_mesh` support history without turning off all Knop histories by targeting mesh-owned artifact roles instead of `AnyGovernedArtifactPolicyTarget`.
- Use existing `WEAVE_TIMING=1` runtime timing output for provider/cache timing counters.
- Do not support metadata-inline `sfcfg:hasConfig` / `sfcfg:hasInheritableConfig` in this task.
- Do not require sibling target policies to be mutually compatible. Versioning is per-Knop; fail closed only on per-target config errors or concrete planned-file conflicts.

## Contract Changes

- `version`, `generate`, and recursive `weave` operations can apply Knop-local and inherited config to each resolved target, not only when the operation has exactly one target.
- Runtime effective-config loading gains a provider/cache abstraction keyed by designator/scope path.
- Runtime planning/rendering call sites that currently receive one `EffectiveConfig` for an entire operation should either request config per target or receive pre-derived per-target policy bundles.
- Config resolution trace data should remain able to explain which mesh/Knop source contributed to a target's effective config.
- `WEAVE_TIMING=1` should include aggregate provider/cache timing counters when this provider is used.
- Command overrides should continue to beat application defaults, mesh config, mesh-local config sources, Knop-local config, and inherited Knop config for every target.
- No Semantic Flow Framework behavior-spec change is required unless this exposes new portable CLI behavior that should be specified black-box. This is primarily Weave runtime correctness.

## Testing

- Unit-test the provider with two sibling Knops whose inherited/local config sources set conflicting policies; `configForTarget("alice")` and `configForTarget("bob")` must differ correctly.
- Unit-test cache behavior enough to prove repeated requests for the same scope reuse resolved/compiled config and requests for different scopes do not leak config across targets.
- Unit-test scope-key handling so provider/cache `/`, lower-level root `""`, descendant designator paths, and the mesh/global config key do not collapse into one another.
- Unit-test command overrides across two target scopes to prove the override beats both local and inherited config.
- Integration-test `weave generate` or `generatePreparedPages` for two selected targets with different inherited ResourcePage presentation/generation settings.
- Integration-test whole-mesh generation where different Knop pages render from different inherited config while mesh support pages use mesh/global config.
- Assert that whole-mesh page selection considers page owners' effective configs rather than filtering all `hasResourcePage` facts through one global config.
- Assert that final rendered HTML reflects per-page presentation config, not only that page models were assembled under different target configs.
- Regression-test that config resolution failures for selected/generated page targets fail closed.
- Decide and test the best-effort child-hint behavior separately: optional child hint read/path failures may be skipped, but provider/config failures should fail closed unless the implementation proves the failed config cannot affect page selection or rendering policy.
- Integration-test `weave version` with multiple weaveable candidates whose support history or ResourcePage generation policies differ by target config.
- Regression-test that a Knop-local or inherited target config cannot change mesh-owned support history decisions such as `_mesh/_inventory` history materialization during a multi-target operation.
- Regression-test that mesh config can target mesh-owned artifact roles without suppressing Knop-owned histories.
- Failure-test a Knop-local exact-artifact policy target outside that Knop's authority scope, verifying it fails closed with useful scope context.
- Regression-test that sibling targets with different valid per-Knop policies do not fail a compatibility preflight merely because the policies differ.
- Regression-test recursive version planning where an earlier planned write changes metadata/config attachments, proving later targets' effective config still uses the operation-start snapshot rather than staged overlay metadata.
- Regression-test the existing single-target inherited Knop config behavior so this refactor does not break the landed safe slice.
- Add at least one failure-path test for unsafe or invalid Knop config sources reached through a multi-target operation, verifying the operation fails closed with target/scope context in the diagnostic.
- Run focused runtime/config tests first, then the normal Weave check/lint workflow for touched runtime modules.

## Non-Goals

- Do not introduce a new config vocabulary.
- Do not implement metadata-inline `sfcfg:hasConfig` / `sfcfg:hasInheritableConfig`.
- Do not implement future config-artifact-to-config-artifact chaining, even though chained config artifacts remain desirable later.
- Do not add a cross-target policy compatibility solver.
- Do not make portable mesh config grant broader local path or remote URL trust.
- Do not implement remote config-source fetching.
- Do not make planned writes update effective config mid-operation.
- Do not introduce Oxigraph or a durable config graph store for this slice.
- Do not solve historical "effective config at state/time" behavior.
- Do not broaden page-source exact/fallback semantics here; use [[wa.task.2026.2026-04-08_1545-resource-page-definition-and-sources]] for that follow-on.

## Implementation Plan

- [x] Re-read [[wd.general-guidance]], [[wa.completed.2026.2026-05-27_1246-config-source-discovery-and-resolution]], [[wa.completed.2026.2026-05-27_1914-knop-config-source-discovery-and-inheritance]], and the current runtime comments in `src/runtime/weave/page_generation.ts` and `src/runtime/weave/version_execution.ts`.
- [x] Inventory every runtime call path that loads `EffectiveConfig` once and then processes multiple targets.
- [x] Add an `EffectiveConfigProvider` or equivalent helper in the runtime config/execution layer, with operation-wide mesh/default/source inputs and per-target Knop scope lookup.
- [x] Separate mesh/global effective config from target-scoped effective config in the provider API.
- [x] Use `/` for the provider/cache root Knop key, descendant designator paths without a leading slash for ordinary Knops, and a special mesh/global key such as `MESH_CONFIG`; normalize provider/cache `/` to the existing lower-level `""` root designator path when calling `KnopConfigScopeInput` and designator-path helpers.
- [x] Cache ancestor metadata reads, resolved Knop config inputs, and compiled effective config by normalized scope path.
- [x] Preserve command override handling as operation-wide input applied above every provider result.
- [x] Seed target metadata overrides from the initial operation-start candidate set only; do not pass staged loop candidate metadata from the recursive version overlay into provider config resolution.
- [x] Update `prepareVersionExecution` so each candidate planning pass composes planner policy inputs from the correct scopes: mesh/global effective config for mesh-owned support artifacts, target effective config for payload and Knop-owned artifacts, and target or mesh config for ResourcePage work depending on which page or page fact is being planned.
- [x] Keep mesh-support-only version planning on mesh/global config.
- [x] Refactor whole-mesh page selection so candidate `hasResourcePage` facts are collected broadly, mapped to an owning designator/resource and artifact role, then filtered with the effective config for that owner scope.
- [x] Update `generatePreparedPages` / page model assembly so generated page models are assembled with the config for their own designator path, while mesh support pages use mesh/global config.
- [x] Update ResourcePage rendering so each page receives its own presentation config, either by carrying presentation config on page render inputs or by adding an `optionsForPage` / provider callback to the render batch.
- [x] Audit `loadBestEffortGenerateDesignatorContexts` so selected/generated targets fail closed on config errors while optional child-hint context skips only genuinely optional read/path/runtime failures.
- [x] Add `WEAVE_TIMING=1` counters for provider cache hits/misses and per-target config resolution phases.
- [x] Add unit tests for provider scoping, command override precedence, cache isolation, and unsafe source failure diagnostics.
- [x] Add integration coverage for multi-target `generate` and multi-candidate `version` using different inherited Knop config sources.
- [x] Run focused tests, then `deno task check` and `deno task lint` for the Weave repo.
- [x] Update [[wd.todo]] and any user/developer docs only if visible behavior or authoring guidance changes.
- [x] Provide a detailed semantic-commit-style commit message for the Weave repo.
