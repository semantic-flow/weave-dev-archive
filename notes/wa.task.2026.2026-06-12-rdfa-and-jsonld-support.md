---
id: j8yk1ua4e50lhqgek48ruhv
title: 2026 06 12 Rdfa and Jsonld Support
desc: ''
updated: 1781290982127
created: 1781290935033
---

## Goals

- (optionally) embed RDFa and/or JSON-LD in the generated pages

## Summary

## Discussion

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

```json
{
  "@context": {
    "sflo": "https://semantic-flow.github.io/sflo/ontology/",
    "schema": "https://schema.org/",
    "skos": "http://www.w3.org/2004/02/skos/core#"
  },
  "@graph": [
    {
      "@id": "https://semantic-flow.github.io/mesh-branch-fantasy-rules/ontology/wisdom",
      "@type": "skos:Concept",
      "sflo:hasResourcePage": {
        "@id": "https://semantic-flow.github.io/mesh-branch-fantasy-rules/ontology/wisdom/index.html"
      },
      "skos:prefLabel": "Wisdom",
      "skos:definition": "A measure of perception, intuition, judgment, and awareness.",
      "skos:related": {
        "@id": "https://semantic-flow.github.io/mesh-branch-fantasy-rules/ontology/intelligence"
      }
    },
    {
      "@id": "https://semantic-flow.github.io/mesh-branch-fantasy-rules/ontology/wisdom/index.html",
      "@type": [
        "sflo:ResourcePage",
        "schema:WebPage"
      ],
      "schema:about": {
        "@id": "https://semantic-flow.github.io/mesh-branch-fantasy-rules/ontology/wisdom"
      }
    }
  ]
}
```

## Open Issues

## Decisions

## Contract Changes

## Testing

## Non-Goals

## Implementation Plan

- [ ]

## RULED 2026-08-06 (Dave) — PARKED

No consumer has requested embedded RDF; Stagecraft's exercised use is validation and packaging, and its own requirements note says not to prioritize embedding absent direct need. Custom SFLO/RDFS assertions do not unlock a Google rich result, so search is not a justification.

The deeper reason to wait: the page document model is lossy. It already drops literal datatypes and language tags, incoming triples, blank-node identity, inventory predicates and ordinals, and digest/manifestation evidence. A faithful graph cannot be reconstructed from rendered panels — it needs a new parsed-dataset seam at `sourcePanelsForFacts` first. Embedding a lossy projection and calling it the graph would be worse than embedding nothing.

**If revived: JSON-LD** in one `<script type="application/ld+json">` block. RDFa is rejected because it couples graph correctness to presentation markup — hiding or restyling a panel would silently remove triples, and the custom-page renderer bypasses the shared document model. Microdata is dismissed outright.

**Revival triggers:** Dave ruling the page-identity graph normative for every generated page, or a named consumer supplying an acceptance test of the form "extract these exact triples from the HTML."

The cheap first slice remains available: the three-triple page-identity graph, ~547 bytes/page, ~0.77 MiB across SFLO's 1,467 pages, clean seam, no new dependency. Contrast full-payload embedding at 29.41 MiB on a 24.61 MiB publication.

Full design space, markup sketches, determinism/escaping plan, and the seven rulings a revival would need are in the 2026-08-02 codex analysis.
