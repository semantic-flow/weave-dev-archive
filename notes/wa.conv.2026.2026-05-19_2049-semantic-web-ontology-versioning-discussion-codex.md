---
id: semantic-web-ontology-ve-6z3108
title: 'semantic web ontology versioning discussion'
desc: ''
created: 1779248962435
updated: 1779248962435
participants: [codex.gpt-5.5]
conversationEventKinds: [message.assistant, message.user, tool.call, tool.result]
---

# User_2026-05-19_2044_34

semantic web ontology versioning discussion

I feel like prefix declarations are the only common way to specify which version of an ontology a given RDF file uses. But most terms will perserve through ontologies, and there's no mechanism that I know of to indicate that sflo:hasManifestation from https://semantic-flow.github.io/sflo/ontology/releases/v0.1.0 has the same meaning as the one from https://semantic-flow.github.io/sflo/ontology/releases/v0.1.1

You could have a bit of metadata in the file, like "compatibleWith <https://semantic-flow.github.io/sflo/ontology/releases/v0.1.0>" but what would the subject be? Alice is compatible with anything. 

You could keep ontology compatibility in SF metadata, but that's not very portable. 

Basically the conclusion I've come to is that ontologies can have no breaking changes. Once a term is minted, its meaning must stay the same. It can be dropped from an ontology perhaps, and it can be mapped with something like a supersededBy predicate. But there might still be extant use.

You see this with all the big ontologies with version numbers in their IRIs: those version numbers never change.

Can you call it ontological backward-compatibility? i.e., once a term is published, maybe you can tweak its meaning, but it never goes away. 

I guess at least you can use SHACL to say which combinations of terms are legitimate.

# gpt-5.5_2026-05-19_2044_47

I’ll ground this in your repo’s own ontology guidance first, then I’ll answer the versioning question directly. This is one of those places where the standards give you vocabulary, but the real design choice is social and operational.

I found the repo guidance: it explicitly says the ontology is key, and also notes that until v1.0 the project is avoiding backward-compatibility shims. The open-tab path appears to be from the workspace root rather than this repo root, so I’m checking the actual checked-in ontology copy now.

The local ontology already has a general `sflo:compatibleWith`, but with deliberately broad domain/range. That broadness is useful for artifact compatibility, less useful for the very specific question “which term contract am I relying on?”

I think you’re basically right, with one important correction: prefix declarations are not really ontology-version metadata. They are just Turtle syntax. After parsing, only absolute IRIs remain.

In current SFLO, this is good news. The term is not:

```text
https://semantic-flow.github.io/sflo/ontology/releases/v0.1.0/hasManifestation
```

It is:

```text
https://semantic-flow.github.io/sflo/ontology/hasManifestation
```

because the ontology uses the stable base namespace in [semantic-flow-core-ontology.ttl](../../sflo/semantic-flow-core-ontology.ttl:1), while the release IRI identifies a historical ontology state in the metadata around [line 24](../../sflo/semantic-flow-core-ontology.ttl:24). So “`sflo:hasManifestation` from v0.1.0” really means “the definition of the same term as published in ontology document v0.1.0.”

That leads to the rule I’d use:

> Term IRIs are stable contracts. Ontology versions are snapshots of the published contract text and axioms.

So yes: call it **ontology backward compatibility**, **semantic backward compatibility**, or **term IRI stability**. “Ontological backward-compatibility” is understandable, though I’d probably use “backward-compatible ontology evolution” in docs because it sounds less metaphysical.

For your “what is the subject?” question: the subject is not Alice. It is the RDF document, named graph, dataset, artifact, or historical state that contains/describes Alice.

For portable RDF:

```turtle
<https://example.org/alice/bio.ttl>
  a dcat:Dataset, sflo:RdfDocument ;
  dcterms:conformsTo <https://semantic-flow.github.io/sflo/ontology/releases/v0.1.0> .
```

For Semantic Flow metadata, the subject could be a `DigitalArtifact`, `HistoricalState`, or `LocatedFile`:

```turtle
<https://example.org/alice/bio/releases/v1.2.0>
  a sflo:HistoricalState ;
  dcterms:conformsTo <https://semantic-flow.github.io/sflo/ontology/releases/v0.1.0> ;
  sflo:compatibleWith <https://semantic-flow.github.io/sflo/ontology/releases/v0.1.0> .
```

I’d prefer `dcterms:conformsTo` when making a real conformance claim. `sflo:compatibleWith` is useful when the claim is looser: “intended to interoperate with,” not necessarily “validated against.”

The strongest policy is:

- Never change the intended meaning of an existing public term in a breaking way.
- Add new terms freely.
- Clarify comments carefully.
- Deprecate instead of deleting.
- Keep old term pages dereferenceable.
- Use `owl:deprecated true`, `dcterms:isReplacedBy`, `schema:supersededBy`, `owl:equivalentClass`, or `owl:equivalentProperty` where appropriate.
- Treat SHACL as the place to version stricter “valid combinations of terms.”

The subtle pushback: “no breaking changes” is not just about not deleting terms. Adding an `rdfs:range`, making a property functional, adding disjointness, narrowing a SHACL shape, or changing a class hierarchy can break consumers too. So the compatibility promise should cover meaning, entailments, and validation profile separately.

