# Semantic Meshes: Stable Identifiers, Pseudo-Immutable Evidence, and Dynamic Sense-Finding

Draft target: FOMI 2026 short paper, CEUR-ART LaTeX, one-column mode.

## Author Metadata

- Author: Dave Richardson
- Affiliation: Spectacular Voyage LLC
- ORCID: 
- Email: dave@spectacular.voyage
- Venue:
- Submission type: FOMI

## Abstract

The Semantic Flow framework seeks to make the Semantic Web more usable and FAIR by addressing long-standing problems with identifier persistence and ambiguity; digital artifact modeling; and ease of publication. This paper introduces the Semantic Mesh: a hierarchical identifier surface with RDF-described support structures and publication conventions designed to help humans and computers make sense of IRIs. Semantic Meshes support easy IRI minting; can carry arbitrary digital artifacts, with special support for artifact lineages and RDF documents; and provide straightforward dereferenceability through ResourcePages without any dependence on request or response headers or hash IRIs. At the center of the model is a faceted, five-layer `DigitalArtifact` model that builds on DCAT 3 versioning and distribution concepts while adding named internal lineages with settled historical states, and makes a cleaner distinction between representation-level byte manifestations and explicit located files. The Semantic Flow ontology embeds the DigitalArtifact model along with conventions for specifying the needed or acceptable facets via "resolution coordinates". We illustrate the model with the Fantasy Rules ontology fixture and introduce Weave, a command-line implementation for creating and updating Semantic Meshes and generating ResourcePages for mesh-managed IRIs.


## Keywords

Semantic Web, Linked Data, FAIR, cooler IRIs, pseudo-immutability, DCAT, ontology publishing, static sites, dereferenceability

## 1. Introduction



This paper makes n contributions:

- Presents the DigitalArtifact Model to describe things that can be represented as a series of bytes
- Addresses the httpRange-14 and CoolURIs issues by separating DigitalArtifacts from other identifiable things, and proposing the use of directory index resolution to provide descriptions and other sense-making content via HTML resource pages, while preserving the "Be unambiguous" principle 
- Introduces the Semantic Mesh
- Positions the Semantic Flow framework and ontology relative to DCAT 3, FRIR/WEMI, Cool URIs, 

## 2. The Faceted DigitalArtifact Model

start concrete
DigitalArtifact
ArtifactHistory
HistoricalState
ArtifactManifestation
LocatedFile
manifestation vs file location
skip-level links

### Figure: Artifact Chain and Skip-Level Links

TODO

### 2.1 Change, Lineage, and Coordinates

TODO


### Table: Publication Coordinates

TODO


### 2.2 DCAT Alignment

DigitalArtifact as dcat:Dataset
HistoricalState via dcat:hasVersion
ArtifactManifestation as dcat:Distribution
LocatedFile as the extra byte-location distinction
ArtifactHistory as named internal lineages

### table: DCAT vs SFLO


## 3 Identifiers, Referents, and ResourcePages

- not every identifier names a DigitalArtifact (aka Information Resource, Web document)
- term IRIs, class IRIs, person IRIs have senses and support artifacts
- support artifacts are not the referent, but they can get their own IRIs

### 3.1 Primary and Supporting Identifiers



### 3.2 Be Unambiguous

- "Web architecture tells you that for a thing resource (URI) it is inappropriate to return a 200 because there is, in fact, no suitable representation for those resources." [CoolURIs]
  - ultimately it doesn't really matter, but what does githubpages do? i.e., does the response header indicate the index.html is what's returned?
