---
id: weave-honor-mesh-config-w4t2da
title: 'weave honor mesh config'
desc: ''
created: 1779745709470
updated: 1779745709470
participants: [codex.gpt-5.5]
conversationEventKinds: [message.assistant, message.user, tool.call, tool.result]
---

# User_2026-05-25_1134_45

weave honor mesh config

Please review [[wa.task.2026.2026-05-24_2304-honor-mesh-config]] and let me know if you have suggestions. 

Some of this task will be a review pass over the history-related terms in the ontology. It's really untested, and we should improve it without concern for backward compatibility

Can you explain how hasHistoryTrackingDefault, hasDefaultHistoryTrackingPolicy, and hasHistoryTrackingPolicy work with examples? I'm not sure we need a "default" if the default is basically hasHistoryTrackingPolicy at the application or mesh level.

Regarding open issues:
- yes, option that persists policy
- if history tracking policy is applied at the mesh level, we shouldn't have to specify it for each artifact role/type, but maybe we need additional vocabulary for "all types"?

# gpt-5.5_2026-05-25_1134_52

I’ll read the task note plus the repo guidance and product vision first, then inspect the ontology terms around history policy so the suggestions are grounded in what’s actually modeled.

I’ve got the task shape. Next I’m checking the actual SFLO vocabulary definitions and nearby docs, because the “default versus policy” distinction is exactly the kind of thing that can drift between intent and ontology.

The ontology does distinguish a global fallback from role-specific defaults, but the names make that too easy to misread. I’m checking the built-in config and parser now so I can separate “what the RDF says” from “what Weave currently does.”

I reviewed the task note and the live terms. My main suggestion: keep the concept of a config-level default, but tighten its merge semantics and clean up the confusing names. I would not add an “all artifact types” role yet.

**How The Terms Work**

`hasDefaultHistoryTrackingPolicy` is the config-level fallback:

```ttl
<> a sfcfg:MeshConfig ;
  sfcfg:hasDefaultHistoryTrackingPolicy sfcfg:historyTrackingPolicy_versioned .
```

Meaning: artifacts governed by this config should be versioned unless a more specific applicable policy says otherwise.

`hasHistoryTrackingDefault` attaches role-specific defaults:

```ttl
<> a sfcfg:ApplicationConfig ;
  sfcfg:hasDefaultHistoryTrackingPolicy sfcfg:historyTrackingPolicy_currentOnly ;
  sfcfg:hasHistoryTrackingDefault [
    a sfcfg:ArtifactRolePolicy ;
    sfcfg:hasArtifactRole sfcfg:artifactRole_payload ;
    sfcfg:hasHistoryTrackingPolicy sfcfg:historyTrackingPolicy_versioned
  ] .
```

Meaning: payload artifacts are versioned; unmentioned roles fall back to current-only.

`hasHistoryTrackingPolicy` is the actual policy value on a policy-bearing node. In practice today, Weave uses it inside `ArtifactRolePolicy` nodes, while `hasDefaultHistoryTrackingPolicy` is used directly on `Config`. See the ontology definitions in [semantic-flow-config-ontology.ttl](../../sflo/semantic-flow-config-ontology.ttl:293) and the current defaults in [defaults/application.ttl](../../../../../defaults/application.ttl:8).

I would not collapse `hasDefaultHistoryTrackingPolicy` into `hasHistoryTrackingPolicy` on `MeshConfig`. Since `ConfigArtifact` is also a `DigitalArtifact`, `<> sfcfg:hasHistoryTrackingPolicy ...` can be read as “track history for this config artifact itself,” not “this config exports a default for governed artifacts.” The “default” term earns its keep as the scoping operator.

**Recommendation**

For `weave mesh create --history-tracking-policy versioned`: yes, add the option and persist it.

But I’d persist this:

```ttl
<> a sfcfg:MeshConfig ;
  sfcfg:hasDefaultHistoryTrackingPolicy sfcfg:historyTrackingPolicy_versioned .
```

Then define merge semantics so an upper-layer default for a policy family resets lower-layer role defaults in that family. In other words, a mesh-local default should mean “this mesh’s baseline,” not “only roles the application defaults forgot about.” That gives you “history on everywhere” without writing every artifact role into `_mesh/_config/config.ttl`.

