---
id: a8k2v6m1q9r4x7p3n5s0tjc
title: 2026 08 14_1949 Content Digest Contract
desc: ''
created: 1786769777136
---

## Goals

- Define the content-digest methods supported by a Semantic Flow release.
- Establish one exact, interoperable wire grammar for `hasContentDigest`, `expectsContentDigest`, and `observedContentDigest`.
- Give `hasContentDigest` a precise domain covering exact manifestation bytes and independently checkable located-file bytes.
- Make the manifestation, location, resolution-expectation, and resolution-observation layers unambiguous.
- Align the SFLO ontology and SHACL, Semantic Flow behavior specs, and Weave runtime behavior.
- Preserve the existing digest property IRIs and already-canonical `sha256:<64 lowercase hex>` values.

## Summary

SFLO v0.3.0 defines three content-digest properties:

- `hasContentDigest`, currently with domain `rdfs:Resource`
- `expectsContentDigest`, with domain `ArtifactResolutionSpec` and `sh:maxCount 1`
- `observedContentDigest`, with domain `ArtifactResolutionObservation`

All three use `xsd:string`. Their comments recommend algorithm-qualified literals, while SHACL currently accepts a generic `algorithm:value` form at warning severity.

A downstream consumer wants to use `hasContentDigest` as SFLO's general predicate for the digest of bytes represented by resources outside the rest of its SF machinery. Before doing so, it needs SFLO to publish a digest-method vocabulary, identify the supported method and exact lexical form, and give `hasContentDigest` a meaningful domain.

This task keeps algorithm-qualified literals, adds a release-extensible method vocabulary, supports SHA-256 only in the next minor contract, and applies the exact grammar to all three properties. It also separates manifestations, located files, repository locators, expected bytes, and observed bytes: content-digest claims belong on exact `ArtifactManifestation` or `LocatedFile` bearers; pre-resolution requirements belong on `ArtifactResolutionSpec`; computed resolution evidence belongs on `ArtifactResolutionObservation`.

The domain choice is intentionally meaningful. Under RDFS semantics, a resource using `hasContentDigest` is inferred to be a `ContentDigestBearer`. SFLO directly classifies `ArtifactManifestation` and `LocatedFile` as bearers; downstream vocabularies may add narrower bearer subclasses when they can define one determinate or retrievable byte stream. `DigitalArtifact`, `ArtifactHistory`, `HistoricalState`, and repository locator classes are not bearers merely by belonging to the artifact or resolution model.

## Discussion

### Representation and Method Vocabulary

Keep the existing algorithm-qualified `xsd:string` representation:

`sha256:<64 lowercase hexadecimal digits>`

Do not split the method into a second property and do not introduce structured digest assertion nodes. Either alternative would migrate existing values without solving a current requirement. Keeping the token in the literal also leaves the value self-describing outside its original RDF graph.

Add an open controlled vocabulary following SFLO's existing direct-typed-member convention:

- `ContentDigestMethod`
- `contentDigestMethod_sha256`
- `contentDigestMethodToken`

`contentDigestMethod_sha256` is a `ContentDigestMethod`, not a `DigitalArtifact`. Digest values remain literals and are not modeled as artifacts or nodes.

Use `contentDigestMethodToken` rather than the more generic `digestMethodToken`, following the explicit relational naming guidance in [[ont.dev.guidance]]. Make it a functional datatype property whose value is the lowercase token preceding `:` in the wire form.

The vocabulary is extensible across releases. Each release's SHACL grammar is closed over the methods that release actually supports. Do not use `owl:NamedIndividual`, `owl:oneOf`, or SKOS for this vocabulary.

### Digest Bearers and the Manifestation Byte-Identity Invariant

Add `ContentDigestBearer` as the domain of `hasContentDigest`, with a narrow definition: a resource whose own represented or retrievable byte stream is the subject of a digest claim. Make `ArtifactManifestation` and `LocatedFile` subclasses. Do not make the whole `DigitalArtifactFacet` lattice or `DigitalArtifact` a subclass.

Tighten its semantics so one `ArtifactManifestation` denotes one exact byte sequence. Multiple `LocatedFile` resources may serve or locate the same manifestation only when they provide byte-identical replicas. If the bytes differ, the resources represent different manifestations even when they share a media type or broad serialization family.

