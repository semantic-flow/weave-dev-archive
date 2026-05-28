---
id: weave-per-target-effecti-gldhjq
title: 'weave: per-target-effective-config-resolution'
desc: ''
created: 1779943848051
updated: 1779943848051
participants: [codex.gpt-5.5]
conversationEventKinds: [message.assistant, message.user, tool.call, tool.result]
---

# User_2026-05-27_2059_47

weave: per-target-effective-config-resolution

We are going to implement [[wa.task.2026.2026-05-27-2031-per-target-effective-config-resolution]] . Please review that an let me know if you have any feedback or questions.

# gpt-5.5_2026-05-27_2059_56

I’ll review the task note alongside the repo guidance first, then I’ll spot-check the current config code so the feedback is grounded in what the system already does.

The task note is pretty thorough and already has the key architectural shape. I’m going to check the two prior config-source task notes and the current runtime sites now, because the main risk is usually hidden in an existing call path rather than in the stated plan.

One thing I’ve already noticed: generation currently passes one `resourcePagePresentation` into the render step after collecting all models. That makes the page-generation part a bit trickier than just threading a provider into model assembly; rendering may need per-model presentation or grouping.

I’ve got enough to give useful feedback, but I’m pulling a few line-numbered references so the recommendations are easy to act on in the task note.

**Overall**

The task is solid and worth doing next. The provider/cache boundary is the right move, and the note correctly avoids pretending a single `EffectiveConfig` can cover recursive or multi-target runs. I do have a few pieces of feedback before implementation.

**Feedback**

1. Tighten the mesh-scoped versus target-scoped policy split.

The task discusses this, but the implementation checklist still says to derive support history, naming, ResourcePage generation, and presentation config from the candidate’s target config. I’d sharpen that.

Right now `supportHistoryPoliciesFromEffectiveConfig` includes mesh roles like `meshMetadata`, `meshInventory`, and `config` alongside Knop roles in [execution_config.ts](../../../../../src/runtime/weave/execution_config.ts:107). `prepareVersionExecution` then passes that one bundle into every `planVersion` call in [version_execution.ts](../../../../../src/runtime/weave/version_execution.ts:250).

If we simply swap in the target’s config there, Alice’s Knop-local config could accidentally control `_mesh/_inventory` or `_mesh/_meta` behavior. I’d explicitly combine:

- mesh/global config for mesh-owned support artifacts
- target config for payload and Knop-owned support artifacts
- target config for target-owned page presentation/generation

2. The root scope key change needs a migration choice.

The task says use `/` for the root Knop scope. Current runtime uses `""` as the root scope key in [execution_config.ts](../../../../../src/runtime/weave/execution_config.ts:81), and `discoverKnopConfigSources` expects those scope keys when deriving Knop paths/subjects in [config_sources.ts](../../../../../src/runtime/config/config_sources.ts:242).

Question: do you want `/` to become the internal `KnopConfigScopeInput.scopeKey`, or only the provider/cache key? My recommendation: use `/` at the provider/cache API boundary, but normalize to the existing `""` shape when calling lower-level `config_sources.ts`, unless we deliberately update all existing tests/helpers in the same slice.

3. Page generation needs more than provider threading.

`generatePreparedPages` currently loads one config, passes it to model collection, and then renders every page with one presentation profile in [page_generation.ts](../../../../../src/runtime/weave/page_generation.ts:88) and [page_generation.ts](../../../../../src/runtime/weave/page_generation.ts:185). `renderResourcePages` also receives one options object for all pages in [pages.ts](../../../../../src/runtime/weave/pages.ts:218).

So I’d amend the task to say generated page models or render inputs must carry per-page presentation config, or rendering must call an `optionsForPage`/provider callback. Otherwise model assembly may become target-scoped but rendering will still flatten everything back to one presentation.

4. Whole-mesh page path selection is a hidden hotspot.

`collectResourcePageModels` lists all generated page paths from mesh inventory using one config in [page_model_assembly.ts](../../../../../src/runtime/weave/page_model_assembly.ts:115). With per-target ResourcePage generation policy, that list itself may differ by target. The implementation probably needs to collect candidate page facts broadly, identify the owning designator/resource, then filter with the effective config for that owner.

5. Preserve snapshot semantics carefully during recursive versioning.

