# Candidate References for the FOMI/FOIS Semantic Flow Papers

Review scope: surveyed the public-notes vault's 644 `ar.*` notes with keyword and backlink passes around Semantic Web publication, ontology versioning, provenance, artifacts, static linked data publication, Git-backed RDF collaboration, SHACL, and ontology engineering. I also checked the topic/product notes most likely to point into the relevant literature, especially [[t.cs.semantic-web.versioning]], [[t.cs.semantic-web.temporal]], [[t.cs.semantic-web.ontology.modular]], [[t.cs.semantic-web.iri]], [[prdct.jekyll-rdf]], [[prdct.quit]], [[prdct.provencance-authoring-versioning-ontology-pav]], [[prdct.functional-requirements-for-information-resources-frir]], [[prdct.dcat-data-catalog.application-profiles]], [[prdct.shacl]], and [[prdct.ontology-development-kit]].

Working position: the paper should not pretend Semantic Flow invented dereferenceable identifiers, static linked data publishing, ontology versioning, provenance, or Git-backed RDF collaboration. The stronger story is that the Semantic Flow framework integrates these research lines into a byte-grounded publication model centered on governed digital artifacts, explicit histories, source bindings, and generated ResourcePages. Weave is the filesystem-native reference implementation used to demonstrate that model.

## Citation Strategy

For each 5-9 page paper, cite a tight spine in the paper body and keep this file as the richer reserve. Each paper probably wants about 8-10 sharp references, not a full literature-review bibliography. The references should be doing work:

- positioning `DigitalArtifact`, `ArtifactHistory`, `HistoricalState`, `ArtifactManifestation`, and `LocatedFile` against prior work on information resources, WEMI/FRIR, provenance, and Web identifiers without implying that Semantic Flow implements the FRBR/WEMI stack
- explaining Semantic Flow's sparse/skip-level properties as a deliberate modeling affordance: publishers can materialize the full artifact/state/manifestation/file chain when it carries evidence, or use direct coordinate links when intermediate layers would not help
- showing that ontology publication problems are known: URI design, content negotiation, documentation, metadata, versioning, and FAIR vocabulary publication
- positioning Semantic Flow as a model/framework in a known practice space: Git-based and CI/CD-oriented ontology/RDF workflows, static Linked Data publication, and collaborative ontology engineering; positioning Weave as the implementation/demo vehicle for the FOIS paper and as implementation evidence for the FOMI paper
- keeping Semantic Flow vocabulary legible by explaining local terms when first used, then citing the nearest prior art

## Recommended Citation Spine

Use these first unless a paper's emphasis changes. The FOMI paper will probably use the FRIR/WEMI/versioning half more heavily; the FOIS demo paper will probably use the static-publication/tooling half more heavily. The first ten are the likely shared set; the later items are reserves for emphasis shifts.

