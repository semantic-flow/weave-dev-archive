---
id: kwcinhs7z9zr3kpj40c8riv
title: 2026 05 25_2230 Coderabbit
desc: ''
updated: 1779773448142
created: 1779773445462
---
Verify each finding against current code. Fix only still-valid issues, skip the
rest with a brief reason, keep changes minimal, and validate.

## Disposition

- [x] Removed deprecated `word-break: break-word` from the default stylesheet metadata header and `pre` rules, keeping `overflow-wrap: anywhere`.
- [x] Hyphenated "default-generated" in [[wu.resource-pages]].
- [x] Added shared `xsd:dateTime` lexical validation for import and integrate observation timestamps, with rejection tests.
- [x] Fixed the exact extraction source test helper to match `alice/data`.
- [x] Reworked runtime import writes to stage working bytes and RDF sidecars before a rollback-capable commit.
- [x] Avoided dangling `sflo:hasResolutionObservation` links when source evidence produces no observation facts.
- [x] Removed stale extraction-source observation blocks when replacing runtime extraction source bindings.
- [x] Allowed target-only ReferenceSource links while still rejecting multiple requested target states.
- [x] Sorted ResourcePage panel selection target page kinds, target artifact roles, and data requirements.
- [c] Cancelled the `currentColor` casing nit: CSS keywords are case-insensitive, and `currentColor` is the conventional spelling used by the CSS specs and common authoring practice.
- [c] Cancelled the font-family quote nit: quoted family names are valid CSS, clearer for names with spaces, and not worth churn.

Inline comments:
In `@defaults/stylesheet.css`:
- Line 22: The CSS rule for the selector .wf-metadata th uses the deprecated
value word-break: break-word; remove that declaration and rely on the existing
overflow-wrap: anywhere (or, if you want aggressive breaking, replace it with
word-break: break-all) so update the .wf-metadata th rule to drop word-break:
break-word (or swap to word-break: break-all) and keep overflow-wrap: anywhere
for modern, supported behavior.
- Line 85: The CSS rule for the pre selector uses the deprecated value
word-break: break-word; remove that declaration and rely on overflow-wrap:
anywhere (already present) or replace with word-break: break-all if you want
aggressive breaking; update the pre rule (and any parent pre rules that also
include word-break: break-word) to eliminate the deprecated value so only
overflow-wrap: anywhere (or word-break: break-all when appropriate) remains.

In `@documentation/notes/wu.resource-pages.md`:
- Line 23: Replace the phrase "Most pages are default generated pages." with the
hyphenated compound "Most pages are default-generated pages." so the modifier is
properly hyphenated; locate the exact sentence "Most pages are default generated
pages." in the document and update it to "Most pages are default-generated
pages." ensuring surrounding punctuation and capitalization are unchanged.

In `@src/core/import/import.ts`:
- Around line 420-425: The observedAt value is only checked for non-empty text
(via normalizeNonEmptyLiteral) but not validated as a proper xsd:dateTime, so
invalid timestamps may be emitted; update the code that sets observedAt (the
observedAt variable using observation.observedAt and normalizeNonEmptyLiteral)
to run an xsd:dateTime/ISO-8601 validator (e.g., validateXsdDateTime or
equivalent) and reject/normalize values that do not pass (return undefined or
surface an error) before the serialization block that emits xsd:dateTime (the
serializer code handling observedAt around the xsd:dateTime emission). Replace
or augment normalizeNonEmptyLiteral with the validator, or add a small helper
(validateXsdDateTime) and call it both when assigning observedAt and before
serializing, ensuring only valid xsd:dateTime values are emitted.

In `@src/core/integrate/integrate.ts`:
- Around line 438-443: The observedAt value is only checked for non-empty text
via normalizeNonEmptyLiteral but later rendered as an xsd:dateTime, risking
invalid typed literals; update the observedAt handling (the observedAt variable
and the code that serializes it where you emit xsd:dateTime) to validate the
value against an xsd:dateTime/ISO-8601 pattern before accepting it—add or reuse
a helper like isValidXsdDateTime(value) and only call
normalizeNonEmptyLiteral/assign observedAt if the helper returns true (otherwise
set undefined or throw/log a validation error) so only valid dateTimes are
emitted.

In `@src/core/weave/weave_test.ts`:
- Around line 3742-3743: The test helper's regex still matches "alice/bio" so
the replacement no-ops for fixtures that use "alice/data"; update the helper
regex in src/core/weave/weave_test.ts to match the literal string
`sflo:hasTargetArtifact <alice/data>` (or change the captured group from
`alice/bio` to `alice/data`) so the template line ``sflo:hasTargetArtifact
<alice/data> ;\n  sflo:hasRequestedTargetState <${sourceStatePath}> .`` is
correctly replaced; ensure the updated pattern targets that exact substring used
in the test helper.

