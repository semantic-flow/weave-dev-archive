---
id: eifjeuo2o0xjm7fvlkajf95
title: 2026 05 28 0030 Rdf and Turtle Cleanup
desc: ''
updated: 1779953472503
created: 1779953472503
---

## Goals

- Make RDF parse/render boundaries explicit so production callers do not have to remember hidden Turtle context such as `@base`, `sflo:`, or `sfcfg:`.
- Move semantic planner inputs away from stringly Turtle snippets where a fact/quad representation is the honest data shape.
- Keep Weave's generated TTL readable and consistent: `a` for `rdf:type`, compact SFLO/config prefixes where appropriate, mesh-local IRIs as relative `<path>` terms, and external IRIs as absolute IRIs.
- Preserve byte-for-byte carried unknown Turtle blocks when append-onlyish inventory behavior depends on original source text.
- Create a small RDF/JS-shaped seam that can use N3 today and later support an Oxigraph-backed implementation without rewriting every planner/resolver.
- Keep the first implementation slice small enough to land during normal release hardening.

## Summary

We just tripped over a representative boundary bug: carried Knop support facts were semantically correct but rendered as absolute SFLO predicate IRIs, and when we compacted them to `sflo:*`, the internal planner parse path needed explicit prefix context.

That is the deeper smell. Several code paths pass Turtle snippets as strings even when the caller and callee are really exchanging facts. The snippets may be full Turtle documents, fragments that assume a mesh base, fragments that assume standard prefixes, or byte-preserved blocks whose original text matters. Today those distinctions live in caller memory and ad hoc helper names.

This task introduces a small local RDF/Turtle context layer. The first concrete target should be `inventory_append_planner`: production callers should be able to pass parsed facts/quads or local `WeaveRdfFact` values directly, while Turtle-string parsing remains available as a convenience/helper boundary. Rendering should also go through one preferred Weave formatter instead of each caller deciding whether to emit absolute IRIs, prefixed names, or mesh-local relative terms.

This is not a replacement of N3. N3 already gives us RDF/JS terms and quads and is good enough for the current slice. The design should avoid making raw N3 parser calls and raw Turtle snippets the broad application API, so an Oxigraph-backed dataset/query adapter remains plausible later.

## Discussion

### Current Pain

Current code has several levels of RDF representation:

- Durable Turtle files and fixture text.
- Byte-preserved Turtle blocks carried from existing inventory documents.
- Parsed N3 `Quad` arrays used for semantic checks.
- Rendered one-line Turtle facts used as append-planner input.
- Hand-built string renderers in import/extract/integrate/weave paths.

The representation choice is often right locally, but the boundaries are blurry. For example, `renderKnopInventoryWithPreservedSupportArtifacts` semantically knows it is carrying support facts, but it currently has to render those facts to Turtle so `planInventoryAppend` can parse them back into quads. That forces the caller to know the planner's parse context and makes output style leak into semantic comparison.

### Desired Boundary

Use Turtle strings at file and text-preservation boundaries. Use facts/quads at semantic boundaries.

Good examples:

- A full inventory file is Turtle text when read from disk.
- A carried unknown subject block should keep `{ originalBlock: string, parsedQuads: readonly Quad[] }`.
- A requested settled fact for the append planner should be a fact/quad value, not a Turtle string.
- Renderer output should be Turtle text only after policy/planning has already decided what facts to write.

### N3, RDF/JS, and Oxigraph

N3 is already in `deno.json` and exposes RDF/JS-style terms/quads. That is the right dependency to use for this slice.

The useful abstraction is not "hide N3 completely." It is "do not make every application caller responsible for parser construction, prefix setup, term formatting, and quad keying." A small local layer can accept/return RDF/JS-ish terms today and later adapt to Oxigraph for indexing/querying where it helps.

Potential shape:

```ts
export interface WeaveRdfFact {
  readonly subject: NamedNode;
  readonly predicate: NamedNode;
  readonly object: NamedNode | Literal;
}

export interface ParsedTurtleBlock {
  readonly originalTurtle: string;
  readonly quads: readonly Quad[];
}
```

The exact names can change, but the intent should hold: semantic code exchanges fact objects; byte-preservation code carries original Turtle alongside parsed quads.

### Standard Context

Weave needs one standard place to define:

- `rdf:` / `a`
- `sflo:`
- `sfcfg:`
- `xsd:`
- mesh-base handling for relative IRIs
- file-base handling for host/local config files when mesh base is not correct

This should not silently apply the mesh base everywhere. The helper names should distinguish a full Weave mesh Turtle document, a Weave mesh Turtle snippet, and arbitrary host-local Turtle.

### Formatting

The formatter should optimize for Weave's generated TTL style, not for generic pretty-printing:

- Use `a` for `rdf:type`.
- Use `sflo:*` for SFLO ontology terms.
- Use `sfcfg:*` for config vocabulary terms.
- Use `xsd:*` for common datatypes when safe.
- Use mesh-local relative IRIs when an IRI starts with the mesh base.
- Use absolute `<https://...>` IRIs for external terms.
- Preserve literal language/datatype information.

This is a small fact renderer, not a full Turtle document canonicalizer.

### Relationship To Append-Onlyish Inventory