- [[ar.functional-requirements-for-information-resource-provenance-on-the-web]] - Key touchpoint for the DigitalArtifact model, but not the model itself. FRIR extends FRBR/WEMI to electronic information resources and distinguishes work/resource, expression/content, manifestation/bytes, and item/copy/location while integrating with PROV-O. Semantic Flow uses this as contrast: it keeps the concrete byte/file/provenance concerns, leaves expression/work modeling to the publisher, and adds an explicit temporal `HistoricalState` layer for artifact publication.
- [[ar.works-expressions-manifestations-items-an-ontology]] - Useful companion to FRIR because OpenWEMI relaxes overly rigid FRBR assumptions. Use when explaining why Semantic Flow avoids committing each DigitalArtifact to a WEMI role: a DigitalArtifact may be modeled as an expression of something more abstract by a publisher, but Semantic Flow itself does not require that interpretation.
- [[ar.w3.design-issues.linked-data]] - Foundational Linked Data rules: HTTP URIs, dereferenceability, useful RDF, links to other URIs. Cite early in introduction/motivation.
- [[ar.w3.cool-uris]] - Use for the public identifier and information-resource problem, especially if explaining why Weave treats identifiers, pages, and bytes as related but distinct.
- [[ar.best-practices-for-implementing-fair-vocabularies-and-ontologies-on-the-web]] - Strong practical citation for ontology URI design, metadata, version IRIs, content negotiation, documentation, and FAIR vocabulary publication. This is probably the most useful applied-publication reference.
- [[ar.w3.best-practice-recipes-for-publishing-rdf-vocabularies]] - Older W3C recipe citation for RDF vocabulary publication patterns. Use if space permits or if we need a W3C-backed publication-pattern reference.
- [[ar.jekyll-rdf-template-based-linked-data-publication-with-minimized-effort-and-maximum-scalability]] - Best citation for static HTML publication of RDF resources under their IRIs. Use to position ResourcePages and static-host friendliness.
- [[ar.the-acimov-methodology-agile-and-continuous-integration-for-modular-ontologies-and-vocabularies]] - Best citation for Git-based, CI/CD-oriented ontology engineering and publication. Use for repository-native workflow, validation, documentation generation, and modular ontology artifacts.
- [[ar.analysing-multiple-versions-of-an-ontology]] - Best ontology-versioning citation in the vault. Use for the claim that ontology diffs and version comparison are a serious ontology engineering concern, not just file bookkeeping.
- [[ar.decentralized-collaborative-knowledge-management-using-git]] - Strong citation for Git-inspired RDF dataset collaboration, branching, merging, synchronization, and provenance. Use to distinguish Weave from RDF-store-centered collaboration: Weave focuses on publication state and artifact resolution.
- Reserve: [[ar.delta-an-ontology-for-the-distribution-of-differences-between-rdf-graphs]] - Useful if the paper discusses deltas, weak/strong diffs, or future work around artifact state comparison.
- Reserve: [[ar.modular-ontology-modeling]] - Useful for the applied ontology framing: modularity, reuse difficulty, conceptual clarity, provenance of reused resources, records-based modeling, and tooling support.
- Reserve: [[ar.defining-n-ary-relations-on-the-semantic-web]] - Use if `ArtifactResolutionSpec` needs an explicit modeling-pattern citation.
- Reserve: [[ar.base-platform-for-knowledge-graphs-with-free-software]] - Useful CEUR citation for the FOSS KG tooling landscape and the pain of composing specialized creation, serving, viewing, exploration, documentation, and maintenance tools into a platform.
- Reserve: [[ar.semiceu.semic-s-semantic-modelling-reference-architecture-and-tool-recommendations]] - Not exactly academic literature, but excellent practice reference for tool architecture around semantic modeling, WIDOCO, Papyrus, VocBench, and SHACL.
- Reserve: [[ar.voc-bench-3-a-collaborative-semantic-web-editor-for-ontologies-thesauri-and-lexicons]] - Use only if comparing Weave to editing/collaboration tools.

## DigitalArtifact and Provenance

The DigitalArtifact model is the conceptually richest angle for FOIS. It should be introduced in plain language before using local vocabulary:

> In Semantic Flow, a `DigitalArtifact` is a mesh-governed information artifact, such as an ontology file, SHACL file, reference catalog, source registry, page definition, or generated page. It is not merely a file path. It has identity in the mesh and may have histories, states, byte manifestations, located file copies, source bindings, and provenance.

Candidate references:

- [[ar.functional-requirements-for-information-resource-provenance-on-the-web]] - Primary. Maps cleanly to the resource/content/bytes/copy distinction and to provenance for HTTP information resources, but lacks Semantic Flow's temporal publication-state layer. Use heavily as a touchpoint and contrast.
- [[ar.works-expressions-manifestations-items-an-ontology]] - Primary/supporting. Helps explain WEMI without forcing strict FRBR disjointness or a full stack in every case. Use carefully: Semantic Flow intentionally leaves `frbr:Expression`-like modeling out of the core artifact layer.
- [[ar.ontology-infrastructure-for-the-semantic-web]] - Reserve. Good for foundational ontology humility: the challenge is isolating ontological options and formal relationships, not choosing one monolithic category set.
- [[ar.comparison-of-ontological-representations-of-relations-between-digital-and-physical-artifacts-in-manufacturing-domain]] - Reserve only. Title is very relevant, but the local note has no usable detail yet. Worth looking up later if DigitalArtifact becomes the headline contribution.
- [[ar.advanced-ontology-topics-events-roles-artifacts]] - Reserve. Potential conceptual support for artifacts/roles/events, but inspect before citing.
- [[ar.environment-programming-in-multi-agent-systems-an-artifact-based-perspective]] - Reserve, probably not for this paper. It treats artifacts as first-class computational environment entities, which is interesting but may pull the reviewer into a different literature.
- [[ar.agents-and-artifacts-a-meta-model-for-agent-oriented-computing]] - Reserve, same caution as above.
- [[ar.instrumenting-multi-agent-organisations-with-organisational-artifacts-and-agents]] - Reserve, same caution as above.

Suggested use in paper:

- Use FRIR/WEMI to explain the ambiguity around identifier, content, bytes, and copies; then make clear that Semantic Flow does not map `DigitalArtifact` to `frbr:Expression`. Different formats and packaging choices are `ArtifactManifestation`s in Semantic Flow; whether they are expressions of a more abstract work is a publisher/domain modeling choice.
- Emphasize that FRIR does not provide the temporal `HistoricalState` layer that Semantic Flow needs for release-state browsing and artifact histories.
- Explain sparse/skip-level modeling as part of the contrast with WEMI-shaped stacks. Semantic Flow can publish the full chain from `DigitalArtifact` through `ArtifactHistory`, `HistoricalState`, `ArtifactManifestation`, and `LocatedFile`, but it also allows direct links such as `hasManifestation`, `locatedFileForState`, `locatedFileForArtifact`, `hasWorkingLocatedFile`, and resolution targets such as `targetLocatedFile` when the full chain would be over-modeled.
- Consider a two-column figure/table comparing FRIR and Semantic Flow directly: FRIR handles information-resource provenance across work/content/bytes/copies; Semantic Flow handles byte-grounded artifact publication with `HistoricalState`, `ArtifactManifestation`, `LocatedFile`, and coordinate-bearing `ArtifactResolutionSpec` resources.
- Use PROV-O through FRIR/PAV/product notes if needed, but do not introduce a long provenance ontology detour in a demo paper.
- Avoid making `DigitalArtifact` sound like a complete theory of every domain-specific digital object distinction. The honest claim is still strong: it is a broad Semantic Flow category and practical publication model for governed digital artifacts in a SemanticMesh.

## Web Identifiers and Linked Data Publication

These references ground dereferenceability, ResourcePages, static hosting, and ontology publication conventions.

- [[ar.w3.design-issues.linked-data]] - Primary. Cite for Linked Data principles.
- [[ar.w3.cool-uris]] - Primary. Cite for persistent HTTP URIs and the information-resource distinction.
- [[ar.best-practices-for-implementing-fair-vocabularies-and-ontologies-on-the-web]] - Primary. Cite for modern FAIR vocabulary/ontology publication guidance: URI design, versioning, permanent URIs, metadata, documentation, content negotiation, and multiple serializations.
- [[ar.w3.best-practice-recipes-for-publishing-rdf-vocabularies]] - Primary/supporting. Cite for W3C vocabulary publication recipes.
- [[ar.jekyll-rdf-template-based-linked-data-publication-with-minimized-effort-and-maximum-scalability]] - Primary. Cite for static linked data resource pages and the human/machine publication bridge.
- Jeni Tennison, "Using 'Punning' to Answer httpRange-14" - Primary/supporting web reference for the distinction between content located by a URI and the socially governed sense associated with URI use. Use carefully: Semantic Flow should say URIs have or acquire a sense in context, not that they literally refer to a sense.
- [[ar.identitas-semantics-free-and-human-readable-identifiers]] - Supporting. Useful if we discuss readable identifiers, labels, and avoiding opaque-only IRIs.
- [[ar.linked-data-templates]] - Reserve. Could support read-write linked data interface work, but Weave's current paper is publication-oriented rather than app-interface-oriented.
- [[ar.adaptive-linked-data-driven-web-com-ponents-building-flexible-and-reusable-semantic-web-interfaces]] - Reserve. Useful if we discuss future UI components or ResourcePage panels as reusable semantic interfaces.
- [[ar.ldflex-a-read-write-linked-data-abstraction-for-front-end-web-developers]] - Reserve. Good for the "developer experience matters for Linked Data adoption" argument, but likely outside the demo's core.
- [[ar.third-generation-web-apis-bridging-the-gap-between-rest-and-linked-data]] - Reserve. Useful if positioning future API/daemon work; probably not central to the current static publication demo.

