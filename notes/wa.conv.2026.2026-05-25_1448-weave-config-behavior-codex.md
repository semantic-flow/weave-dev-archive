---
id: weave-config-behavior-bu33nm
title: 'weave config behavior'
desc: ''
created: 1779745706027
updated: 1779745706027
participants: [codex.gpt-5.5]
conversationEventKinds: [message.assistant, message.user, tool.call, tool.result]
---

# User_2026-05-25_1434_31

weave config behavior

[[wa.task.2026.2026-05-24_2304-honor-mesh-config]] is the name of the task, but it suggest broader considerations around how configuration works in the Semantic Flow platform.

Please review the task, let me know if you have any questions, and then let me know if we need to step back first and make sure our approach to config and the config ontology needs a higher-level polish first. I think we might want a spec document, so I created[sf.spec.2026-05-25-config-behavior.md](../../semantic-flow-framework/notes/sf.spec.2026-05-25-config-behavior.md) (not mentioned in the task.)

Maybe once the spec is defined we don't need to step back.

# gpt-5.5_2026-05-25_1434_50

I’ll ground this in the repo guidance first, then read the task and the new spec note together so we can decide whether the implementation task is ready or wants a higher-level config pass.

The task note is doing a lot of useful design work already, but the new spec note is empty, so I’m going to check the current config ontology and a little of the loader surface before giving you a recommendation. That should keep the answer from floating off into pure architecture vapor.

I’m seeing the same implementation gap the task describes: the execution helper currently accepts only a history override and starts from `loadWeaveDefaultEffectiveConfig()`. I’m reading just enough deeper to separate “needs ontology work” from “needs spec plus implementation discipline.”

I reviewed the task, current ontology, and the current Weave config path.

My read: yes, write the spec first, but I would not do a big “step back and redesign config” first. The task already has a good direction. What’s missing is a crisp portable behavior spec that turns the current design notes into invariants: layer precedence, policy binding semantics, fail-closed validation, mesh-local `_mesh/_config/config.ttl`, command overrides, and ResourcePage presentation profile behavior.

A few concrete reasons:

- The desired policy-binding model in the task is not yet in `semantic-flow-config-ontology.ttl`: terms like `sfcfg:PolicyBinding`, `sfcfg:TrackingPolicyDefinition`, `sfcfg:GovernedArtifactPolicyContext`, `sfcfg:ArtifactRolePolicyContext`, and `sfcfg:policyPriority` do not exist yet.
- The current ontology still says `sfcfg:hasHistoryTrackingPolicy` assigns policy to “a config or role policy,” while the task wants to reject direct history policy assignment on `Config`. That should be polished before implementation.
- The code confirms the implementation gap: [execution_config.ts](../../../../../src/runtime/weave/execution_config.ts:30) loads only Weave defaults plus an optional all-role history override. No mesh-root-aware config loading yet.
- The ResourcePage presentation model is close, but currently only knows the default profile and the metadata panel is still driven by command opt-in. So the spec should define supported profile identities like `semantic-site-default`, `semantic-site-all-panels`, and `semantic-site-no-panels` before wiring CLI behavior.

My questions before implementation:

1. Should the new policy-binding vocabulary land in the config ontology as part of this task, or should the spec first define expected behavior using candidate terms and then ontology changes follow?
2. For this slice, is the policy-binding model only for history tracking, or should ResourcePage generation also move away from direct `hasDefaultResourcePageGenerationPolicy` / `hasResourcePageGenerationDefault` semantics?
3. For authored `ResourcePageDefinition`s, should mesh-wide presentation profiles automatically add generated panels, or should authored page-local `hasGeneratedResourcePagePanelSelection` remain the only opt-in for custom pages?
4. The task’s tests mention Knop-local precedence, but the implementation non-goal says only built-in defaults, mesh-local config, and command overrides. I’d defer Knop-local tests unless you want this task to expand.

Recommendation: use `sf.spec.2026-05-25-config-behavior.md` as the stepping-back point. Keep it high-level and portable, then make the task an implementation slice under that spec. I would include sections for config source layers, policy binding and conflict resolution, mesh-local loading, command override precedence, ResourcePage presentation profiles, fail-closed validation, and explicit non-goals. After that, I think we can move straight into ontology/code changes without a broader rethink.

# User_2026-05-25_1453_51

do we really want both GovernedArtifactPolicyContext and ArtifactRolePolicyContext?

What we probably do need is an internal policy API. What do you think?

Answers:

- yes, the binding vocabulary should be established first, or at least co-established with the spec.
- the policy-binding vocabulary should be generalized for history, resourcepage generation, and likely other config concerns
- everything should be uniform: mesh-wide profiles (e.g. presentation) can affect as well as application-level, knop-inheritable, and knop-local level. I'm not aware of any exceptions where something can only be defined at a particular level 
- the "global built-in defaults" language is only saying to preserve the application-level "slim histories" as the default. If you can handle it, knop-local precedence (and knop-inherited precedence) should be in scope. You're going to want an efficient and queryable in-memory data structure to hold config; if it doesn't scale to large meshes on a command-line invocation, we will implement the API soon enough.

I removed "- Do not redesign the entire config ontology." from the non-goals because I am open to that; also removed "- Do not add an all-artifact-role selector term in this slice." because of course we need an all-artifact-role selector.

Thoughts?

# gpt-5.5_2026-05-25_1454_02

I’m going to answer this at the model/API level rather than editing yet. The shape you’re describing changes the center of gravity from “parse some config terms” to “compile layered policy bindings,” which is the right place to be a little fussy.

# gpt-5.5_2026-05-25_1455_05

Yes, I think you’re right about the internal policy API. The implementation should not grow by adding one-off parsing paths for history, then ResourcePage generation, then presentation, then naming. That way lies soup.

My current take: we probably do **not** want both `GovernedArtifactPolicyContext` and `ArtifactRolePolicyContext` as separate conceptual siblings. We want a generalized **policy target/selector** model.

The important distinction is:

- **scope**: where the config binding comes from and how far it applies: application, mesh-local, mesh-inheritable, knop-inherited, knop-local, command override
- **target selector**: what kind of thing inside that scope the policy applies to: any artifact, artifacts with role `payload`, exact artifact IRI, generated ResourcePages of kind `identifier`, etc.
- **policy family/value**: history tracking, ResourcePage generation, ResourcePage presentation profile, naming policy, and later concerns

So `GovernedArtifactPolicyContext` feels like it mixes scope and selector. A mesh-local config binding is already scoped to the mesh. A knop-local config binding is already scoped to that knop. The selector should just say “artifacts” or “artifacts with these roles” or “this exact artifact.”

I would model this more like:

```ttl
<> a sfcfg:MeshConfig ;
  sfcfg:hasPolicyBinding <#history-versioned-for-all-artifacts> .

<#history-versioned-for-all-artifacts> a sfcfg:PolicyBinding ;
  sfcfg:bindsPolicy <#history-versioned-policy> ;
  sfcfg:appliesToPolicyTarget <#all-artifacts> .

<#history-versioned-policy> a sfcfg:PolicyDefinition ;
  sfcfg:hasPolicyFamily sfcfg:policyFamily_historyTracking ;
  sfcfg:hasHistoryTrackingPolicy sfcfg:historyTrackingPolicy_versioned .

<#all-artifacts> a sfcfg:ArtifactPolicyTarget ;
  sfcfg:hasArtifactRoleSelector sfcfg:artifactRoleSelector_any .
```

Then a role exception is the same selector type, just narrower:

```ttl
<#runtime-meta-artifacts> a sfcfg:ArtifactPolicyTarget ;
  sfcfg:hasArtifactRole sfcfg:artifactRole_runtimeMeta .
```

I’d slightly push back on calling it an “all-artifact-role” term. “All” is not an artifact role; it is a selector over roles. I’d rather have something like `sfcfg:artifactRoleSelector_any`, `sfcfg:AnyArtifactRoleSelector`, or an explicit `sfcfg:matchesAnyArtifactRole true`. I also think omitted role should **not** mean all roles, because omitted-data-as-wildcard is exactly the kind of thing that makes fail-closed config hard to reason about.

For precedence, I’d revise the task’s earlier direction a bit. Once we have an explicit “any artifact role” selector, same-layer role-specific exceptions should probably override same-layer any-role baselines by selector specificity without needing `policyPriority "10"` everywhere. Otherwise ordinary config gets noisy fast. My preferred resolution order:

1. Config layer/scope precedence: command override > knop-local > knop-inherited > mesh-local/inherited > application defaults.
2. Selector specificity: exact artifact > artifact role > any artifact.
3. Explicit `policyPriority`, for conflicts at the same selector specificity.
4. Fail closed if two bindings still tie and produce incompatible values.

For the internal API, I’d aim for something like:

```ts
type PolicyFamily =
  | "historyTracking"
  | "resourcePageGeneration"
  | "resourcePagePresentation"
  | "historyNaming"
  | "stateNaming"
  | "manifestationNaming";

interface PolicyBinding {
  id: string;
  family: PolicyFamily;
  value: string;
  selector: PolicySelector;
  layer: ConfigLayerRole;
  scope: ConfigScope;
  priority: number;
  source: ConfigSourceRef;
}

interface PolicyIndex {
  resolve(query: PolicyQuery): EffectivePolicy;
  explain(query: PolicyQuery): PolicyResolutionTrace;
}
```

The hot path should query a compiled in-memory index, not wander RDF graphs repeatedly. For scale, index bindings by family first, then by selector kind/role/page kind/exact IRI. Knop inheritance probably wants a path/prefix-aware scope index so a command can ask “which inherited configs apply to this designator path?” without scanning the whole mesh every time.

So yes: co-establish binding vocabulary with the spec, and make the Weave implementation compile RDF config into a uniform policy engine. I’d treat the old direct defaults like `hasDefaultHistoryTrackingPolicy` / `hasHistoryTrackingDefault` as either application-profile shorthand that compiles into bindings, or as deprecated transitional vocabulary. The long-term center should be policy bindings plus selectors.

# User_2026-05-25_1502_50

I'm not sure "presentation profile" is a complex concern like these others... it might just be "github pages gets a .nojekyll file". It is an example of a mesh-only config knob.

ConfigResolutionConfig is something that is most likely to be defined at application and maybe mesh level, kind of a meta-policy. We don't need to get to far into that right now, just remember that it's around.

There's also ontology terms around <ConfigResolution> that are only designed for in-memory storage, never persisted. We can ignore that side of things for now too if you'd like. But I want to know you're opinion.

# gpt-5.5_2026-05-25_1502_58

I’ll peek at the `ConfigResolution*` vocabulary so I can separate what’s portable/authored from what’s runtime diagnostic. Then I’ll give you the opinionated version.

# gpt-5.5_2026-05-25_1503_22

Yes, this changes my opinion a bit: we should not force every config knob through policy bindings.

I’d split config into two buckets:

**1. Layered policy**
Use the internal policy API for things that can reasonably vary by scope/target and need precedence/selector semantics:

- history tracking
- ResourcePage generation
- ResourcePage presentation/chrome/layout
- naming policy
- maybe access/fetch/cache behavior later, carefully

These want bindings, selectors, layers, conflict handling, and explainability.

**2. Plain scoped settings**
Keep direct config terms for things that are essentially singleton facts about a config scope:

- `sfcfg:hasPublicationProfile sfcfg:publicationProfile_githubPages`
- `sfcfg:workspaceRootRelativeToMeshRoot "../"`
- maybe “this mesh uses this config resolution profile,” if mesh-level narrowing is allowed
- maybe application-level defaults for resolver behavior

These do not need `PolicyBinding` unless they become target-selective or compositional. GitHub Pages creating `.nojekyll` is a perfect example: that is a mesh publication setting, not a general policy family. It should stay boring.

I think I also need to be more precise about “presentation profile.” There are two different things that sound similar:

- **Publication profile**: GitHub Pages, none, etc. Mesh publication surface behavior. `.nojekyll` lives here.
- **ResourcePage presentation config/profile**: generated page layout/chrome/panels/templates. This one can absolutely be application, mesh, inherited, or knop-local.

So I would not put `PublicationProfile` into the policy engine right now. I would put ResourcePage presentation defaults into it, because “all pages in this mesh use no generated panels, except this subtree uses references/history panels” sounds like normal layered config.

For `ConfigResolutionConfig`: agreed. It’s meta-policy and probably special. It can be authored at application level, maybe mesh level, but portable mesh config must not be able to expand trust or loader reach. I’d keep it adjacent to the policy system, not inside it. Resolver settings decide how policy/config is loaded and merged; they are not just another policy family resolved by the thing they configure. Bootstrapping trap, small but real.

For `ResolvedConfig` / `ConfigResolutionRecord`: my opinion is we should keep them out of this slice and out of persisted mesh config. They’re useful vocabulary for diagnostics, audit snapshots, maybe deliberate provenance artifacts later. But the implementation should first have native internal structures, with RDF serialization only when we intentionally materialize an explanation.

So the internal API probably wants two compiled outputs:

```ts
interface CompiledConfig {
  settings: ScopedConfigSettings;
  policies: PolicyIndex;
  resolution: ConfigResolutionRuntimeProfile;
}
```

Where `settings` handles singleton facts like publication profile, `policies` handles layered target-selective behavior, and `resolution` handles bootstrap/meta behavior.