A digest on an `ArtifactManifestation` is a claim about the exact bytes represented by that manifestation. A digest on a `LocatedFile` is a standing, independently falsifiable claim about bytes retrieved through that file identity or location. Either RDF assertion may be wrong. When both bearer levels declare a digest using the same supported method, the values must agree; disagreement is a SHACL violation because the file is asserted to provide that manifestation's exact bytes.

Consequences include:

- formatting, line-ending, compression, canonicalization, packaging, or serialization changes that alter bytes create a different manifestation
- moving or mirroring identical bytes may create another `LocatedFile` without creating another manifestation
- a `HistoricalState` may have multiple manifestations and therefore has no single intrinsic content digest
- a continuing `DigitalArtifact` or `ArtifactHistory` has no single intrinsic content digest
- a `LocatedFile` identifies a retrievable file identity or location whose byte claim can be checked independently
- a `RepositorySourceLocator` identifies repository coordinates; it is not the content those coordinates resolve

Revise `LocatedFile` so its comment distinguishes a retrievable file/location identity from the exact representation identity supplied by `ArtifactManifestation`. Revise the `RepositorySourceLocator` comment to remove its current permission to carry `hasContentDigest`.

Do not add `ContentAddressedArtifact`. A new content-addressed artifact class would introduce an artifact-identity model that this task does not need. Sparse or external file use cases can use a `LocatedFile` bearer directly; predicates that accept an exact facet should say so rather than relying on range inference to turn a file into a governing `DigitalArtifact`.

### Downstream Migration and Attestation Query

Stagecraft currently carries a common chain in which an `ArtifactManifestation` links to a digest-bearing `LocatedFile`. Under the approved two-bearer contract that file-level claim remains canonical. A consumer may also move or copy the same digest to the manifestation when it wants to assert exact representation identity; if both are present they must match.

Document a worked example with one authored file and one published mirror represented as two byte-identical `LocatedFile` resources for one `ArtifactManifestation`, including matching digest claims.

No new direct observation-to-bearer property is needed in the first slice. Existing vocabulary supports a standard property-path join from `ArtifactResolutionObservation` through `observedArtifactResolutionSpec` to `targetLocatedFile` or `targetManifestation`, then to the bearer's standing digest claim. Include that query in the behavior spec so consumers need not rediscover it.

### Resolution Semantics and Naming

Keep the name `ArtifactResolutionSpec`.

The spec may begin with artifact-, history-, state-, manifestation-, file-, URL-, path-, or repository-level coordinates and resolve them to a byte stream. Calling it `ManifestationResolutionSpec` would incorrectly suggest that the input or selected target must already be a modeled manifestation. Calling it `ByteResolutionSpec` would obscure the artifact/state selection expressed by resolution modes such as `artifactResolutionMode_latestState`.

Clarify its comment instead: an `ArtifactResolutionSpec` selects and retrieves bytes associated with its artifact or source coordinates, potentially by selecting an `ArtifactManifestation`, but successful resolution does not require minting a persistent manifestation merely to record operational evidence.

For resolution data:

- `expectsContentDigest` is a pre-existing requirement supplied by a caller, authored policy, or durable source binding
- `observedContentDigest` is a digest computed from bytes actually resolved during an event
- `hasContentDigest` is a standing digest claim about bytes represented by an `ArtifactManifestation` or retrieved through a `LocatedFile`

Do not promote an observed digest into an expectation after the fact. In particular, Weave integration must stop copying a digest it just computed into `expectsContentDigest` merely to pin the same operation retroactively.

When an expected and observed digest exist for one linked resolution, mismatch is failed verification. The bytes must not be used as a successful result, and the operation must not persist the mismatch as though it were a successful resolution observation. Failure-event provenance remains a separate future concern.

Repository locators that resolve directory trees do not acquire digest semantics through this contract. A tree digest needs an explicit canonical byte representation and is out of scope.

### Family Coverage and Wire Grammar

Apply the supported grammar to all three properties:

- `hasContentDigest`
- `expectsContentDigest`
- `observedContentDigest`

Use this exact SHACL pattern at `sh:Violation` severity:

`^sha256:[0-9a-f]{64}$`

Keep `expectsContentDigest` at `sh:maxCount 1`. Do not make the other two properties globally functional because future releases may support simultaneous digests using different algorithms.

Within any one subject or observation, at most one distinct value may be asserted for the same digest method. Enforce same-method uniqueness with SHACL rather than making the multi-method properties globally functional.

### Compatibility and Release Boundary

This is an intentional pre-1.0 contract correction, not a compatibility shim:

- data using `hasContentDigest` on a subject not otherwise typed as a bearer now infers `ContentDigestBearer`; active SF data should use an explicit bearer class when one is known
- existing `LocatedFile` digest claims remain canonical and do not need migration solely because of the domain change
- Weave-generated source bindings and fixtures using repository-locator digests must migrate or be regenerated
- computed integration digests must move to `ArtifactResolutionObservation`
- only caller- or policy-supplied digest requirements remain in `expectsContentDigest`
- runtime input and persisted RDF must use the exact lowercase SHA-256 form

Do not add a transition-warning period. The contract belongs in the next minor release after v0.3.0. Do not change ontology version metadata, mint release resources, tag, or publish as part of this feature task; those actions remain governed by [[ont.dev.release-runbook]].

## Open Issues

No modeling issues remain open for the first slice. Additional algorithms, tree/RDF canonicalization, failure-event provenance, and any future split between artifact selection and byte retrieval require separate decisions.

## Decisions

- Keep algorithm-qualified `xsd:string` digest literals.
- Add `ContentDigestMethod`, `contentDigestMethod_sha256`, and functional datatype property `contentDigestMethodToken`.
- Model neither the SHA-256 method nor digest values as `DigitalArtifact` resources.
- Support only lowercase SHA-256 in the next release contract.
- Keep the method vocabulary open across releases while closing each release's SHACL grammar over its supported members.
- Cover `hasContentDigest`, `expectsContentDigest`, and `observedContentDigest` with the same release grammar.
- Add `ContentDigestBearer` as the sole `rdfs:domain` of `hasContentDigest`.
- Make `ArtifactManifestation` and `LocatedFile` subclasses of `ContentDigestBearer`.
- Define an `ArtifactManifestation` as one exact byte sequence, possibly served through multiple byte-identical `LocatedFile` replicas.
- Treat byte differences as distinct manifestations even when format, media type, or conceptual content are otherwise the same.
- Treat a `LocatedFile` digest as an independently checkable standing claim about retrievable bytes.
- Require matching manifestation/file digest values when both assert the same supported method.
- Reject multiple distinct standing or observed digest values for the same method on one subject.
- Treat `RepositorySourceLocator` as a coordinate resource rather than a content-digest bearer.
- Keep standing, expected, and observed digest assertions on `ContentDigestBearer`, `ArtifactResolutionSpec`, and `ArtifactResolutionObservation`, respectively.
- Keep the `ArtifactResolutionSpec` name and clarify its byte-resolution semantics rather than renaming the family.
- Do not promote computed observations into expectations.
- Treat expected/observed mismatch as failed verification.
- Do not add `ContentAddressedArtifact`, structured digest nodes, canonicalization vocabulary, or additional algorithms in this task.
- Land the contract through the next minor release without compatibility aliases or a transition-warning profile.

## Contract Changes

### SFLO Core Ontology

- Add `ContentDigestMethod`.
- Add `contentDigestMethod_sha256` with token `"sha256"`.
- Add functional datatype property `contentDigestMethodToken`.
- Add `ContentDigestBearer`.
- Make `ArtifactManifestation` and `LocatedFile` subclasses of `ContentDigestBearer`.
- Change the domain of `hasContentDigest` from `rdfs:Resource` to `ContentDigestBearer`.
- Revise `ArtifactManifestation` to express the exact-byte-sequence and replica invariant.
- Revise `LocatedFile` to distinguish its retrievable file/location role from manifestation identity while retaining file-level digest claims.
- Revise `RepositorySourceLocator` to remove locator-level digest semantics.
- Revise `ArtifactResolutionSpec` to clarify artifact/source selection and byte retrieval.
- Revise all three digest-property comments to state the exact wire form and the standing/expected/observed distinctions.