Suggested use in paper:

- Cite Linked Data and Cool URIs in the first two paragraphs.
- Cite Tennison when explaining why the identified resource, the retrievable content, and the page describing/presenting the resource should not be collapsed. This supports the `ResourcePage` / `hasResourcePage` pattern without requiring 303 redirects or hash-URI discipline.
- Cite FAIR vocabulary best practices when describing the problem Weave solves.
- Cite Jekyll RDF when explaining generated static ResourcePages.

## Versioning, History, and Artifact State

Weave's `ArtifactHistory` and `HistoricalState` terminology needs to be explained as publication history, not a replacement for ontology diff theory.

- [[ar.analysing-multiple-versions-of-an-ontology]] - Primary. Cite for the seriousness of ontology version comparison and semantic vs syntactic change.
- [[ar.decentralized-collaborative-knowledge-management-using-git]] - Primary. Cite for Git-backed RDF collaboration, provenance, branching, merging, and synchronization.
- [[ar.distributed-collaboration-on-rdf-datasets-using-git]] - Supporting. Earlier/related Quit Store work. Use one of the two Quit references unless doing a fuller related-work paragraph.
- [[ar.delta-an-ontology-for-the-distribution-of-differences-between-rdf-graphs]] - Supporting/reserve. Useful for RDF graph deltas and weak/strong diffs.
- [[ar.w3.delta-an-ontology-for-the-distribution-of-differences-between-rdf-graphs]] - Duplicate/alternate W3C note for the same delta topic; use only one.
- [[ar.star-vers-versioning-and-timestamping-rdf-data-by-means-of-rdf-star-an-approach-based-on-annotated-triples]] - Reserve. Useful if using RDF-star or timestamped triples becomes relevant later.
- [[ar.valid-time-rdf]] - Reserve. Very rich for temporal RDF, but probably too deep for this demo unless we discuss valid-time vs release-state explicitly.
- [[ar.rdf-for-temporal-data-management-a-survey]] - Reserve. Local note is empty; inspect before citing.
- [[ar.a-survey-for-managing-temporal-data-in-rdf]] - Reserve. Potential broader temporal RDF citation if state/history language needs a survey.
- [[ar.detailed-comparison-of-seven-approaches-for-the-annotation-of-time-dependent-factual-knowledge-in-rdf-and-owl]] - Reserve. Useful if we need to justify not modeling temporal truth with Weave histories.
- [[ar.representing-time-in-rdf]] - Reserve. Useful if we need a lightweight temporal/context modeling citation.

Suggested use in paper:

- Say Weave records explicit artifact publication states. It does not currently claim to solve semantic ontology differencing.
- Use major ontology revisions as the clearest example of why publication state needs to be RDF-visible. A breaking release may affect term meanings, imports, examples, validation shapes, generated documentation, dependent meshes, deprecated-term dereferenceability, and migration guidance. Semantic Flow can make the major-version boundary inspectable as artifact states and manifestations, but the semantic migration analysis remains ontology-engineering work.
- If challenged on "history," point to ontology versioning and RDF/Git collaboration literature, then emphasize Weave's narrower publication-state contribution.
- Do not let valid-time RDF literature take over the paper. Release/history state is not the same thing as temporal truth.

## Ontology Engineering, Modularity, and Tooling

