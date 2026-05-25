---
id: vnlq32ahptl195aysm430vk
title: 2026 05 25_1542 Honor Mesh Config Claude
desc: ''
updated: 1779748956804
created: 1779748954462
---

Things I'd tighten before implementation
1. "Applicable" is doing a lot of work in the precedence rule. The Discussion (~L120) says "a higher layer's applicable policy wins even when a lower layer has a more explicit context," and the Contract Changes (L170) says "an upper-layer applicable policy binding resets lower-layer applicable bindings for that same policy family." Read together this is ambiguous about whether a mesh-local role-specific binding (e.g., runtimeMeta → metadataOnly) resets application's payload binding. It shouldn't — different contexts cover different artifacts. Recommend pinning the rule per-artifact, not per-family: "An upper-layer binding whose context covers artifact A masks lower-layer bindings whose context also covers A." Then governed-artifact-context naturally beats lower-layer role-context for every artifact, and a mesh-local role-only binding leaves untouched artifacts on app defaults.

2. Define "governed." The doc asserts that config artifacts governed by a mesh are included in sfcfg:GovernedArtifactPolicyContext (L79), but never says what makes an artifact governed. Is it strictly "lives under meshRoot"? Does it include reusable config referenced by IRI but not co-located? Does it include the _mesh metadata itself? One sentence in Discussion would prevent future arguments.

3. The "disallow direct tracking policy on Config" scope is inconsistent. Summary (L21) reads as a global rule; the Testing section (L187) restricts it to MeshConfig. The current defaults/application.ttl:6-29 relies heavily on direct hasDefaultHistoryTrackingPolicy + hasHistoryTrackingDefault on <application>. Either commit to migrating ApplicationConfig too (bigger blast radius, requires updating the parser at src/runtime/config/effective_config.ts) or scope the prohibition to MeshConfig and say so explicitly.

4. Within-layer ambiguity will bite authors. Default priority 0 + fail-closed on equal-priority conflicts is correct, but if a mesh authors both a GovernedArtifactPolicyContext binding and an ArtifactRolePolicyContext binding for payload, both at default priority, the config fails closed. That's the intended shape for runtime-meta exceptions per the L86-L116 example (priority 10). Make this an explicit authoring rule in the task: "Mesh-local artifact-role bindings that coexist with a governed-artifact binding MUST set a non-zero priority." And make the error message reference the conflicting context IRIs by name.

5. The new entry-point signature contradicts a resolved decision. L46 proposes loadEffectiveConfigForExecution({ meshRoot, historyTrackingPolicyOverride, includeSemanticFlowMetadataOverride }), but the Resolved Questions (L133-L135) commit to representing metadata-panel inclusion through a presentation profile, not a boolean. Either drop includeSemanticFlowMetadataOverride from the entry point (and let the CLI flag swap in a presentation profile override layer) or rename it to something like resourcePagePresentationProfileOverride. Otherwise the CLI flag survives as a boolean and the "supported profile variant" decision is half-applied.

6. semantic-site-all-panels quietly does two jobs. It selects all panels and eliminates the panelDataRequirement_semanticFlowMetadataOptIn gate on the metadata panel (see defaults/application.ttl:153-158). Worth saying explicitly — otherwise reviewers reading "all panels" will assume the opt-in gate still suppresses the metadata panel even under that profile.

7. semantic-site-no-panels needs one more sentence. "Suppress generated panels while keeping the page shell available" (L135) is the right intent, but list what survives: outer/inner template + title/breadcrumbs (if those are template concerns rather than panel selections). Otherwise the test in L189 is under-specified.

8. Naming policies don't fit the binding model. L188-L189 mention overriding naming policies (history/state/manifestation), but those are singletons set directly on Config — they're not contextual. The task should say explicitly that the policy-binding/context redesign applies only to tracking + generation policies, and naming policies remain direct properties merged by layer precedence. Otherwise the merge semantics gets blurred.

Sizing concern
This task bundles three substantial pieces of work: (a) mesh-config loading + layer merge, (b) the ontology redesign for policy bindings/contexts, and (c) new presentation profiles + the metadata-panel hook. (b) is the biggest unknown — adding PolicyBinding, TrackingPolicyDefinition, GovernedArtifactPolicyContext, ArtifactRolePolicyContext etc. to sfcfg, updating the parser, and writing new merge code. Worth considering splitting: a first pass that honors _mesh/_config/config.ttl under the existing hasHistoryTrackingDefault shape — unblocking the SFLO replay's --history-tracking-policy and --include-semantic-flow-metadata repetition — then a second pass for the binding/context redesign. The doc currently couples them, which means SFLO has to wait for the ontology work to land.

Smaller nits
L141 audit-log requirement is vague — say where it lands (runtimeMeta? stderr? both?).
Open Issue L147 names "exact CLI command names" but those will gate the implementation; recommend deciding them in this task rather than deferring (e.g., weave mesh set-history-tracking-policy, weave mesh set-resource-page-presentation).
The two PolicyContext classes (GovernedArtifactPolicyContext, ArtifactRolePolicyContext) might benefit from a common sfcfg:PolicyContext superclass for SHACL — worth a line in Remaining Open Issues.
Implementation Plan should add an explicit "decide governed-by semantics" item before the merge-code item, since (2) above is load-bearing for the parser.