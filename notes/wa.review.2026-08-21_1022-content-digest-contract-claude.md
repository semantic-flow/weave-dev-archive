---
id: k3f9t2ncdrev8q1x5m7b0aa
title: 2026 08 21_1022 Content Digest Contract Claude
desc: 'Adversarial review of the uncommitted content-digest contract across sflo, the framework specs, Weave, and the task note'
created: 1787332942941
---

## Scope

Adversarial review of the content-digest contract as delivered in sflo `936bf8b6` (ontology, SHACL, guardrail tests), Semantic Flow Framework `144db3d` (specs), the uncommitted Weave working tree, and [[ont.task.2026.2026-08-14_1949-content-digest-contract]]. Focus areas: RDF/RDFS/OWL semantics, SHACL-versus-prose enforcement, future multi-method behavior, cross-document contradictions, digest-property placement, LocatedFile/ArtifactManifestation identity, repository-locator leakage, and missing negative tests. Weave's focused digest tests were run during review (60/60 pass). No files were edited.

## BLOCKING

### B1. `observedContentDigest` constraints do not fire for the data the prose claims they cover

In `semantic-flow-core-shacl.ttl`, every `observedContentDigest` constraint (grammar, same-method uniqueness, expected/observed agreement) lives only in `ArtifactResolutionObservationShape`, which uses `sh:targetClass sflo:ArtifactResolutionObservation` (line 662) — while `hasContentDigest` and `expectsContentDigest` use `sh:targetSubjectsOf` (lines 558, 595). Plain SHACL performs no RDFS domain entailment, so a malformed `observedContentDigest` (for example `sha256:XYZ`, or a value leaked onto a repository locator) on a node lacking the explicit type passes validation entirely. The only forced-typing path is `ArtifactResolutionObservationLinkShape` (line 608), which covers just nodes linked via `hasResolutionObservation`. This contradicts [[sf.spec.2026-08-21-content-digest]] line 96 ("SHACL applies the exact supported grammar to ... observedContentDigest at violation severity") and the decision-log claim that "all three digest properties use the same release-scoped lexical grammar at SHACL violation severity."

Fix: add a `sh:targetSubjectsOf sflo:observedContentDigest` literal shape mirroring `ContentDigestLiteralShape`.

### B2. The SHACL enforcement layer is never executed by any test

`assertSparqlViolation` (`tests/shacl_guardrails_test.ts:1004` in sflo) only asserts that a `sh:sparql` node with a matching `sh:message` substring and Violation severity exists. The behavioral test ("selected content-digest checks distinguish matching and mismatched evidence", line 385) exercises `validateDigestConsistency` (line 1054), a hand-rolled TypeScript reimplementation — the shipped `sh:select` strings are never parsed or run, so a reversed property direction or typo'd variable in any of the four new SPARQL constraints would pass CI. The reimplementation also diverges from the SPARQL it stands in for: `sameMethodDifferentValue` uses `value.split(":", 1)`, treating two colon-less values as different methods, while `STRBEFORE(STR(?d), ':')` returns `""` for both and treats them as the same method. The task's testing requirement ("mismatched pairs violate the consistency constraint", task line 222) is satisfied only by proxy logic.

### B3. Weave records an empty `observedArtifactResolutionSpec`

The ontology states "Concrete resolved coordinates are recorded through observedArtifactResolutionSpec" (`semantic-flow-core-ontology.ttl:523`) and the spec says an implementation "should put the most concrete coordinates it actually knows" there ([[sf.spec.2026-08-21-content-digest]] line 92). But repo-backed integrate builds the observation as `{observedContentDigest}` only (`src/runtime/integrate/integrate.ts:445-450`) while holding repository URL/ref/commit/path and the working-relative path in hand; the rendered evidence is `sflo:observedArtifactResolutionSpec [ a sflo:ArtifactResolutionSpec ]` — an observation of unstated bytes. The tests bake this in (`src/core/integrate/integrate_test.ts:302`, `tests/e2e/integrate_cli_test.ts:407`). Consequences: the evidence loses replay value the moment the mutable `working` binding changes, and the attestation property-path join advertised to Stagecraft (`observedArtifactResolutionSpec/(targetLocatedFile|targetManifestation)`) can never match anything Weave writes — import records only a `targetLocalRelativePath` string; integrate records nothing.

## ADVISORY

### A1. Expected/observed agreement is validated through one link direction only