These references place Weave in the ontology engineering/tool support ecosystem.

- [[ar.the-acimov-methodology-agile-and-continuous-integration-for-modular-ontologies-and-vocabularies]] - Primary. Best fit for Git, CI/CD, modular ontology engineering, checks, documentation generation, and artifact publication.
- [[ar.modular-ontology-modeling]] - Primary/supporting. Good for modularity, reuse difficulty, provenance of reused resources, and records-based modeling.
- [[ar.voc-bench-3-a-collaborative-semantic-web-editor-for-ontologies-thesauri-and-lexicons]] - Supporting. Cite if contrasting Weave with ontology/thesaurus editing and workflow tools.
- [[ar.an-infrastructure-for-collaborative-ontology-development]] - Reserve. Inspect if adding a related work paragraph on collaborative ontology infrastructure.
- [[ar.distributed-engineering-of-ontologies-diligent]] - Reserve. Classic methodology line; inspect before citing.
- [[ar.ontology-development-101-a-guide-to-creating-your-first-ontology]] - Reserve. Familiar methodology citation but likely too introductory for this demo.
- [[ar.criteria-and-evaluation-for-ontology-modularization-techniques]] - Reserve. Use if modularity becomes a stronger claim.
- [[ar.bobdc.using-owlincludes]] - Reserve. Useful blog-level support for include/import ergonomics; not a primary academic citation.
- [[ar.where-to-publish-and-find-ontologies-a-survey-of-ontology-libraries]] - Supporting/reserve. Useful if discussing ontology registries and discoverability.
- [[ar.douroucouli.a-lightweight-ontology-registry-system]] - Reserve. Potential registry reference.
- [[ar.base-platform-for-knowledge-graphs-with-free-software]] - Supporting. CEUR paper on assembling a free/open-source KG platform from specialized tools; mentions Protégé for ontology authoring, Widoco for documentation/visualization, and several serving/exploration tools. Useful for arguing that Semantic Flow complements rather than replaces the existing tool stack.
- [[ar.semiceu.semic-s-semantic-modelling-reference-architecture-and-tool-recommendations]] - Supporting. Practical architecture/tool recommendation reference. Good for showing Semantic Flow grew out of the same tool ecosystem, but do not lean on it as a peer-reviewed theoretical citation.

Suggested use in paper:

- ACIMOV plus FAIR vocabulary best practices can carry most of the "existing ontology engineering research already wants repository-native CI/CD publication" burden.
- Use VocBench only to avoid implying Weave is an editor. "Complementary to editors such as VocBench..."

## SHACL, Validation, and Modeling Constraints

Use only if the demo shows validation or SHACL artifacts prominently.

- [[ar.a-formal-approach-for-customization-of-schema-org-based-on-shacl]] - Supporting. Shows SHACL as a formal/domain customization and pattern mechanism, not just validation plumbing.
- [[ar.arxiv.shacl-a-description-logic-in-disguise]] - Reserve. More theoretical SHACL/description logic angle; likely too deep for demo.
- [[ar.topquadrant.why-i-use-shacl-for-defining-ontology-models]] - Reserve. Practitioner perspective; useful for internal rationale, weaker as academic citation.
- [[ar.owl-profiles-rule-based-reasoning-and-handling-reasoning-with-the-jena-api]] - Reserve. Use only if OWL profiles or Jena implementation details enter the paper.
- [[ar.pellet-an-owl-reasoner]] - Reserve. Probably unnecessary.

Suggested use in paper:

- If mentioning SHACL only as "one kind of DigitalArtifact Weave can govern," no citation needed beyond specs or FAIR/ACIMOV context.
- If claiming SHACL participates in the modeling method, add one SHACL citation.

## RDF Modeling Patterns for Explaining Semantic Flow Terms

Useful if reviewers need more context for relators such as `ArtifactResolutionSpec`.

