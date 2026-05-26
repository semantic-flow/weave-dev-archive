---
id: sflo-ontology-and-weave-2y6ni4
title: 'sflo ontology and weave runtime config support'
desc: ''
created: 1779773647136
updated: 1779773647136
participants: [codex.gpt-5.5]
conversationEventKinds: [message.assistant, message.user, tool.call, tool.result]
---

# User_2026-05-25_1705_12

sflo ontology and weave runtime config support

Please review: 

[[sf.spec.2026-05-25-config-behavior]]
[[wa.task.2026.2026-05-25_1609-config-policy-ontology-and-runtime]]

And let me know if you have any questions or refinements before we implement.

# gpt-5.5_2026-05-25_1705_19

I’ll read the repository guidance first, then the two Dendron notes you named, and I’ll come back with questions plus any design refinements that seem worth making before implementation.

Found the spec in the framework dependency notes and the task in the archive dependency, alongside the local Weave guidance. I’m pulling those together now so the review is grounded in both repos’ conventions.

The task/spec pair is internally pretty coherent; I’m doing one more pass against the actual ontology/runtime files so any push-back is about implementation pressure, not abstract taste.

One concrete tension is already visible: the current config ontology has an `ArtifactRolePolicy` direct-default model and even a `configLayerRole_reusableConfig` value, while the new spec wants explicit bindings and says reuse is not a layer. That’s a good candidate for cleanup rather than compatibility plumbing.

I reviewed both notes plus the current ontology/runtime shape. I think the direction is sound and worth implementing cleanly. No big conceptual blocker, but I’d refine a few things before we start.

**Refinements**
- Add an explicit `sfcfg:hasPolicyBinding` attachment property. The task names `PolicyBinding`, `bindsPolicy`, and `appliesToPolicyTarget`, but the config-to-binding edge should be first-class.
- Use explicit `PolicyFamily` resources, even if we also add subclasses. That keeps the runtime `PolicyIndex` generic and makes diagnostics less magical.
- Do not reuse core `sflo:hasTargetArtifact` for exact policy targets. Its domain is `sflo:ArtifactResolutionTarget`, so using it on config targets would infer the wrong type. Prefer a config-side property like `sfcfg:targetsArtifact` or `sfcfg:hasExactArtifactTarget`.
- Remove `configLayerRole_reusableConfig` from the default precedence profile. It can be a provenance/source role elsewhere, but keeping it ordered as a layer fights the new spec’s “reuse is not a layer” rule.
- Treat config-scope ResourcePage presentation defaults as policy bindings. Keep page-local `sfcfg:hasResourcePagePresentationConfig` on `sflo:ResourcePageDefinition` as direct adjacency.
- Let `semantic-site-no-panels` have zero generated panel selections. Current parser rejects empty panel selections, and SHACL currently warns on zero selections; Weave must relax the parser for that profile.
- Add a non-opt-in metadata data requirement or equivalent for `semantic-site-all-panels`. The current runtime only makes the metadata panel available through `includeSemanticFlowMetadata`, so “all panels” needs a real policy-level path, not a hidden boolean.
- Make the first runtime API query by target object, not only by artifact role. If we keep role-only helpers as the core surface, exact-artifact and ResourcePage selectors will be awkward immediately.

**Questions**
1. For old direct default terms like `hasDefaultHistoryTrackingPolicy`, should I remove live Weave authoring/parser support outright, while leaving released ontology history alone? My vote: yes.
2. For `ResourcePageGenerationPolicy_defer`, should Weave omit `sflo:hasResourcePage` facts like `suppress`, or preserve promises but skip rendering until requested? This needs one crisp rule before tests.
3. For first-slice diagnostics, is structured logger output plus an in-memory resolution trace enough? I’d avoid persisted `ResolvedConfig`/`ConfigResolutionRecord` for now.
4. Should exact artifact policy targets be accepted in mesh config in this slice, or should we add the vocabulary and parser but reject them until query call sites can provide full governed-artifact context? My preference: implement them now, using mesh inventory as the governance authority.

I’d implement this in the clean model, not as a bridge through the old vocabulary. The notes are already pointing that way; I’d just make those few decisions explicit before coding.

# User_2026-05-25_1724_23

- yes, add hasPolicyBinding

