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