For SFLO specifically, the existing docs already make the right distinction: Semantic Flow’s release identity is the `HistoricalState`, while OWL’s `owl:versionIRI` should point to fetchable versioned bytes [ont.use-case.dereferenceable-ontology.md](../../sflo/notes/ont.use-case.dereferenceable-ontology.md:59). That gives you a clean model: stable term IRIs, versioned ontology documents, and versioned SHACL graphs for constraint regimes.

# User_2026-05-19_2049_13

maybe our ontology metadata should have a dcat:previousVersion for each HistoricalState so if we ever drop a term, they could trace back for meaning. Kinda scraping the barrel here.

# gpt-5.5_2026-05-19_2050_01

Yeah, I think that is worth doing. Not barrel-scraping, more like adding a boring little spine that future tools can follow.

`dcat:previousVersion` is real DCAT 3 vocabulary: it is for a chain of version snapshots, is equivalent to `pav:previousVersion`, and is a subproperty of `prov:wasRevisionOf` in the W3C spec. That fits SFLO `HistoricalState`s almost exactly.

I’d model it like this:

```turtle
<https://semantic-flow.github.io/sflo/ontology>
  dcat:hasVersion <https://semantic-flow.github.io/sflo/ontology/releases/v0.1.0>,
                  <https://semantic-flow.github.io/sflo/ontology/releases/v0.1.1> ;
  dcat:currentVersion <https://semantic-flow.github.io/sflo/ontology/releases/v0.1.1> .

<https://semantic-flow.github.io/sflo/ontology/releases/v0.1.1>
  a owl:Ontology, sflo:HistoricalState ;
  dcat:previousVersion <https://semantic-flow.github.io/sflo/ontology/releases/v0.1.0> ;
  sflo:previousHistoricalState <https://semantic-flow.github.io/sflo/ontology/releases/v0.1.0> ;
  dcat:version "0.1.1" .
```

And I’d probably make `sflo:previousHistoricalState` a subproperty of `dcat:previousVersion`, or assert both. Current SFLO already has the SF-specific version chain property in [semantic-flow-core-ontology.ttl](../../sflo/semantic-flow-core-ontology.ttl:104), but `dcat:previousVersion` is better for portable discovery.

The one caveat: this helps someone trace historical ontology documents, but it does not fully solve dropped term meaning. For that, the stable term IRI still needs to remain dereferenceable, ideally with deprecation/replacement metadata. The version chain is the fallback audit trail; the term page is the humane answer.

So my recommendation:

- Use `dcat:hasVersion` / `dcat:currentVersion` on the stable ontology artifact.
- Use `dcat:previousVersion` on each release `HistoricalState`.
- Keep `sflo:previousHistoricalState` if it carries SF-specific constraints or local clarity.
- Never let old term IRIs 404; mark them deprecated/superseded instead.

Source: W3C DCAT 3 defines `dcat:previousVersion`, `dcat:hasVersion`, `dcat:currentVersion`, and `dcat:version` for version chains: https://www.w3.org/TR/vocab-dcat-3/

# User_2026-05-19_2057_21

I wanted to avoid dcat:currentVersion because loading two different version, could both say currentVersion. I think we can solve this generally with a relator that specifies a validity period, or rather a validity start, but you'd have to discover the next version to know whether validity has ended. You could also find a version/state which isn't the object of a dcat:previousVersion statement, but the open-world assumption is hard.

One thing I think we should do is link SHACL versions/states to their corresponding ontology versions/states. Right?

# gpt-5.5_2026-05-19_2057_33

You’re teasing out the thing RDF is bad at by default: time-varying “current” facts look like ordinary eternal triples unless you deliberately put them somewhere contextual. I’ll answer this as a modeling choice, not a code edit.

# gpt-5.5_2026-05-19_2058_26

Yes. I’d absolutely link SHACL states to ontology states.

And I agree with avoiding `dcat:currentVersion` inside immutable release files. That triple is temporally fragile. If every old release snapshot says “I was current when published,” then union-loading old and new snapshots gives you multiple “current” versions. RDF has no built-in “this triple was current as of graph publication time” unless you preserve graph context.

The clean split is:

- Immutable release/state docs say: `dcat:previousVersion`, `dcterms:issued`, `owl:versionInfo`, `dcat:version`.
- Mutable catalog/current inventory says: `dcat:currentVersion`.
- Consumers who load immutable snapshots alone never get stale “current” claims.

A validity relator is possible, but I’d keep that out unless there’s a real consumer. Also, “validity” is a little slippery here: v0.1.0 does not stop being valid when v0.1.1 is released; it only stops being current. So I’d model currentness as publication status, not semantic validity.

For SHACL-to-ontology linkage, I’d use either `prof:isProfileOf` or `dcterms:requires`, depending on how formal you want to be.

A good simple pattern:

