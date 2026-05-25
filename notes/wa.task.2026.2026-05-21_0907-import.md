---
id: fqbhy6al0tztz6w1d90ch3v
title: 2026-05-21_0907 import
desc: ''
updated: 1779379649208
created: 1779379649208
---

## Goals

- Implement the first runnable `weave import` command for materializing outside-origin bytes into a governed local working artifact.
- Make the first implementation small enough to land quickly: copy/fetch bytes, write a governed local working file, register or update the artifact, and stop before versioning/page generation.
- Treat the carried Bob `a.20/a.21` import-boundary fixture described in [[wa.completed.2026.2026-04-13_1245-bob-import-boundary-for-page-source]] as historical/contextual coverage; if the fixture ladder is regenerated into a `b.*` series, make Bob `b.20/b.21` either a real remote-origin import fixture or stop calling that step import.
- Make import behavior explicit across whole-mesh repos, sidecar repos, and branch-based meshes, because the right destination for the regular working file differs by topology.
- Preserve the semantic boundary between `import`, `integrate`, `payload update`, `workingAccessUrl`, and page-source resolution.
- Keep source acquisition explicit and fail-closed: import may actively read/fetch the requested source, but normal `weave` / `generate` should not start following remote sources just because import metadata exists.
- Record import provenance through a named `sflo:ImportSource` source-registry binding rather than an unnamed `ArtifactResolutionTarget`, while keeping the shared resolution vocabulary for URL, repository, digest, and observation evidence.
- Depend on [[ont.completed.2026.2026-05-24_1256-artifact-resolution-observations]] for the shared `ArtifactResolutionObservation` model instead of widening extraction-specific observed-source vocabulary inside this task.

## Summary

`weave import` should be the friendly copy-acquisition surface that Weave is currently missing. The command should take bytes from a local path, `file:` URL, or explicitly requested HTTP(S) URL, place those bytes at an explicitly chosen governed working file, and make that file the current working surface for a designator path.

This is different from `weave integrate`: `integrate` registers an existing working source where it already lives, including extra-mesh local sources and floating repository locators. `import` creates the governed local working copy first, then records the artifact around that local copy. That distinction matters for branch-published and page-customization workflows where generated pages should follow a stable local artifact boundary, not a contributor's source checkout or a live remote URL.

There is one core import operation:

- acquire bytes from an explicit outside source
- copy those bytes into the chosen `--working-file`
- create or update the Knop/payload artifact at the requested designator path so that local file is the active working locator
- record the observed origin and byte fingerprint so the import is more than a shell download followed by `weave integrate`

The source-registry record for that acquisition should be an `sflo:ImportSource`, a source-registry application concern that subclasses `sflo:ArtifactResolutionTarget`. That keeps import aligned with the shared resolution vocabulary without making every source binding a vague bare `ArtifactResolutionTarget`.

This is also different from [[wa.task.2026.2026-05-20_2152-workingAccessUrl]]. `workingAccessUrl` is a current-byte locator that later runtime operations might follow when remote access policy permits it. `import` is an explicit acquisition command: the operator says "take these bytes now and put them here." After import succeeds, ordinary versioning and page generation should follow the governed local working file.

## Discussion

The earlier first-use example was Bob page content:

```sh
weave import \
  "https://raw.githubusercontent.com/djradon/public-notes/refs/heads/main/user.bob-newhart.md" \
  bob/page-main \
  --working-file bob-page-main.md
```

That command shape is still useful as an example of the intended acquisition boundary: materialize `bob-page-main.md`, register `bob/page-main` as a governed payload artifact if it is not already registered, and record the remote origin in the Knop source registry. Bob's ResourcePageDefinition can then target `bob/page-main`; page generation follows the local `sflo:hasWorkingLocatedFile <bob-page-main.md>` surface rather than fetching the URL during generation.

The carried `20-bob-page-imported-source` fixture is ahead of the productized command and should not be treated as the final import contract. The relevant Accord lane uses the `a.20-bob-page-imported-source` fixture branch, whose state proves a narrower page-source boundary: it adds `bob-page-main.md` and `bob/_knop/_page/page.ttl` with `sflo:targetLocalRelativePath "bob-page-main.md"`. That lane does not create a `bob/page-main` payload artifact, does not create `bob/page-main` Knop support, and does not record the raw GitHub URL as provenance. Older/non-`a` refs contain different historical shapes, including raw URL metadata, but the `a.*` lane and current conformance manifest are the relevant carried fixture surface. Weave also does not currently have a runnable `weave import` CLI/runtime path that can produce a first-class import state. This task should close the command gap for the new import path and add provenance expectations where a fixture or replacement fixture claims outside-origin acquisition.

