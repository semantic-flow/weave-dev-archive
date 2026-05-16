---
id: 2zk4eg4jfb5ib8ilmujj7re
title: 2026 05 16_1413 Coderabbi
desc: ''
updated: 1778966049674
created: 1778966049674
---

Verify each finding against current code. Fix only still-valid issues, skip the
rest with a brief reason, keep changes minimal, and validate.

Inline comments:
In @.github/workflows/release-manual.yml:
- [x] Line 63: The workflow references the non-existent tag denoland/setup-deno@v2;
update every occurrence of the action usage (lines where you have "uses:
denoland/setup-deno@v2") to a specific published patch tag such as
denoland/setup-deno@v2.0.4 (or another exact v2.x.x release) so the action
resolves correctly at runtime; leave other actions like actions/checkout@v6,
actions/setup-node@v6, actions/upload-artifact@v7, actions/download-artifact@v8
unchanged.
  - Also updated CI and moved workflow Deno pins to 2.7.14.

In `@documentation/notes/release-notes.v0.1.0.md`:
- [x] Around line 11-35: The release notes file currently contains placeholder
"TODO" entries under the headings "## Highlights", "## Breaking Or Changed
Behavior", "## Artifacts", "## Validation", "## Known Limitations", and "##
Next"; replace each TODO with concrete content for v0.1.0: fill "## Highlights"
with key features and user-visible changes, "## Breaking Or Changed Behavior"
with any API/behavior changes and migration notes, "## Artifacts" with
build/releases/assets and links, "## Validation" with test/QA status or coverage
notes, "## Known Limitations" with known issues and workarounds, and "## Next"
with planned follow-ups or roadmap items so the v0.1.0 release notes are
complete and accurate.

In `@documentation/notes/wu.repository-options.md`:
- [x] Line 5: Remove the manual update to the "updated" timestamp in the document:
revert or delete the line that sets `updated: 1778817945812` so the note does
not contain an explicit updated field; rely on Dendron to manage the `updated`
metadata automatically and ensure no other hard-coded `updated` entries remain
in this note.

In `@scripts/assemble-npm-packages.ts`:
- [x] Around line 278-283: The resolveRootPath function currently treats only POSIX
absolute paths (path.startsWith("/")) as absolute; update it to use Node's
cross-platform path.isAbsolute (or path.isAbsolute from 'path') to detect
Windows (e.g., "C:\\..." or UNC "\\\\...") and POSIX absolute paths, then return
the path unchanged when isAbsolute(path) is true; otherwise continue to
join(root, path). Ensure the file imports/uses the same path module symbol
(e.g., isAbsolute and join) that the rest of the file uses.

In `@scripts/build-binaries.ts`:
- [x] Around line 138-143: The resolveRepoPath function currently checks absolute
paths with path.startsWith("/") which fails on Windows; update resolveRepoPath
to use isAbsolute from `@std/path` (import isAbsolute) and return the input path
when isAbsolute(path) is true, otherwise return join(repoRoot, path); ensure the
import for join remains and replace the startsWith check with isAbsolute(path)
to correctly handle Windows-style absolute paths like C:\.

In `@scripts/bump-version.ts`:
- [x] Around line 94-104: The exported function bumpVersion currently assumes CLI
validated XOR of options.version and options.increment; add an explicit check at
the start of bumpVersion to enforce exclusivity: if neither options.version nor
options.increment is provided, or if both are provided, throw a clear,
descriptive error (e.g., "Either version or increment must be provided, but not
both") before calling requireVersionString/incrementVersion; reference the
options parameter and the variables options.version and options.increment so
callers get a consistent validation regardless of invocation source.

In `@scripts/package-binaries.ts`:
- [x] Around line 233-237: The resolveRootPath function incorrectly treats only
POSIX-style absolute paths by checking path.startsWith("/"); replace that check
with Node's cross-platform check (path.isAbsolute) so Windows paths (e.g.
C:\...) are detected as absolute and returned unchanged; update the
resolveRootPath implementation to call path.isAbsolute(path) and return path
when true, otherwise return join(root, path), and ensure the 'path' module is
imported where resolveRootPath is defined.

In `@scripts/publish-npm-packages.ts`:
- [x] Around line 288-292: The resolveRootPath function currently detects absolute
paths with path.startsWith("/") which fails on Windows; import and use
isAbsolute from `@std/path` (or node's path.isAbsolute equivalent) to check if the
provided path is absolute, returning it directly when true and otherwise
returning join(root, path); apply the same change for the identical patterns in
scripts/smoke-npm-install.ts (function around line ~340),
scripts/package-binaries.ts (around ~233) and scripts/assemble-npm-packages.ts
(around ~278) so all platform-aware absolute path checks use isAbsolute instead
of startsWith.

In `@scripts/smoke-npm-install.ts`:
- [x] Around line 51-54: The switch's "case \"--\": break;" only exits the switch,
not the surrounding argument-processing loop, so trailing args are still parsed;
update the handler for argument variable "arg" (the switch block) to stop
parsing when "--" is seen—either by breaking out of the outer loop (e.g., set a
stopParsing flag checked by the loop or perform a loop-level break/return)
instead of just breaking the switch, ensuring no further options are processed
after the delimiter.

In `@src/cli/run.ts`:
- [x] Around line 939-950: The deploy output never logs result.updatedPaths (only
materializedSource.updatedPaths), so publish updates like CNAME/.nojekyll
silently; update the printing logic in the block that calls
describeGHPagesDeployBootstrapResult(result) to iterate and console.log
result.updatedPaths exactly once (e.g., after printing result.createdPaths), and
stop separately re-printing materializedSource.updatedPaths since
result.updatedPaths already includes those; you can still print
materializedSource.createdPaths if needed, but ensure
materializedSource.updatedPaths is not double-logged.

In `@src/core/extract/extract.ts`:
- [x] Around line 239-245: normalizeNonEmptyLiteral currently trims but does not
escape control characters, allowing values containing \n, \r, or \t to produce
invalid Turtle; update normalizeNonEmptyLiteral to call the Turtle escaping
helper (escapeTurtleString) or extend that helper to also escape control chars
(at minimum replace \n, \r, \t with their escaped forms) and return the escaped
trimmed string so all evidence literals emitted (including where
normalizeNonEmptyLiteral is used) produce valid Turtle; apply the same fix to
the other similar occurrence noted around lines 895-897.

In `@src/runtime/deploy/gh_pages.ts`:
- [x] Around line 592-600: normalizeCname currently only trims and rejects
empties/newlines; change it to validate that the value is a bare hostname and
throw GHPagesDeployInputError for anything else. Specifically, inside
normalizeCname(value: string) after trimming, reject if the string contains a
scheme ("://"), path ("/"), port (":" followed by digits), whitespace, or
illegal chars (e.g. underscores), and enforce a hostname regex (labels of
letters/digits/hyphen, separated by dots, length limits) — if the check fails,
throw new GHPagesDeployInputError("cname must be a valid bare hostname"); keep
the existing empty/newline checks but replace the permissive return with this
stricter validation.

In `@src/runtime/weave/weave.ts`:
- [c] Around line 525-553: The helper loadEffectiveConfigForExecution currently
always calls loadWeaveDefaultEffectiveConfig() — change it to start from the
mesh/workspace resolved effective config instead of the defaults (i.e., replace
the call to loadWeaveDefaultEffectiveConfig() with the function that returns the
mesh/resolved workspace EffectiveConfig), then construct the new
EffectiveConfigValue from that resolved config (preserve its sources,
configResolution, namingPolicies, resourcePageRegenerationConfigPolicy and
resourcePageGenerationPolicyForArtifactRole) and only apply
historyTrackingPolicyOverride to defaultHistoryTrackingPolicy and
historyTrackingByRole (using ALL_ARTIFACT_ROLES), leaving all other policies
from the resolved config intact.
  - Cancelled for this review pass: there is no resolved workspace EffectiveConfig loader yet; this belongs in the grand config synthesis work rather than a release-review patch.
- [x] Around line 1383-1433: loadKnopSourceRegistryArtifact currently joins
workspaceRoot with workingLocalRelativePath straight from inventory, allowing
path traversal; before reading the file with readTextFileWithOverlay ensure the
same local-path policy used elsewhere is applied to
sourceRegistryState.workingLocalRelativePath (e.g., validate/normalize it
against workspaceRoot, reject or resolve any ../ segments outside the workspace,
or call the existing helper used for payload/reference-catalog path checks), and
only then call readTextFileWithOverlay; update error messages to reflect the
validated path and keep existing Deno.errors.NotFound handling in
loadKnopSourceRegistryArtifact.

In `@tests/e2e/deploy_gh_pages_cli_test.ts`:
- [x] Line 6: The test builds cliPath using new URL("src/main.ts",
repoRoot).pathname which yields Windows-incompatible paths; change cliPath to
use fromFileUrl(new URL("src/main.ts", repoRoot)) from `@std/path` instead, and
add the corresponding import for fromFileUrl (ensure
tests/e2e/deploy_gh_pages_cli_test.ts imports fromFileUrl) so the file URL is
converted to a platform-correct file path before use.

---

Outside diff comments:
In `@src/runtime/mesh/inventory.ts`:
- [c] Around line 281-308: The optional sourceRegistryTurtle is being turned into an
empty sourceRegistryQuads which makes subsequent lookups fail; change the
creation of sourceRegistryQuads so that when sourceRegistryTurtle is undefined
it falls back to using inventoryQuads instead of [], i.e. replace the current
ternary so the false branch returns inventoryQuads and otherwise calls
parseInventoryQuads(meshBase, sourceRegistryTurtle, messages.parseErrorMessage);
keep the later checks using hasNamedNodeObject(extractionSourceIri,
RDF_TYPE_IRI, SFLO_EXTRACTION_SOURCE_IRI) and the existing error throw with
messages.missingExtractionSourceMessage.
  - Cancelled: this would preserve legacy inline extraction-source facts in Knop inventories. Current direction is `_knop/_sources`; inline fallback should stay retired.

In `@src/runtime/weave/weave.ts`:
- [x] Around line 1332-1340: The code currently falls back to
toPayloadHistoricalSnapshotPath(...) for non-latest pinned states, which uses
the working filename and breaks for non-default manifestation names; change
selectedHistoricalSnapshotPath resolution to first look up the snapshot path
recorded in the artifact/inventory for the specific state (e.g. check
sourcePayloadArtifact.historicalSnapshots or a manifest/manifestation map keyed
by state such as sourcePayloadArtifact.historicalSnapshotPathMap[sourceState] or
sourcePayloadArtifact.manifestationByState[selectedHistoricalStatePath]) and use
that if present, only falling back to
toPayloadHistoricalSnapshotPath(selectedHistoricalStatePath,
sourcePayloadArtifact.workingLocalRelativePath) when no inventory entry exists;
update the logic around selectedHistoricalSnapshotPath to prefer
inventory-resolved snapshot paths over deriving from the working filename.

In `@tests/e2e/payload_update_cli_test.ts`:
- [c] Around line 32-36: The transient test input file is being written into
workspaceRoot (via sourcePath + Deno.writeTextFile using
readMeshAliceBioBranchFile), which contaminates workspace snapshot assertions;
change the test to write the temp source into a separate temp directory (e.g.,
create a temp dir with Deno.makeTempDir and set sourcePath there) or ensure the
file is removed before calling listRelativeFiles(...) by calling
Deno.remove(sourcePath) once the test step that needs it finishes; update
references to sourcePath/transitionCase accordingly so the test uses the
isolated temp path or deletes the file prior to snapshot comparison.
  - Cancelled: current Alice `a.10-alice-bio-updated` intentionally includes root `alice-bio-v2.ttl`; moving the test input outside the workspace makes the fixture comparison false.

---

Nitpick comments:
In `@src/core/integrate/integrate_test.ts`:
- [x] Around line 140-141: The test's semantic-normalization replacement currently
targets the concrete subject string "<_mesh/_inventory/_history001/_s0002/ttl>",
which breaks when fixture IDs are renumbered; update the replacement in
integrate_test.ts to use a path-agnostic pattern (e.g. match the "_s" segment
with digits or a wildcard) instead of the literal "_s0002/ttl" so both
occurrences ("<_mesh/_inventory/_history001/_s0002/ttl> a
sflo:ArtifactManifestation, sflo:RdfDocument ;" and
"<_mesh/_inventory/_history001/_s0002/ttl> rdf:type sflo:RdfDocument,
sflo:ArtifactManifestation ;") are normalized by the regex/pattern and not tied
to a specific fixture number.

In `@src/core/knop/create.ts`:
- [c] Around line 255-280: When attempting legacy shape detection in createKnop (the
try block that calls assertHasLegacyCurrentMeshInventoryShapeForKnopCreate), the
catch currently swallows KnopCreateInputError and falls back silently; update
the catch to log a debug-level message including the error details (e.g.,
error.message or error.stack) and context (that legacy detection failed and
we're falling back to assertHasWorkingCurrentMeshInventoryShapeForKnopCreate)
before continuing, while still rethrowing any non-KnopCreateInputError as
currently implemented.
  - Cancelled: we are trying not to deepen legacy support before v0.1.0, and this core planner path does not have an appropriate debug logger.

In `@src/runtime/extract/extract.ts`:
- [x] Around line 1743-1745: The escapeTurtleString function only escapes
backslashes and double quotes which can yield invalid Turtle when values contain
control characters; update escapeTurtleString to also escape newline, carriage
return, tab (and optionally backspace/form-feed) sequences (e.g., replace \n ->
\\n, \r -> \\r, \t -> \\t) and ensure replacements run on the original string
(or use a single pass replacer) so every control char is converted before
returning; locate the escapeTurtleString function and add these additional
escapes to its replacement logic.
- [x] Around line 1747-1749: The splitTurtleBlocks function currently uses
turtle.trim().split(/\n\s*\n/g) which splits on any blank line and can
incorrectly break Turtle multiline string literals or miss other block
boundaries; update the function by adding a clear comment above
splitTurtleBlocks stating that it deliberately splits on blank lines, that it
will not preserve blank lines inside multiline string literals (and therefore
may split inside """...""" or '''...''' literals), and note that a full
Turtle-aware parser would be needed to avoid this; optionally mention intended
acceptable input shape for callers (extraction source blocks) so future
maintainers know this limitation.
- [x] Around line 2005-2008: The current conditional mixes checks on
term.datatype.value and isUrlLiteral(term.value) such that when datatype ===
XSD_ANY_URI_IRI but isUrlLiteral(...) is false the function returns undefined;
change the logic so that if term.datatype.value === XSD_ANY_URI_IRI you return
term.value unconditionally, otherwise only return term.value when
isUrlLiteral(term.value) is true. Update the conditional that currently uses
XSD_ANY_URI_IRI and isUrlLiteral to a simpler OR-based check (or two-branch
check) so the XSD_ANY_URI_IRI case always yields term.value and non-anyURI
relies on isUrlLiteral.

In `@src/runtime/weave/pages.ts`:
- [x] Around line 2394-2403: The code calls
classifyHistoryComponentResourcePage(resourcePath, historyGroups) twice causing
duplicate work; fix by calling it once, storing the result in a local variable
(e.g., const classified = classifyHistoryComponentResourcePage(resourcePath,
historyGroups)), then if (classified) return classified; otherwise return the
rdfClass("sflo:DigitalArtifact", `${SFLO_NAMESPACE}DigitalArtifact`). Update the
block containing classifyHistoryComponentResourcePage, resourcePath,
historyGroups and the rdfClass fallback accordingly.
