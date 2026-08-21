---
id: m7q2v9k4x1r8p5t3n6h0bca
title: 2026 08 21_1048 Cross Engine Shacl Conformance
desc: 'Run one SFLO-owned SHACL fixture corpus through PySHACL, shacl-engine, and Jena without adopting Oxigraph as a Weave runtime dependency'
created: 1787334528000
---

## Goals

- Prove that the SFLO content-digest SHACL contract behaves consistently under PySHACL, JavaScript `shacl-engine`, and Apache Jena SHACL.
- Keep one authoritative fixture corpus and expectation manifest in SFLO rather than reimplementing equivalent cases independently in each consumer.
- Exercise the exact shipped `semantic-flow-core-shacl.ttl` and `semantic-flow-core-ontology.ttl` without weakening or rewriting shapes for a particular engine.
- Record semantic receipts that compare conformance, severity, focus, result path, and constraint identity without comparing engine-specific report serialization.
- Verify Stagecraft's production `shacl-engine`/Oxigraph configuration before SFLO v0.4.0 without making Oxigraph a Weave or SFLO runtime dependency.
- Distinguish validator portability from the separate, parked question of an Oxigraph-backed Weave graph cache.

## Summary

The adversarial digest review [[wa.review.2026-08-21_1022-content-digest-contract-claude]] correctly required executable SHACL-SPARQL coverage rather than structural assertions plus a hand-written parallel evaluator. SFLO now runs eleven isolated cases through pinned PySHACL.

Stagecraft reports `shacl-engine` over Oxigraph as its engine of record, with Jena/Riot as backup. Neither Weave nor SFLO currently uses Oxigraph. Weave uses N3/RDF/JS-shaped quads and operation-scoped TypeScript read models; the parked [[wa.task.2026.2026-05-27_1314-oxigraph]] concerns possible future graph caching and query-heavy runtime paths. That architectural question is not a prerequisite for using `shacl-engine`, whose core accepts an RDF/JS dataset independently of the dataset implementation.

The vendored Accord implementation is useful precedent but not the same configuration: it uses `shacl-engine@1.1.2` core with an Accord-owned `sh:sparql` hook because the package's optional SPARQL plugin did not import cleanly under Deno's normal npm-cache path. No Oxigraph dependency is present there. The conformance plan must therefore test both the validator contract and Stagecraft's actual adapter rather than infer compatibility from package names.

## Discussion

### Engine and Storage Are Separate Axes

`shacl-engine` is the SHACL processor. Oxigraph may supply storage or SPARQL evaluation behind a Stagecraft adapter, but `shacl-engine` does not require Oxigraph merely to validate an RDF/JS dataset.

The test matrix should separate:

| Runner | SHACL core | SHACL-SPARQL | Dataset/query substrate | Purpose |
|---|---|---|---|---|
| PySHACL | PySHACL | PySHACL/RDFlib | RDFlib | SFLO's independent executable CI oracle |
| JavaScript reference | `shacl-engine` core | Explicit registered hook | Minimal RDF/JS dataset first | Prove the JS engine independent of Stagecraft infrastructure |
| Stagecraft production | `shacl-engine` core | Stagecraft's registered hook | Oxigraph-backed adapter | Prove the real downstream gate |
| Jena | Jena SHACL | Jena ARQ | Jena dataset | Independent Java backup and release-candidate check |

Riot validates RDF syntax and can remain a preflight tool, but Riot alone is not a SHACL execution engine. The Jena row must run Jena SHACL, not merely `riot --validate`.

### Canonical Fixture Corpus

Move the currently embedded PySHACL cases into an engine-neutral SFLO fixture tree, for example:

```text
tests/shacl/content-digest/
  cases.json
  valid-manifestation-file.ttl
  valid-downstream-bearer.ttl
  valid-historical-observation-after-expectation-change.ttl
  valid-manifestation-target-resolution.ttl
  invalid-untyped-observed-grammar.ttl
  invalid-standing-same-method.ttl
  invalid-observed-same-method.ttl
  invalid-manifestation-file-mismatch.ttl
  invalid-repository-standing-digest.ttl
  invalid-repository-expected-observed.ttl
  warning-untyped-bearer.ttl
```

