---
id: 1cz6zxv6mlfcra9yte2bjh1
title: 2026 05 25_1541 Config Behavior Spec Claude
desc: ''
updated: 1779748905940
created: 1779748903595
---

Substantive issues
1. Selector-specificity rule disagrees with the Weave task. Policy Resolution step 3 (sf.spec.2026-05-25-config-behavior.md:105) makes selector specificity a tiebreaker within a layer, and explicitly says "sfcfg:policyPriority is not the mechanism that makes a role-specific policy beat an any-artifact policy" (L110). The Weave task takes the opposite stance for its slice — see wa.task L120: "More specific contexts can either be compiled to a higher priority by convention or explicitly carry a higher sfcfg:policyPriority; for this slice, explicit priority is less surprising." Either the framework spec wins (Weave drops the "explicit priority" stance and adds structural specificity to its resolver), or the spec is too prescriptive for implementations that want a flatter same-layer model. Pick one before either doc lands.

2. L42 and L112 contradict each other on the surface. L42: "Higher-precedence layers win over lower-precedence layers for the same policy family and target." L112: a mesh-local "any artifact" history binding overrides lower-layer role-specific defaults — those targets are not the same. Resolve by either (a) tightening L42 to say "for an overlapping target," or (b) keeping L42 strict and using a sentence in L112 like "an upper-layer binding masks lower-layer bindings for every target its selector covers." Without one of those tweaks, a reader can defensibly come away with either an extensional or intensional view of "same target."

3. Define "governed artifact" or "any artifact in the current config scope." L89 leans on this phrase to define the broadest artifact selector, and L112 uses "any artifact in that mesh" as the canonical example, but neither pins what makes an artifact governed by a scope. Is it path-prefix under meshRoot? Does it include reusable-config artifacts referenced by IRI but stored under a different mesh? Does the _mesh metadata artifact qualify? The Weave task has the same gap (it inherits it from here). Worth one paragraph in the Conceptual Model.

4. Resolver-config trust boundary needs an operational rule. L28 and L143 correctly say portable config must not use resolver config to expand trust, but the spec never says what a mesh-local sfcfg:ConfigResolutionConfig declaration IS allowed to do. Two reasonable shapes — pick one explicitly:

Mesh-local ConfigResolutionConfig is ignored entirely by trusted resolvers; only application/operational layers can supply resolver config.
Mesh-local ConfigResolutionConfig may narrow (tighten) the active resolver config but never widen it (so a mesh can demand stricter unknown-term rejection or a lower maxConfigReferenceDepth, but not the inverse).
The current wording — "may be narrowed by trusted or portable config where allowed" (L28) — leaves "where allowed" undefined.

5. Layer list and the rest of the spec disagree about knopInheritable and reusable config. The ordered layers at L34-40 list "Knop-inheritable config" implicitly via Knop layers, but L46 explicitly says Knop-inheritable is an outbound offer — not a layer consumed by the declaring Knop. L19 similarly says reuse is not a separate config kind. Yet defaults/config-resolution.ttl and the Weave task (L34) list reusableConfig and knopInheritable as named layer roles. Reconcile: either the layer list is the source of truth and reusable/inheritable get real layer positions explained, or those vocabulary terms get reframed as "attachment roles" rather than "layer roles."

6. Vocabulary drift: "target selector" vs "policy context." The spec uses "policy target," "selector shape," "selector value" (L83-95). The Weave task uses "policy context" with concrete classes (GovernedArtifactPolicyContext, ArtifactRolePolicyContext). Same idea, different name. Pick one before the ontology lands — these terms will become RDF class names users author.

7. Validation section (L134) drops scoped-settings validation. L57 says scoped settings get fail-closed validation, but the central Validation list at L134-143 is framed around policy-family resolution. Fold scoped-settings rules into the central list (unsupported values, duplicate singletons, unsafe path values) — otherwise implementers will skim the central list and miss the L57 requirement.

8. Operation-request override interaction is half-specified. L128 says command overrides are highest precedence "unless a stricter invariant or operation-request override policy rejects the request." But sfcfg:operationRequestOverridePolicy isn't introduced anywhere, and historyTrackingPolicy_required is the only concrete invariant in flight (sfcfg ontology L277-L279). Either reference operationRequestOverridePolicy explicitly in this spec or remove the hedge — otherwise implementers will independently invent semantics for "stricter invariant."

Smaller nits
L17 introduces sfcfg:ResolvedConfig and sfcfg:ConfigResolutionRecord as outputs, then never references them again. The Runtime Shape section (L147) is the natural place to say ConfigResolutionRecord is the materialized form of the explanation that L159 describes.
L74 enumerates initial policy families; the Weave task should be required to use these exact family identifiers in its policy-binding schema so a cross-implementation diagnostic format is possible.
L97 lists ResourcePage selectors (page-kind, target-class, artifact-role, exact-page, authored/generated) but never says whether selector specificity for ResourcePages has a documented total order. If two bindings target {page-kind=identifier} and {artifact-role=payload}, which is "more specific"? Either define the lattice or say "ResourcePage selector specificity is implementation-defined for this revision."
L82 says publication profiles are not policy bindings — good. Add the symmetric statement: "workspaceRootRelativeToMeshRoot is not a policy binding either." Right now only publication profile is called out.
L165 lists deferrable items but omits inheritable-config projection, which the Weave slice also defers. Add it to the deferred list for clarity.
Where the spec is doing real work
The Scoped Settings vs Layered Policies distinction (L23-26) is the right cut and matches the Weave instinct to keep publicationProfile separate from policy bindings.
The "reuse is not a separate kind of config" rule (L19-21) is load-bearing and well-stated — it prevents a tempting trust-escalation bug where Knop-local config could promote itself to mesh scope just by being referenced.
The internal command-override-as-layer framing (L130) gives implementations a clean way to model overrides without polluting persisted mesh state.
Refusing implicit "any" in target selectors (L85-95) is the right call and will save the ontology from a slow drift toward selector ambiguity.