The task’s snapshot rule is good. The implementation should avoid passing staged/overlay metadata from later recursive candidate loads into config resolution, because that could make config order-dependent. Existing single-target behavior passes candidate metadata at [version_execution.ts](../../../../../src/runtime/weave/version_execution.ts:138); for the provider, I’d seed any target metadata overrides from the initial, operation-start candidate set, not from the staged loop after planned writes have been applied.

**Questions**

- Should `/` fully replace `""` in `KnopConfigScopeInput`, or should `/` be provider-only and normalized downward?
- For target-scoped config trying to set mesh-owned roles, should the runtime ignore those target-layer values for mesh support work, or should the compiler/config layer reject them in that context? I lean toward ignoring them for mesh-owned decisions at the runtime derivation boundary, because the same config may validly express broader role policies elsewhere.
- For best-effort child context loading during page generation, should per-child config resolution failures be swallowed like other best-effort context errors, or fail closed? I lean fail closed for selected/generated targets, but best-effort-only child hints may be acceptable to skip.

No file changes made.

# User_2026-05-27_2109_52

I'm having trouble understanding #1. "If we simply swap in the target’s config there"... doesn't resolution take care of things? How would Alice’s Knop-local config accidentally control _mesh/_inventory

# gpt-5.5_2026-05-27_2110_12

Yes, resolution takes care of conflicts **inside one effective config**. The issue is choosing **which effective config to ask** for a mesh-owned decision.

`EffectiveConfig` does not know “Alice owns this decision.” It just answers questions like:

```ts
historyTrackingPolicyForArtifactRole("meshInventory")
```

So if we derive all support policies from Alice’s target-scoped config, this helper will ask Alice’s effective config for mesh roles too:

[execution_config.ts](../../../../../src/runtime/weave/execution_config.ts:107)

That means Alice’s Knop-local or inherited config could include something like:

```turtle
<#binding> sfcfg:appliesToPolicyTarget <#meshInventory> .
<#meshInventory> sfcfg:hasArtifactRole sfcfg:artifactRole_meshInventory .
<#policy> sfcfg:hasHistoryTrackingPolicy sfcfg:historyTrackingPolicy_currentOnly .
```

Then `supportHistoryPolicies.meshInventory` becomes `currentOnly`, and `planVersion` currently passes that bundle into mesh inventory planning:

[version_execution.ts](../../../../../src/runtime/weave/version_execution.ts:250)

So Alice’s target config could affect whether `_mesh/_inventory/_history001/...` is created or suppressed. Resolution did its job: it picked the winning `meshInventory` policy in Alice’s effective config. The bug would be that runtime asked Alice’s config for a mesh-owned artifact decision.

The distinction I’m making is:

- **effective config scope**: mesh/global vs Alice vs Bob
- **policy target**: payload, knopInventory, meshInventory, any governed artifact, exact artifact, etc.

Those are related, but not the same. For mesh-owned support artifacts, runtime should ask mesh/global config. For Alice’s payload and Alice’s Knop-owned support artifacts, runtime should ask Alice’s target config.

# User_2026-05-27_2111_27

Isn't there an EffectiveConfig per object?

# gpt-5.5_2026-05-27_2111_46

Not quite. It’s an `EffectiveConfig` per **config scope**, not per mesh object/artifact.

For this task, the proposed scopes are roughly:

- mesh/global config
- root Knop config
- `alice` Knop config
- `alice/data` Knop config
- `bob/data` Knop config

Each `EffectiveConfig` can still answer many policy questions:

```ts
config.historyTrackingPolicyForArtifactRole("payload")
config.historyTrackingPolicyForArtifactRole("meshInventory")
config.resourcePageGenerationPolicyForArtifactRole("knopInventory")
```

So an Alice-scoped `EffectiveConfig` is not “only policies for Alice’s object.” It is “the config stack visible from Alice’s scope.” If Alice’s Knop-local config contains a broad or role-targeted policy for `meshInventory`, that policy is visible when you ask Alice’s config about `meshInventory`.

That’s why the caller still has to choose the right scope for the decision:

- `_mesh/_inventory` decision: ask mesh/global config
- `alice/data` payload decision: ask `alice/data` config
- `alice/data/_knop/_inventory` decision: ask `alice/data` config

The accidental-control case happens if runtime does:

```ts
const aliceConfig = await provider.configForTarget("alice/data");
const policies = supportHistoryPoliciesFromEffectiveConfig(aliceConfig);
```