Please remove `PolicyFamily` from the proposed config policy-binding model.

Rationale: conflict detection and merge semantics are not correctly determined by broad policy families. Two policy definitions being in the same family does not mean they conflict, and two policies in a broad area like ResourcePage presentation may set different slots with different merge/cardinality rules. The meaningful unit for resolution is the policy slot/value predicate, such as `sfcfg:hasHistoryTrackingPolicy` or `sfcfg:hasResourcePageGenerationPolicy`, not a family label.

The runtime should identify which policy slot a `PolicyDefinition` sets, either by inspecting supported value predicates directly or through an explicit slot/property marker if that proves useful. Conflicts are then detected between applicable bindings that set the same policy slot for overlapping targets after layer precedence, selector specificity, and optional priority resolution.

`PolicyFamily` can be reintroduced later as non-normative grouping metadata for UI/logging/docs if needed, but it should not be part of the normative resolution model now. Removing it keeps the ontology smaller, avoids a misleading abstraction, and aligns validation with SHACL-style per-predicate constraints.

# gpt-5.5_2026-05-25_1724_35

Agreed. I’ll fold that decision into the behavior spec and task note now, so implementation doesn’t accidentally bake in a broad family abstraction.

I found all the lingering “family” language. I’m going to keep harmless prose where it means a merge-profile category only if it won’t confuse the normative model, and switch the actual resolution model to policy slots/value predicates.

The framework spec now says resolution is per policy slot/value predicate and explicitly excludes `PolicyFamily` from the normative model. Next I’m aligning the Weave task note so the implementation checklist points the same way.

The notes are aligned now. I also tightened one existing ontology comment from “property-family” to “property-slot” so the live config vocabulary doesn’t leave a misleading breadcrumb.

Diff looks tidy: only the requested spec/task wording plus one ontology comment. I’m running the SFLO test task because the Turtle file changed, even though it’s just a comment.

Done. I updated the proposed model to use policy slots/value predicates instead of `PolicyFamily`, and added `sfcfg:hasPolicyBinding` to the binding vocabulary list.

Changed:
- [sf.spec.2026-05-25-config-behavior.md](../../semantic-flow-framework/notes/sf.spec.2026-05-25-config-behavior.md:26): normative resolution now talks about policy slots, slot predicates, and per-slot conflicts.
- [wa.task.2026.2026-05-25_1609-config-policy-ontology-and-runtime.md](wa.task.2026.2026-05-25_1609-config-policy-ontology-and-runtime.md:95): task/open issues now include `hasPolicyBinding` and explicitly reject normative `PolicyFamily`.
- [semantic-flow-config-ontology.ttl](../../sflo/semantic-flow-config-ontology.ttl:517): changed the existing merge-profile comment from property-family to property-slot.

Validation: `deno task test` passed in `sflo` with 23 tests passing.

Heads up: `weave-dev-archive` already had two modified conversation notes I did not touch. Suggested commit summaries:
- `docs(config): resolve policy bindings by slot predicate`
- `docs(config): record hasPolicyBinding and remove PolicyFamily`
- `docs(config): align merge profile wording with policy slots`

# User_2026-05-25_1811_16

I committed on weave, but I think with the wrong commit message. Can you ammend the commit message, I think the work was for the manifest support in another conversation.

# gpt-5.5_2026-05-25_1811_27

I’ll inspect the latest Weave commit and the current worktree so we only amend the intended repo and don’t disturb the ontology/spec edits in the dependencies.

The latest Weave commit is indeed labeled as config work, but its files are fixture ladder plus raw-source panel manifest support. There are also uncommitted changes in two Weave integration tests, so I’ll amend only the message with `--only` and leave those untouched.

I’m going to amend the subject/body to describe the actual slice: manifest-backed fixture replay steps plus binary-safe raw source panels. The current uncommitted test edits will stay out of the amend.

Amended the latest Weave commit message.

New HEAD: `8399c70 feat(fixtures): support manifest-backed replay sources`

Body now describes:
- manifest-declared integrate commands for Alice Bio sidebar/favicon transitions
- page-definition fixture source description update
- binary-safe raw-source panel handling

I left the current uncommitted Weave integration test edits and dependency repo changes untouched. No tests run for the message-only amend.