That feels cleaner than making `PolicyBinding` the One Ring. It gives us uniformity where uniformity buys us something, without making `.nojekyll` wear a tiny governance robe.

# User_2026-05-25_1504_21

I agree with your all-artifact role pushback, and yes, omitted doesn't mean any. We dpm

# gpt-5.5_2026-05-25_1505_05

Yep, exactly. I’d make that a hard modeling rule:

- Omitted selector fields do **not** imply wildcard.
- “Any artifact” must be explicit.
- “Any artifact role” is not itself a role; it’s a selector mode or selector node.
- A malformed/incomplete selector should fail validation, not silently broaden.

I think the clean ontology move is something like:

```ttl
<#all-artifacts> a sfcfg:ArtifactPolicyTarget ;
  sfcfg:hasArtifactRoleSelector sfcfg:artifactRoleSelector_any .
```

and role-specific:

```ttl
<#payload-artifacts> a sfcfg:ArtifactPolicyTarget ;
  sfcfg:hasArtifactRole sfcfg:artifactRole_payload .
```

Maybe even better, use classes for selector shapes if SHACL validation will be involved:

```ttl
sfcfg:AnyArtifactPolicyTarget
sfcfg:ArtifactRolePolicyTarget
sfcfg:ExactArtifactPolicyTarget
```

That makes “what fields are required” much easier to validate fail-closed.

Also, I think your message got cut off at “We dpm”. If that was “we don’t…”, I’m with you in spirit, but I don’t want to fill in the rest too confidently.

# User_2026-05-25_1506_02

Was going to say "we don't need policyPriority everywhere, and it's explicitly not to discriminate between "all" and "role-based" selections... it's for when there are conflicting policy statements at the same level. It's optional.

# gpt-5.5_2026-05-25_1506_26

Yes, agreed. That is the right boundary for `policyPriority`.

I’d define precedence like this:

1. **Config layer / scope precedence** decides between application, mesh, inherited, knop-local, command override, etc.
2. **Selector specificity** decides between target shapes within the same layer: exact artifact beats artifact role beats any artifact.
3. **`sfcfg:policyPriority`** only breaks ties among bindings in the same layer, same policy family, and same effective selector specificity.
4. If priority is absent, treat it as `0`.
5. If two same-layer/same-specificity bindings still tie and conflict, fail closed.

So `policyPriority` is not the mechanism that makes role-specific beat any-artifact. The model should know that structurally.

That means ordinary config can stay clean:

```ttl
<#history-versioned-for-all-artifacts> a sfcfg:PolicyBinding ;
  sfcfg:bindsPolicy <#history-versioned-policy> ;
  sfcfg:appliesToPolicyTarget <#all-artifacts> .

<#runtime-meta-current-only> a sfcfg:PolicyBinding ;
  sfcfg:bindsPolicy <#history-current-only-policy> ;
  sfcfg:appliesToPolicyTarget <#runtime-meta-artifacts> .
```

No priority needed there. Add `policyPriority` only when two bindings are legitimately competing in a way the selector shape cannot settle.

# User_2026-05-25_1507_50

OK what's next. Maybe try the spec first. If it's clear and succinct, I can review it before we start modifying the config ontology and also give myself a good mental model of our conceptual model. ;)

# gpt-5.5_2026-05-25_1508_57

I’ll draft the spec as a conceptual contract, not an implementation task dump: small vocabulary, resolution rules, and what counts as policy versus plain settings. I’ll re-read the current task/spec file first so I don’t accidentally overwrite your newest tweaks.

# gpt-5.5_2026-05-25_1510_04

I’ve got enough context now. I’m going to write this as a reviewable spec, with implementation details kept out unless they define externally observable behavior.

# gpt-5.5_2026-05-25_1513_13

The first draft is in place. I’m doing a quick pass now for consistency with the decisions we just made, especially “publication profile is a scoped setting” and “priority only handles same-specificity conflicts.”

# gpt-5.5_2026-05-25_1513_45

One thing looked slightly too loose on the pass: the `Knop-inheritable` layer should not accidentally beat `Knop-local` just because it is listed later. I’m tightening that wording so inherited/offered config remains overrideable locally.

# gpt-5.5_2026-05-25_1514_38

Drafted the config behavior spec here: [sf.spec.2026-05-25-config-behavior.md](../../semantic-flow-framework/notes/sf.spec.2026-05-25-config-behavior.md).

It now lays out the conceptual split between scoped settings and layered policies, treats `ConfigResolutionConfig` as resolver meta-config, keeps `ResolvedConfig` / `ConfigResolutionRecord` as derived runtime/diagnostic outputs, defines explicit policy target rules, and nails down the resolution order: layer, selector specificity, optional `policyPriority`, then fail-closed on unresolved conflicts.

A few spots I’d especially like your review on:

- Whether `ResourcePage presentation defaults` belongs as a layered policy family. I think yes; publication profile stays a scoped mesh setting.
- Whether the Knop-inheritable wording is right: outbound by default, can apply to self only by inheritance policy, and remains overrideable by Knop-local config.
- Whether the spec is intentionally abstract enough before we choose exact ontology term names.

I didn’t touch the ontology or task note yet. I also didn’t run lint/tests since this was a documentation-only spec draft. In the framework repo, this spec file is currently untracked, and `notes/sf.vision.md` was already modified separately.

Suggested commit message for `semantic-flow-framework`:

```text
docs(config): specify layered config behavior

- define scoped settings versus layered policy bindings
- document config layer precedence and explicit target selector rules
- clarify policyPriority as same-specificity conflict resolution only
- separate publication profiles from ResourcePage presentation policy
- mark ResolvedConfig and ConfigResolutionRecord as derived runtime outputs
```

# User_2026-05-25_1519_55

Is "reusable config at the attachment point that referenced it" really a layer? Also, I don't think knop-inheritable config should apply at the declaring knop. Or should it?

# gpt-5.5_2026-05-25_1520_04

I agree with both pushes. I’m going to tighten the spec: reusable config is provenance/expansion at an attachment point, not a universal precedence layer; and Knop-inheritable stays descendant-facing unless separately attached as Knop-local too.

# gpt-5.5_2026-05-25_1520_18

Yes on both.

I updated the spec accordingly. `reusable config` is now described as a referenced source that participates at the attachment point, not as its own universal precedence layer. That matches the existing ontology comment better: it can be provenance/source role, but merge happens where it is attached.

And I agree that Knop-inheritable config should not apply at the declaring Knop. It is cleaner if it means “offer to descendants.” If the author wants the same behavior locally and for descendants, they can attach it twice: once as Knop-local, once as Knop-inheritable, or reference the same reusable config from both attachment points.

The config layer list is now:

1. built-in/application defaults
2. trusted operational config
3. mesh-local
4. mesh-inheritable projected into descendants
5. Knop-inherited from ancestors
6. Knop-local
7. command overrides

# User_2026-05-25_1521_41

I feel like all config is reusable config, and something that is declared at the knop level could maybe be scoped at the mesh level? What do you think?

# gpt-5.5_2026-05-25_1523_09

I think you’re right, and this suggests a cleaner split:

- **Config content**: the RDF declarations themselves. In principle, all config can be reusable.
- **Config source**: where the content came from: inline, mesh file, Knop file, referenced artifact, application default, etc.
- **Config attachment**: the important bit. This is what gives the config its effective scope and layer: mesh-local, mesh-inheritable, Knop-local, Knop-inheritable, operational, command override.

So I’d stop treating “reusable config” as a kind of config. It’s just config used through a reference. The attachment point is what matters.

On “something declared at the Knop level could maybe be scoped at the mesh level”: I’d draw a firm line.

If you mean: a config artifact physically stored under a Knop can be explicitly referenced by mesh config and therefore used at mesh scope, yes. Storage location should not be authority by itself.

If you mean: Knop-local config can declare “actually, I apply mesh-wide,” I’d say no. That creates scope laundering. A lower/local attachment should be able to narrow behavior inside its authority, not promote itself upward.

So the rule I’d want:

> Effective scope comes from the attachment point, not from the referenced config’s self-description. A config source may narrow or validate its intended use, but it cannot expand the authority of the attachment that loaded it.

That lets all config be reusable without making reusable config magical. It also keeps mesh-wide choices explicit: something applies mesh-wide only because mesh-level config attached or referenced it.

# User_2026-05-25_1523_41

Agreed. Please update to reflect.

# gpt-5.5_2026-05-25_1523_56

I’ll fold that into the conceptual model and layer section: config is reusable as content, while attachment determines scope/layer. That should remove the lingering “reusable config as a kind of layer” smell.

# gpt-5.5_2026-05-25_1525_02

Updated the spec to reflect that model.

Key changes in [sf.spec.2026-05-25-config-behavior.md](../../semantic-flow-framework/notes/sf.spec.2026-05-25-config-behavior.md):

- All authored config is now described as potentially reusable.
- Reuse is not a separate config kind and does not define precedence.
- Effective scope/layer comes from the attachment point.
- A referenced config source can constrain intended use, but cannot expand the authority of the attachment that loaded it.
- Knop-local config cannot promote itself to mesh-wide behavior.
- Referenced config participates at the scope/layer of the attachment point, not as a universal layer.

I also smoothed the wording around Knop-inheritable config so it says shared behavior should be attached from both relevant attachment points, rather than relying on “reusable config” as a special category.

# User_2026-05-25_1526_17

Is "ResourcePage generation" just the boolean "should a resource page be generated?

# gpt-5.5_2026-05-25_1526_38

Mostly yes, but I would not make it a plain boolean.

Conceptually it answers: **what should the runtime do about ResourcePage materialization for this target?**

Current values already suggest more than true/false:

- `generate`: produce/update the ResourcePage as part of normal generation
- `suppress`: do not produce one, and do not assert `sflo:hasResourcePage`
- `defer`: allowed/expected, but not generated in this operation/profile
- `onRequest`: generate only when explicitly requested or when a serving/runtime layer needs it

So it’s boolean-ish at the simplest level, but it’s really a small lifecycle/materialization policy. I’d keep it as `ResourcePageGenerationPolicy`, not `includeResourcePage true/false`.

I do think “ResourcePage presentation defaults” is a different family: assuming a page is generated, what chrome/layout/panels/templates/styles should it use?

The distinction is useful:

- Generation: should a page exist / be materialized?
- Presentation: what should the generated page look like / include?

For the first Weave slice, we may only implement `generate` and `suppress`, but the ontology already has room for the richer behavior.

# User_2026-05-25_1529_18

Agreed. Under policy resolution, can we say "For a given policy query, resolution proceeds in this order for the default/application-level ConfigResolutionConfig" or something like that.

# gpt-5.5_2026-05-25_1529_24

Good call. I’ll make that rule explicitly tied to the default/application resolver profile so we don’t accidentally declare it as ontology law forever.

# gpt-5.5_2026-05-25_1529_36

Updated that sentence.

It now reads:

> Under the default/application-level `sfcfg:ConfigResolutionConfig`, resolution for a given policy query proceeds in this order:

That keeps the rule anchored to the default resolver profile instead of sounding like universal ontology law.

# User_2026-05-25_1544_10

I had Claude review the spec at [wa.review.2026-05-25_1541-config-behavior-spec-claude.md](wa.review.2026-05-25_1541-config-behavior-spec-claude.md) and the task at [wa.review.2026-05-25_1542-honor-mesh-config-claude.md](wa.review.2026-05-25_1542-honor-mesh-config-claude.md) 

Please review the reviews and let me know if you have any pushback.

# gpt-5.5_2026-05-25_1544_39

I’ll read both review notes against the current spec/task and then give you the “yes, adopt” versus “I’d push back” parts.

# gpt-5.5_2026-05-25_1548_53

Claude’s reviews are useful. I’d adopt most of the spec review, with two real pushbacks.

**Adopt**
The big corrections I agree with:

- Change layer precedence wording from “same target” to “overlapping target.” Better: an upper-layer binding masks lower-layer bindings only for resources its selector actually covers.
- Define “governed artifact.” My suggested rule: an artifact is governed by a scope when it is a mesh/Knop-managed `sflo:DigitalArtifact` attached to that scope through inventory/support/payload structure. `_mesh` support artifacts qualify. Merely referencing an external config artifact does not make that external artifact governed by the referencing mesh.
- Make portable `ConfigResolutionConfig` behavior explicit. I’d say portable config may only narrow resolver behavior, and initial Weave may reject/ignore unsupported resolver-config terms until implemented.
- Rename away from “policy context” toward “policy target” / “selector.” The spec’s wording is better than the task’s `GovernedArtifactPolicyContext` direction.
- Clarify ResourcePage selector specificity. I’d use “set containment” if possible: selector A is more specific than B if every target matching A also matches B. If selectors are incomparable, priority can break ties; otherwise conflict fails closed.
- Add `workspaceRootRelativeToMeshRoot` to the explicit “not a policy binding” examples.
- Make `ConfigResolutionRecord` the materialized form of the policy-resolution explanation.
- Fix the task’s `includeSemanticFlowMetadataOverride` entry-point sketch. Internally it should be a presentation-profile or panel-selection override, not a boolean-shaped config concept.

**Pushback**
I disagree with Claude’s task point 4: same-layer “any artifact” plus role-specific exception should **not** require non-zero `policyPriority`. Structural specificity should handle that. Priority is for conflicts at the same layer and same effective specificity. Requiring priority for ordinary role exceptions would make the authoring model noisier than it needs to be.