The current command override already expands across all known roles in [execution_config.ts](../../../../../src/runtime/weave/execution_config.ts:30), so I’d keep CLI override semantics as “all roles for this run.” Mesh config can be the durable equivalent, but represented compactly.

**Ontology Cleanup Suggestions**

If backward compatibility is not a constraint, I’d strongly consider renaming `hasHistoryTrackingDefault`. It is too close to `hasDefaultHistoryTrackingPolicy`. Better names:

```ttl
sfcfg:hasArtifactRoleHistoryTrackingPolicy
```

or, if `ArtifactRolePolicy` remains generic:

```ttl
sfcfg:hasArtifactRolePolicy
```

I’d also tighten `hasHistoryTrackingPolicy` so it is clearly the leaf value on an `ArtifactRolePolicy`, an exact artifact policy, or another specific policy holder. I would remove or reword the “config or role policy” comment because config-level use is exactly what is causing the ambiguity.

One more thing: `historyTrackingPolicy_required` feels like two concepts mashed together: a history mode plus an override/invariant rule. Since the config ontology already gestures at required invariants and merge profiles, I’d either split that into a separate non-overridable constraint or rename it to something unmistakably mode-like.

**Task Note Suggestions**

In the task note, I’d change the current open issue around lines 68-74 to prefer option 1: mesh-local defaults replace lower-layer role defaults for that policy family. Then explicit per-role entries become only for mesh-local exceptions, not for “all roles.”

Example:

```ttl
<> a sfcfg:MeshConfig ;
  sfcfg:hasDefaultHistoryTrackingPolicy sfcfg:historyTrackingPolicy_versioned ;
  sfcfg:hasHistoryTrackingDefault [
    a sfcfg:ArtifactRolePolicy ;
    sfcfg:hasArtifactRole sfcfg:artifactRole_runtimeMeta ;
    sfcfg:hasHistoryTrackingPolicy sfcfg:historyTrackingPolicy_currentOnly
  ] .
```

That means “version everything, except runtime metadata.”

For the ResourcePage metadata panel, I’d lean toward a built-in presentation profile variant rather than a boolean term. Same reason: it composes better with “all ordinary generated panels,” references, raw source, history, etc. A flag is convenient at CLI level, but durable config should probably select a presentation profile.

I’d add tests for:

- mesh default `versioned` makes every artifact role resolve to `versioned`
- mesh default plus one mesh role exception
- command override still wins over mesh config
- direct `hasHistoryTrackingPolicy` on `MeshConfig` is either rejected or explicitly defined
- `required` policy behavior, if kept

So: your instinct is right. We should not need to spell out every artifact role just to say “this mesh tracks history everywhere.” The cleanest path is a compact mesh-level default plus explicit family-reset merge semantics.

# User_2026-05-25_1139_57

agreed, update and suggest other open issues resolutions.

# gpt-5.5_2026-05-25_1140_08

I’ll update the task note rather than touch ontology/runtime code in this pass: lock in the mesh-level default semantics, resolve the CLI persistence question, and add a compact “proposed resolutions” pass for the remaining open issues.

I’m going to make the note more decisive: the “all roles” behavior will come from merge semantics, not from enumerating roles or adding a synthetic all-types role. I’ll also keep the remaining items as suggested resolutions where they still deserve implementation-time confirmation.

The note now has a new suggested-resolutions section and the contract/testing language has been tightened around the chosen semantics. I’m doing a quick consistency pass now for stale wording like “likely” or old all-role-output assumptions.

The file lives under the archive dependency, which looks like its own repo rather than the main Weave worktree, so the main repo status doesn’t show the edit. I’m checking that nested repo diff directly.

Updated [wa.task.2026.2026-05-24_2304-honor-mesh-config.md](wa.task.2026.2026-05-24_2304-honor-mesh-config.md:68).

Key changes:
- Locked in mesh-local default semantics: `hasDefaultHistoryTrackingPolicy` resets lower-layer role defaults for that policy family.
- Rejected explicit all-role `hasHistoryTrackingDefault` output and an “all artifact roles” vocabulary term for this slice.
- Added `Suggested Resolutions` for the remaining open issues, including presentation profile variants, `_mesh/_config/config.ttl` scope, deferring workspace/machine config, audit logging, and minimal config-resolution implementation.
- Added ontology-review guidance for clarifying/renaming `hasHistoryTrackingDefault` and deciding whether direct `hasHistoryTrackingPolicy` on `Config` should be valid.
- Updated contract, testing, non-goals, and implementation plan accordingly.