After this import implementation lands, a complete fixture-ladder regeneration can use a new `b.*` series to test fixture-helper generalization against the existing `a.*` baseline. The integrate source binding migration has already landed in [[wa.completed.2026.2026-05-24_1301-integrate-source-binding-update]], so import is the remaining source-family gate before framework fixture meshes should be regenerated. In that deferred flow, Bob should probably become the whole-mesh remote-origin import fixture instead of being avoided as "already used." A `b.20-bob-page-imported-source` step can intentionally differ from `a.20`: acquire Bob Markdown from the floating raw `main` URL, write `bob-page-main.md`, create or update `bob/page-main` as a governed payload artifact, record the source URL and observed digest in `bob/page-main/_knop/_sources`, and let Bob's page definition source the page from that governed artifact or its local working file. `b.21` can then prove `weave` renders from the governed local bytes and does not fetch the remote URL.

Bob should not supply `--expected-digest` in that deferred fixture. That gives the first remote-origin fixture a useful shape: import from a non-pinned outside source, compute the observed digest for record keeping, and preserve the copied local bytes as the governed surface. This is a weaker replay contract than a commit-pinned fixture, but it proves an important path that users will actually run. If the floating upstream changes later, replay should surface the newly observed digest clearly rather than pretending the fixture was deterministic.

The floating Bob URL currently used in notes is:

```text
https://raw.githubusercontent.com/djradon/public-notes/refs/heads/main/user.bob-newhart.md
```

The next import fixtures should still deliberately extend the three repository-option ladders:

- whole-mesh repo fixture: use Bob `b.20/b.21` as the real remote-origin import fixture if the ladder moves to a `b.*` regen series
- sidecar repo fixture: add content that comes from or lands beside the source repo while the mesh lives in the sidecar/docs layout
- branch-based mesh fixture: add content that can be imported across a local source branch and publication branch boundary without requiring live remote fetch during page generation

Carol Burnett content from public-notes is still useful, but it should become the repeated replacement/versioning path rather than the first remote-origin import:

```sh
weave import \
  "https://raw.githubusercontent.com/djradon/public-notes/f46d85187ed7781917b73dd7779b756e2d2b7494/user.carol-burnett.md" \
  carol/page-main \
  --working-file carol/page-main.md
```

The file is useful because it is not just a tiny placeholder. It has YAML frontmatter, headings, Markdown emphasis, lists, remote image references, and an explicit Bob relationship. That makes it a better fit for later custom ResourcePage work than another minimal Alice note.

After import lands, Carol should become six fixture steps: an import and weave pair for each of the three Carol page versions. The first import creates `carol/page-main` and its governed working file. The second and third imports use `--replace-working` against the same designator and working file, refresh the source-origin/digest metadata, and then weave a new historical state from the replaced bytes. That sequence gives `--replace-working` real fixture coverage without overloading the first implementation task.

Carol is also a strong candidate for ResourcePage panel inclusion, such as proving a Markdown/content panel can sit beside the references panel. That panel work can ride on the three-version Carol ladder if it is ready, or remain a later follow-on.

The Carol Markdown also points at remote images, including a Bob/Carol image that would make a good later imported asset. Do not make the first import slice recursively fetch referenced images or other linked assets. A future scraper-like import mode could parse referenced content, import selected linked assets under an explicit asset root, compute separate digests, and record source registry entries for each imported asset.

That topology split is now part of the task, not just test plumbing:

- In a whole-mesh repo, the regular working file can be a normal mesh-relative file under the active mesh root.
- In a sidecar repo, the copied working file may belong either inside the sidecar/docs mesh or in an explicitly selected sidecar-adjacent source folder. The one-time import should not persist an absolute source path by default.
- In a branch-based mesh, the copied working file may belong in the active mesh/publication checkout or in an explicitly selected source worktree/layout. Import should not become an implicit git checkout, commit, or branch synchronization command.

This can be easy if the first implementation avoids pretending to solve every import-shaped problem:

- require an explicit `--working-file <meshRelativePath>` for the first slice rather than inferring path names too early
- keep the first CLI `--working-file` mesh-root-relative as the simple user-facing destination contract, but do not freeze the planner/API to that single destination shape
- let the core API model an explicit import destination locator so non-CLI callers can request sidecar-adjacent or branch source-worktree writes under the same path-safety and source-grant policy used for approved local working locators
- support plain byte copy/fetch without transforms
- keep `weave import` create/update behavior focused on current working files, not historical states
- record source-origin metadata in the Knop source registry plus an observed content digest, but do not require full remote-runtime policy before the user can run an explicit import command
- fail if the target file exists unless the user explicitly asks to overwrite or update it
- defer copying from an already registered artifact/current source until a real workflow needs that rarer operation
- leave page-definition repointing, `weave version`, `weave generate`, and publication validation as separate commands

`weave import` deserves to exist as a distinct operation. It is not just `curl` or `wget` followed by `weave integrate`: it couples explicit acquisition with governed placement, source-origin recording, observed digest calculation, optional digest verification, artifact registration, and later default path inference.

The non-CLI API case makes this boundary more important, not less. A caller using a library, daemon, web UI, or automation API should not have to manually reproduce "fetch bytes, choose a safe mesh-relative destination, write the file, compute a digest, register/update the artifact, and record provenance" as separate host-side steps. The import planner/runtime should expose that as one coherent operation, with CLI wiring as one caller of the same API.

### ImportSource and IntegrationSource

Import and integrate both use the same source-registry / artifact-resolution family, but they are different application concerns.

`ImportSource` should describe source bytes actively acquired by `weave import` and copied into a governed local working surface. It should live in the target Knop's `_knop/_sources/sources.ttl` registry and be linked from that registry with `sflo:hasSourceBinding`.

`IntegrationSource` should describe source bytes registered by `weave integrate` where those bytes already live. It is the sibling concern for sidecar and branch-published source binding: integrate leaves bytes in place and records source policy; import copies bytes and records acquisition evidence.

Both should subclass `sflo:ArtifactResolutionTarget` so they can reuse:

- `sflo:targetAccessUrl` for URL acquisition/source coordinates when appropriate
- `sflo:targetLocalRelativePath` for portable, policy-approved local source coordinates when appropriate
- `sflo:hasTargetRepositorySource` / `sflo:hasRepositorySourceFloatingLocator` for repository-backed source coordinates
- `sflo:expectsContentDigest` for requested/expected byte identity
- `sflo:hasResolutionObservation` for intentionally recorded acquisition evidence
- `sflo:hasArtifactResolutionMode` and requested target state/history terms when a source is resolved through the normal artifact-resolution path

The cross-cutting observation vocabulary is owned by [[ont.completed.2026.2026-05-24_1256-artifact-resolution-observations]]. Import should record concrete acquisition evidence as an `sflo:ArtifactResolutionObservation` linked from the `sflo:ImportSource`, including the observed content digest, observation time, and observer where available. The important command contract is that every successful import records the concrete digest it observed while acquiring bytes.

For the first import slice, implement `ImportSource` for import provenance. `IntegrationSource` now exists in the shared ontology, and [[wa.completed.2026.2026-05-24_1301-integrate-source-binding-update]] has landed. Import should reuse the settled source-registry rendering/parsing shape instead of introducing a second one-off bridge.

### Learnings from IntegrationSource Migration

The completed integrate migration gives import a concrete local pattern to mirror:

- Render the Knop source registry as `sflo:hasSourceBinding <...#payload-source>` plus a concrete source subclass, not a bare `sflo:ArtifactResolutionTarget`.
- Use the stable observation fragment pattern `<sourceBindingIri>-observation-001` when a command intentionally records evidence.
- Keep requested/expected evidence on the source binding: origin URL or repository locator, target artifact, local target locator when appropriate, resolution mode, and `sflo:expectsContentDigest`.
- Keep observed evidence on the linked `sflo:ArtifactResolutionObservation`, especially `sflo:observedContentDigest`.
- If `sflo:observedAt` is emitted, render it as an `xsd:dateTime` literal and make the runtime clock injectable or otherwise deterministic in tests.
- Do not add observations for source bindings that are merely working/floating policy records. Import is different from working-only integrate: every successful import should record an observed digest because acquisition evidence is part of the command contract.
- Add an import-specific parser surface analogous to `listIntegrationSourceInventoryStates`, or a shared source-registry helper if the implementation naturally wants one. The important outcome is that runtime callers can retrieve `ImportSource` coordinates and linked observation evidence without inspecting raw RDF class names.
- Keep remote access bounded to the import command. `sflo:targetAccessUrl` on an `ImportSource` is acquisition provenance; it must not make `weave`, `generate`, ResourcePage rendering, or `workingAccessUrl` resolution start fetching that URL.