I also push back on the sizing suggestion to first implement mesh config under the old `hasHistoryTrackingDefault` shape. That would unblock SFLO faster, but it risks teaching the code the wrong model right before we replace it. I’d split spec/ontology from Weave implementation, yes, but I would not add a deliberate old-shape implementation unless SFLO replay becomes urgent enough to justify throwaway work.

**Task-Specific Fixes**
The task is now stale relative to the spec in several places. I’d update it before implementation:

- Replace `GovernedArtifactPolicyContext` / `ArtifactRolePolicyContext` language with policy targets/selectors.
- Remove the claim that explicit priority is less surprising for role-specific exceptions.
- Define masking per queried artifact/resource, not per policy family globally.
- Decide whether naming policies are policy bindings. I’m leaning no for now: they are layered scoped settings unless/until they become target-selective.
- Explicitly say `semantic-site-all-panels` both selects all generated panels and removes the metadata opt-in gate.
- Define what survives in `semantic-site-no-panels`: page shell/chrome, title/breadcrumbs/canonical metadata, but no generated data panels.
- Add a plan item to define governed-artifact semantics before merge-code work.

Net: Claude found real cleanup. The spec is basically on the right track; the task needs more revision than the spec.

# User_2026-05-25_1557_47

OK, make any last spec revisions, and then... 

Do we create a new clean task for the ontology changes and maybe accompanying code changes? Or just tweak the existing task?

# gpt-5.5_2026-05-25_1558_48

I’ll make the last spec cleanup from Claude’s useful points, but keep our deliberate choices: structural specificity beats priority, and old-shape implementation is not the preferred shortcut. Then I’ll give you a recommendation on task shape.

# gpt-5.5_2026-05-25_1559_53

I’m going to make the spec a little less overcommitted on naming policy too. Naming can still be layered behavior, but unless it becomes target-selective, it doesn’t need to wear the full policy-binding outfit.

# gpt-5.5_2026-05-25_1602_41

I made the last spec revisions in [sf.spec.2026-05-25-config-behavior.md](../../semantic-flow-framework/notes/sf.spec.2026-05-25-config-behavior.md):

- portable `ConfigResolutionConfig` can narrow, not widen, resolver behavior
- precedence now says overlapping targets, not “same target”
- governed artifact is defined
- referenced config is attachment-scoped, not a layer
- naming policies are not forced into policy bindings yet
- ResourcePage selector specificity uses containment, with priority only for same-layer conflicts
- command override text names `sfcfg:OperationRequestOverridePolicy`
- `ConfigResolutionRecord` is called out as the materialized explanation form
- initial Weave slice explicitly defers inheritable projection and arbitrary referenced config resolution

On the task: create a new clean task.

I would not keep tweaking the existing task as the implementation source of truth. It has too much now-stale conceptual scaffolding: `PolicyContext`, `GovernedArtifactPolicyContext`, priority-as-specificity, reusable config as a layer, and a boolean-ish metadata override signature. Editing all of that in place will leave a note that reads like sedimentary rock.

My recommendation:

- Keep the existing `wa.task...honor-mesh-config` as the exploratory/design-history task.
- Create a new task for “config policy ontology and Weave mesh-config runtime” that cites the spec and the two review notes.
- Make the new task narrow and current: ontology vocabulary first, then Weave policy index/compiled config, then mesh-local `_mesh/_config/config.ttl` support.
- Split implementation phases inside the new task, but not separate task files unless it gets too large.

I only changed the spec. I did notice unrelated framework repo changes in the Alice Bio conformance files; I left those alone.

# User_2026-05-25_1610_47

I called it wa.task.2026.2026-05-25_1609-config-policy-ontology-and-runtime because it's not just about mesh-config. Please fill it out.

# gpt-5.5_2026-05-25_1611_26

I’ll turn it into the clean implementation task, anchored on the new spec and the review notes rather than reusing the older mesh-config wording. I’ll check the task template and the stub first so the shape matches the archive conventions.

# gpt-5.5_2026-05-25_1611_34

The framework spec is now in the shape this task should point at. I’m going to replace the stub body with a current task note that treats the old mesh-config task as background, not as the implementation contract.

# gpt-5.5_2026-05-25_1612_06

I’m going to make the task fairly concrete, with phases and contract changes, but leave exact RDF class names as an explicit decision point where the ontology edit should settle them. That should make it implementable without pretending the names are already final.

# gpt-5.5_2026-05-25_1613_08

The new task is filled in. I’m doing a quick readback now for obvious contradictions with the spec, and I’ll leave the existing older task untouched.

# gpt-5.5_2026-05-25_1613_26

Filled out the new task: [wa.task.2026.2026-05-25_1609-config-policy-ontology-and-runtime.md](wa.task.2026.2026-05-25_1609-config-policy-ontology-and-runtime.md).

I framed it as the current implementation contract, with the older honor-mesh-config task as background. It now covers ontology vocabulary, scoped settings versus policy bindings, governed artifacts, selector specificity, ResourcePage profiles, runtime `CompiledConfig` / `PolicyIndex`, command overrides, testing, non-goals, and an implementation checklist.

I didn’t run lint/tests since this was a task-note edit only. I also left the other modified/untracked archive notes alone.

# User_2026-05-25_1625_27

OK, ready for implementation? New conversation or this one? Should we do a weave version bump + PR to main first?

# gpt-5.5_2026-05-25_1625_36

I’ll sanity-check the current worktree state before giving a recommendation, because “version bump + PR first” depends a lot on whether the pending changes are coherent or mixed with other fixture work.

# gpt-5.5_2026-05-25_1627_00

Yes, ready conceptually.

I’d start a **new conversation** for implementation. This one has useful design history, but it also has old model residue. The new thread should point at:

- `[[sf.spec.2026-05-25-config-behavior]]`
- `[[wa.task.2026.2026-05-25_1609-config-policy-ontology-and-runtime]]`
- maybe the old `[[wa.task.2026.2026-05-24_2304-honor-mesh-config]]` only as background

On version bump + PR: **yes, I’d do that first if the current Weave release branch is otherwise ready**. You’re on `weave` branch `next/v0.2.0`, and the actual Weave worktree looks clean. The config-policy implementation is going to be a real cross-cutting change, so I’d rather not mix it with release-note/fixture/versioning work.

Suggested sequence:

1. Finish/currently validate the v0.2.0 release state.
2. Bump/version and PR `next/v0.2.0` to `main`.
3. Start a new branch/conversation for config policy ontology/runtime.
4. First implementation pass should probably be ontology terms + default config migration + Weave `CompiledConfig` skeleton, before wiring every command path.