No tests run; this was documentation-only. Note: the nested `weave-dev-archive` repo also has an existing untracked Alice task note, which I left alone.

Suggested commit message for `weave-dev-archive`:

```text
docs: resolve mesh config policy direction

- define mesh-local default history policy as a policy-family reset
- document persisted mesh create history policy behavior
- prefer ResourcePage presentation variants over boolean metadata config
- narrow mesh-config scope to built-in defaults, mesh-local config, and command overrides
- add ontology review and test expectations for history-policy terms
```

# User_2026-05-25_1340_35

OK, we're headed in the right direction. Does "open issues" need updating?

# gpt-5.5_2026-05-25_1340_47

Yes. The current “Open Issues” section is now mostly a graveyard of questions we just answered, which makes the task look less settled than it is. I’ll split it into resolved design questions plus the few things that are still genuinely open.

That’s cleaner now: answered design questions are no longer masquerading as blockers. I’m doing one quick readback around the revised section so I don’t leave contradictory text behind.

Yes, and I updated it.

The note now has:
- `Resolved Questions` for the decisions we’ve settled.
- `Remaining Open Issues` only for the things still genuinely undecided: history-term naming, direct `hasHistoryTrackingPolicy` on `Config`, exact presentation profile names/panels, whether `mesh create` gets a persisted presentation-profile option, and audit/log shape.

Updated here: [wa.task.2026.2026-05-24_2304-honor-mesh-config.md](wa.task.2026.2026-05-24_2304-honor-mesh-config.md:88)

No tests run; doc-only edit.

# User_2026-05-25_1343_26

If it's a default, "hasArtifactRoleHistoryTrackingDefault" but I'm still confused why default is necessary, given the config cascade from application config -> mesh config -> knop local default/inheritable config... doesn't seem like we ne ed a separate default concept.

# gpt-5.5_2026-05-25_1343_40

You’re right to push on this. “Default” is doing two different jobs in the current vocabulary, and that’s why it feels mushy.

I think we need **one fallback concept**, but probably not two “default”-named properties.

The cascade answers: “which config layer wins?”
Application → mesh → inherited Knop → local Knop → command override.

But inside a winning/applicable config layer, we still need a way to say: “what applies when no role-specific or artifact-specific rule matches?”

That can be modeled as:

```ttl
<> a sfcfg:MeshConfig ;
  sfcfg:hasDefaultHistoryTrackingPolicy sfcfg:historyTrackingPolicy_versioned ;
  sfcfg:hasArtifactRoleHistoryTrackingPolicy [
    sfcfg:hasArtifactRole sfcfg:artifactRole_runtimeMeta ;
    sfcfg:hasHistoryTrackingPolicy sfcfg:historyTrackingPolicy_currentOnly
  ] .
```

But I agree `hasArtifactRoleHistoryTrackingDefault` is a bad name. It combines “role-specific” and “default” and invites exactly this confusion.

My current preference:

- Keep `hasDefaultHistoryTrackingPolicy` or replace it with an even clearer fallback name.
- Rename `hasHistoryTrackingDefault` to **`hasArtifactRoleHistoryTrackingPolicy`**.
- Do not put “Default” in the artifact-role property name.

So the mental model becomes:

- `hasDefaultHistoryTrackingPolicy`: fallback for this config scope.
- `hasArtifactRoleHistoryTrackingPolicy`: role-specific policy in this config scope.
- `hasHistoryTrackingPolicy`: the actual policy value on a policy node.

If we want to be more aggressive, we could also rename the fallback to:

```ttl
sfcfg:hasFallbackHistoryTrackingPolicy
```

That may be clearer than “default” because it distinguishes “fallback within this config scope” from “inherited default from lower config layers.”

So I’d revise the note away from `hasArtifactRoleHistoryTrackingDefault` and toward `hasArtifactRoleHistoryTrackingPolicy`, with maybe a stronger suggestion that `hasDefaultHistoryTrackingPolicy` should be considered for rename to `hasFallbackHistoryTrackingPolicy`.

# User_2026-05-25_1344_58

"we still need a way to say: “what applies when no role-specific or artifact-specific rule matches?”" -- Application just needs to catch everything, so if it's not defined somewhere else, application always provides the fallback.

