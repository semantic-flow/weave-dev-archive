---
id: 9p65fo83rc0xds4dr542ss0
title: Exact-Byte Binary Payload Advancement
desc: 'Cut 2026-08-01 14:11 by Jimbo from the wd.todo binary item, wa.completed.2026.2026-07-21_1322 r1 F3, and requirement 4 of the Stagecraft requirements collection. Drafted by codex read-only analysis (code-verified at 73a26cc); awaiting spec adjudication (wa.dave-court card) before implementation.'
updated: 1785618665282
created: 1785618665282
---


## Goals

- Preserve opaque binary payload bytes exactly across `versionPayloads` admission, plan-only overlay use, working-file update, first HistoricalState creation, later HistoricalState advancement, retry/no-op comparison, and historical snapshot assertions.
- Correct the shared core/runtime path so CLI `version` and the programmatic API create byte-identical binary snapshots without decode/re-encode loss, for both cardinality-one and coherent multi-target requests.
- Preserve the existing UTF-8 text/RDF contract unchanged while ensuring binary payloads and their manifestations/located files are not falsely rendered as `sflo:RdfDocument`.
- Establish one end-to-end binary fixture spine proving first advancement, later advancement, preservation of prior snapshots, exact-byte working state, no-op rerun, and bounded file-backed snapshot verification.

## Summary