The constraint at `semantic-flow-core-shacl.ttl:637-638` joins solely via `?requestedSpec sflo:hasResolutionObservation $this`. An observation that points at the expectation-bearing spec only through its mandatory `observedArtifactResolutionSpec` (exactly what spec line 92 encourages when the requested spec is the most concrete one) escapes the check, since `hasResolutionObservation` is optional. Spec line 98 claims agreement is validated "when both are present for a linked resolution" — the SHACL delivers half of that.

### A2. Mismatch-as-Violation is structurally incompatible with "appendable evidence records"

The ontology calls observations "appendable evidence records" (`semantic-flow-core-ontology.ttl:523`), yet the constraint at shacl line 637 compares the spec's current `expectsContentDigest` against every linked observation. With a mutable `working` binding, a legitimately updated expectation turns honest historical observations into violations without any verification having failed. Weave avoids this today only because it overwrites the fixed id `-observation-001` (`src/core/integrate/integrate.ts:602`) — that is, by not appending. The constraint also has no outcome guard, so the deferred failure-provenance extension (task line 135) cannot land later without weakening this exact shape.

### A3. The two-bearer prose has no SHACL teeth outside repository locators

Only the repository locator shapes received `sh:maxCount 0`. A digest on `HistoricalState`, `DigitalArtifact`, or `ArtifactHistory` — placements the decision log rules out, and which the old `hasContentDigest` comment used to bless (`HistoricalState` was a named example), so legacy data exists — passes validation silently. Task line 180 forbids only a closed rejection constraint; a Warning-severity "subject SHOULD be typed a known ContentDigestBearer" check, mirroring the typed-subject check for `expectsContentDigest` at shacl line 591, would enforce the "assert the narrower bearer type" guidance without closing the extension boundary.

### A4. `ManifestationLocatedFileDigestConsistencyShape` targets by class, not by link

With `sh:targetClass sflo:ArtifactManifestation` (shacl line 570), a node asserting `locatedFileForManifestation` and a digest but lacking the explicit type escapes the agreement check. The prose conditions on the link ("a LocatedFile that provides an ArtifactManifestation..."), so `sh:targetSubjectsOf sflo:locatedFileForManifestation` is both more faithful and strictly more robust.

### A5. Locator-digest prohibition is a migration dead end with a generic diagnostic

`src/runtime/mesh/inventory.ts:1384-1386` rejects a locator-level `hasContentDigest` with the caller's generic `parseErrorMessage` ("Could not parse source registry"). Pre-contract Weave itself wrote those digests (task line 126), so an existing mesh now fails to load with a message that names neither the digest nor the fix — and because inventory parsing is a prerequisite, re-running `weave integrate` cannot regenerate the registry; the user must hand-edit Turtle. This is the pattern the condition-specific-diagnostics work (PR #42) just moved away from.

### A6. Extract hashes decoded text, not the exact byte sequence

`src/runtime/extract/extract.ts:1338-1345` digests `TextEncoder().encode(contents)` where `contents` came from `Deno.readTextFile` (lines 1117, 1188, 1327) — UTF-8 BOMs are stripped and invalid bytes replaced before hashing. This violates spec line 99 ("compute SHA-256 over the exact resolved byte sequence without text normalization") and diverges from the resolver (`resolver.ts:689`, raw bytes) and integrate (`integrate.ts:543-550`, raw bytes): for a BOM'd source file, extract's `observedContentDigest` will never match a resolver-verified expectation for the same file.

### A7. Extract's observation evidence violates the observation shape it is typed into

Core and runtime extract render `sflo:observedAt` as a plain string (`src/core/extract/extract.ts:930-934`) on nodes explicitly typed `sflo:ArtifactResolutionObservation`, while the shape requires `xsd:dateTime` at Violation severity (shacl lines 647-653); integrate and import correctly append `^^xsd:dateTime`. Pre-existing, but it defeats the task's "generated digest values are ... SHACL-conforming" goal (task line 234) for the observation family this contract formalizes.

### A8. Negative tests promised by the task are missing

- Task line 230 requires rejection tests for "uppercase, abbreviated SHA-256, and unsupported algorithms": every new Weave negative test uses uppercase only (`src/runtime/artifact_resolution/resolver_test.ts:139-153`, `src/core/import/import_test.ts:115-158`). No test feeds a 63-hex or `sha512:`/`md5:` value to `--source-digest`, import `expectedDigest`, extract `sourceDigest`, or the resolver.
- Task line 232's "legitimate LocatedFile claims remain supported" is untested — no Weave test demonstrates a `LocatedFile` `hasContentDigest` claim being accepted or preserved.
- In sflo, `contentDigestFixture("a","b","c","d")` triggers the manifestation/file and expected/observed mismatches together (`tests/shacl_guardrails_test.ts:389-393`); neither is exercised in isolation, so a check firing on the wrong condition would still pass.