```turtle
@prefix dcat: <http://www.w3.org/ns/dcat#> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix prof: <http://www.w3.org/ns/dx/prof/> .
@prefix sh: <http://www.w3.org/ns/shacl#> .

<https://semantic-flow.github.io/sflo/ontology/shacl/releases/v0.1.1>
  a sh:ShapesGraph, prof:Profile, sflo:HistoricalState ;
  dcat:version "0.1.1" ;
  dcat:previousVersion <https://semantic-flow.github.io/sflo/ontology/shacl/releases/v0.1.0> ;
  prof:isProfileOf <https://semantic-flow.github.io/sflo/ontology/releases/v0.1.1> ;
  dcterms:requires <https://semantic-flow.github.io/sflo/ontology/releases/v0.1.1> .
```

I’d be careful with `dcterms:conformsTo`: data conforms to a SHACL profile; the SHACL graph does not really “conform to” the ontology. The SHACL graph profiles, constrains, or requires the ontology.

If you want the strict PROF model, the “profile” can be a separate resource and the SHACL Turtle file can be a `prof:ResourceDescriptor` artifact. But for SFLO’s current lightweight release metadata, typing the SHACL state as a `prof:Profile` is probably good enough.

Sources: DCAT 3 version properties: https://www.w3.org/TR/vocab-dcat-3/ ; W3C PROF `prof:isProfileOf`: https://www.w3.org/TR/dx-prof/

# User_2026-05-19_2101_09

one problem with dcat:previousVersion for Semantic Flow is we now have multiple histories. I guess we can gloss previousVersion to mean "previous version in this history" and if we needed previousChronologicalVersion/previousAbsoluteVersion we could invent it.

# gpt-5.5_2026-05-19_2101_32

Yes, I think that gloss is defensible.

`dcat:previousVersion` should mean **previous version in the same lineage**, not “previous thing this project published.” That is very close to DCAT’s own wording: a version chain of snapshots of a resource. In Semantic Flow terms, the “resource” being versioned is not only the `DigitalArtifact`; it can be the `DigitalArtifact` within a particular `ArtifactHistory`.

So:

```turtle
:ontology/releases/v0.1.1
  sflo:previousHistoricalState :ontology/releases/v0.1.0 ;
  dcat:previousVersion :ontology/releases/v0.1.0 .
```

means:

> previous historical state in this artifact history.

If a `DigitalArtifact` has multiple histories, each history gets its own independent `dcat:previousVersion` chain. That is fine as long as the history membership is discoverable:

```turtle
:ontology/releases
  sflo:hasHistoricalState :ontology/releases/v0.1.0,
                          :ontology/releases/v0.1.1 .

:ontology/experimental
  sflo:hasHistoricalState :ontology/experimental/foo,
                          :ontology/experimental/bar .
```

I’d probably make the SFLO rule explicit:

> `dcat:previousVersion` on an `sflo:HistoricalState` denotes the previous state within the same `sflo:ArtifactHistory`.

And then SHACL can warn/error if a state points to a previous version outside any shared `ArtifactHistory`.

I would not rush to add `previousChronologicalVersion`. Chronology across histories is a different relation: it’s publication sequencing, not version lineage. If it ever matters, I’d name it something like `previousPublishedState` or model a separate release/event/catalog sequence. “Absolute version” feels like it will lie the moment there are backports, prereleases, branch histories, republished bytes, or sidecar artifacts moving on different schedules.

So the compact policy:

- `sflo:previousHistoricalState`: Semantic Flow lineage edge.
- `dcat:previousVersion`: portable version-chain edge, same-history semantics.
- `dcat:prev`: maybe for ordered collections/series, not version lineage.
- Future project-wide chronology belongs in a release index or publication event model, not in the state lineage itself.

# User_2026-05-19_2107_30

previousHistoricalState is already subproperty rdfs:subPropertyOf prov:wasRevisionOf, we could also make it subproperty dcat:previousVersion.

# gpt-5.5_2026-05-19_2107_47

Yes, that’s a good move.

Since DCAT 3 defines `dcat:previousVersion` as a subproperty of `prov:wasRevisionOf`, this is a tighter, more portable version of what `sflo:previousHistoricalState` already means. You can safely give it both superproperties:

```turtle
<previousHistoricalState> a owl:ObjectProperty, owl:FunctionalProperty ;
  rdfs:subPropertyOf dcat:previousVersion ;
  rdfs:subPropertyOf prov:wasRevisionOf ;
  rdfs:domain <HistoricalState> ;
  rdfs:range <HistoricalState> ;
  rdfs:label "previous historical state" ;
  rdfs:comment "Links a HistoricalState to the immediately preceding HistoricalState in its ArtifactHistory." .
```

Strictly, the `prov:wasRevisionOf` line becomes redundant by inference, but I’d keep it. It makes the provenance intent obvious even to tooling or readers that don’t load the DCAT ontology.

The semantic gloss should be:

> `sflo:previousHistoricalState` is the previous `sflo:HistoricalState` in the same `sflo:ArtifactHistory`; it is also a `dcat:previousVersion` relation for portable version-chain discovery.

One small caution: because `dcat:previousVersion` is a generic version-chain property, SFLO’s stronger same-history rule should live in the SFLO property comment and SHACL. The ontology can say “this is a previous version”; SHACL can say “and for Semantic Flow, it must be in the same history.”