This task is the dedicated follow-up required by [[wa.completed.2026.2026-07-21_1322-programmatic-version-api]] r1 F3, the Current Work item in [[wd.todo#current-work-and-next-pick]], and requirement 4 in [[wa.task.2026.2026-06-30_1108-stagecraft-driven-semantic-flow-requirements#evidence-backed-stagecraft-persistence-requirements-2026-08-01]]. The current `versionPayloads` release deliberately supports only UTF-8 text/RDF payloads; binary admission was deferred until working update, first/later history creation, no-op comparison, and snapshots could be owned as one exact-byte slice.

The r1 F3 code claim remains correct at repository commit `73a26cc`: the runtime loader retains `currentPayloadBytes`, the no-op comparator already compares binary bytes, and first payload versioning already emits a `PlannedBinaryFile`, but later advancement emits decoded `currentPayloadTurtle` through `createdFiles`. The current code audit also exposes an adjacent semantic defect not named in F3: both later-payload renderer branches reconstruct binary payload, manifestation, and located-file facts with `sflo:RdfDocument`.

The primary product path in this task is the existing composed `versionPayloads` operation: its admission-copied bytes must remain authoritative through pre-write planning, working update, and history creation. The shared planner/runtime correction also makes file-backed CLI `version` safe for existing binary working payloads. Whether the standalone `payload update <file>` command is generalized from text to binary in the same slice is called out explicitly under Open Issues rather than silently implied.

## Discussion

### Current byte-preservation boundary

| Surface | Current behavior | Task implication |
| --- | --- | --- |
| Programmatic admission | Each `Uint8Array` is copied, but every copy is immediately decoded with fatal UTF-8 behavior (`src/api/version_payloads.ts:236-267`); LOAD then admits only a text-like extension with non-empty decoded text (`src/api/version_payloads.ts:483-490`). | Retain admission copying, but defer decoding until the resolved target is classified. Opaque binary bytes must never pass through `TextDecoder`. |
| Programmatic overlay and working update | The admitted content is staged in `TextFileOverlay` and converted to a text `PlannedFile` (`src/api/version_payloads.ts:319-347`). The combined manifest permits binary creates but models working updates only as `PlannedFile`, and all non-`binary-create` entries use `Deno.writeTextFile` (`src/api/version_payloads.ts:564-608`, `src/api/version_payloads.ts:662-685`, `src/api/version_payloads.ts:934-945`). | Add a byte-capable overlay/read seam and a binary-update write shape. A binary working update must use `Deno.writeFile` with the admitted copy. |
| Runtime payload loading | Non-text-like working files retain raw `currentPayloadBytes`, and non-text historical snapshots retain `latestHistoricalSnapshotBytes` (`src/runtime/weave/artifact_loaders.ts:106-120`, `src/runtime/weave/artifact_loaders.ts:139-157`). The loader nevertheless also creates lossy `currentPayloadTurtle` with a non-fatal decoder, and a staged binary value would currently be re-encoded from overlay text (`src/runtime/weave/artifact_loaders.ts:199-220`). | Preserve the byte fields and stop treating lossy decoded text as an eligible binary planning value. Binary overlay reads must return the exact staged bytes. |
| Retry/no-op comparison | `payloadContentsMatchLatest` selects byte comparison whenever `currentPayloadBytes` exists and compares length plus every byte (`src/core/weave/weave.ts:1001-1065`). | This is already byte-safe. Regression-pin it with byte sequences that decode to the same replacement-character text so a future text fallback cannot pass. |
| First payload history | `planFirstPayloadWeave` selects `currentPayloadBytes`, omits the text snapshot, and emits the historical payload through `createdBinaryFiles` (`src/core/weave/weave.ts:1169-1333`). A focused unit test already covers this planner output (`src/core/weave/weave_test.ts:1811-1869`). | Preserve this behavior and extend proof through the real API/runtime writer; the current unit proof cannot exercise binary API admission. |
| Later payload history | `planLaterPayloadWeave` always emits the new payload snapshot as `contents: payloadArtifact.currentPayloadTurtle` in `createdFiles` (`src/core/weave/weave.ts:1778-1839`). | This confirms r1 F3. Later binary snapshots must follow the first-history `createdBinaryFiles` path using `currentPayloadBytes`. |
| Later payload rendering | The first-payload renderer already has content-kind-aware artifact, manifestation, and located-file helpers (`src/core/weave/payload_renderers.ts:43-97`). The later renderer accepts no payload content-kind option (`src/core/weave/payload_renderers.ts:510-520`); its simple second-state template hard-codes `sflo:RdfDocument` on the payload, manifestations, and located files (`src/core/weave/payload_renderers.ts:582-697`). Its multi-history branch does the same for the payload and located files (`src/core/weave/payload_renderers.ts:258-365`, `src/core/weave/payload_renderers.ts:404-490`) and for every rendered manifestation (`src/core/weave/payload_renderers.ts:1024-1034`). | Carry the resolved payload content kind into every later renderer branch and preserve non-RDF typing for existing and newly added binary states. |
| File-backed batch capture | Multi-target CLI preparation reads each working payload with `Deno.readFile`, hashes the raw bytes with SHA-256, and repeats that raw-byte hash before planning proceeds (`src/runtime/weave/version_execution.ts:698-851`). | This is already binary-safe within its landed multi-target scope. Retain and add a binary mutation probe; do not replace it with decoded comparison. |
| Standalone payload update | Core update accepts `replacementPayloadTurtle: string`, requires `sflo:RdfDocument`, and produces only `PlannedFile` updates (`src/core/payload/update.ts:11-28`, `src/core/payload/update.ts:137-203`). Runtime reads the source and atomically stages/writes it as text (`src/runtime/payload/update.ts:226-259`, `src/runtime/payload/update.ts:407-459`). | This path is not already binary-safe. Its inclusion must be ruled explicitly; API working update alone must not be described as standalone CLI-update support. |

`PlannedBinaryFile` and `VersionPlan.createdBinaryFiles` already provide a binary-created-output shape (`src/core/planned_file.ts:1-9`, `src/core/weave/version_plan.ts:3-9`), and both the coherent batch merger and runtime writer already preserve binary creates. The missing plan vocabulary is binary updates, needed for the programmatic working file and, if included, explicit overwrite or standalone payload update.

### Byte authority and capture

For `versionPayloads`, each caller-supplied view remains copied at ADMIT. The admitted copy is authoritative; later caller mutation and mutation of a shared backing buffer are invisible. Text/RDF targets are strictly decoded and retain their existing non-empty/content validation. Binary targets skip decoding, whitespace rules, and RDF parsing.

The API does not gain a changed-under-capture comparison against the old working file. Planning uses the admitted overlay, and another writer changing disk after plan-green remains outside the caller-owned single-writer contract from [[wd.programmatic-version-api]] and [[wd.decision-log]]. File-backed CLI batches retain their existing raw-byte initial-hash/verify window.

“Snapshot verification” has two distinct meanings here: runtime input-snapshot verification remains the bounded raw-byte CLI capture guard; historical snapshot verification is an executable acceptance assertion that reads the resulting files and compares exact bytes. This task does not introduce a runtime read-after-write durability guarantee.

### Content-kind boundary

The implementation needs one shared content classification instead of duplicating the API’s extension regex and the loader’s private `isTextLikePayloadPath`. The proposed rule is:

- An inventory-declared `sflo:RdfDocument` remains text/RDF and must satisfy the existing strict UTF-8 and content validation.
- A non-RDF payload with a supported text-like working extension remains text and keeps the existing UTF-8/non-empty behavior.
- A non-RDF payload with a non-text-like working extension is opaque binary. Valid-looking UTF-8 does not reclassify it as text.
- Opaque binary may contain NULs, invalid UTF-8, every byte value, and zero bytes; a zero-length binary payload is valid and must compare and persist as zero length.
- File extension continues to determine the default manifestation segment and snapshot filename, but never authorizes transcoding or byte normalization.

### Historical and semantic invariants

Normal binary advancement must add only the requested first or next HistoricalState, its manifestation, its located snapshot, and the usual owned support/inventory progression. Every older payload snapshot remains byte-identical. The current payload artifact, its binary manifestations, and their located files remain non-`sflo:RdfDocument`; support Turtle remains RDF and continues through the text validation path.

## Acceptance Criteria

- `versionPayloads` accepts an eligible mesh-local opaque binary payload using its existing `Uint8Array` request field, including invalid UTF-8, NUL-containing, high-bit, and zero-length byte sequences.
- Every request item is copied independently at ADMIT, respecting view offset and length. Caller mutation, shared buffers, and overlapping views cannot alter planned or written bytes.
- Binary items remain byte-valued through overlay loading and planning. No `TextDecoder`/`TextEncoder`, string comparison, Unicode normalization, newline conversion, or whitespace rule participates in their persistence.
- Any ADMIT, LOAD, or PLAN refusal leaves binary and text working files, historical snapshots, inventories, and support files unchanged, preserving the whole-request pre-write refusal contract.
- A first binary history creates a binary historical snapshot exactly equal to the admitted bytes and writes the working file exactly when its bytes differ.
- A later binary history creates the next binary historical snapshot exactly equal to the admitted bytes, advances the expected current/latest relations, and leaves every earlier snapshot byte-identical.
- Simple second-state and general later/multi-history rendering preserve the binary payload’s non-RDF artifact, manifestation, and located-file types while support artifacts remain RDF.
- Binary no-op comparison uses exact length-and-byte identity. Identical bytes plus identical resolved naming return `alreadyCurrent` with no created or updated paths; different bytes that decode to the same replacement-character string still advance.
- Mixed text/binary coherent batches work at cardinality one and greater than one, remain deterministically ordered, merge shared support outputs once, and allow already-current binary members to coexist with applied text or binary members.
- `dryRun: true` performs the same binary LOAD/PLAN/preflight path, forecasts binary working updates and snapshots accurately, and writes nothing.
- The existing file-backed multi-target snapshot guard hashes binary working files as raw bytes, refuses a change during its covered capture window before writes, and retains the existing post-verification boundary.
- Text/RDF behavior remains regression-identical: fatal UTF-8 validation, non-empty eligibility, planned RDF parsing, no-op results, working update, first/later snapshots, and typed refusal families do not weaken.
- Binary write failures report the existing `WeaveApiError` write-stage partial-path evidence, distinguishing completed binary creates from completed binary updates without inferring recovery from filename conventions.
- Any binary combination not supported by the final overwrite or standalone-update ruling refuses before mutation; it may not fall through to a text planner.
- [[sf.spec.2026-07-21-programmatic-version-api]], [[wd.programmatic-version-api]], [[wd.codebase-overview]], user-facing command documentation if affected, and release notes describe the final binary boundary without retaining the v1 “no binary payload support” fence.

## Open Issues

- [ ] **Standalone `payload update` inclusion.** Proposed lean: include it. r1 F3 identified its text-only read/plan/write path, and leaving it unchanged prevents true CLI update-plus-version equivalence for binary payloads. If excluded, Goals, Contract Changes, Testing, and documentation must say that “working update” means only the composed `versionPayloads` write and that operators must update binary working files outside `payload update`.
- [ ] **Binary `overwriteExistingState`.** Proposed lean: include single-item binary overwrite because the public API already exposes overwrite and `src/core/weave/payload_overwrite.ts:145-155` currently routes it through text. If implementation risk warrants deferral, binary overwrite must refuse before planning with a documented stable code and receive its own follow-up; silent decode is forbidden.
- [ ] **Canonical content classifier.** Proposed lean: one shared helper governed by inventory `sflo:RdfDocument` status plus the established text-like extension set, used by API eligibility, overlay loading, current/latest snapshot loading, update, and tests. Do not introduce MIME sniffing.
- [ ] **Plan shape.** Choose `updatedBinaryFiles` alongside existing `createdBinaryFiles` or a discriminated planned-file union. Lean: explicit `createdBinaryFiles`/`updatedBinaryFiles` fields for the smallest compatible change, with collision and result-path handling centralized across both.
- [ ] **Release vehicle.** Proposed lean: ship as the next minor release after `0.6.0`, because this broadens a stable public API and the packaged `@semantic-flow/weave-lib` contract.

## Decisions

- **Inherited:** `Uint8Array` remains the only programmatic payload representation. There is no binary string/base64 overload.
- **Inherited:** caller-owned single-writer serialization remains the concurrency contract. No API/CLI lock, filesystem transaction, journaling, or rollback mechanism is added.
- **Inherited:** admitted API copies are authoritative and are planned before any working or history write. The API does not compare them to a second capture of the pre-call working file.
- **Exactness rule:** equality means identical byte length and identical byte at every offset. No decoded-text equivalence is acceptable for binary content.
- **Zero-length rule:** an eligible opaque binary payload may be zero length; the existing rejection of empty/whitespace-only text remains unchanged.
- **Semantic typing rule:** opaque binary payload artifacts, manifestations, and located files are not rendered as `sflo:RdfDocument`; no new binary-specific ontology class is required by this task.
- **History rule:** normal first/later advancement is append-only with respect to prior payload snapshots. Existing support/inventory policy remains governed by the resolved configuration.
- **Verification rule:** raw-byte historical assertions and the existing bounded file-backed capture guard are required; runtime read-after-write durability verification is not.
- **API rule:** this is an additive eligibility expansion of `versionPayloads`, not a new public operation or request shape.

## Contract Changes

- `VersionPayloadsRequest`, `VersionPayloadItem`, `VersionPayloadsResult`, and `PayloadVersionOutcome` retain their existing public shapes. Eligible binary content becomes supported through the existing `bytes: Uint8Array` field.
- `unsupported-content` no longer means that every invalid-UTF-8 byte sequence is rejected. Invalid UTF-8 remains a refusal for text/RDF targets; eligible opaque binary targets bypass UTF-8 validation. Content-kind mismatch remains a LOAD refusal.
- `dryRun`, canonical result ordering, `applied`/`alreadyCurrent`, request-level created/updated path lists, partial-write evidence, and all existing identity/source/naming restrictions remain unchanged.
- Internal plan/write contracts gain byte-valued update support and later-state binary creation. RDF preflight operates only on planned text/RDF files.
- Shared `version` planning stops decoding later binary snapshots and later renderers preserve their content kind. This corrects CLI/runtime behavior for existing binary working payloads without changing CLI invocation syntax.
- If the standalone-update issue is included, `payload update <file> <designatorPath>` reads and atomically installs raw bytes for eligible binary targets while preserving current text/RDF validation for text targets.
- [[sf.spec.2026-07-21-programmatic-version-api]] is amended before implementation because binary admission changes externally visible Semantic Flow behavior. [[wd.programmatic-version-api]] carries Weave-specific classification, overlay, plan, writer, error, and example details.
- The packaged `@semantic-flow/weave-lib` contract and its off-tree smoke gain a binary case; the CLI wrapper/library distinction in [[wd.library-packaging]] does not change.

## Testing

- Follow [[wd.testing]] with a spec-first update, fail-on-old regression tests, focused core tests, real-filesystem integration tests, and packaged-library smoke coverage.
- Build one deterministic opaque-binary fixture with bytes chosen to expose text corruption, including NUL, invalid UTF-8, high-bit bytes, and sequences such as `0x80` versus `0x81` that both non-fatally decode to the replacement character.
- The end-to-end spine starts with a binary payload that has no declared history, calls `versionPayloads` to create `_s0001`, advances it with different bytes to `_s0002`, reruns the identical second request for `alreadyCurrent`, and at each rung asserts with `Deno.readFile` that the working file and selected snapshot equal the admitted bytes and that all prior snapshots remain unchanged.
- Extend that spine with `dryRun`, caller-buffer mutation after admission, a zero-length binary state, and a mixed text/binary coherent batch. Each test must state which old behavior it defeats.
- Add core planner tests proving first and later binary payloads are emitted only through binary plan collections, batch merging detects text/binary path collisions, and byte-identical retries skip.
- Add renderer tests for both the simple `_s0002` branch and the general later/multi-history branch, proving payload artifacts, manifestations, and located files do not gain `sflo:RdfDocument` while RDF support files retain it.
- Add file-backed multi-target capture probes that mutate a binary working file after the initial raw hash and after verified capture, preserving the existing refusal and authority boundaries.
- Add write-failure injection at binary working update, binary snapshot create, and binary support/update boundaries, checking completed-created, completed-updated, and possibly-touched paths.
- If standalone `payload update` is included, add source-file integration and CLI end-to-end tests that copy non-UTF-8 bytes exactly and then version them without text conversion.
- Add an off-tree Node/npm-library smoke that supplies an opaque `Uint8Array`, checks working and historical bytes, and reruns to `alreadyCurrent`.
- Preserve and run all existing text/API/CLI equivalence tests. Before merge run `deno task fmt`, `deno task ci`, and any Accord-backed transition added for the portable behavior.

## Non-Goals

- MIME detection, content sniffing, transcoding, compression, newline normalization, image processing, preview generation, or interpreting opaque binary payload contents.
- New ontology vocabulary solely to label “binary”; existing non-`RdfDocument` payload/manifestation/located-file semantics are sufficient for this slice.
- Remote, repository/floating, or outside-policy working-source mutation.
- Recursive targets, payload-IRI request identity, new-payload creation, or changes to naming/policy precedence.
- Locks, transactions, journaling, rollback, automatic recovery from partial writes, or runtime read-after-write durability guarantees.
- Whole-mesh validation, page-generation redesign, extraction of terms from binary payloads, or broad ResourcePage raw-source presentation changes.
- Generalizing every `PlannedFile` consumer when an explicit created/updated binary field is sufficient.
- Unrelated append-only inventory work, extractor defects, validation scaling, renderer cleanup, or Stagecraft press-side wiring.
- JSR publication or other packaging work beyond keeping the existing npm library contract correct.

## Implementation Plan

- [ ] Adjudicate the classifier, standalone-update, overwrite, plan-shape, and release issues; amend [[sf.spec.2026-07-21-programmatic-version-api]] and [[wd.programmatic-version-api]] before code.
- [ ] Add the deterministic binary fixture and failing end-to-end first → later → no-op byte-preservation spine, including the decode-collision and zero-length probes.
- [ ] Centralize payload content classification and generalize the planning overlay/read path to stage and retrieve copied bytes without decode/re-encode conversion.
- [ ] Extend plan, collision/preflight, dry-run manifest, result-path, partial-write, and writer handling for binary updates while preserving existing binary creates.
- [ ] Route later binary payload snapshots through the byte-valued create path; regression-pin the already-correct first-history and byte-comparison paths.
- [ ] Make both later payload renderer branches content-kind-aware and verify that prior and new binary manifestations/located files remain non-RDF.
- [ ] Implement and test binary overwrite and standalone `payload update` according to the adjudicated rulings; any excluded combination must fail closed before mutation.
- [ ] Add mixed-batch, snapshot-capture, failure-injection, text-regression, CLI, and off-tree npm-library coverage.
- [ ] Update [[wd.codebase-overview]], command/API documentation, behavior spec, release note, and [[wd.todo]]; run `deno task fmt` and `deno task ci`.
- [ ] Run a fresh close review, adjudicate findings, land, then let the planning seat close/rename the task note and update queue/backlog receipts.

