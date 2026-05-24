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
- Treat the carried Bob `20/21` import-boundary fixture described in [[wa.completed.2026.2026-04-13_1245-bob-import-boundary-for-page-source]] as historical/contextual coverage, but do not use Bob page content as the next new import fixture.
- Make import behavior explicit across whole-mesh repos, sidecar repos, and branch-based meshes, because the right destination for the regular working file differs by topology.
- Preserve the semantic boundary between `import`, `integrate`, `payload update`, `workingAccessUrl`, and page-source resolution.
- Keep source acquisition explicit and fail-closed: import may actively read/fetch the requested source, but normal `weave` / `generate` should not start following remote sources just because import metadata exists.

## Summary

`weave import` should be the friendly copy-acquisition surface that Weave is currently missing. The command should take bytes from a local path, `file:` URL, or explicitly requested HTTP(S) URL, place those bytes at an explicitly chosen governed working file, and make that file the current working surface for a designator path.

This is different from `weave integrate`: `integrate` registers an existing working source where it already lives, including extra-mesh local sources and floating repository locators. `import` creates the governed local working copy first, then records the artifact around that local copy. That distinction matters for branch-published and page-customization workflows where generated pages should follow a stable local artifact boundary, not a contributor's source checkout or a live remote URL.

There is one core import operation:

- acquire bytes from an explicit outside source
- copy those bytes into the chosen `--working-file`
- create or update the Knop/payload artifact at the requested designator path so that local file is the active working locator
- record the observed origin and byte fingerprint so the import is more than a shell download followed by `weave integrate`

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

But Bob content has already been used in the carried import/page-source line, so it should not be the next new fixture. The next import fixtures should deliberately extend the three repository-option ladders:

- whole-mesh repo fixture: add a small new source that belongs naturally in the whole mesh, such as an Alice-related note that can become a normal payload without relying on Bob
- sidecar repo fixture: add content that comes from or lands beside the source repo while the mesh lives in the sidecar/docs layout
- branch-based mesh fixture: add content that can be imported across a local source branch and publication branch boundary without requiring live remote fetch during page generation

A stronger next whole-mesh candidate is Carol Burnett content from a commit-pinned public-notes URL:

```sh
weave import \
  "https://raw.githubusercontent.com/djradon/public-notes/f46d85187ed7781917b73dd7779b756e2d2b7494/user.carol-burnett.md" \
  carol/page-main \
  --working-file carol/page-main.md \
  --expected-digest sha256:5634ffc14165c55ab43c2af38b9d6395e22c8385f54d4a94a7d22d83c99afee7
```

The file is useful because it is not just a tiny placeholder. It has YAML frontmatter, headings, Markdown emphasis, lists, remote image references, and an explicit Bob relationship. That makes it a better import/content fixture for later custom ResourcePage work than another minimal Alice note. The import fixture may intentionally depend on GitHub rather than running a local HTTP server, because that proves the real acquisition path. Keep the URL commit-pinned and require the expected digest so the dependency is on GitHub availability, not mutable source content.

The Carol Markdown also points at remote images, including a Bob/Carol image that would make a good later imported asset. Do not make the first import slice recursively fetch referenced images or other linked assets. A future scraper-like import mode could parse referenced content, import selected linked assets under an explicit asset root, compute separate digests, and record source registry entries for each imported asset.

That topology split is now part of the task, not just test plumbing:

- In a whole-mesh repo, the regular working file can be a normal mesh-relative file under the active mesh root.
- In a sidecar repo, the regular working file may belong in the sidecar/docs mesh while the source bytes come from the adjacent source repo or another local path; the one-time import should not persist an absolute source path by default.
- In a branch-based mesh, the regular working file belongs to the active mesh/publication checkout or explicitly selected local branch layout; import should not become an implicit git checkout, commit, or branch synchronization command.

This can be easy if the first implementation avoids pretending to solve every import-shaped problem:

- require an explicit `--working-file <meshRelativePath>` for the first slice rather than inferring path names too early
- keep `--working-file` mesh-root-relative as the normal destination contract; if a user wants bytes staged in a sidecar-adjacent source folder or another branch/worktree, that staging belongs outside import and can be followed by `integrate`
- support plain byte copy/fetch without transforms
- keep `weave import` create/update behavior focused on current working files, not historical states
- record source-origin metadata in the Knop source registry plus an observed content digest, but do not require full remote-runtime policy before the user can run an explicit import command
- fail if the target file exists unless the user explicitly asks to overwrite or update it
- defer copying from an already registered artifact/current source until a real workflow needs that rarer operation
- leave page-definition repointing, `weave version`, `weave generate`, and publication validation as separate commands

This is the practical line for whether import deserves to exist. If the command only downloads/copies bytes and then calls the same artifact registration path as `integrate`, it is not much better than `curl` or `wget` plus `weave integrate`. The useful import value is that it couples the copy boundary with governed placement, origin recording, digest calculation or verification, and later default path inference.

The non-CLI API case makes this boundary more important, not less. A caller using a library, daemon, web UI, or automation API should not have to manually reproduce "fetch bytes, choose a safe mesh-relative destination, write the file, compute a digest, register/update the artifact, and record provenance" as separate host-side steps. The import planner/runtime should expose that as one coherent operation, with CLI wiring as one caller of the same API.

There is a real design edge: because HTTP(S) import is in the first slice, Deno dev/test permissions need to allow network access for the import tests and `deno task dev:root` may need a permission adjustment or documentation. Native/package users will experience this as a normal command, but source-tree development uses Deno permissions.

The current carried Bob fixture uses older namespace vocabulary in some historical refs, especially outside the current `a.*` scenario lane. Implementation should follow the current Weave/SFLO namespace constants and should not copy stale fixture namespace strings into new production code. The stale non-`a.*` fixture refs are probably more confusing than useful now; prune them in a separate Semantic Flow Framework fixture-hygiene pass after confirming no active scenario index or documentation still points at them.

## Open Issues

- Which new source content should extend the sidecar and branch-based fixture ladders? The first whole-mesh candidate is the commit-pinned Carol Burnett Markdown note from public-notes.
- Should linked assets referenced by imported Markdown, such as the Bob/Carol image in the Carol note, eventually be importable as a recursive/scraper-like option? Recommendation: yes later, but not in the first slice.

## Decisions

- `weave import` is an active acquisition command. It reads or fetches source bytes at command time and writes a governed local working file.
- `weave import` owns the copy-to-working-file boundary: after source bytes have been copied to the selected `--working-file`, Weave should create or update the Knop/payload artifact under the specified designator path and make that local file the active working locator.
- `--working-file` is mesh-root-relative. Keep that as the baseline destination model for whole-mesh, sidecar, and branch-published layouts rather than adding named destination modes.
- The first implementation should require `--working-file`; default path inference can be added later after examples settle.
- A minimum useful import records source origin metadata and a computed byte fingerprint. If import cannot record provenance/fingerprint, the workflow is too close to manual download plus `integrate`.
- The import planner/runtime should be a first-class API operation, not just CLI sugar. Non-CLI callers should be able to request acquisition, governed placement, artifact registration, provenance, and digest verification as one operation.
- Support explicit HTTP(S) fetch in the first slice. Import may acquire bytes from an arbitrary explicit HTTP(S) source URL under bounded import policy; that does not broaden `integrate`, `weave`, `generate`, page-source resolution, or ambient `workingAccessUrl` following.
- Use `--replace-working` as the explicit flag for replacing an existing working file or existing artifact working surface.
- When `--replace-working` imports into an existing payload artifact, update both the local working file and source-origin/digest metadata.
- Optional `--expected-digest sha256:...` is part of the first CLI surface. Import should always compute an observed digest; when the expected digest is supplied, mismatch is a hard failure.
- Store import-origin metadata in the Knop source registry, not as the active current locator on the payload artifact. The payload artifact should carry the governed local `sflo:hasWorkingLocatedFile`; the source registry should carry origin URL/repository details, observed digest, expected digest when supplied, and observation time.
- Do not assert `sflo:RdfDocument` for imported Markdown, images, or other non-RDF bytes. The first import slice can avoid the false type without a full content-kind cleanup by asserting `sflo:RdfDocument` only when the source is actually RDF, based on explicit content kind, content type, or conservative extension detection.
- `weave import` should not run `weave`, `weave version`, or `weave generate` automatically.
- The first CLI shape should be:

```sh
weave import <source> <designatorPath> --working-file <meshRelativePath> [--mesh-root <meshRoot>] [--expected-digest sha256:<hex>] [--replace-working]
```

- The imported artifact's current bytes are the governed local `sflo:hasWorkingLocatedFile`, not the outside source.
- HTTP(S) source URLs should be recorded as source registry provenance, not as the imported artifact's active `sflo:workingAccessUrl`. Page generation should continue following the local working file unless a later task explicitly implements remote current-byte following.
- Existing local path access grants are still relevant for ongoing extra-mesh source binding through `integrate`, but import should not require a persistent local-path grant merely to copy explicitly requested bytes into the mesh.
- Remote integrate is deferred for now. Explicit import is the preferred remote-origin acquisition path until a real use case proves that `integrate` itself needs to fetch or bind remote URLs.
- Copying bytes from an already registered artifact/current-source binding into a new local working file is deferred; that is a rarer "copy existing artifact" workflow, not part of the first import contract.
- Import must be topology-aware. Whole-mesh, sidecar, and branch-based meshes can share the same core operation, but tests and validation should prove the chosen working-file destination is correct for each layout.
- Bob page content should not be used as the next new import fixture; use new content that legitimately extends the existing whole-mesh, sidecar, and branch-based fixture ladders.
- Use the commit-pinned Carol Burnett Markdown note as the first whole-mesh import/content candidate, with optional CLI `--expected-digest` supplied in the fixture as `sha256:5634ffc14165c55ab43c2af38b9d6395e22c8385f54d4a94a7d22d83c99afee7`.
- Allow the import fixture/e2e path to depend on live GitHub through a commit-pinned raw URL and fixture-supplied expected digest. This avoids a local HTTP server and exercises the real external acquisition path.
- Keep lower-level import tests local or dependency-injected where practical so semantic failures remain distinguishable from GitHub/network availability failures.
- First-slice overwrite behavior should be explicit. A command that would replace an existing working file should fail unless `--replace-working` is supplied.
- First-slice implementation should preserve existing path safety rules: imported working files must be mesh-relative files, must not escape the mesh root, and must not land under reserved support segments unless a later command intentionally handles support artifacts.

## Contract Changes

- Implement the first runnable `weave import` CLI/runtime command; today import exists as a conceptual and fixture operation, not as a supported command surface.
- Add core/runtime import planning for materializing source bytes to a governed local working file and registering the designator path as a governed payload artifact when missing.
- Make import destination validation explicitly cover whole-mesh, sidecar, and branch-based mesh layouts rather than assuming every fixture is whole-repo-shaped.
- The command should produce a stdout summary plus created/updated paths, matching the existing CLI style.
- If the designator path is new, import should create the same essential support surface as a first-time payload integration: Knop metadata, Knop inventory, payload artifact registration, mesh inventory update, and the working file.
- If the designator path already exists and the caller supplies the chosen update/overwrite flag, import should replace the working file and refresh import-origin metadata without minting a historical state.
- Import should compute and report an observed digest for every acquired source; when `--expected-digest` is supplied, the command should fail if the observed digest differs.
- HTTP(S) import should use bounded fetch behavior: scheme restriction, timeout, maximum byte size, redirect decision, digest calculation, and clear diagnostics.
- Import provenance should be rendered through the Knop source registry as an `sflo:ArtifactResolutionTarget` binding for the imported artifact. For URL sources, use `sflo:targetAccessUrl` on that binding for the source URL; record the copied mesh-local file as the payload artifact's active `sflo:hasWorkingLocatedFile`.
- `sflo:targetAccessUrl` in an import source-registry binding is provenance/acquisition evidence for the import operation. It does not make ResourcePageDefinition `targetAccessUrl` rendering supported, and it does not authorize later remote fetching by `weave` or `generate`.
- First-slice content typing should avoid false RDF claims. Support artifacts such as inventories and source registries remain `sflo:RdfDocument`; imported payloads and their working files should only be typed as `sflo:RdfDocument` when the imported bytes are RDF.
- Add user documentation for `weave import` and link it from [[wu.cli-reference]], [[wu.repository-options]], and the relevant release notes when the command ships.
- Tighten existing docs language that mentions "integration/import" so it does not imply the command exists before this task lands.