## NIT

- The `ContentDigestBearer` comment (`semantic-flow-core-ontology.ttl:59-61`) defines a bearer as a resource whose byte stream "is the subject of a content-digest claim," but subclassing makes every `LocatedFile` and `ArtifactManifestation` a member with or without any claim; "can be the subject" is what is meant.
- The revised `ArtifactResolutionSpec` comment (ontology line 519) now names manifestations as first-class targets, but the pre-existing concrete-target Warning SPARQL (shacl line 213) does not count `targetManifestation` (or `targetAccessUrl`) as satisfying evidence, so a manifestation-targeted spec with a local path warns spuriously.
- The uniqueness message "A ContentDigestBearer MUST NOT declare different content-digest values..." (shacl line 554) over-states the typing: the shape fires on any subject of `hasContentDigest`, typed bearer or not.

## Process Note

The sflo and framework halves of the contract are committed locally but unpushed (`936bf8b6`, `144db3d`); only Weave and the archive task note remain in working trees, matching the task's read-only-index remark at line 255.

## Disposition — 2026-08-21

The review was received after the initial cross-repository commits. Every blocking finding and every substantive advisory was accepted or resolved as follows.

### Blocking

- **B1 accepted.** `ArtifactResolutionObservationShape` now also targets every subject of `observedContentDigest`, so its datatype, grammar, uniqueness, required observed-spec, and timestamp constraints fire without RDFS entailment or an explicit observation type.
- **B2 accepted.** The parallel TypeScript digest-consistency evaluator was removed. SFLO CI now installs pinned PySHACL and executes the shipped SHACL-SPARQL against isolated positive and negative fixtures, including untyped malformed observations, same-method duplicates, link-direction errors, repository leakage, and downstream bearer subclasses.
- **B3 accepted.** Repository-backed `integrate` now records the concrete `targetLocalRelativePath` actually read in `observedArtifactResolutionSpec`; tests no longer bless an empty observed spec.

### Advisory

- **A1 and A2 accepted as one lifecycle correction.** The timeless expected/observed SHACL comparison was removed rather than widened. Runtimes enforce mismatch before successful use or persistence; SHACL must not compare a mutable current expectation against every appendable historical observation or preclude future failed-attempt evidence.
- **A3 accepted.** A warning-level SPARQL constraint asks digest subjects to carry an explicit `ContentDigestBearer` type or downstream subclass without closing the extension boundary.
- **A4 accepted.** Manifestation/file agreement now targets subjects of `locatedFileForManifestation`, not only explicitly typed manifestations.
- **A5 accepted.** Weave now reports the prohibited repository-locator digest and both migration destinations instead of the generic source-registry parse error.
- **A6 accepted.** Extract hashes `Uint8Array` source bytes before decoding; a UTF-8 BOM regression proves it no longer hashes decoded/re-encoded text.
- **A7 accepted and broadened.** Core extract, runtime extract, and weave extraction-source renderers emit `observedAt` as `xsd:dateTime`; core input validation rejects invalid lexical values.
- **A8 accepted.** Canonical-digest tests now cover uppercase, abbreviated values, and unsupported algorithms across the shared validator and operation surfaces; Weave covers digest-bearing `LocatedFile` acceptance, isolated mismatch behavior, exact-byte extraction, and CLI `--source-digest` rejection. PySHACL fixtures isolate each SHACL-SPARQL condition.

### Nits

- The bearer definition now says its bytes **can be** the subject of a claim.
- The local-resolution warning now recognizes `targetManifestation` as an exact coordinate. `targetAccessUrl` remains intentionally non-exempt because an access URL may be mutable and still needs an explicit working mode or digest expectation.
- The uniqueness message now says "a subject" rather than presuming explicit bearer typing.

The original Process Note was accurate at review time but is now historical: Weave landed as `c0daa57e`, and the initial task/archive follow-through landed as `0d9e78d` and `d7859b2` before this disposition pass.

### Validation Receipts

- SFLO: format and lint clean; type checks pass; 30 Deno RDF/structural guardrails pass; 11 isolated fixtures execute through PySHACL; release validation passes for the current v0.3.0 metadata.
- Weave: focused review suite passes 116 tests; full `deno task ci` passes 834 tests with format, lint, type checks, and coverage generation.

The Weave disposition landed as `2ec7606` (`fix(provenance): address content-digest review findings`).