### SFLO SHACL

- Replace the generic warning-level digest pattern with `^sha256:[0-9a-f]{64}$` at violation severity for all three properties.
- Retain datatype constraints and the `expectsContentDigest` maximum cardinality.
- Do not add a closed subject-type constraint that rejects downstream `ContentDigestBearer` subclasses; the domain remains an extensible open-world boundary.
- Add a violation when an `ArtifactManifestation` and one of its `locatedFileForManifestation` values declare different SHA-256 digests.
- Add violations for multiple distinct standing or observed digest values using the same method on one subject.
- Add or refine expected-versus-observed consistency validation where the requested spec and observation are linked.

### Semantic Flow Framework

Create or update [[sf.spec.2026-08-21-content-digest]] covering:

- the exact supported wire form
- manifestation byte identity, located-file claims, and byte-identical replicas
- standing, expected, and observed digest placement
- fail-closed mismatch behavior
- repository locator and tree-resolution exclusions
- the RDFS consequence of using `hasContentDigest`
- a worked authored-file/published-mirror example and observation property-path query

Correct [[sf.spec.2026-04-04-integrate-behavior]] and [[sf.spec.2026-05-18-publication-source-binding]], which currently permit or describe computed digest evidence as `expectsContentDigest` and locator-level `hasContentDigest`.

### Weave

- Stop emitting `hasContentDigest` on `RepositorySourceLocator`.
- Stop promoting a newly computed integration digest into `expectsContentDigest`.
- Keep caller- or policy-supplied expected digests on the relevant `ArtifactResolutionSpec`.
- Record computed resolution digests on an `ArtifactResolutionObservation` linked from the source spec.
- Emit or preserve `hasContentDigest` only for `ArtifactManifestation` or `LocatedFile` bearers, never for repository locators.
- Align expected-digest validation with the lowercase-only grammar; reject uppercase, short, malformed, and unsupported-algorithm input rather than persisting or case-normalizing it as canonical RDF.
- Preserve fail-closed mismatch behavior before resolved bytes are used or successful observation evidence is written.
- Remove obsolete canonical rendering/parsing assumptions rather than adding long-lived pre-1.0 compatibility shims.
- Regenerate or migrate active fixtures that contain locator-level digests or retroactively authored expectations.

## Testing

In `sflo`:

- Assert the new method class, member, token property, domain, range, and functional-property declaration.
- Assert every direct SFLO-authored `ContentDigestMethod` member has exactly one unique token.
- Assert all three digest shapes use the same pattern and violation severity.
- Assert the supported vocabulary tokens and SHACL grammar remain in parity.
- Test accepted lowercase SHA-256 examples.
- Test rejection of uppercase hex, wrong lengths, non-hex characters, missing prefixes, empty values, and unsupported algorithms.
- Assert the bearer domain, `ArtifactManifestation`/`LocatedFile` subclass edges, and updated class comments/relationships through RDF graph guardrails where appropriate.
- Test that matching manifestation/file digest pairs conform and mismatched pairs violate the consistency constraint.
- Test same-method uniqueness while leaving the properties open to future multi-method values.
- Add structural guardrails for expected/observed consistency constraints.
- Run `deno task ci`.

In Weave:

- Test acceptance of the exact canonical expected-digest form.
- Test rejection of uppercase, abbreviated SHA-256, and unsupported algorithms.
- Test that caller-supplied expectations remain expectations while computed integration values become observations.
- Test that repository locators no longer render `hasContentDigest` while legitimate `LocatedFile` claims remain supported.
- Test fail-closed expected/observed mismatch.
- Test that generated digest values are lowercase and SHACL-conforming.
- Run focused artifact-resolution, import, integrate, inventory, and fixture tests followed by `deno task ci`.

Across active repositories:

- Search live RDF, specs, examples, and fixtures for locator-level `hasContentDigest`, retroactively generated `expectsContentDigest`, and non-canonical digest literals.
- Historical conversation and completed-task notes remain historical and must not be mechanically rewritten.

## Implementation Progress

