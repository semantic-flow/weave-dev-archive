---
id: c8d55c05-e67f-4e6a-9f60-5dea15532e6e
title: Founding Referent Data Claude Review
desc: 'Read-only Claude Opus max-effort adversarial review of the first FoundingReferentData task draft'
created: 1787423391000
---

## Scope

Read-only Claude Opus review of [[wa.task.2026.2026-08-22_1112-founding-referent-data]] against the live SFLO ontology and SHACL, ontology guidance and the historical `ReferentMetadata` removal, the Semantic Flow `knop.create` behavior spec, and Weave's create, inventory-preservation, validation, digest, and test code. The requested lenses were semantic distinctness, vocabulary and RDF entailments, the exact-`D`/no-blank-node profile, exact-byte lifecycle, create/adopt boundaries, later operation interactions, 552-entry scale, and missing tests or scope traps. No files were edited by the reviewer.

## BLOCKING

### B1. The founding document's base IRI is unspecified, and the worked example is unresolvable

Task lines 26–35 show a bare relative subject `<characters/new-npc>` with no `@base` and no `stage:` prefix. Line 88 requires "every triple's subject is exactly the public referent IRI formed from `meshBase + designatorPath`". Those reconcile only if a base is pinned, and the task never says whose.

Every Weave-authored document emits `@base <${meshBase}> .` (`src/core/knop/create.ts:155`, `:168`, `:822`) and every Weave reader supplies it (`src/core/weave/rdf_helpers.ts:28`, `new Parser({ baseIRI: meshBase })`). Neither mechanism is available here:

- Weave cannot prepend `@base` — Decisions line 163 digests "the exact accepted file bytes"; prepending changes them.
- Parsing with `baseIRI: meshBase` doesn't help the client. Line 132 and Testing line 226 require the client to read `data.ttl` bytes directly. A client resolving against the document's own URL (RFC 3986) gets `…/characters/new-npc/_knop/_founding/characters/new-npc`, not `D`.

The subject rule, the digest rule, and the client-discovery rule cannot all hold.

**Fix:** rule it explicitly. Cleanest: require all terms to be absolute IRIs and reject relative IRIs in the first profile — self-contained bytes, base-independent, and the subject check becomes a string comparison. Fix the example accordingly.

### B2. Ordinary `weave` silently drops the founding slot, and nothing in the plan catches it

Weave regenerates the Knop inventory wholesale (`src/core/weave/knop_inventory_renderers.ts:41-139`) and re-attaches only what the preservation pass knows. `src/core/weave/knop_support_renderers.ts` knows exactly two artifacts:

- `:50-56` `SUPPORT_SINGLE_VALUED_PREDICATES`
- `:450-478` `renderKnopSupportLinkFacts` — source registry + reference catalog only
- `:89-93` — early-returns the rendered inventory unchanged when neither is present

A Knop whose only extra artifact is `_founding` hits that early return. `hasFoundingReferentData`, the artifact typing, `hasWorkingLocatedFile`, and `hasContentDigest` are all discarded on first weave, with no error. `data.ttl` stays on disk, orphaned — destroying the purpose stated at line 12.

Line 201 says "teach … preservation to accept the optional artifact," which understates it. Testing line 225 only checks create-time state; nothing asserts survival across weave.

**Fix:** state preservation as a required contract change, and require a round-trip test: create-with-founding → weave → slot, typing, located file, and digest all still present; `data.ttl` byte-identical.

### B3. Exact-byte preservation is not expressible in the models being extended

Decisions line 163 / Contract Changes line 197 require exact bytes and a digest over the originals. The path is string-typed end to end:

- `src/core/planned_file.ts:1-4` — `PlannedFile { contents: string }`
- `src/core/knop/create.ts:64` — `createdFiles: readonly PlannedFile[]`
- `src/runtime/knop/create.ts:249` — `Deno.writeTextFile(...)`

`PlannedBinaryFile { contents: Uint8Array }` already exists at `src/core/planned_file.ts:6-9` and is unused by this task. Through the string path: `Deno.readTextFile` decodes with a default `TextDecoder`, which strips a UTF-8 BOM; non-well-formed UTF-8 becomes U+FFFD; lone surrogates round-trip lossily. Digest-over-source and digest-over-written then diverge. Line 195's "extend the models" is too vague to force this.

**Fix:** require founding bytes as `Uint8Array` through request → plan → write via `PlannedBinaryFile` / `Deno.readFile` / `Deno.writeFile`; digest exactly the bytes that will be written; test a BOM-prefixed and a CRLF document.

### B4. Reopens an Active SFLO decision without engaging it or reconciling the notes it contradicts

Three live sources say the opposite:

- `dependencies/github.com/semantic-flow/sflo/notes/ont.summary.core.md:130` — "Substantive RDF about a referent should normally live in a payload artifact or dataset rather than in a support artifact."
- `dependencies/github.com/semantic-flow/sflo/notes/ont.reference-links.md:104` — same.
- `dependencies/github.com/semantic-flow/sflo/notes/ont.dev.decision-log.md:245-258` — 2026-04-02, active — `ReferentMetadata` was removed because it "kept reopening the question of whether substantive RDF belonged in support artifacts or payload artifacts." Also `weave-dev-archive/notes/ont.completed.2026.2026-04-01-ReferenceCatalog.md:118`.

`FoundingReferentData` is structurally substantive referent RDF in a Knop-owned support artifact. Lines 14 and 102–112 assert distinctness, but create-only + flat profile + single subject is new policy, not an argument that the 2026-04-02 reasoning lapsed. The actually defensible reason — a payload would make `D` byte-bearing, contradicting Stagecraft's own `stage:byteBearing false` — appears only obliquely in Non-Goals line 237.