- [[ar.defining-n-ary-relations-on-the-semantic-web]] - Supporting. Good for explaining relator-like resources that carry source coordinates, resolution mode, fallback policy, and provenance around a target relation.
- [[ar.don-t-like-rdf-reification-making-statements-about-statements-using-singleton-property]] - Reserve. Only cite if discussing why not use singleton-property/reification approaches.
- [[ar.representing-classes-as-property-values-on-the-semantic-web]] - Reserve. Use only if class/property modeling issues appear.
- [[ar.c-owl-contextualizing-ontologies]] - Reserve. Contextualization literature, probably too broad.
- [[ar.a-reusable-ontology-for-fluents-in-owl]] - Reserve. Temporal/contextual modeling, likely not central.

Suggested use in paper:

- A short footnote or parenthetical can say `ArtifactResolutionSpec` is a relation resource carrying additional parameters, a common RDF modeling pattern for n-ary relations.

## Foundational Ontology Background

Probably do not cite these in the demo unless the paper shifts toward "formal ontology foundations of DigitalArtifact."

- [[ar.formal-ontologies-and-information-systems]] - FOIS lineage/context.
- [[ar.applied-ontology]] - Journal/topic context rather than a specific claim.
- [[ar.basic-formal-ontology-july-2023]] - BFO reference, likely not needed.
- [[ar.bfo-2-0-specification-and-users-guide]] - BFO reference, likely not needed.
- [[ar.dolce-a-descriptive-ontology-for-linguistic-and-cognitive-engineering]] - DOLCE reference, useful only if grounding artifact/event/category choices.
- [[ar.gfo-the-general-formal-ontology]] - GFO reference, likely not needed.
- [[ar.general-formal-ontology-gfo-a-foundational-ontology-integrating-objects-and-processes]] - GFO reference, likely not needed.
- [[ar.ufo-unified-foundational-ontology]] - UFO reference, likely not needed.
- [[ar.towards-ontological-foundations-for-conceptual-modeling-the-unified-foundational-ontology-ufo-story]] - UFO/conceptual modeling background, likely not needed.
- [[ar.foundational-ontologies-in-action]] - Could be useful if framing "applied formal ontology in tooling," but probably not necessary.
- [[ar.some-open-issues-after-twenty-years-of-formal-ontology]] - Reserve for broader positioning.

Suggested use in paper:

- For FOIS reviewers, a small nod to formal ontology is useful, but this is a demo paper. Cite FRIR/WEMI first because it frames the prior ambiguity around information resources, content, bytes, and copies; then show where Semantic Flow intentionally diverges.

## Standards and Specification Notes

These are not all `ar.*` academic papers, but they may be needed in the final bibliography.

- [[ar.w3.rdf-1-1-concepts-and-abstract-syntax]] - Cite if defining RDF graphs, IRIs, or datasets beyond ordinary background.
- [[ar.w3.rdf11-primer]] - Introductory only; likely not needed.
- [[ar.w3.rdf-identifiers]] - Reserve for identifier detail.
- [[ar.semiceu.dcat-ap-3-0]] - Reserve if publication metadata/cataloging becomes important.
- [[ar.w3.vocab-data-cube]] - Not relevant unless examples use data cube material.

## Paper Claim Map

Use this map while drafting so every citation has a job.