`cases.json` should identify for each case:

- stable case id
- data file
- expected conforming status under the agreed warning policy
- expected maximum severity
- expected constraint/message identifier or stable message fragment
- expected focus-node and result-path IRIs when stable
- whether the result is a warning or violation

The manifest must not contain engine-specific blank-node ids, report ordering, stack traces, or serialized report bytes.

### Execution Profile

All runners must use one documented graph assembly profile:

- data graph: the case Turtle
- shapes graph: the checked-out `semantic-flow-core-shacl.ttl`
- ontology graph: the checked-out `semantic-flow-core-ontology.ttl`
- inference: none unless a later case explicitly tests inference
- SHACL advanced/SPARQL features: enabled
- warnings: reported and compared explicitly rather than silently promoted, ignored, or allowed by engine defaults
- network and imports: disabled; all graphs come from the checked-out source tree

The first spike must verify how each engine supplies an ontology graph. If an engine lacks a separate ontology-graph input, its adapter should create the same effective graph mix-in deliberately and filter reports to fixture focus nodes. Do not silently enable RDFS inference as a substitute; that would mask property-targeting defects such as the one found in B1 of the review.

### Normalized Result Contract

Each runner should emit a small JSON receipt with this shape or a deliberately equivalent one:

```json
{
  "engine": "pyshacl",
  "engineVersion": "0.40.0",
  "sfloCommit": "...",
  "caseId": "invalid-manifestation-file-mismatch",
  "conforms": false,
  "results": [
    {
      "severity": "Violation",
      "focusNode": "https://example.test/manifestation",
      "resultPath": null,
      "constraintComponent": "SPARQLConstraintComponent",
      "messageKey": "manifestation-file-digest-mismatch"
    }
  ]
}
```

Adapters may derive `messageKey` from a fixture-manifest mapping or stable SFLO message text. The comparison layer should ignore result order and engine-created blank-node identifiers.

### Gate Policy

Recommended initial policy:

- PySHACL remains required on every SFLO pull request because it is already small, independent, and automated.
- The JavaScript reference runner becomes required on every pull request once its Deno dependency and SPARQL-hook shape are stable.
- Stagecraft's exact `shacl-engine`/Oxigraph runner and Jena SHACL are required release-candidate checks for SFLO v0.4.0, with receipts checked into or linked from an `ont.report.*` note.
- Promote the Stagecraft and Jena rows into ordinary SFLO CI only if they can run without a private-repository dependency, hidden credentials, or disproportionate setup cost.

No SFLO release should proceed when engines disagree on conforming status or severity for a canonical case. Differences limited to report ordering, blank-node ids, or non-normative message rendering are acceptable after normalization.

## Open Issues

- Can Stagecraft expose its Oxigraph-backed validator as a stable local command or share a minimal public adapter without making SFLO CI depend on the private Stagecraft repository? Recommendation: produce a command that accepts explicit data/shapes/ontology paths and emits the normalized receipt.
- Should SFLO host its own `shacl-engine` adapter or reuse an extracted Accord helper? Recommendation: spike the smallest standalone adapter first; reuse only if the shared code can be packaged without pulling Accord manifest semantics into SFLO.
- Which Jena distribution and command are pinned for the release gate? Recommendation: pin one Apache Jena version and verify the actual Jena SHACL command during the spike; retain Riot only as syntax preflight.
- How should warnings affect `conforms` across engines? Recommendation: compare both the raw engine conformance flag and normalized maximum severity, then define the SFLO release gate as no unexpected Warning or Violation rather than trusting engine-default warning policy.
- Does `shacl-engine` core plus Stagecraft's Oxigraph query hook implement the full SPARQL subset used by SFLO (`STRBEFORE`, property paths, inverse joins, and `$this` prebinding)? The canonical cases must prove each used construct directly.