because that helper derives **both** mesh-owned and Knop-owned support policies from the same config. The fix is probably to split or combine policy derivation so mesh-owned roles come from mesh config and target-owned roles come from target config.

# User_2026-05-27_2134_57

I ran your concern by the PM. It says "he task should distinguish config scope, config layer, and policy target more explicitly.

Mesh config is a global baseline layer visible to all Knop scopes, but it should not steer _mesh versus Knop artifacts except through normal policy targeting. The actual runtime guardrail is scope-correct policy derivation:

mesh-owned roles (meshInventory, meshMetadata, config) must be derived from provider.configForMeshScope()
Knop-owned roles (knopInventory, knopMetadata, referenceCatalog, resourcePageDefinition) must be derived from provider.configForTarget(targetPath)
payload/page decisions use the relevant target scope
CLI overrides remain global top layer
So the implementation checklist should not say “derive support history, naming, ResourcePage generation, and presentation config from the candidate’s target config.” It should say something like:

Compose planner policy inputs from the correct scopes: mesh/global effective config for mesh-owned support artifacts, target effective config for payload and Knop-owned artifacts, and target or mesh config for ResourcePage rendering depending on which page is being rendered.

Also worth adding:

Descendant Knop config must not affect ancestor Knop config.
Knop-local exact artifact targets should be constrained to the Knop’s authority scope; mesh config can target mesh-governed artifacts broadly.
Mesh config can turn off _mesh support history without turning off all Knop histories by targeting mesh-owned artifact roles rather than AnyGovernedArtifactPolicyTarget."

# gpt-5.5_2026-05-27_2135_21

That PM response lands exactly where I was trying to point, and it says it more cleanly: scope, layer, and policy target are three separate axes. I’m going to fold that wording into the task note so the implementation checklist does not accidentally encode the risky shortcut.

I’m making this a documentation-only edit in the archive note: adding an explicit “scope/layer/target” section, tightening version-planning wording, and replacing the checklist item that over-implied “everything from candidate target config.”

That wording is now in the task note. I’m doing a quick diff pass to make sure the edits read coherently and didn’t disturb the Dendron/frontmatter bits.

Agreed, that PM framing is the right one. I updated the task note to make that distinction explicit in [the new scope/layer/target section](wa.task.2026.2026-05-27-2031-per-target-effective-config-resolution.md:70), and tightened the version-planning, decisions, testing, and implementation checklist language.

The key implementation note now says planner inputs must be composed from the right scopes: mesh config for mesh-owned roles, target config for payload/Knop-owned roles, and page rendering config based on the page being rendered. I also added the descendant/ancestor guardrail, exact-artifact authority constraint, and mesh-owned-role regression tests.

No tests run, since this was a task-note edit only.

# User_2026-05-27_2149_57

your other feedback:
2. Yes use ""
3. task note updated for per-page presentation config, are you satisfied with it?
4. same, updated the task. does it look ok now?
5. same

I think your questions are answered now too. Are you satisfied? Can you proceed with implementation in this converation, or should I start a new one?

# gpt-5.5_2026-05-27_2150_12

I’ll do a quick pass over the updated note, especially the bits for root scope, per-page presentation, page-path filtering, and snapshot semantics, then I’ll tell you whether I think it’s ready to implement.

Yes, I’m satisfied now. The updated note answers my concerns:

- `""` stays the lower-level root designator path, with `/` only as provider/cache key.
- Per-page presentation config is called out clearly.
- Whole-mesh page selection now names the hidden risk: don’t prefilter all page facts through one global config.
- Snapshot semantics now explicitly say staged recursive metadata must not feed config resolution.
- The best-effort child-context split is clear enough to implement without hand-waving.

No need to start a new conversation. We can proceed here; the context is still coherent, and the task note is now a good implementation guide.

# User_2026-05-27_2150_35

ok proceed

# gpt-5.5_2026-05-27_2150_44

Great, I’ll implement it in this conversation. I’m going to start by mapping the current config loader and page/version call paths in a little more detail, then I’ll make the provider slice first and use tests to keep the behavior honest.

# gpt-5.5_2026-05-27_2151_16

The current loader can already build a scoped config if given a `knopConfigScopePath`, but it does that as a one-shot function. I’m going to wrap that with a runtime provider first, and where practical I’ll reuse the existing compiler/discovery path rather than cloning config logic.