- SFLO core now defines the method vocabulary, two-bearer domain, exact manifestation/replica invariant, canonical SHA-256 comments, and corrected resolution/locator lifecycle.
- SFLO SHACL now enforces the exact grammar, same-method uniqueness, manifestation/file agreement, expected/observed agreement, and repository-locator prohibition.
- [[sf.spec.2026-08-21-content-digest]] now carries the portable behavior, mirror example, and attestation property-path query; integrate and publication-source-binding specs no longer describe retroactive expectations or locator digests.
- Weave now shares one canonical digest validator, rejects noncanonical expected values, keeps caller expectations distinct from observations, stops rendering repository-locator digests, and fails closed when parsing them from active source registries.
- Existing `LocatedFile` claims remain valid. The carried Weave fixture suites required no branch regeneration; focused expectations and parser fixtures were updated in place.
- Validation completed with SFLO format/lint, type checks, 31 tests, and release validation; SFLO check/test used a temporary validation lock because the restricted environment could not fetch one uncached JSR manifest. Weave `deno task ci` passed with 829 tests.

## Repository Commits

- SFLO: `936bf8b6` — `feat(ontology): define the SHA-256 content-digest contract`
- Semantic Flow Framework: `144db3d` — `docs(spec): define portable content-digest behavior`
- Weave: `c0daa57e` — `fix(provenance): separate expected and observed digests`
- weave-dev-archive: `0d9e78d` — `docs(task): record content-digest contract delivery`

Stagecraft confirmed all four questions in [[ont.disc.2026-08-21_0923-response-to-stagecraft-requirements]] on 2026-08-21: the two-bearer scope, existing attestation property-path, separate DCAT media profile, and `v0.4.0` release-notice boundary are acceptable.

## Non-Goals

- Additional digest algorithms.
- Multihash, SRI, `ni:` URI, or structured digest-node adoption.
- Modeling digest methods or digest values as digital artifacts.
- A `ContentAddressedArtifact` class or automatic bearer status for the entire digital-artifact/facet lattice.
- A general checksum or signature ontology.
- Canonicalization of RDF datasets, directory trees, archives, or other abstract content.
- Treating `DigitalArtifact`, `ArtifactHistory`, `HistoricalState`, or `RepositorySourceLocator` as digest bearers merely because of their existing SFLO class.
- Renaming or splitting `ArtifactResolutionSpec` in this task.
- Changing the existing digest property IRIs.
- Updating the shelved downstream estate.
- Cutting, tagging, or publishing the next SFLO release.
- Adding compatibility aliases for pre-1.0 behavior.

## Implementation Plan

- [x] Re-read [[ont.dev.guidance]], [[ont.summary.core]], [[ont.dev.decision-log]], [[ont.dev.release-runbook]], [[wd.general-guidance]], and this task before editing.
- [x] Record the digest representation, method vocabulary, manifestation byte-identity, resolution evidence, and compatibility decisions in [[ont.dev.decision-log]].
- [x] Add the method vocabulary, `ContentDigestBearer`, bearer subclass edges, and digest domain to `semantic-flow-core-ontology.ttl`.
- [x] Update `ArtifactManifestation`, `LocatedFile`, `RepositorySourceLocator`, `ArtifactResolutionSpec`, and digest-property comments.
- [x] Tighten all three SHACL digest patterns and add manifestation/file and expected/observed mismatch constraints.
- [x] Add SFLO graph, grammar, parity, positive, and negative tests.
- [x] Update [[ont.summary.core]] and other live ontology guidance.
- [x] Add or update the Semantic Flow content-digest behavior spec and correct affected integrate/publication-source-binding specs.
- [x] Update Weave integration/source-binding output so computed digests become observations rather than locator facts or retroactive expectations.
- [x] Align Weave validation, persistence, and parsing behavior with the canonical grammar and placement rules.
- [x] Regenerate or migrate active fixtures affected by repository-locator digest removal or retroactive expectations; retain valid `LocatedFile` claims.
- [x] Search active code, live RDF, specs, and fixtures for old placement and non-canonical literal forms.
- [x] Run `deno task ci` in SFLO and Weave, plus applicable framework documentation checks.
- [x] Prepare detailed semantic commit messages separately for SFLO, Semantic Flow Framework, Weave, and weave-dev-archive.
- [x] Leave release metadata and publication to the next release task.