## Decisions

- This work does not adopt Oxigraph in Weave or SFLO runtime code.
- SFLO owns the canonical conformance cases and expected semantic outcomes.
- Engines execute the shipped shapes unchanged.
- Cross-engine comparison operates on normalized semantic results, not serialized report equality.
- PySHACL remains an independent oracle rather than being replaced merely because Stagecraft uses another engine.
- Stagecraft's production adapter and Jena are release gates first; CI promotion is conditional on reproducibility and cost.
- Engine disagreement blocks the SFLO release until the shapes, fixture expectation, or documented portability boundary is adjudicated.

## Contract Changes

- Add an engine-neutral SFLO SHACL fixture manifest and Turtle cases.
- Refactor the current PySHACL script to consume that manifest and emit normalized JSON receipts.
- Add a JavaScript `shacl-engine` runner or adapter consuming the same manifest.
- Define the command/receipt boundary for Stagecraft's production runner.
- Add a Jena SHACL runner consuming the same manifest.
- Record engine/version/commit receipts for the SFLO v0.4.0 release candidate.
- Update [[ont.dev.release-runbook]] with the cross-engine release gate after the spike proves the commands.

## Testing

- Every fixture executes independently so one violation cannot mask another broken constraint.
- Positive cases cover the two direct bearers, a downstream bearer subclass, expected-versus-historical-observation coexistence, and manifestation-target resolution.
- Negative cases cover all three lexical slots, same-method uniqueness, manifestation/file mismatch without explicit manifestation typing, repository leakage of every digest property, and missing explicit bearer typing at Warning severity.
- Add one deliberately corrupted copy of each SHACL-SPARQL query during runner development to prove that every engine-side test fails when property direction, variable, or filter logic is wrong.
- Verify warning normalization separately from violation normalization.
- Run the corpus with no inference and no network access.
- Record exact engine versions and commands in the release receipt.

## Non-Goals

- Adopting Oxigraph as Weave's general RDF store, parsed-graph cache, or config resolver.
- Replacing N3/RDF/JS terms in current Weave planners.
- Making SFLO CI depend directly on the private Stagecraft checkout.
- Rewriting SFLO shapes into an engine-specific subset.
- Running the full W3C SHACL conformance suite in the first slice.
- Proving that all engines serialize identical validation reports.
- Designing the separate DCAT media-type/byte-size profile.

## Implementation Plan

- [ ] Re-read [[ont.dev.guidance]], [[ont.dev.release-runbook]], [[sf.spec.2026-08-21-content-digest]], [[wa.review.2026-08-21_1022-content-digest-contract-claude]], and this task before editing.
- [ ] Extract the eleven embedded PySHACL cases into the canonical Turtle fixture tree and expectation manifest.
- [ ] Define and test the normalized JSON receipt contract.
- [ ] Refactor the PySHACL runner to consume the manifest and emit one receipt per case.
- [ ] Spike `shacl-engine` core over a minimal RDF/JS dataset without Oxigraph; record dependency size, Deno behavior, and SPARQL-hook requirements.
- [ ] Implement or adapt the JavaScript runner against the canonical manifest.
- [ ] Ask Stagecraft to expose or run its exact `shacl-engine`/Oxigraph adapter against the same checked-out fixture paths and return normalized receipts.
- [ ] Pin and implement the Jena SHACL runner; keep Riot syntax validation as a separate preflight.
- [ ] Compare all receipts and adjudicate every semantic disagreement.
- [ ] Add the proven PR/release-gate commands to SFLO CI and [[ont.dev.release-runbook]] at the scoped frequencies above.
- [ ] Create an `ont.report.*` release-candidate receipt naming SFLO commit, engine versions, commands, and case results.
- [ ] Run SFLO CI and cross-engine release gates from clean worktrees.
- [ ] Prepare separate semantic commit messages for SFLO and weave-dev-archive; Stagecraft owns any private-adapter commit.