# gpt-5.5_2026-05-25_1345_54

Yes. That’s the cleaner model.

If application config is required to provide the total fallback, then mesh/Knop config does **not** need a “default” to guarantee a value exists. What mesh config needs is a **scope-wide policy override**:

```ttl
<> a sfcfg:MeshConfig ;
  sfcfg:hasHistoryTrackingPolicy sfcfg:historyTrackingPolicy_versioned .
```

Meaning: for this config scope, apply `versioned` to governed artifacts unless a more specific policy in the same or later layer wins.

Then role-specific policy becomes:

```ttl
<> a sfcfg:MeshConfig ;
  sfcfg:hasArtifactRoleHistoryTrackingPolicy [
    a sfcfg:ArtifactRolePolicy ;
    sfcfg:hasArtifactRole sfcfg:artifactRole_runtimeMeta ;
    sfcfg:hasHistoryTrackingPolicy sfcfg:historyTrackingPolicy_currentOnly
  ] .
```

So I’d revise my recommendation:

- Drop `hasDefaultHistoryTrackingPolicy`.
- Rename `hasHistoryTrackingDefault` to `hasArtifactRoleHistoryTrackingPolicy`.
- Let `hasHistoryTrackingPolicy` on `Config` mean “scope-wide history tracking policy for artifacts governed by this config,” not “policy for the config artifact itself.”
- If we need to control history of the config artifact as a `DigitalArtifact`, that should be expressed via its artifact role (`artifactRole_config`) or an artifact-specific policy shape, not by overloading the config node.

That gives a three-level policy shape:

```text
scope-wide policy
role-specific policy
artifact-specific policy, later
```

And the cascade handles layer precedence. Much cleaner. The word “default” can go away from this family entirely, except perhaps in prose when describing the application config’s catch-all behavior.

# User_2026-05-25_1348_29

OH I think I'm starting to see... is the issue that we want to separate history tracking policy that applies to config vs applies to the things governed by the config? 

Remember, no blank nodes. Give it a name with a hash identifier if it doesn't merit its own identifier.

# gpt-5.5_2026-05-25_1348_45

Yes, that’s the crux. I’m going to update the note to name that ambiguity directly, remove the blank-node examples, and steer the vocabulary toward named policy nodes with fragment IRIs.

I agree with your read: the ambiguity is not really about “defaults,” it’s about whether a policy triple is about the config artifact itself or about artifacts governed by that config. I’m going to revise the task note around that, using named fragment policy nodes instead of blank nodes.

I’ve swapped the examples to named fragment resources and made the policy ambiguity explicit. I’m checking for leftover “default history” phrasing now, because this is exactly where stale wording can keep the confusion alive.

Yes, exactly. The ambiguity is: “policy for this config artifact” versus “policy this config applies to governed artifacts.” That’s the thing the ontology needs to separate.

I updated the task note to say that directly, and revised the examples to avoid RDF blank nodes. The candidate direction is now:

```ttl
<> a sfcfg:MeshConfig ;
  sfcfg:hasGovernedArtifactHistoryTrackingPolicy sfcfg:historyTrackingPolicy_versioned ;
  sfcfg:hasArtifactRoleHistoryTrackingPolicy <#runtime-meta-history-policy> .

<#runtime-meta-history-policy> a sfcfg:ArtifactRolePolicy ;
  sfcfg:hasArtifactRole sfcfg:artifactRole_runtimeMeta ;
  sfcfg:hasHistoryTrackingPolicy sfcfg:historyTrackingPolicy_currentOnly .
```

I also changed the task language away from “default” for mesh-local history policy and toward “governed-artifact policy.” Application config can still provide the catch-all fallback; mesh config supplies a scope-wide override. No tests run, doc-only edit.

# User_2026-05-25_1350_36

Maybe we just disallow direct policy assignments to config, and always require a context. Like a relator that defines the policy, and another relator that links the policy relator and a context relator.

# gpt-5.5_2026-05-25_1350_53

Yes, I think that’s the cleanest direction.

I’d phrase it as: **no direct operational policy properties on `Config`**. A config contains policy assignments. Each assignment has:

1. the policy value
2. the applicability context

I’d make the assignment itself the main relator, and only split out context into a named node when context has structure or reuse value.

Example:

```ttl
<> a sfcfg:MeshConfig ;
  sfcfg:hasPolicyAssignment <#history-versioned-for-governed-artifacts>,
    <#runtime-meta-current-only> .

<#history-versioned-for-governed-artifacts>
  a sfcfg:HistoryTrackingPolicyAssignment ;
  sfcfg:hasHistoryTrackingPolicy sfcfg:historyTrackingPolicy_versioned ;
  sfcfg:appliesToPolicyContext <#governed-artifacts> .

<#governed-artifacts>
  a sfcfg:GovernedArtifactPolicyContext .

<#runtime-meta-current-only>
  a sfcfg:HistoryTrackingPolicyAssignment ;
  sfcfg:hasHistoryTrackingPolicy sfcfg:historyTrackingPolicy_currentOnly ;
  sfcfg:appliesToPolicyContext <#runtime-meta-artifacts> .

<#runtime-meta-artifacts>
  a sfcfg:ArtifactRolePolicyContext ;
  sfcfg:hasArtifactRole sfcfg:artifactRole_runtimeMeta .
```

This resolves the ambiguity neatly:

- The config artifact’s own history policy would need a context like `sfcfg:ThisConfigArtifactPolicyContext` or an artifact-specific context.
- Governed artifacts use `GovernedArtifactPolicyContext`.
- Role-specific rules use `ArtifactRolePolicyContext`.
- Future class/path/Knop/artifact-specific rules can add new context types without inventing a new property for each case.

Small pushback: I probably would not use two relators unless we truly need reusable policy definitions independent of application. A `HistoryTrackingPolicyAssignment` that links policy value + context is already the policy relator. The context can be its own named resource. That keeps the graph understandable without building a policy meta-framework too early.

This also means we can retire/avoid all of these as direct config properties:

```ttl
sfcfg:hasDefaultHistoryTrackingPolicy
sfcfg:hasGovernedArtifactHistoryTrackingPolicy
sfcfg:hasArtifactRoleHistoryTrackingPolicy
```

And replace them with one pattern:

```ttl
sfcfg:hasPolicyAssignment
```

That feels much more future-proof.

# User_2026-05-25_1353_28

It does seem like reusable policy could be useful. 

And what about resolving conflicting policies. Seems like we need an integer to establish priority, and if not specified the application assumes priority 0. We're using that somewhere else.

# gpt-5.5_2026-05-25_1353_35

That points to a more durable policy model: reusable policy definitions plus assignments/bindings with context and priority. I’ll check the existing vocabulary for the priority/order pattern you mentioned, then fold this into the task note so it doesn’t stay as vapor in chat.

I like that, with one guardrail: priority should resolve conflicts among applicable policy bindings, but it shouldn’t accidentally let application defaults beat explicit mesh policy across the cascade. I’ll update the note toward a reusable policy definition plus policy binding/context model, with missing priority treated as `0` and equal-priority conflicts failing closed.

The note now uses a reusable `PolicyDefinition` plus `PolicyBinding` plus `PolicyContext` shape. I’m tightening the examples to make priority an actual typed integer and checking for stale direct-policy wording.

# User_2026-05-25_1354_43

Couldn't config artifacts be governed under GovernedArtifactPolicyContext without needing their own context?

# gpt-5.5_2026-05-25_1354_51

Yes, that’s a good simplification. I’ll tweak the note so `GovernedArtifactPolicyContext` explicitly includes config artifacts when they are governed artifacts; a separate context is only needed for “this exact config artifact” or another narrower target.

Good catch. I’m going to sharpen that wording: `GovernedArtifactPolicyContext` should include governed config artifacts too; the problem is direct, context-free policy assertions on a `Config`, not that config artifacts require a special context.

Yes. That’s the better formulation.

I updated the note so `GovernedArtifactPolicyContext` explicitly includes governed config artifacts. A separate context is only needed when we want to target something narrower, like:

- all config artifacts: `ArtifactRolePolicyContext` with `artifactRole_config`
- one exact config artifact: future `ArtifactPolicyContext`

The key distinction is now **direct policy properties on `Config` vs contextual policy bindings carried by a config**, not “config artifacts need a separate world.” That feels much less twisty.

# User_2026-05-25_1356_31

It's a good question whether mesh policy can beat explicit policy, but for now yes, let's keep the decision per level... all knop-level beats any mesh level, etc.

# gpt-5.5_2026-05-25_1356_34