One small note: `weave-dev-archive` still has modified conversation notes, but `weave`, `sflo`, and `semantic-flow-framework` currently look clean from `git status`. So the PR-first path seems nicely low-friction.

# User_2026-05-25_1708_52

First comment in the other conversation was "Add an explicit sfcfg:hasPolicyBinding attachment property. The task names PolicyBinding, bindsPolicy, and appliesToPolicyTarget, but the config-to-binding edge should be first-class."

Does that make sense?

# gpt-5.5_2026-05-25_1708_59

Yes, that makes sense. I’d make `sfcfg:hasPolicyBinding` first-class.

Without it, you have policy definitions and targets floating around, but no clear authored edge saying “this config source contributes this binding at this attachment/layer.” You could infer bindings by scanning all `sfcfg:PolicyBinding` nodes in the same document, but that is sloppy RDF hygiene and dangerous once reusable/referenced config enters the picture.

The shape should be explicit:

```ttl
<> a sfcfg:MeshConfig ;
  sfcfg:hasPolicyBinding <#history-versioned-for-governed-artifacts> .

<#history-versioned-for-governed-artifacts> a sfcfg:PolicyBinding ;
  sfcfg:bindsPolicy <#history-versioned-policy> ;
  sfcfg:appliesToPolicyTarget <#governed-artifacts> .
```

I’d define it roughly as:

- domain: `sfcfg:Config`
- range: `sfcfg:PolicyBinding`
- meaning: the config contributes this policy binding at the config’s effective attachment point
- validation: a `PolicyBinding` not reachable from an active config via `hasPolicyBinding` should not affect resolution

That last point matters. It lets reusable config documents contain policy definitions, targets, examples, or dormant bindings without everything automatically becoming active.

# User_2026-05-25_1709_33

I thought we were going to use a relator.

# gpt-5.5_2026-05-25_1709_39

I think we are using a relator: `sfcfg:PolicyBinding` is the relator.

The distinction is:

- `sfcfg:PolicyDefinition` says the reusable policy value: “history tracking = versioned.”
- `sfcfg:PolicyTarget` says what can be matched: “any governed artifact,” “payload artifacts,” etc.
- `sfcfg:PolicyBinding` is the relator that connects policy + target and carries binding-specific metadata like priority.

But we still need an attachment edge from the config to the relator:

```ttl
<> sfcfg:hasPolicyBinding <#binding> .

<#binding> a sfcfg:PolicyBinding ;
  sfcfg:bindsPolicy <#policy> ;
  sfcfg:appliesToPolicyTarget <#target> .
```

That does not stop it from being a relator. It activates/declares which relators this config contributes. Otherwise every `PolicyBinding` node in the graph risks becoming active just because it exists.

So: `PolicyBinding` is the relator; `hasPolicyBinding` is the config-to-relator attachment.

# User_2026-05-25_1710_34

would bindsPolicy be the inverse of hasPolicyBinding?

# gpt-5.5_2026-05-25_1710_56

No. `bindsPolicy` should not be the inverse of `hasPolicyBinding`.

They connect different things:

```ttl
<> sfcfg:hasPolicyBinding <#binding> .

<#binding> sfcfg:bindsPolicy <#policy> ;
  sfcfg:appliesToPolicyTarget <#target> .
```

Meanings:

- `hasPolicyBinding`: `Config -> PolicyBinding`
  - “this config contributes/activates this binding”
- `bindsPolicy`: `PolicyBinding -> PolicyDefinition`
  - “this binding uses this policy definition”

The inverse of `hasPolicyBinding` would be something like:

```ttl
sfcfg:policyBindingInConfig
```

or maybe `sfcfg:policyBindingForConfig`, though I’m not sure we need to name it.

The inverse of `bindsPolicy` would be something like:

```ttl
sfcfg:policyBoundBy
```

Also probably not needed.

A good mental model:

```ttl
Config --hasPolicyBinding--> Binding --bindsPolicy--> PolicyDefinition
                                 |
                                 +--appliesToPolicyTarget--> PolicyTarget
```

So the binding is the relator in the middle.

# User_2026-05-25_1712_21

next comment was "Use explicit PolicyFamily resources, even if we also add subclasses. That keeps the runtime PolicyIndex generic and makes diagnostics less magical." -- can we do those as enums instead of classes?

# gpt-5.5_2026-05-25_1712_27

Yes. I’d make policy families controlled vocabulary instances, not classes.

Something like:

```ttl
sfcfg:PolicyFamily a rdfs:Class .

sfcfg:policyFamily_historyTracking
  a sfcfg:PolicyFamily ;
  rdfs:label "history tracking" .

sfcfg:policyFamily_resourcePageGeneration
  a sfcfg:PolicyFamily ;
  rdfs:label "ResourcePage generation" .

sfcfg:policyFamily_resourcePagePresentation
  a sfcfg:PolicyFamily ;
  rdfs:label "ResourcePage presentation" .
```

Then:

```ttl
<#history-versioned-policy> a sfcfg:PolicyDefinition ;
  sfcfg:hasPolicyFamily sfcfg:policyFamily_historyTracking ;
  sfcfg:hasHistoryTrackingPolicy sfcfg:historyTrackingPolicy_versioned .
```

That is enum-like RDF: `PolicyFamily` is the category class, and the actual enum values are named individuals.

I agree with the comment’s motivation. Explicit family values make the runtime `PolicyIndex` straightforward:

```ts
family: "historyTracking" | "resourcePageGeneration" | "resourcePagePresentation"
```

and diagnostics can say “conflict in policyFamily_historyTracking” without inferring family from class shape. Subclasses can still help validation or documentation, but the family enum should be the runtime key.

# User_2026-05-25_1715_13

what's the point of policyFamily? It's kind of just a marker? What weight does it pull?

# gpt-5.5_2026-05-25_1715_23

`policyFamily` pulls weight mainly in resolution, not in author-facing meaning.

It gives the runtime a clean answer to: **which bindings are competing with each other?**

Example: these two policies might target the same artifact:

```ttl
<#history-versioned-policy>
  sfcfg:hasPolicyFamily sfcfg:policyFamily_historyTracking ;
  sfcfg:hasHistoryTrackingPolicy sfcfg:historyTrackingPolicy_versioned .

<#page-generate-policy>
  sfcfg:hasPolicyFamily sfcfg:policyFamily_resourcePageGeneration ;
  sfcfg:hasResourcePageGenerationPolicy sfcfg:resourcePageGenerationPolicy_generate .
```

They overlap by target, but they do not conflict because they are different families. The policy index can bucket by family first, then resolve only within that bucket.

It also helps with:

- diagnostics: “conflict in history tracking policy” rather than “two opaque policy definitions had incompatible properties”
- validation: each family has expected value predicates
- generic runtime code: one `PolicyIndex.resolve(family, target)` path instead of hard-coding every policy class
- mixed policy definitions: if we ever allow a definition to carry multiple concerns, family tells the resolver which dimension is being queried
- future extension: new policy families can be added without rewriting the binding model

Could we infer family from the value predicate like `hasHistoryTrackingPolicy`? Yes, for simple cases. But then the runtime has to maintain predicate-to-family magic anyway. Making it explicit moves that magic into RDF where authors and diagnostics can see it.

So: it is a marker, but a useful one. It is the bucket key for conflict resolution.

# User_2026-05-25_1719_11

But just because two policies are of the same family doesn't mean they necessarily conflict. I feel like conflict detection is mostly on the predicate level. SHACL. expected value predicates are per-predicate. Your examples targeting the same artifact doesn't clinch anything for me.

# gpt-5.5_2026-05-25_1719_28

I think your pushback is right. I overstated `policyFamily`.

Same family is not enough to define conflict. Conflict is really on the **resolved policy slot**, which is probably the value predicate or a value-predicate-equivalent dimension.

For example, if a future “ResourcePage presentation” family includes:

- `sfcfg:hasDefaultResourcePagePresentationConfig`
- `sfcfg:hasResourcePageStylesheet`
- `sfcfg:hasResourcePagePanelSelection`

those are not automatically mutually conflicting just because they are all presentation-ish. Some may be singleton, some additive, some ordered, some mergeable. SHACL and resolver behavior need to know the predicate/slot, not only the family.

So I’d revise my position:

- `PolicyFamily` is optional grouping/diagnostic metadata.
- It should not be the core conflict key.
- The runtime `PolicyIndex` should primarily key on a **policy slot** or **policy value predicate**.
- Conflict detection should be per slot, with slot-specific cardinality/merge semantics.

A cleaner model might be:

```ttl
<#history-versioned-policy> a sfcfg:PolicyDefinition ;
  sfcfg:setsPolicyProperty sfcfg:hasHistoryTrackingPolicy ;
  sfcfg:hasHistoryTrackingPolicy sfcfg:historyTrackingPolicy_versioned .
```

or we just infer the slot from the value predicate:

```ttl
<#history-versioned-policy> a sfcfg:PolicyDefinition ;
  sfcfg:hasHistoryTrackingPolicy sfcfg:historyTrackingPolicy_versioned .
```

Then the resolver says: this policy definition sets the `hasHistoryTrackingPolicy` slot. Conflicts are between bindings that set the same slot for overlapping targets after layer/specificity/priority resolution.

If we keep family at all, I’d demote it to something like:

```ttl
sfcfg:hasPolicyGroup sfcfg:policyGroup_tracking .
```

Useful for docs/UI/log grouping, not normative conflict behavior.

So yes: the implementation comment’s motivation is good, but `PolicyFamily` may be the wrong abstraction if it becomes more than a label. Predicate/slot is the thing that carries semantic weight.

# User_2026-05-25_1722_57

Give me a prompt justifying the removal of PolicyFamily

# gpt-5.5_2026-05-25_1723_24

Use this:

```text
Please remove `PolicyFamily` from the proposed config policy-binding model.

Rationale: conflict detection and merge semantics are not correctly determined by broad policy families. Two policy definitions being in the same family does not mean they conflict, and two policies in a broad area like ResourcePage presentation may set different slots with different merge/cardinality rules. The meaningful unit for resolution is the policy slot/value predicate, such as `sfcfg:hasHistoryTrackingPolicy` or `sfcfg:hasResourcePageGenerationPolicy`, not a family label.

The runtime should identify which policy slot a `PolicyDefinition` sets, either by inspecting supported value predicates directly or through an explicit slot/property marker if that proves useful. Conflicts are then detected between applicable bindings that set the same policy slot for overlapping targets after layer precedence, selector specificity, and optional priority resolution.

`PolicyFamily` can be reintroduced later as non-normative grouping metadata for UI/logging/docs if needed, but it should not be part of the normative resolution model now. Removing it keeps the ontology smaller, avoids a misleading abstraction, and aligns validation with SHACL-style per-predicate constraints.
```

# User_2026-05-25_1737_25

Next in my sights for simnplification: <ResourcePagePresentationConfig> -- I don't think we should have different kinds of configs other than the level-based ones. I also think we should remove <OperationalConfig>/<WorkspaceOperationalConfig>/<HostLocalOperationalConfig>  -- if there's something specific to an eventual Weave service, it can have its own ontology. But that doesn't belong here in the SF side.

Regarding ArtifactRole. 

Only two of these don't have corresponding core ontology classes: artifactRole_resourcePageStylesheet and artifactRole_resourcePageTemplate. Maybe it would expand later. How are we going to tell that things are ResourcePageStylesheets/Templates if they don't have classes?

# gpt-5.5_2026-05-25_1737_44

I agree with the simplification direction.

`ResourcePagePresentationConfig` should probably stop being a kind of `Config`. It is not a level/attachment scope like application, mesh, or Knop. It is a **policy value/profile** selected by config. I’d rename/reframe it as something like:

```ttl
sfcfg:ResourcePagePresentationProfile
```

or maybe:

```ttl
sfcfg:ResourcePagePresentation
```

Then application/mesh/Knop config can select it through a policy binding or scoped default. The profile itself can still contain panel selections, templates, stylesheets, etc., but it is not “config” in the same sense.

On `OperationalConfig` / `WorkspaceOperationalConfig` / `HostLocalOperationalConfig`: yes, I’d remove them from the Semantic Flow config ontology. They are runtime trust/environment concepts. A future Weave daemon/service ontology can define Weave-specific operational config if needed. The SF-side ontology should describe portable Semantic Flow behavior, not host-local authority.

For `ArtifactRole`: this is the sharper question.

If `artifactRole_resourcePageTemplate` and `artifactRole_resourcePageStylesheet` are real governed artifact roles, then we need a way to classify the actual artifacts. I see three possible paths:

1. **Move/add classes to core**
   Add `sflo:ResourcePageTemplate` and `sflo:ResourcePageStylesheet` as `sflo:DigitalArtifact` subclasses in core. This is clean if templates/stylesheets are first-class Semantic Flow support artifacts, not merely config implementation details.

2. **Remove those artifact roles for now**
   If Weave templates/stylesheets are built-in application assets, not mesh-governed artifacts yet, then they should not be artifact roles yet. Keep them as implementation/default resources until the mesh can actually govern custom templates/stylesheets.

3. **Use a generic support-artifact role assignment**
   Let an artifact carry a role without a matching class. But that feels weaker than the rest of the model and would make ResourcePage templates/stylesheets special in an unhelpful way.

