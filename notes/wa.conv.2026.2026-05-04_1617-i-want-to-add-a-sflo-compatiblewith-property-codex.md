---
id: i-want-to-add-a-sflocomp-2x7vkt
title: 'I want to add a sflo:compatibleWith property.'
desc: ''
created: 1777936664378
updated: 1777936664378
participants: [djradon, codex.gpt-5.5]
conversationEventKinds: [message.assistant, message.user, tool.call, tool.result]
---

# djradon_2026-05-04_1501_19

I want to add a sflo:compatibleWith property.

should its range be a HistoricalState? Do you have any design suggestions?

# gpt-5.5_2026-05-04_1501_25

I’ll read the repo guidance and nearby ontology definitions before giving design advice, since this is a modeling question and the local conventions matter.

The ontology paths from the IDE aren’t directly under this repo root, so I’m checking the workspace layout and dependency copy before answering against the actual TTL vocabulary.

I would not make `sflo:compatibleWith` range `sflo:HistoricalState` unless the property name is explicitly version-pinned.

Reason: `rdfs:range` is inferential. If you declare:

```ttl
sflo:compatibleWith rdfs:range sflo:HistoricalState .
```

then every object of `sflo:compatibleWith` is inferred to be a `HistoricalState`. That makes broad statements awkward or wrong, such as “this inventory is compatible with the fantasy rules ontology” where the target may be the governing `DigitalArtifact`, not one immutable release state.

My suggestion:

```ttl
<compatibleWith> a owl:ObjectProperty ;
  rdfs:domain rdfs:Resource ;
  rdfs:range rdfs:Resource ;
  rdfs:label "compatible with" ;
  rdfs:comment "Relates a resource to another resource it is intended to interoperate with or conform to. Use a more specific compatibility target or HistoricalState-targeting property when the claim is version-pinned." .
```

Then add a narrower property only if you need pinned compatibility:

```ttl
<compatibleWithState> a owl:ObjectProperty ;
  rdfs:subPropertyOf <compatibleWith> ;
  rdfs:domain rdfs:Resource ;
  rdfs:range <HistoricalState> ;
  rdfs:label "compatible with state" ;
  rdfs:comment "Relates a resource to a specific immutable HistoricalState it is compatible with." .
```

I’d also avoid declaring `owl:SymmetricProperty` for now. Compatibility is often directional in practice: “data compatible with schema/version X” does not necessarily mean “schema/version X compatible with data.”

Given the existing ontology already has `ArtifactResolutionTarget`, `hasTargetArtifact`, `hasRequestedTargetHistory`, and `hasRequestedTargetState`, the richer design would be a relator if you need mode/fallback/provenance:

```ttl
:inventory sflo:hasCompatibilityTarget [
  a sflo:CompatibilityTarget ;
  sflo:hasTargetArtifact :fantasyRulesOntology ;
  sflo:hasRequestedTargetState :fantasyRulesOntology_s0001 ;
  sflo:hasArtifactResolutionMode sflo:ArtifactResolutionMode/Pinned
] .
```

So my push-back: don’t overload plain `compatibleWith` with `HistoricalState` range. Use broad `compatibleWith` for simple human-facing claims, and a state-specific subproperty or relator when exact version semantics matter.

# djradon_2026-05-04_1505_37

ok. no compatibleWithState for now.

# gpt-5.5_2026-05-04_1505_42

Agreed. Add only broad `sflo:compatibleWith` for now:

```ttl
<compatibleWith> a owl:ObjectProperty ;
  rdfs:domain rdfs:Resource ;
  rdfs:range rdfs:Resource ;
  rdfs:label "compatible with" ;
  rdfs:comment "Relates a resource to another resource it is intended to interoperate with or conform to. This property does not by itself imply exact version pinning; use a more specific relation or compatibility target pattern if version-pinned compatibility is needed." .
```

That keeps the simple assertion available without prematurely forcing every compatibility target to be a `HistoricalState`.

# djradon_2026-05-04_1506_04

proceed.

# gpt-5.5_2026-05-04_1506_15

I’ll add the broad property to the core ontology and then run the relevant validation/lint path for this repo if available. First I’m checking the exact ontology file and repo state so the edit lands in the source copy, not a generated test fixture.

The ontology repo itself is clean. I’m going to make a scoped change there: add `compatibleWith` as a broad object property near the general-purpose/core vocabulary, without SHACL constraints or a state-specific subproperty.

I’m also updating the compact ontology summary and decision log because this is a vocabulary-level modeling choice, even though the code change is small.

The property and docs are in place. I’m checking syntax now with the repo’s prior Turtle validation convention, then I’ll review the diff and prepare the requested commit message text.

Added broad `sflo:compatibleWith` in [semantic-flow-core-ontology.ttl](../../../../../../ontology/semantic-flow-core-ontology.ttl:200), with `rdfs:Resource` domain/range and no `HistoricalState` pinning. I also updated [ont.summary.core.md](../../../../../../ontology/notes/ont.summary.core.md:132) and recorded the modeling decision in [ont.decision-log.md](../../../../../../ontology/notes/ont.decision-log.md:12).

Validation passed:
`riot --validate semantic-flow-core-ontology.ttl`
`git diff --check`

Reasonable commit message:

```text
ontology: add broad compatibleWith relation

- add sflo:compatibleWith as a general resource-to-resource compatibility property
- avoid HistoricalState range pinning so broad artifact, ontology, and profile compatibility claims remain valid
- document the modeling choice in the core ontology summary and decision log
```

# djradon_2026-05-04_1509_46

ok, now add compatibleWith to [gunaar.ttl](../../mesh-sidecar-fantasy-rules/examples/gunaar.ttl) targeting https://semantic-flow.github.io/mesh-sidecar-fantasy-rules/ontology/releases/v0.0.2 (if I'm not mistaken)

# gpt-5.5_2026-05-04_1509_53

I’ll verify the fantasy-rules ontology release IRI in the local files first, then add the compatibility assertion to `gunaar.ttl` using the existing style.

The local ontology file confirms `v0.0.2`, but its `owl:versionIRI` points to the concrete TTL file under `/ttl/fantasy-rules-ontology.ttl`; the shorter release resource appears via `dcterms:hasVersion`. I’ll target the release resource you named, since that matches your compatibility intent.

I’m editing only the example file, leaving the existing ontology and SHACL changes in that dependency repo untouched.

The assertion is added. I’ll validate the Turtle and check the diff for just this file.

Added `sflo:compatibleWith` to [gunaar.ttl](../../mesh-sidecar-fantasy-rules/examples/gunaar.ttl:2), targeting:

`https://semantic-flow.github.io/mesh-sidecar-fantasy-rules/ontology/releases/v0.0.2`

Validation passed with `riot --validate examples/gunaar.ttl`.

Note: the dependency repo already had unrelated edits in `ontology/fantasy-rules-ontology.ttl` and `shacl/fantasy-rules-shacl.ttl`; I left those alone.