Agreed. I’ll record that as a resolved precedence rule: layer wins first, then priority/context only resolves within the winning layer. That keeps the first implementation deterministic without needing a cross-layer “explicitness” debate yet.

Recorded that decision in the task note.

The policy resolution model now says:
- Resolve by config layer first.
- Higher layer wins even if lower layer has a more explicit context.
- Within one layer and policy family, `sfcfg:policyPriority` resolves conflicts.
- Missing priority is integer `0`.
- Equal-priority conflicting values fail closed.

I also added test expectations for that exact edge case: Knop-local governed-artifact policy beats mesh role-specific policy, and mesh governed-artifact policy beats application role-specific policy.

# User_2026-05-25_1359_19

Update that open issues.

- Regarding the naming, seems like too many classes. Maybe justified, but couldn't we just have one TrackingPolicyDefinition relator class, and specify whether history (or any other type) is included?

# gpt-5.5_2026-05-25_1359_33

That naming pressure is real: a class per policy family will get chunky fast. I’ll update the note so the open issue now leans toward a generic `TrackingPolicyDefinition` with explicit included policy dimensions, while still leaving exact term names open for the ontology pass.

I’m going to reflect that simplification in the examples and the open issue: one reusable `TrackingPolicyDefinition`, tagged with a tracking kind like history, instead of a separate definition class per policy family.

Updated the note in that direction.

Changes made:
- Replaced `sfcfg:HistoryTrackingPolicyDefinition` in the examples with `sfcfg:TrackingPolicyDefinition`.
- Added `sfcfg:hasTrackingPolicyKind sfcfg:trackingPolicyKind_history` to show how the generic policy definition says what kind of tracking policy it includes.
- Updated the open issue to prefer a smaller class set:
  - `PolicyBinding`
  - `TrackingPolicyDefinition`
  - `GovernedArtifactPolicyContext`
  - `ArtifactRolePolicyContext`
  - `ArtifactPolicyContext`

I think this is the better modeling instinct. Separate definition classes per policy family would probably multiply faster than they clarify.

# User_2026-05-25_1405_32

So can we delete the "Finalize the history-policy" and "whether direct on config" open issues?

What's this about the built-in presentation profile names. "semantic-site-publication" seems bad because everything is for publication. Maybe just "all panels" and "no panels" are built-in?

mesh create should expose a profile option. and really we should have API and CLI surfaces for changing the presentation profile or the history profile. 

Your minimum suggestion for audit/logging is good.

# gpt-5.5_2026-05-25_1405_44

Yep. Those are no longer open in the useful sense; the note should stop pretending they are. I’ll convert the presentation-profile and CLI/API surface points into decisions/contract changes, replace `semantic-site-publication`, and leave only genuinely unsettled vocabulary naming where needed.

I’m going to make the open-issues section match the decisions you just made: no more “should direct config policy be allowed,” no more publication-profile name bikeshed, and the API/CLI profile surface becomes part of the contract.

The open issues are much slimmer now. I’m doing a quick sweep for stale `semantic-site-publication`, old direct-config-policy open questions, and audit wording that should now be resolved.

Updated.

Open issues now only include:
- exact reusable policy model term names
- exact API/CLI command names for changing durable history and presentation profiles after mesh creation

I also changed the note to:
- remove `semantic-site-publication`
- use `semantic-site-all-panels` for the SFLO desired config
- add `semantic-site-no-panels`
- record that `mesh create` should expose a persisted presentation/profile option
- record that API and CLI surfaces should exist for changing durable history and ResourcePage presentation profiles
- move the audit/logging minimum into resolved questions

No tests run; doc-only edit.

# User_2026-05-25_1419_20

thanks. new conversation for implementation?

# gpt-5.5_2026-05-25_1419_27

Yes, new conversation is the right move for implementation.

This one did the modeling/design refinement, and the task note now has enough settled direction that implementation can start cleanly. In the new thread, I’d start with:

“Implement `weave honor mesh config` from `wa.task.2026.2026-05-24_2304-honor-mesh-config.md`. Read product vision and `wd.general-guidance` first.”

I’d expect the first implementation slice to be:
1. ontology/config parser adjustment for the policy binding shape
2. mesh-root-aware effective config loading from `_mesh/_config/config.ttl`
3. history policy honored from mesh config
4. focused tests around precedence and priority

Then ResourcePage profile work can be second slice if it starts to sprawl.