For HTTP(S) import, `targetAccessUrl` on `ImportSource` records the outside acquisition source. It is not the imported payload's active current-byte locator and it does not authorize later remote fetch by `weave`, `generate`, or page rendering.

For local path import, avoid persisting host-absolute source paths in public RDF by default. If the source path is portable and policy-approved, `targetLocalRelativePath` may be used on `ImportSource`; otherwise record durable evidence such as observed digest and observation time without publishing an absolute local path.

Repeated `weave import --replace-working` is not the same operation as `weave payload update`. `payload update` is a current-working-surface convenience for an already registered local payload artifact whose source/acquisition story is otherwise unchanged. It replaces the existing governed working bytes and stops there: no HTTP(S) fetch, no import-origin provenance refresh, no expected-digest assertion, no source-registry acquisition event, and no active-locator change. Repeated import with `--replace-working` means "acquire these outside-origin bytes again, overwrite the governed working file, and refresh the import provenance/digest evidence for this artifact." It still does not mint a historical state; `weave version` or the composed `weave` flow owns that.

There is a real design edge: because HTTP(S) import is in the first slice, Deno dev/test permissions need to allow network access for the import tests and `deno task dev:root` may need a permission adjustment or documentation. Native/package users will experience this as a normal command, but source-tree development uses Deno permissions.

The current carried Bob fixture uses older namespace vocabulary in some historical refs, especially outside the current `a.*` scenario lane. Implementation should follow the current Weave/SFLO namespace constants and should not copy stale fixture namespace strings into new production code. The stale non-`a.*` fixture refs are probably more confusing than useful now; prune them in a separate Semantic Flow Framework fixture-hygiene pass after confirming no active scenario index or documentation still points at them.

## Open Issues

- Which new source content should extend the sidecar and branch-based fixture ladders?
- Should linked assets referenced by imported Markdown, such as the Bob/Carol image in the Carol note, eventually be importable as a recursive/scraper-like option? Recommendation: yes later, but not in the first slice.

## Decisions

- `weave import` is an active acquisition command. It reads or fetches source bytes at command time and writes a governed local working file.
- `weave import` owns the copy-to-working-file boundary: after source bytes have been copied to the selected `--working-file`, Weave should create or update the Knop/payload artifact under the specified designator path and make that local file the active working locator.
- `--working-file` is mesh-root-relative. Keep that as the baseline destination model for whole-mesh, sidecar, and branch-published layouts rather than adding named destination modes.
- The first implementation should require `--working-file`; default path inference can be added later after examples settle.
- A minimum useful import records source origin metadata and a computed byte fingerprint; that provenance and digest evidence is part of why import is distinct from manual download plus `integrate`.
- The import planner/runtime should be a first-class API operation, not just CLI sugar. Non-CLI callers should be able to request acquisition, governed placement, artifact registration, provenance, and digest verification as one operation.
- The first CLI may expose only mesh-root-relative `--working-file`, but the planner/API should support an explicit destination locator for mesh-local, approved sidecar-adjacent, and approved branch source-worktree targets. Mesh-local imports can record the active working locator as `sflo:hasWorkingLocatedFile`; approved outside-mesh local destinations should use the existing working-local locator shape rather than forcing the bytes under the mesh root.
- Support explicit HTTP(S) fetch in the first slice. Import may acquire bytes from an arbitrary explicit HTTP(S) source URL under bounded import policy; that does not broaden `integrate`, `weave`, `generate`, page-source resolution, or ambient `workingAccessUrl` following.
- Use `--replace-working` as the explicit flag for replacing an existing working file or existing artifact working surface.
- When `--replace-working` imports into an existing payload artifact, update both the local working file and source-origin/digest metadata.
- `payload update` remains a narrow current-byte replacement command for an existing local payload artifact. Use repeated `import --replace-working` instead when the replacement should reacquire outside-origin bytes and refresh import provenance or digest evidence.
- Optional `--expected-digest sha256:...` is part of the first CLI surface. Import should always compute an observed digest; when the expected digest is supplied, mismatch is a hard failure.
- Store import-origin metadata in the Knop source registry, not as the active current locator on the payload artifact. The payload artifact should carry the governed local `sflo:hasWorkingLocatedFile`; the source registry should carry origin URL/repository details and expected digest when supplied, with observed digest and observation time recorded on an `sflo:ArtifactResolutionObservation` linked from the `sflo:ImportSource`.
- Model that import-origin metadata as an `sflo:ImportSource`, linked from the Knop source registry with `sflo:hasSourceBinding`. `ImportSource` should be an `sflo:ArtifactResolutionTarget` subclass, not a replacement for the shared target/resolution vocabulary.
- Use `sflo:IntegrationSource` as the sibling `ArtifactResolutionTarget` subclass for `weave integrate` source bindings: integrate registers existing source bytes where they live; import acquires/copies bytes into a governed local working file.
- Do not use a bare `sflo:ArtifactResolutionTarget` for new import provenance except as a temporary implementation step during the ontology migration. The durable contract should identify import provenance as `sflo:ImportSource`.
- Do not assert `sflo:RdfDocument` for imported Markdown, images, or other non-RDF bytes. The first import slice can avoid the false type without a full content-kind cleanup by asserting `sflo:RdfDocument` only when the source is actually RDF, based on explicit content kind, content type, or conservative extension detection.
- `weave import` should not run `weave`, `weave version`, or `weave generate` automatically.
- The first CLI shape should be:

```sh
weave import <source> <designatorPath> --working-file <meshRelativePath> [--mesh-root <meshRoot>] [--expected-digest sha256:<hex>] [--replace-working]
```

- For mesh-local import destinations, the imported artifact's current bytes are the governed local `sflo:hasWorkingLocatedFile`, not the outside source. For approved outside-mesh local destinations, the active working locator should be the explicit working-local locator for the copied file, and the outside acquisition source remains provenance only.
- HTTP(S) source URLs should be recorded as source registry provenance, not as the imported artifact's active `sflo:workingAccessUrl`. Page generation should continue following the local working file unless a later task explicitly implements remote current-byte following.
- Existing local path access grants are still relevant for ongoing extra-mesh source binding through `integrate`, but import should not require a persistent local-path grant merely to copy explicitly requested bytes into the mesh.
- Remote integrate is deferred for now. Explicit import is the preferred remote-origin acquisition path until a real use case proves that `integrate` itself needs to fetch or bind remote URLs.
- Copying bytes from an already registered artifact/current-source binding into a new local working file is deferred; that is a rarer "copy existing artifact" workflow, not part of the first import contract.
- Import must be topology-aware. Whole-mesh, sidecar, and branch-based meshes can share the same core operation, but tests and validation should prove the chosen destination locator is correct for each layout.
- Defer full fixture-ladder regeneration until after this import provenance lands. Integrate source binding has already landed; the first import implementation should still have focused unit/runtime/e2e coverage without requiring a `b.*` regen as a landing gate.
- In the deferred `b.*` fixture pass, use Bob `b.20/b.21` as the first whole-mesh remote-origin import fixture rather than keeping Bob in the current `a.*` half-state.
- Bob `b.20` should use the floating raw `main` URL without `--expected-digest`; import should still compute and record the observed digest for provenance.
- Use Carol in the deferred fixture pass for the richer replacement/versioning ladder: three import/weave pairs over the same `carol/page-main` working file, with the second and third imports using `--replace-working`.
- Keep lower-level import tests local or dependency-injected where practical so semantic failures remain distinguishable from GitHub/network availability failures.
- First-slice overwrite behavior should be explicit. A command that would replace an existing working file should fail unless `--replace-working` is supplied.
- First-slice implementation should preserve existing path safety rules: imported working files must be mesh-relative files, must not escape the mesh root, and must not land under reserved support segments unless a later command intentionally handles support artifacts.

## Contract Changes