SFLO Contract Changes are purely additive: no amendment to the summary, none to reference-link guidance, and no ontology decision-log entry.

**Fix:** add a Discussion subsection answering "why not a payload artifact"; schedule amendments to both guidance notes and a dated decision-log entry naming the decision it narrows.

## MAJOR

### M1. Create-only plus no update or delete is unrecoverable

The draft fixes a create-only lifecycle, but Weave has no delete command and create refuses when targets exist. One wrong template means manual surgery across 552 directories plus hand-repair of the mesh inventory. Decide the minimum escape hatch, or ship a documented manual-repair procedure.

### M2. Three Open Issues are already decided elsewhere in the note

Adoption is open but assumed and tested; bounds are open but enforced and tested; `SemanticFlowResource` is open but already a superclass in Contract Changes. Resolve them before queueing.

### M3. The digest is decorative

Nothing in Weave verifies a standing `hasContentDigest`. The draft's claim that the digest "makes later accidental mutation detectable" is false as scoped. Either add verification to `validate` or withdraw the claim.

### M4. `--founding-data <path>` bypasses the established local-path policy

Every other caller-supplied-path command loads local-path policy first. The draft mentions neither policy nor whether the path resolves from the command working directory or `--mesh-root`.

### M5. No Accord manifest or fixture branch

The current `knop.create` spec names the acceptance layer, the Alice Bio examples have conformance manifests, and the CLI end-to-end test drives from it. The task amends the spec but adds no manifest and no fixture branch, so the founding path gets no acceptance coverage at the layer used for every other Knop operation.

### M6. The 552-entry smoke has nothing to measure with

Memory instrumentation is weave-only. `knop.create` has no timing or memory instrumentation, and the existing pending-heavy harness writes files directly rather than calling `executeKnopCreate`. Either scope the smoke to wall-clock or add instrumentation explicitly.

### M7. Log content-freeness has an open leak path

Runtime create logs `error.message` to both loggers, and N3 parse errors can embed offending tokens and lines. Require fixed diagnostics and a test asserting that no source substring reaches logs.

### M8. Root designator unhandled

Root creates are supported. For root, `D = meshBase`, which ends in `/`, colliding with slashless-canonical preferences. Rule the exact subject IRI, or refuse founding data on the root Knop in slice one.

## ADVISORY

### A1. `knop add-reference` is the closest precedent and an unconsidered alternative surface

It plans a Knop-owned support artifact, creates its file, updates inventory, and reuses preservation across inventory shapes. A `knop add-founding-data` command would leave create untouched. The create-time approach may be simpler, but needs an explicit ruling.

### A2. The global DigitalArtifact shape already enforces one working located file

Only the `LocatedFile` plus `RdfDocument` typing half is new.

### A3. Severity is unspecified for the optional slot

Current optional Knop slots use `sh:Warning`; say so explicitly.

### A4. Quadratic MeshInventory rewrite is structurally certain

Current create performs whole-inventory parse/scans and a full rewrite for every Knop. At roughly 200 bytes per Knop the reviewer estimates about 30 MB of aggregate parse/write over 552 creates. Deferring batch create is tenable, but the task should state the expected shape so the receipt confirms or falsifies it.

### A5. Atomicity needs more than preflight

Create writes files sequentially and overwrites inventory last. A third file widens the mid-write window. Testing covers only preflight failure, not a mid-write failure or rollback.

### A6. `SemanticFlowResource` typing knowingly conflicts with deferred pages

The summary says pages should accompany every `SemanticFlowResource`, and woven inventories give every artifact a page. This may be acceptable temporarily but must be recorded explicitly.

### A7. Check commands do not match repository convention

Use `deno task fmt` and `deno task ci` for Weave. SFLO's `ci` does not include all named SHACL engines, so name Jena and comparison commands explicitly when plural-engine receipts are required.

### A8. Extract a shared SHA-256 helper

Four copies already exist; do not add a fifth.

### A9. Stagecraft motivation is not verifiable in the repositories

The 552-entry requirement and predicates appear only in the draft. The prior Stagecraft response listed Knop-per-high-volume-identifier guidance among explicit non-asks. Reconcile that prior statement with this new ask.

## NIT

- The vocabulary naming open issue answers itself; the proposed names already follow ontology naming guidance.
- Add a `toFoundingReferentDataPath` helper following sibling path helpers.
- Add negatives for relative versus absolute IRIs, a `D#fragment` subject, workspace path escape, language-tagged and typed literal objects, and a comments-only zero-triple document.
- Add the ordinary backlog/queue documentation follow-through required by general guidance.
- The draft's observation that current KnopMetadata shape checks are open is correct.

## Verdict

**NO-GO as written.**

B1 and B2 each defeat the task's stated purpose, B3 makes the digest contract unenforceable through the models being extended, and B4 leaves live ontology guidance self-contradicting.

The reviewer judged the core idea defensible: a narrow, Knop-owned, create-only slice is distinct from `KnopMetadata`, `ReferenceCatalog`, and `PayloadArtifact`; the exact-subject/no-blank-node profile is a real constraint rather than `ReferentMetadata` respelled; the SHACL/file-membership analysis is correct; and deferring batch create is tenable.

Re-review after resolving the four blockers, ruling M1/M2/M4/M5, implementing or withdrawing M3, and explicitly scoping M6/M8. The expected next verdict would be GO WITH CHANGES.