## Testing

- Unit-test import request normalization: designator path, `--working-file`, root designator handling, reserved path rejection, empty source rejection, and overwrite/update flag behavior.
- Core-plan tests for the new-artifact path: created working file, Knop support files, payload artifact block, mesh inventory update, source registry origin metadata, and RDF-vs-non-RDF content typing.
- Runtime tests for local filesystem source import and `file:` URL import.
- Runtime/e2e tests for the three topology ladders: whole-mesh repo, sidecar repo, and branch-based mesh.
- Add a GitHub-backed e2e/import fixture using the commit-pinned Carol URL, plus focused lower-level tests for timeout/404 diagnostics, max-size rejection, digest mismatch, and no live remote fetch during later page generation.
- Add a Carol import/content test path that uses the commit-pinned raw GitHub URL with the expected digest. Network/GitHub failures should be reported clearly as external acquisition failures.
- Add or tighten provenance assertions so an import fixture that claims HTTP(S) acquisition verifies the recorded origin URL or repository/source metadata, not only the local copied file.
- Add regression coverage that imported Markdown is not typed as `sflo:RdfDocument`.
- Keep Bob import/page-source coverage as historical regression coverage if it remains useful, but add new non-Bob fixture content for the next import e2e path.
- Add regression coverage that import does not persist absolute local source paths in public RDF by default.
- Run focused tests for import, integrate, payload update, page-definition weave/generate, then `deno task lint`, `deno task check`, and `deno task test`.

## Non-Goals

- Do not implement ambient remote following for `sflo:workingAccessUrl`; that belongs to [[wa.task.2026.2026-05-20_2152-workingAccessUrl]].
- Do not implement remote-fetching `integrate` as part of this task; import is the explicit acquisition path for now.
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

- [ ] Re-read [[wd.general-guidance]], [[wd.testing]], [[wu.repository-options]], [[wa.completed.2026.2026-04-13_1245-bob-import-boundary-for-page-source]], and this note before editing.
- [x] Decide the exact update flag name and whether HTTP(S) fetch is in the first implementation slice: use `--replace-working` and include explicit bounded HTTP(S) fetch.
- [ ] Choose new non-Bob fixture content that legitimately extends the sidecar and branch-based fixture ladders; use the Carol Burnett note as the first whole-mesh import/content candidate.
- [ ] Add `src/core/import/` planner types and validation for source description, designator path, topology-aware working file path, source registry origin metadata, content-kind/RDF typing, and create/update mode.
- [ ] Reuse existing path/designator helpers and existing integrate/payload-update rendering where possible instead of copying large Turtle renderers.
- [ ] Add `src/runtime/import/` source acquisition for local paths, `file:` URLs, bounded HTTP(S) fetch, observed digest calculation, optional expected digest verification, and `--replace-working` behavior.
- [ ] Implement atomic write behavior for the imported working file and generated support files, matching the existing runtime patterns for create/update commands.
- [ ] Add CLI wiring in `src/cli/run.ts` for `weave import`.
- [ ] Update Deno dev/test permissions and documentation for bounded GitHub-backed HTTP(S) import, likely `--allow-net=raw.githubusercontent.com`.
- [ ] Add focused unit, runtime, integration, and e2e tests, including one path each for whole-mesh, sidecar, and branch-based meshes.
- [ ] Add [[wu.cli-reference.import]] and update [[wu.cli-reference]], [[wu.environment-variables]], [[wu.repository-options]], and release notes.
- [ ] Keep the carried Bob `20-bob-page-imported-source` / `21-bob-page-imported-source-woven` expectations as context/regression only; do not use Bob content as the primary new fixture path.
- [ ] Run `deno task fmt`, `deno task lint`, `deno task check`, and `deno task test`.
- [ ] Provide a commit message with a summary line and detailed bullets for CLI, runtime/planner, docs, and tests.