- the problem with "Machines should get RDF data and humans should get a readable representation, such as HTML." is that it is inherently ambiguous. An identifier alone no longer identifies something, it requires a request header as well. 
- "The challenge is to find a solution that allows us to find the describing documents if we have just the resource's URI, using standard Web technologies." gets the problem exactly right.
- The additional wrinkle is that information resources/web documents are also things in the world, and you may want to refer to them in the abstract as well.
- The SF proposed solution is simple: ID -> Resource Page (HTML document which may have embedded RDF) that describes a thing OR ID+/index.html refers to the Resource Page itself OR ID -> file/informational resource
  - don't hide the source from humans! Learn the "view source" lesson
  - in fact, the HTML page should both be able to expose RDF (so humans don't have to literally view source and search through all that other stuff) as well as silently embed it for machines. 

### 3.3 Resource Pages and Embedded RDF

- about the referent
- can be any content, but the point is to establish the sense of the corresponding IRI
  - should contain RDF metadata via RDFa and/or JSON-LD embedded in a script tag that differentiates the corresponding IRIs referent (a skos:Concept, rdfs:Class, foaf:Person) from the resource page itself (a sflo:ResourcePage)
  - may include FAIR metadata, which in the case of the Semantic Meshes, includes human and machine-interpretable links to supporting data

### Listing: RDFa

```html
<!doctype html>
<html lang="en"
  prefix="
    sflo: https://semantic-flow.github.io/sflo/ontology/
    schema: https://schema.org/
    skos: http://www.w3.org/2004/02/skos/core#
  ">

<head>
  <meta charset="utf-8" />
  <title>Wisdom</title>
</head>

<body
  resource="https://semantic-flow.github.io/mesh-branch-fantasy-rules/ontology/wisdom/index.html"
  typeof="sflo:ResourcePage schema:WebPage">

  <link
    property="schema:about"
    href="https://semantic-flow.github.io/mesh-branch-fantasy-rules/ontology/wisdom" />

  <main>
    <h1>Wisdom</h1>

    <section
      resource="https://semantic-flow.github.io/mesh-branch-fantasy-rules/ontology/wisdom"
      typeof="skos:Concept">

      <link
        property="sflo:hasResourcePage"
        href="https://semantic-flow.github.io/mesh-branch-fantasy-rules/ontology/wisdom/index.html" />

      <p property="skos:prefLabel">Wisdom</p>

      <p property="skos:definition">
        A measure of perception, intuition, judgment, and awareness.
      </p>

      <p>
        Wisdom complements
        <a
          property="skos:related"
          href="https://semantic-flow.github.io/mesh-branch-fantasy-rules/ontology/intelligence">
          Intelligence
        </a>,
        which represents reasoning, memory, and analytical ability.
      </p>

    </section>
  </main>
</body>
</html>
```

### Listing: JSON-LD in script tag

```json
{
  "@context": {
    "sflo": "https://semantic-flow.github.io/sflo/ontology/",
    "schema": "https://schema.org/",
    "skos": "http://www.w3.org/2004/02/skos/core#"
  },
  "@graph": [
    {
      "@id": "https://example.org/vocab/alignment",
      "@type": "skos:Concept",
      "sflo:hasResourcePage": {
        "@id": "https://example.org/vocab/alignment/index.html"
      }
    },
    {
      "@id": "https://example.org/vocab/alignment/index.html",
      "@type": ["sflo:ResourcePage", "schema:WebPage"],
      "schema:about": {
        "@id": "https://example.org/vocab/alignment"
      }
    }
  ]
}
```

### 3.3 Cooler IRIs

- github pages redirect
- role of fragment identifiers
- fragment identifiers should resolve to an HTML anchor





## 4. Semantic Mesh

- DigitalArtifact: integrate vs import

### 4.1 Knops and Support Artifacts

now introduce the whole publication structure
hierarchical IRI surface
Knops
support artifacts: reference catalogs, source registries, inventories, metadata, page definitions

### 4.2 Static Hosting and Repository Topologies 
- portability

### 4.1 "Repository Options"


## 5. Worked Example: The Fantasy Rules Ontology Surface

ontology document as DigitalArtifact: the perfect use case; surrounding mesh can include SHACL, examples, and documentation, all as DigitalArtifacts. The terms in an ontology are mostly non-artifacts, but some are also artifacts; ontologists may want to reference SF lineages, states, and manifestations in their ontology header data.
terms as primary identifiers that are not DigitalArtifacts
generated ResourcePages
release/state/manifestation/file coordinates
maybe one RDFa or JSON-LD listing
SFLO only as a brief reflexive note, not the main example


## 6. Implementation: Weave

CLI creates/updates meshes
generates ResourcePages
integrates source files
versions states
validates/generates static output

## 7. Related Work

Cool URIs / httpRange-14
DCAT 3
FRIR/WEMI
FAIR vocabulary publishing
Pubby/LodView/Widoco/LODE/pyLODE/Jekyll RDF/Walder/Harshp
Git/ACIMOV/repository-native ontology work

## 8. Limitations

TODO

## 9. Conclusion

TODO

## Generative AI Declaration

During the preparation of this work, the author used Codex (primarily GPT-5.5 "Extra High reasoning") to plan the work; help gather references from his notes; search past conversations for decisions and context; and review for accuracy, grammar, and clarity. Codex also served as a discussion partner, answering countless questions, providing opinions, and helping to generate ideas. Codex was instrumental in the development of the Semantic Flow platform and ontology and the Weave application, and was able to act as a domain expert in that regard. The author typed every word of this paper himself and takes full responsibility for its content.

## Acknowledgements

TODO

## References

TODO

## LaTeX Conversion Notes

- Use CEUR-ART LaTeX, one-column mode.
- Convert section headings directly to `\section{}` / `\subsection{}`.
- Convert code/listing blocks to CEUR-compatible listings or verbatim blocks.
- Convert figures and tables to numbered LaTeX floats with captions.
- Keep final paper within 5-9 pages including references.