In `@src/runtime/import/import.ts`:
- Around line 175-184: The current sequence writes the working file with
writeWorkingFile(meshRoot, plan, workingFileExisted) before completing
writeCreatedFiles and writeUpdatedFiles, risking partial commits on later
failure; change the flow to stage all target file writes first (perform
writeCreatedFiles and writeUpdatedFiles into temporary paths or a staging area
and ensure assertCreateTargetsDoNotExist/assertUpdatedTargetsExist pass), then
atomically write or rename the working file (or perform an atomic commit/rename
of all staged files) so that writeWorkingFile is only applied after all
created/updated file writes succeed; update the functions involved
(writeWorkingFile, writeCreatedFiles, writeUpdatedFiles, and any staging
helpers) to support staging/atomic swap and rollback on error.

---

Outside diff comments:
In `@src/core/weave/extraction_source_blocks.ts`:
- Around line 36-112: The code currently adds the sflo:hasResolutionObservation
fact in the facts array inside renderExtractionSourceBlocks (using
observationPath and sourceEvidence) even when
renderExtractionSourceObservationBlock(observationPath, sourceEvidence) returns
undefined, causing a dangling reference; change the construction so you first
call renderExtractionSourceObservationBlock(observationPath, sourceEvidence) to
get observationBlock and only include the ["sflo:hasResolutionObservation",
`<${observationPath}>`] fact when observationBlock is truthy, then build
sourceBlock from the adjusted facts array and return either
`${sourceBlock}\n\n${observationBlock}` or sourceBlock as before.

In `@src/runtime/extract/extract.ts`:
- Around line 1793-1825: The code emits a separate observation block
`<${extractionSourcePath}-observation-001>` but replaceExtractionSourceBinding
only replaces the ExtractionSource block, leaving stale observation blocks
behind; update replaceExtractionSourceBinding to also find and replace (or
remove) the associated observation block named
`<${extractionSourcePath}-observation-001>` when rewriting an extraction source,
using the same deterministic observationPath naming as in this helper (or match
the `-observation-001` suffix), so that both the `<${extractionSourcePath}>`
block and its observation block are replaced together; reference the
functions/values renderExtractionSourceObservationBlock, extractionSourcePath
and observationPath when locating and replacing the old blocks.

In `@src/runtime/mesh/inventory.ts`:
- Around line 1049-1091: The code incorrectly returns undefined when
referenceTargetStatePaths.size === 0 (making valid target-only reference sources
fail); remove the early "return undefined" and instead only throw if
referenceTargetStatePaths.size > 1 so that zero requested-state entries are
allowed; locate the block handling referenceTargetPaths and
referenceTargetStatePaths (symbols: referenceTargetPaths,
referenceTargetStatePaths, resolveOptionalUniqueNamedNodePath,
SFLO_HAS_REQUESTED_TARGET_STATE_IRI,
messages.missingReferenceLinkMessage/missingReferenceTargetMessage) and change
the logic to accept an empty referenceTargetStatePaths (treat as no requested
state) while still enforcing exactly-one for referenceTargetPaths and erroring
when more than one requested-state is present.

---

Nitpick comments:
In `@defaults/stylesheet.css`:
- Line 54: The font-family declaration in the rule "code, pre { font-family:
"SFMono-Regular", Consolas, "Liberation Mono", monospace; }" uses unnecessary
quotes around SFMono-Regular and Liberation Mono; update the rule to use
unquoted identifiers for valid CSS names (e.g., change "SFMono-Regular" and
"Liberation Mono" to SFMono-Regular and Liberation Mono respectively) so the
font-family becomes a list of proper identifiers followed by monospace.
- Line 44: The CSS uses the keyword currentColor with mixed casing in the
.wf-term rule; update the value to lowercase (currentcolor) wherever it appears
(e.g., the .wf-term selector and the other occurrence noted) to normalize
keyword casing for consistency across the stylesheet.

In `@src/runtime/config/effective_config.ts`:
- Around line 984-1033: The arrays derived from RDF object sets
(targetPageKinds, targetArtifactRoles, and dataRequirements) must be sorted to
produce a stable ResourcePagePanelSelectionProfile; update the return
construction so that targetPageKinds and targetArtifactRoles use
.map(...).sort() (or call .sort() on their resulting arrays) and sort the
dataRequirements array (e.g., replace the current dataRequirements usage with
dataRequirements.sort() or assign dataRequirements = dataRequirements.sort()
before return) so all three are lexically ordered; reference the variables/data
constructors: dataRequirements, targetPageKinds, targetArtifactRoles, and the
helper collectKnownNamedNodes that produces .identity values.