- "Semantic Flow builds on Linked Data publication principles." Cite [[ar.w3.design-issues.linked-data]], [[ar.w3.cool-uris]], [[ar.best-practices-for-implementing-fair-vocabularies-and-ontologies-on-the-web]].
- "Ontology publication needs metadata, documentation, versioning, and content negotiation, not only RDF serialization." Cite [[ar.best-practices-for-implementing-fair-vocabularies-and-ontologies-on-the-web]] and optionally [[ar.w3.best-practice-recipes-for-publishing-rdf-vocabularies]].
- "DigitalArtifact separates artifact identity, historical publication state, byte manifestations, located files, and provenance while leaving expression/work modeling to the publisher." Cite [[ar.functional-requirements-for-information-resource-provenance-on-the-web]] and [[ar.works-expressions-manifestations-items-an-ontology]] as touchpoints/contrasts.
- "Semantic Flow supports sparse publication by allowing consistency-preserving skip-level properties instead of forcing every layer to be materialized." Use this as a DigitalArtifact design claim; cite FRIR/OpenWEMI only as contrast, not as foundation.
- "Static generated pages can be a legitimate Linked Data publication strategy." Cite [[ar.jekyll-rdf-template-based-linked-data-publication-with-minimized-effort-and-maximum-scalability]].
- "Repository-native ontology engineering and CI/CD are established concerns." Cite [[ar.the-acimov-methodology-agile-and-continuous-integration-for-modular-ontologies-and-vocabularies]].
- "Versioning and change analysis matter for ontologies." Cite [[ar.analysing-multiple-versions-of-an-ontology]].
- "Major ontology revisions are public contract changes, not only repository diffs." Cite [[ar.analysing-multiple-versions-of-an-ontology]] and use Semantic Flow's `HistoricalState`/manifestation/ResourcePage stack as the publication-layer response.
- "Git-backed RDF collaboration has prior work." Cite [[ar.decentralized-collaborative-knowledge-management-using-git]] or [[ar.distributed-collaboration-on-rdf-datasets-using-git]].
- "Parameterized relation resources are a normal RDF modeling pattern." Cite [[ar.defining-n-ary-relations-on-the-semantic-web]] if `ArtifactResolutionSpec` needs defense.

## Suggested Final Bibliography Shortlist

For the FOMI DigitalArtifact paper, target these ten first:

1. [[ar.functional-requirements-for-information-resource-provenance-on-the-web]]
2. [[ar.works-expressions-manifestations-items-an-ontology]]
3. [[ar.w3.design-issues.linked-data]]
4. [[ar.w3.cool-uris]]
5. [[ar.best-practices-for-implementing-fair-vocabularies-and-ontologies-on-the-web]]
6. [[ar.w3.best-practice-recipes-for-publishing-rdf-vocabularies]]
7. [[ar.jekyll-rdf-template-based-linked-data-publication-with-minimized-effort-and-maximum-scalability]]
8. [[ar.the-acimov-methodology-agile-and-continuous-integration-for-modular-ontologies-and-vocabularies]]
9. [[ar.analysing-multiple-versions-of-an-ontology]]
10. [[ar.decentralized-collaborative-knowledge-management-using-git]]
Reserve swaps if the draft asks for them:

- [[ar.modular-ontology-modeling]] - swap in if the DigitalArtifact section leans into modularity, reuse, records, or conceptual clarity.
- [[ar.defining-n-ary-relations-on-the-semantic-web]] - swap in if `ArtifactResolutionSpec` needs defense as a relation resource.
- [[ar.base-platform-for-knowledge-graphs-with-free-software]] - swap in if the draft keeps a paragraph on existing KG/ontology publishing and exploration tools.
- [[ar.semiceu.semic-s-semantic-modelling-reference-architecture-and-tool-recommendations]] - swap in if the tooling architecture story needs a practice reference.
- [[ar.voc-bench-3-a-collaborative-semantic-web-editor-for-ontologies-thesauri-and-lexicons]] - swap in if we explicitly compare Weave with collaborative ontology editors.
- [[ar.delta-an-ontology-for-the-distribution-of-differences-between-rdf-graphs]] - swap in if artifact state comparison or RDF patching enters the paper.

If page pressure is brutal, keep FRIR over OpenWEMI for the DigitalArtifact story, but phrase it as a touchpoint and contrast rather than a foundation.

## Follow-Up Checks

- Fetch proper BibTeX or CSL metadata for the shortlist before final PDF assembly.
- Verify whether FOIS/CEUR treats W3C DesignIssues pages as acceptable web references; if not, use W3C Recommendations/Notes where possible.
- Inspect [[ar.comparison-of-ontological-representations-of-relations-between-digital-and-physical-artifacts-in-manufacturing-domain]] if we decide DigitalArtifact is the headline contribution rather than a supporting model.
- Decide whether the paper should cite PAV or PROV-O directly from product/spec notes in addition to FRIR. FRIR may be enough for a demo paper.