Append-onlyish inventory needs semantic comparison plus byte-preservation. RDF facts are good for no-op/conflict/missing decisions. Original Turtle blocks are still required when carrying unknown support facts byte-for-byte.

Do not let a renderer rewrite carried unknown blocks just because it can serialize equivalent quads.

## Open Issues

- Should the first public-ish helper type be `WeaveRdfFact`, raw RDF/JS `Quad`, or a `FactLike` shape that can be converted to a quad? Recommendation: use a small local fact type at higher-level boundaries and keep N3 `Quad` in parser-adjacent helpers.
- Should the new helpers live in `src/core/rdf` or `src/core/weave`? Recommendation: standard prefixes, parser context, term rendering, and quad keys belong in `src/core/rdf`; inventory-specific append policy remains in `src/core/weave`.
- How much existing code should be migrated in the first slice? Recommendation: `inventory_append_planner` and `knop_support_renderers` only, with tests. Do not touch import/extract/integrate bulk renderers until the boundary proves itself.
- Which prefixes are "standard" for snippets? Recommendation: `rdf`, `sflo`, `sfcfg`, and `xsd`; allow callers to opt into more, but do not silently invent application-specific prefixes.
- Should blank nodes be allowed in append-planner requested facts? Current append behavior wants settled named-node/literal facts. Keep blank nodes out of requested append facts unless a specific use case appears.
- Should parser helpers return mutable arrays or immutable/read-only views? Prefer readonly in signatures; N3 returns arrays internally.

## Decisions

- Turtle strings are the durable file format and the byte-preservation format; they should not be the default representation for semantic planner input.
- Use N3/RDF/JS terms for the first implementation. Do not add a new RDF dependency in this task.
- Design the helper boundary so an Oxigraph-backed implementation can be introduced later for dataset/query-heavy paths.
- Preserve existing generated output style unless a focused test locks in an intended compactness fix.
- Do not build or adopt a general Turtle pretty-printer in this task.
- Do not re-render carried unknown blocks when the requirement is byte preservation.

## Contract Changes

- No user-facing CLI or ontology contract change is intended.
- Internal API change: `planInventoryAppend` should gain a facts/quads input path so production callers do not need to pass `requestedSettledFactsTurtle`.
- Existing tests may continue to use Turtle-string convenience helpers where that keeps fixtures readable.
- Generated TTL should remain semantically equivalent and, for touched paths, should prefer existing Weave compact style over absolute SFLO/config IRIs.

## Testing

- Add unit tests for standard Weave Turtle snippet parsing:
  - `sflo:*` predicates parse without each caller hand-adding prefix directives.
  - `sfcfg:*` terms parse when needed.
  - mesh-local relative IRIs resolve against the mesh base.
- Add unit tests for preferred term/fact rendering:
  - `rdf:type` renders as `a`.
  - SFLO predicates/classes render as `sflo:*`.
  - config terms render as `sfcfg:*`.
  - mesh-local objects render as relative `<path>`.
  - external IRIs remain absolute.
  - literals preserve language/datatype.
- Add append-planner regression tests where requested facts are provided as fact/quad objects, not Turtle snippets.
- Keep or add the Knop support preservation regression:
  - no absolute `<https://semantic-flow.github.io/sflo/ontology/hasReferenceCatalog>` predicate leaks into appended compact support facts.
  - carried unknown support blocks stay byte-for-byte unchanged.
- Run focused tests for:
  - `src/core/weave/inventory_append_planner_test.ts`
  - `src/core/weave/knop_support_renderers_test.ts`
  - `src/core/knop/add_reference_test.ts`
  - `tests/integration/extract_test.ts` for the source-reference path.
- Before closing, run `deno task fmt`, `deno task lint`, and `deno task check`.

## Non-Goals

- Do not replace N3 with Oxigraph in this task.
- Do not build a full RDF store abstraction across the runtime.
- Do not canonicalize or reformat every generated TTL file.
- Do not regenerate fixtures solely for formatting churn.
- Do not make append-onlyish inventory depend on RDF serializer output for byte-preserved unknown blocks.
- Do not change Semantic Flow ontology vocabulary or SHACL constraints.

## Implementation Plan

- [ ] Add or expand `src/core/rdf` helpers for standard Weave Turtle directives, parser context, RDF term/fact construction, quad keys, and preferred one-line fact rendering.
- [ ] Add tests for the helper behavior before migrating planner callers.
- [ ] Extend `planInventoryAppend` to accept requested settled facts as fact/quad objects in addition to, or instead of, `requestedSettledFactsTurtle`.
- [ ] Keep a Turtle-string convenience path for tests and current callers that truly start from Turtle text.
- [ ] Update `renderKnopInventoryWithPreservedSupportArtifacts` so it passes requested facts to the planner without serializing them to parser-dependent Turtle first.
- [ ] Keep carried support blocks represented as original Turtle text plus parsed quads, and append original blocks when byte preservation is required.
- [ ] Replace local one-off support fact renderers with the shared preferred term/fact renderer.
- [ ] Add regressions for compact appended support facts and byte-preserved unknown support blocks.
- [ ] Run the focused tests listed above.
- [ ] Run `deno task fmt`, `deno task lint`, and `deno task check`.
