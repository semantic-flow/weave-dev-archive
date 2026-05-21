---
id: fqbhy6al0tztz6w1d90ch3v
title: 2026-05-21_0907 import
desc: ''
updated: 1779379649208
created: 1779379649208
---

## Goals

- Add a first-class `weave import` command for materializing outside-origin bytes into a governed mesh-local working artifact.
- Make the first implementation small enough to land quickly: copy/fetch bytes, write a mesh-local working file, register or update the artifact, and stop before versioning/page generation.
- Backfill the command/runtime story for the carried Bob `20/21` import-boundary fixture described in [[wa.task.2026.2026-04-13_1245-bob-import-boundary-for-page-source]].
- Preserve the semantic boundary between `import`, `integrate`, `payload update`, `workingAccessUrl`, and page-source resolution.
- Keep source acquisition explicit and fail-closed: import may actively read/fetch the requested source, but normal `weave` / `generate` should not start following remote sources just because import metadata exists.

## Summary

`weave import` should be the friendly copy-acquisition surface that Weave is currently missing. The command should take bytes from a local path, `file:` URL, or explicitly requested HTTP(S) URL, place those bytes at a mesh-local working file, and make that file the current governed working surface for a designator path.

This is different from `weave integrate`: `integrate` registers an existing working source where it already lives, including extra-mesh local sources and floating repository locators. `import` creates the in-mesh working copy first, then records the governed artifact around that local copy. That distinction matters for branch-published and page-customization workflows where generated pages should follow a stable local artifact boundary, not a contributor's source checkout or a live remote URL.

This is also different from [[wa.task.2026.2026-05-20_2152-workingAccessUrl]]. `workingAccessUrl` is a current-byte locator that later runtime operations might follow when remote access policy permits it. `import` is an explicit acquisition command: the operator says "take these bytes now and put them here." After import succeeds, ordinary versioning and page generation should follow the mesh-local working file.

## Discussion

The first useful slice is Bob page content:

```sh
weave import \
  "https://raw.githubusercontent.com/djradon/public-notes/refs/heads/main/user.bob-newhart.md" \
  bob/page-main \
  --working-file bob-page-main.md
```

That command should materialize `bob-page-main.md`, register `bob/page-main` as a governed payload artifact if it is not already registered, and record the remote origin as source metadata such as `sflo:workingAccessUrl`. Bob's ResourcePageDefinition can then target `bob/page-main`; page generation follows the local `sflo:hasWorkingLocatedFile <bob-page-main.md>` surface rather than fetching the URL during generation.

This can be easy if the first implementation avoids pretending to solve every import-shaped problem:

- require an explicit `--working-file <meshRelativePath>` for the first slice rather than inferring path names too early
- support plain byte copy/fetch without transforms
- keep `weave import` create/update behavior focused on current working files, not historical states
- record HTTP(S) source URL metadata when the source is a URL, but do not require full remote-runtime policy before the user can run an explicit import command
- fail if the target file exists unless the user explicitly asks to overwrite or update it
- leave page-definition repointing, `weave version`, `weave generate`, and publication validation as separate commands

There is a real design edge: if HTTP(S) import lands in the first slice, Deno dev/test permissions need to allow network access for the import tests and `deno task dev:root` may need a permission adjustment or documentation. Native/package users will experience this as a normal command, but source-tree development uses Deno permissions.

The current carried Bob fixture uses older namespace vocabulary in some historical branches. Implementation should follow the current Weave/SFLO namespace constants and should not copy stale fixture namespace strings into new production code.

## Open Issues

- Should `--working-file` be required for the first slice, or should Weave infer a default from the designator path and source extension? Recommendation: require it first; add inference later after examples settle.
- Should the first slice support HTTP(S) fetch immediately, or should it support local/file sources plus optional origin metadata first? Recommendation: support HTTP(S) if bounded fetch and test permissions are straightforward; otherwise land local/file import first and keep the command shape ready for HTTP(S).
- What should the update flag be called when the designator or working file already exists: `--overwrite`, `--replace-working`, or `--update`?
- Should an import into an existing payload artifact update only the working file, or also refresh source-origin metadata? Recommendation: update both when the command is explicitly replacing working bytes.
- Should imported Markdown-bearing artifacts continue using the current `PayloadArtifact`, `DigitalArtifact`, and `RdfDocument` carried shape until content-kind modeling improves? Recommendation: yes for this task; do not mix import with a type-model cleanup.
- Should `--expected-digest sha256:...` be part of the first CLI surface for HTTP(S) URLs? Recommendation: yes if HTTP(S) import ships in the first slice, because mutable branch URLs are otherwise intentionally latest-ish.
- Which source-origin metadata belongs in the Knop source registry versus directly on the payload artifact? The Bob fixture records the remote source URL on the governed artifact; if the current ontology has a more precise source registry shape, use it deliberately and update the note.

## Decisions

- `weave import` is an active acquisition command. It reads or fetches source bytes at command time and writes a mesh-local working file.
- `weave import` should not run `weave`, `weave version`, or `weave generate` automatically.
- The first CLI shape should be:

```sh
weave import <source> <designatorPath> --working-file <meshRelativePath> [--mesh-root <meshRoot>]
```

- The imported artifact's current bytes are the mesh-local `sflo:hasWorkingLocatedFile`, not the outside source.
- HTTP(S) source URLs, when supported, should be recorded as origin/current-source metadata such as `sflo:workingAccessUrl`, but page generation should continue following the local working file unless a later task explicitly implements remote current-byte following.
- Existing local path access grants are still relevant for ongoing extra-mesh source binding through `integrate`, but import should not require a persistent local-path grant merely to copy explicitly requested bytes into the mesh.
- First-slice overwrite behavior should be explicit. A command that would replace an existing working file should fail unless an update/overwrite flag is supplied.
- First-slice implementation should preserve existing path safety rules: imported working files must be mesh-relative files, must not escape the mesh root, and must not land under reserved support segments unless a later command intentionally handles support artifacts.

## Contract Changes

- Add a `weave import` CLI command.
- Add core/runtime import planning for materializing source bytes to a mesh-local working file and registering the designator path as a governed payload artifact when missing.
- The command should produce a stdout summary plus created/updated paths, matching the existing CLI style.
- If the designator path is new, import should create the same essential support surface as a first-time payload integration: Knop metadata, Knop inventory, payload artifact registration, mesh inventory update, and the working file.
- If the designator path already exists and the caller supplies the chosen update/overwrite flag, import should replace the working file and refresh import-origin metadata without minting a historical state.
- HTTP(S) import should use bounded fetch behavior: scheme restriction, timeout, maximum byte size, redirect decision, digest calculation, and clear diagnostics.
- Add user documentation for `weave import` and link it from [[wu.cli-reference]], [[wu.repository-options]], and the relevant release notes when the command ships.
- Tighten existing docs language that mentions "integration/import" so it does not imply the command exists before this task lands.

## Testing

- Unit-test import request normalization: designator path, `--working-file`, root designator handling, reserved path rejection, empty source rejection, and overwrite/update flag behavior.
- Core-plan tests for the new-artifact path: created working file, Knop support files, payload artifact block, mesh inventory update, and recorded origin metadata.
- Runtime tests for local filesystem source import and `file:` URL import.
- If HTTP(S) lands in the first slice, add a local HTTP-server integration test for successful import, timeout/404 diagnostics, max-size rejection, digest mismatch, and no live remote fetch during later page generation.
- Add e2e coverage for `weave import ... bob/page-main --working-file bob-page-main.md` followed by a normal `weave` / `generate` flow where Bob's page uses the governed local artifact.
- Add regression coverage that import does not persist absolute local source paths in public RDF by default.
- Run focused tests for import, integrate, payload update, page-definition weave/generate, then `deno task lint`, `deno task check`, and `deno task test`.

## Non-Goals

- Do not implement ambient remote following for `sflo:workingAccessUrl`; that belongs to [[wa.task.2026.2026-05-20_2152-workingAccessUrl]].
- Do not implement direct page rendering from `targetAccessUrl`.
- Do not make `weave import` a deploy command, git command, publication-profile command, or branch-publishing wrapper.
- Do not add transformations, content negotiation, RDF-to-Markdown conversion, or custom HTTP `Accept` handling in the first slice.
- Do not infer commits, push branches, or make repository cleanliness decisions in this task.
- Do not redesign payload/content-kind ontology typing as part of this command.
- Do not rename this task note to a completed note unless explicitly requested.

## Implementation Plan

- [ ] Re-read [[wd.general-guidance]], [[wd.testing]], [[wu.repository-options]], [[wa.task.2026.2026-04-13_1245-bob-import-boundary-for-page-source]], and this note before editing.
- [ ] Decide the exact update flag name and whether HTTP(S) fetch is in the first implementation slice.
- [ ] Add `src/core/import/` planner types and validation for source description, designator path, working file path, origin metadata, and create/update mode.
- [ ] Reuse existing path/designator helpers and existing integrate/payload-update rendering where possible instead of copying large Turtle renderers.
- [ ] Add `src/runtime/import/` source acquisition for local paths and `file:` URLs; add HTTP(S) bounded fetch if included in the first slice.
- [ ] Implement atomic write behavior for the imported working file and generated support files, matching the existing runtime patterns for create/update commands.
- [ ] Add CLI wiring in `src/cli/run.ts` for `weave import`.
- [ ] Update Deno dev/test permissions or documentation if HTTP(S) import requires `--allow-net`.
- [ ] Add focused unit, runtime, integration, and e2e tests.
- [ ] Add [[wu.cli-reference.import]] and update [[wu.cli-reference]], [[wu.environment-variables]], [[wu.repository-options]], and release notes.
- [ ] If the command backfills the Bob fixture path, verify the resulting shape against the carried `20-bob-page-imported-source` / `21-bob-page-imported-source-woven` expectations and note any intentional namespace/model drift.
- [ ] Run `deno task fmt`, `deno task lint`, `deno task check`, and `deno task test`.
- [ ] Provide a commit message with a summary line and detailed bullets for CLI, runtime/planner, docs, and tests.