- Implement the first runnable `weave import` CLI/runtime command; today import exists as a conceptual and fixture operation, not as a supported command surface.
- Add core/runtime import planning for materializing source bytes to a governed local working file and registering the designator path as a governed payload artifact when missing.
- Make import destination validation explicitly cover whole-mesh, sidecar, and branch-based mesh layouts rather than assuming every fixture is whole-repo-shaped.
- Keep the first CLI `--working-file` path mesh-root-relative, but design the core import request with an explicit destination locator so API callers can target approved sidecar-adjacent and branch source-worktree files.
- The command should produce a stdout summary plus created/updated paths, matching the existing CLI style.
- If the designator path is new, import should create the same essential support surface as a first-time payload integration: Knop metadata, Knop inventory, payload artifact registration, mesh inventory update, and the working file.
- If the designator path already exists and the caller supplies the chosen update/overwrite flag, import should replace the working file and refresh import-origin metadata without minting a historical state.
- Repeated import should be covered separately from `payload update`: both replace current working bytes, but only import refreshes acquisition provenance, observed digest, optional expected digest evidence, and source registry metadata.
- Import should compute and report an observed digest for every acquired source; when `--expected-digest` is supplied, the command should fail if the observed digest differs.
- HTTP(S) import should use bounded fetch behavior: scheme restriction, timeout, maximum byte size, redirect decision, digest calculation, and clear diagnostics.
- Add or require SFLO vocabulary for `sflo:ImportSource` as an `sflo:ArtifactResolutionTarget` subclass. Coordinate this with [[ont.completed.2026.2026-05-24_1256-artifact-resolution-observations]], which also covers `sflo:IntegrationSource` and the shared observation vocabulary.
- Add or require SFLO observation vocabulary so import can record observed digest evidence through an `sflo:ArtifactResolutionObservation` linked from `sflo:ImportSource`; the first implementation should not have to misuse an extraction-only digest property.
- Import provenance should be rendered through the Knop source registry as an `sflo:ImportSource` binding for the imported artifact. For URL sources, use `sflo:targetAccessUrl` on that binding for the source URL. Record the copied file as the payload artifact's active working locator: `sflo:hasWorkingLocatedFile` for mesh-local destinations, or the approved working-local locator shape for outside-mesh local destinations.
- `sflo:targetAccessUrl` in an `sflo:ImportSource` source-registry binding is provenance/acquisition evidence for the import operation. It does not make ResourcePageDefinition `targetAccessUrl` rendering supported, and it does not authorize later remote fetching by `weave` or `generate`.
- First-slice content typing should avoid false RDF claims. Support artifacts such as inventories and source registries remain `sflo:RdfDocument`; imported payloads and their working files should only be typed as `sflo:RdfDocument` when the imported bytes are RDF.
- Add user documentation for `weave import` and link it from [[wu.cli-reference]], [[wu.repository-options]], and the relevant release notes when the command ships.
- Tighten existing docs language that mentions "integration/import" so it does not imply the command exists before this task lands.

## Testing

- Unit-test import request normalization: designator path, `--working-file`, root designator handling, reserved path rejection, empty source rejection, and overwrite/update flag behavior.
- Core-plan tests for the new-artifact path: created working file, Knop support files, payload artifact block, mesh inventory update, `sflo:ImportSource` source-registry origin metadata, and RDF-vs-non-RDF content typing.
- Runtime tests for local filesystem source import and `file:` URL import.
- Runtime/e2e tests for the three topology ladders: whole-mesh repo, sidecar repo, and branch-based mesh. The CLI path can cover mesh-root-relative `--working-file`; core/API tests should cover approved sidecar-adjacent and branch source-worktree destinations.
- Add focused lower-level and runtime/e2e tests for timeout/404 diagnostics, max-size rejection, digest mismatch, provenance recording, and no live remote fetch during later page generation without requiring a full fixture-ladder regen.
- Defer the GitHub-backed `b.*` fixture ladder until after import lands: Bob floating-source import/weave for `b.20/b.21`, then Carol six-step replacement coverage.
- Add or tighten provenance assertions so an import fixture that claims HTTP(S) acquisition verifies the recorded origin URL or repository/source metadata, not only the local copied file.
- Add regression coverage that import provenance is typed `sflo:ImportSource` and linked from the Knop source registry with `sflo:hasSourceBinding`.
- Add regression coverage that import observed digest evidence validates on an `sflo:ArtifactResolutionObservation` linked from `sflo:ImportSource`.
- Add regression coverage that imported Markdown is not typed as `sflo:RdfDocument`.
- Keep Bob `a.20/a.21` as historical comparison coverage; use Bob `b.20/b.21` for honest remote-origin import in the deferred fixture-regeneration pass.
- Add regression coverage that import does not persist absolute local source paths in public RDF by default.
- Run focused tests for import, integrate, payload update, page-definition weave/generate, then `deno task lint`, `deno task check`, and `deno task test`.