My preference: if custom ResourcePage templates/stylesheets are intended to become mesh-governed artifacts, add core classes for them. If not, remove `artifactRole_resourcePageTemplate` and `artifactRole_resourcePageStylesheet` for now.

I would not keep those roles while removing the only classes that identify them. That leaves the ontology with roles the runtime can name but the mesh cannot clearly assert.

# User_2026-05-25_1752_44

Remove "OperationalConfig / WorkspaceOperationalConfig / HostLocalOperationalConfig" and any related single-purpose properties or prose.

Is there anything analogous to <ResourcePagePresentationConfig>  for History or any of the other concerns? If not, why not? I thought we already had a way to capture sets of configs relevant to a config domain, but it's easy to get confused here. 

Regarding ArtifactRole... anything can be a mesh-governed artifact. And actually we do have a "<ResourcePageTemplate>" class, and subclasses. The key is that these other things (inventories, metadata, local-config, supporting-config, reference catalog, page definitions)  are "supporting artifacts", with their corresponding special dirs under the knop. If the only use case is controlling inventories or resource page generation/profiles for supporting artifacts (that don't have their own config), then we should remove everything that doesn't correspond to a supporting artifact other than perhaps _payload. I'm not clear what artifactRole_runtimeMeta is. What do you think?

# gpt-5.5_2026-05-25_1752_53

I’ll make the ontology simplification for the operational-config classes, then I’ll answer the ResourcePage/profile and artifact-role questions against the actual vocabulary. The runtime-meta role in particular is worth checking before we either keep or cut it.

# gpt-5.5_2026-05-25_1753_40

I’m going to remove the explicit operational config class layer from the SFLO config ontology now, but I’m going to be conservative about broader access-rule vocabulary unless it directly depends on those classes. The access rules may still be useful as portable mesh expectations, even if host-local Weave trust policy moves to a Weave/service ontology.

# gpt-5.5_2026-05-25_1755_19

I’m going to remove the operational access-rule block too, keeping only `workspaceRootRelativeToMeshRoot` as a portable mesh layout setting. That matches your “SF side shouldn’t own Weave/host runtime trust” line better than just deleting three classes and leaving the operational policy machinery behind.

# gpt-5.5_2026-05-25_1758_21

The ontology is clean of the explicit operational terms now. I’m also going to adjust the behavior spec/task wording so we don’t leave “operational config” as if it were still an SF config type.

# gpt-5.5_2026-05-25_1801_05

I removed the SF-side operational config layer from the ontology and cleaned the nearby spec/task prose.

Changed:
- Deleted `OperationalConfig`, `WorkspaceOperationalConfig`, `HostLocalOperationalConfig`.
- Removed `configLayerRole_machineLocalOperational` and `configLayerRole_workspaceOperational`.
- Removed the local/remote access-rule block from SF config ontology.
- Kept `workspaceRootRelativeToMeshRoot`, but reframed it as portable mesh layout, not access/trust policy.
- Updated `sf.spec.2026-05-25-config-behavior` and the new task to say host/service/runtime trust belongs outside portable SF config.

On `ResourcePagePresentationConfig`: I think it is now an outlier. There isn’t an analogous `HistoryConfig` bundle. History has value vocabularies like `HistoryTrackingPolicy`; naming has value vocabularies; ResourcePage generation has value vocabularies. Presentation is different only because it bundles templates, stylesheets, and panel selections. I’d reframe it as `ResourcePagePresentationProfile`, not a `Config` subclass. Config selects or binds to a profile; the profile is not itself a config level.

On `ArtifactRole`: I agree with your instinct. Roles should probably be limited to canonical Semantic Flow governed artifact slots: payload plus real support artifacts. `artifactRole_runtimeMeta` is especially suspicious; I don’t see a corresponding core artifact class or canonical support surface, so I’d remove it unless Weave has a concrete governed runtime metadata artifact. For templates/stylesheets, either move those classes into core and define them as first-class support artifacts, or remove the roles for now. I would not keep roles whose only identity lives in presentation config machinery.

# User_2026-05-25_1803_59

Maybe ResourcePagePresentationPolicy is more uniform than ResourcePagePresentationProfile ?

# gpt-5.5_2026-05-25_1804_24

Yes, `ResourcePagePresentationPolicy` is probably more uniform.

It would line up with:

- `HistoryTrackingPolicy`
- `ResourcePageGenerationPolicy`
- `ResourcePageRegenerationConfigPolicy`
- `HistoryNamingPolicy`
- `StateNamingPolicy`
- `ManifestationNamingPolicy`

And it avoids making presentation sound like a special bundle/config-kind. The resource can still contain structured details:

```ttl
<#semantic-site-all-panels> a sfcfg:ResourcePagePresentationPolicy ;
  sfcfg:hasOuterResourcePageTemplate <...> ;
  sfcfg:hasInnerResourcePageTemplate <...> ;
  sfcfg:hasResourcePageStylesheet <...> ;
  sfcfg:hasResourcePagePanelSelection ... .
```

Then config can bind/select it like any other policy value:

```ttl
<#presentation-policy> a sfcfg:PolicyDefinition ;
  sfcfg:hasResourcePagePresentationPolicy <#semantic-site-all-panels> .
```

I slightly prefer `Policy` over `Profile` here because it expresses runtime choice: “use this presentation policy when rendering generated ResourcePages.” `Profile` sounds like a named preset, which is true for built-ins but too narrow for authored/custom cases.

I’d rename:

- `ResourcePagePresentationConfig` -> `ResourcePagePresentationPolicy`
- `hasDefaultResourcePagePresentationConfig` -> probably `hasResourcePagePresentationPolicy` or `hasDefaultResourcePagePresentationPolicy`
- `hasResourcePagePresentationConfig` on `ResourcePageDefinition` -> maybe `hasResourcePagePresentationPolicy`

The “default” variant may become unnecessary once policy bindings exist, because the target/layer says when it applies. For page-local direct adjacency, `hasResourcePagePresentationPolicy` is enough.

# User_2026-05-25_1807_03

I agree, no default.OK, update the ontology, and let me know what code fixes are required, but don't make them yet.

# gpt-5.5_2026-05-25_1808_08

I’ll update only the ontology vocabulary now: rename the presentation bundle from a config kind to a policy value, remove the “default presentation config” property, and leave code untouched. Then I’ll enumerate the Weave code/docs fallout separately.

# gpt-5.5_2026-05-25_1808_49

I’m going to rename the presentation object/property in the ontology now and remove the default-specific property entirely, since defaulting should come from config attachment/policy resolution rather than the property name.

# gpt-5.5_2026-05-25_1810_55

The ontology rename is clean in the ontology file itself. I’m doing one targeted pass over Weave code/defaults now so I can give you the exact follow-up fixes without touching them.