## Non-Goals

- Do not implement ambient remote following for `sflo:workingAccessUrl`; that belongs to [[wa.task.2026.2026-05-20_2152-workingAccessUrl]].
- Do not implement remote-fetching `integrate` as part of this task; import is the explicit acquisition path for now.
- Do not refactor completed `integrate` source-binding output inside this import task. [[wa.completed.2026.2026-05-24_1301-integrate-source-binding-update]] has landed; implement import against that settled shared source-registry shape.
- Do not implement copying from an already registered artifact/current-source binding in the first slice.
- Do not implement direct page rendering from `targetAccessUrl`.
- Do not make `weave import` a deploy command, git command, publication-profile command, or branch-publishing wrapper.
- Do not switch branches, commit files, synchronize sidecars, or infer repository publication policy in the import command.
- Do not add transformations, content negotiation, RDF-to-Markdown conversion, or custom HTTP `Accept` handling in the first slice.
- Do not recursively fetch images or other linked assets referenced by imported content in the first slice.
- Do not infer commits, push branches, or make repository cleanliness decisions in this task.
- Do not redesign payload/content-kind ontology typing as part of this command. The larger cleanup would thread explicit content-kind/media-type information through integrate, import, payload renderers, versioning, and page generation, and would likely add or adopt vocabulary for Markdown, images, and generic media types. This task should only stop adding false `sflo:RdfDocument` claims for imported non-RDF bytes.
- Do not rename this task note to a completed note unless explicitly requested.

## Implementation Plan

- [x] Re-read [[wd.general-guidance]], [[wd.testing]], [[wu.repository-options]], [[wa.completed.2026.2026-04-13_1245-bob-import-boundary-for-page-source]], and this note before editing.
- [x] Decide the exact update flag name and whether HTTP(S) fetch is in the first implementation slice: use `--replace-working` and include explicit bounded HTTP(S) fetch.
- [d] After this import implementation lands, regenerate a `b.*` fixture lane and make Bob `b.20/b.21` the whole-mesh remote-origin import fixture with floating-source observed-digest provenance, then compare it against `a.20/a.21`.
- [x] Add or coordinate SFLO vocabulary for `ImportSource` as an `ArtifactResolutionTarget` subclass, plus import acquisition observations per [[ont.completed.2026.2026-05-24_1256-artifact-resolution-observations]].
- [x] Confirm [[wa.completed.2026.2026-05-24_1301-integrate-source-binding-update]] has landed before implementing import source-registry output.
- [x] Add `src/core/import/` planner types and validation for source description, designator path, topology-aware destination locator, source registry origin metadata, content-kind/RDF typing, and create/update mode.
- [x] Reuse existing path/designator helpers and existing integrate/source-registry/payload-update rendering where possible instead of copying large Turtle renderers.
- [x] Add an import source-registry parser path analogous to `listIntegrationSourceInventoryStates`, or extract a shared parser if that keeps `ImportSource` and `IntegrationSource` behavior aligned.
- [x] Add `src/runtime/import/` source acquisition for local paths, `file:` URLs, bounded HTTP(S) fetch, observed digest calculation, optional expected digest verification, and `--replace-working` behavior.
- [x] Implement atomic write behavior for the imported working file and generated support files, matching the existing runtime patterns for create/update commands.
- [x] Add CLI wiring in `src/cli/run.ts` for `weave import`.
- [x] Update Deno dev/test permissions and documentation for bounded GitHub-backed HTTP(S) import, likely `--allow-net=raw.githubusercontent.com`.
- [x] Add focused unit, runtime, integration, and e2e tests, including one path each for whole-mesh, sidecar, and branch-based meshes.
- [x] Add [[wu.cli-reference.import]] and update [[wu.cli-reference]], [[wu.environment-variables]], [[wu.repository-options]], and release notes.
- [d] After import lands, add the Carol three-version import/weave ladder, using `--replace-working` for the second and third imports; layer ResourcePage panel inclusion or linked-asset exploration on top only if that scope is ready.
- [x] Run `deno task fmt`, `deno task lint`, `deno task check`, and `deno task test`.
- [x] Provide a commit message with a summary line and detailed bullets for CLI, runtime/planner, docs, and tests.
