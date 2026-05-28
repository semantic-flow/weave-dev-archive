---
id: sxp044nqy4mepwb80wv6i8y
title: 2026 05 25_0958 Alice 14 19 Ladder Redesign
desc: ''
updated: 1779728403457
created: 1779728403457
---

## Goals

- Redesign Alice Bio rungs `14` through `19` so the main custom Alice page story is artifact-backed from the beginning instead of temporarily using loose `targetLocalRelativePath` Markdown sources.
- Import and weave governed `alice/bio` Markdown, and integrate/weave the shared `sidebar` content artifact, before Alice's custom `ResourcePageDefinition` points at them.
- Keep `alice/data` as Alice's RDF Turtle dataset and `alice/bio` as Alice's authored Markdown biography, so the fixture clearly separates data from narrative content.
- Use the freed `18` and `19` slots for a small, useful new artifact story: integrate and weave `favicon.ico` as the first image/binary `DigitalArtifact` in the Alice Bio ladder.
- Keep `targetLocalRelativePath` available as an ontology/runtime feature, but stop presenting it as the recommended page-source pattern for authored content that should be governed.
- Keep this redesign focused on the Alice 14-19 rung story; defer mesh-wide ResourcePage layout policy to [[wa.task.2026.2026-05-24_2304-honor-mesh-config]] and Carol-specific work to [[wa.completed.2026.2026-05-25_0849-carol]].

## Summary

The current Alice Bio `14` through `19` sequence proves useful mechanics but tells the wrong modeling story. `14` drops `alice/alice.md` and `mesh-content/sidebar.md` as loose mesh-local files, `15` weaves the custom Alice page from those loose sources, `16` and `17` introduce and weave governed `alice/page-main` and `sidebar` artifacts, and only `18` and `19` repoint the page definition to the governed artifacts.

The corrected ladder should introduce governed content artifacts first, then define the page against those artifacts:

- `14`: import `alice/bio` from the public notes GitHub source and integrate `sidebar` as governed Markdown payload artifacts.
- `15`: weave `alice/bio` and `sidebar`.
- `16`: define Alice's custom page with authored regions sourced through `sflo:hasTargetArtifact <alice/bio>` and `sflo:hasTargetArtifact <sidebar>`.
- `17`: weave Alice's custom page definition and update `/alice`.
- `18`: integrate `favicon.ico` as a governed non-RDF image/binary payload artifact.
- `19`: weave the favicon artifact, producing its artifact history and generated ResourcePages without incorrectly typing the bytes as RDF.

The favicon pair is a good replacement for the current "repoint page source" pair because the repoint becomes unnecessary once the ladder is ordered correctly. It also exercises a new and useful content kind. We already have runtime ResourcePage chrome that looks for a mesh-root `favicon.ico`, but that file is currently just an ambient file. Making it a governed artifact lets the mesh identify, version, and inspect the favicon bytes.

## Discussion

### Proposed Rung Story

`14-alice-bio-imported` should start from `13-bob-extracted-woven` and run the equivalent of:

```sh
weave import \
  "https://raw.githubusercontent.com/djradon/public-notes/db9a48933f0e6b208baeab7190cef75d1194634f/user.alice-ghostley.md" \
  alice/bio \
  --working-file alice-bio.md \
  --expected-digest sha256:0fcd9fe25c5598686557806cfdacc9c765176f315f780fa96644c3f251b49137

weave integrate mesh-content/sidebar.md --designator-path sidebar
```

The floating source requested by humans is `https://raw.githubusercontent.com/djradon/public-notes/refs/heads/main/user.alice-ghostley.md`, but the conformance rung should use a commit-pinned URL plus the expected digest above. The `main` branch resolved to commit `db9a48933f0e6b208baeab7190cef75d1194634f` when this task was written.

The replay profile can materialize `mesh-content/sidebar.md` from `.assets`; the import command should acquire `alice-bio.md` and record `sflo:ImportSource` provenance for `alice/bio`. This rung should update `_mesh/_inventory/inventory.ttl` and create the Knop support for both artifacts, but it should not yet change Alice's page definition or public `/alice` rendering.

`15-alice-bio-woven` should weave both governed content artifacts:

```sh
weave --history-tracking-policy versioned --target designatorPath=alice/bio --target designatorPath=sidebar
```

This should create `alice/bio` and `sidebar` histories, historical Markdown located files, and ResourcePages for those artifacts. Alice's own identifier page should still be unchanged because no custom page definition has been added yet.

`16-alice-page-customized` should add `alice/_knop/_page/page.ttl` and `alice/_knop/_assets/alice.css`, register the `ResourcePageDefinition` and `KnopAssetBundle`, and point the page regions directly at the governed artifacts:

```ttl
<#main-source> a sflo:ResourcePageSource ;
  sflo:hasTargetArtifact <https://semantic-flow.github.io/mesh-alice-bio/alice/bio> ;
  sflo:hasArtifactResolutionMode <https://semantic-flow.github.io/sflo/ontology/artifactResolutionMode_working> .

<#sidebar-source> a sflo:ResourcePageSource ;
  sflo:hasTargetArtifact <https://semantic-flow.github.io/mesh-alice-bio/sidebar> ;
  sflo:hasArtifactResolutionMode <https://semantic-flow.github.io/sflo/ontology/artifactResolutionMode_working> .
```

This rung should no longer create `alice/alice.md`, and the custom page definition should never assert `sflo:targetLocalRelativePath "alice/alice.md"` or `sflo:targetLocalRelativePath "mesh-content/sidebar.md"`.

`17-alice-page-customized-woven` should weave Alice's page definition and update `/alice` from the artifact-backed authored regions.

`18-favicon-integrated` should add a mesh-root `favicon.ico` file and register a payload artifact, probably at designator path `favicon`. The designator path should be stable and dereferenceable as `/favicon`, while the working located file remains `/favicon.ico` so generated pages can continue to use the conventional browser favicon URL.

`19-favicon-woven` should weave the `favicon` artifact. With filename-derived manifestation naming, the settled bytes should naturally land under an `ico` manifestation such as `favicon/_history001/_s0001/ico/favicon.ico`.

### Binary/Image Artifact Support

This is likely the first fixture that forces non-RDF binary artifact correctness. Current import behavior has a notion of RDF-like versus non-RDF payloads, but the integrate planner currently renders integrated payloads and located files as `sflo:RdfDocument` unconditionally. That would be wrong for `favicon.ico`.

The same correction should not remove RDF typing from `alice-data.ttl`: Alice's Turtle dataset at `alice/data` is still an RDF graph document and should remain typed as `sflo:RdfDocument`. The correction is to distinguish content kinds instead of assuming every payload is RDF. `alice/bio` Markdown and `favicon.ico` icon bytes should not be typed as RDF documents; `alice/data` should.

The redesign should either land a small integrate enhancement first or include it here:

- detect or accept whether an integrated source is RDF-like
- render non-RDF payloads as `sflo:PayloadArtifact, sflo:DigitalArtifact` without `sflo:RdfDocument`
- render non-RDF working and historical located files as `sflo:LocatedFile` without `sflo:RdfDocument`
- keep support artifacts such as metadata, inventories, and source registries as RDF documents
- preserve byte-for-byte copy/version behavior for binary files

No new core ontology class is required just to model an icon. `sflo:PayloadArtifact`, `sflo:DigitalArtifact`, `sflo:ArtifactManifestation`, and `sflo:LocatedFile` are enough for the first slice. If useful later, the mesh can add descriptive terms such as `dcterms:format` or `schema:encodingFormat`, but this task should not block on a richer media-type model.

### Relationship To Mesh-Wide Layout

This task should not add the references panel or all generated panels page-by-page. The mesh-wide layout policy belongs in [[wa.task.2026.2026-05-24_2304-honor-mesh-config]]. After that task lands, a later Alice Bio rung can change `_mesh/_config/config.ttl` and prove that the references panel appears wherever reference data exists.

## Open Issues

- Confirm the final rung names. The names above are descriptive placeholders; keeping the numeric slots `14` through `19` is more important than preserving the old labels.
- Confirm whether `alice/page-main` should disappear from this part of the ladder entirely. The current preference is to use imported `alice/bio` as the main authored region source and reserve `alice/page-main` for a future short page-intro artifact only if that need becomes real.
- Confirm the favicon artifact designator. `favicon` is the current preference because it gives the artifact a clean `/favicon` ResourcePage while keeping the working bytes at conventional root path `favicon.ico`.
- Decide whether `weave integrate` should infer non-RDF from file extension/content sniffing, expose an explicit `--content-kind` or `--rdf-document=false` option, or share import's RDF-like detection rules.
- Decide whether the favicon asset should be a real icon committed in `.assets/18-favicon-integrated/favicon.ico`, copied from the current mesh root if one exists, or generated specifically for the fixture.
- Decide whether `favicon.ico` should be added to source-only rung material when seeding future `b.00` branches, or only introduced at rung `18`.
- Confirm whether generated ResourcePages should link the favicon as soon as the root file exists, or whether only pages regenerated after rung `18` need to show the icon.

## Decisions

- Keep the direct-path `targetLocalRelativePath` page-source behavior in lower-level tests or an explicitly named loose-source fixture, not in the main Alice Bio custom page story.
- Use governed `alice/bio` and `sidebar` artifacts before Alice's custom page definition is introduced.
- Use a commit-pinned import source for Alice's Markdown bio in conformance, even if the human-facing source is the floating `main` raw GitHub URL.
- Keep `alice-data.ttl` / `alice/data` as the RDF dataset, and use `alice-bio.md` / `alice/bio` for authored Markdown biography content.
- Use `18` and `19` for `favicon.ico` rather than preserving a redundant page-source repoint step.
- Prefer designator path `favicon` with working located file `favicon.ico` unless implementation details expose a better mesh-level asset convention.
- Do not type `favicon.ico`, its payload artifact, its manifestation, or its located files as `sflo:RdfDocument`.

## Contract Changes

- Alice Bio conformance manifests for rungs `14` through `17` should stop asserting `targetLocalRelativePath "alice/alice.md"` and `targetLocalRelativePath "mesh-content/sidebar.md"` as the custom page source model.
- The fixture ladder script should reorder the `14` through `17` transition definitions so governed content artifacts exist before the custom Alice page definition references them.
- The old `18`/`19` page-source repoint transition should disappear because the page definition should be artifact-backed from its first state.
- Add `18`/`19` conformance manifests for integrating and weaving a non-RDF `favicon.ico` payload artifact.
- Add or update runtime/core behavior so integrate can represent non-RDF payload artifacts without false `sflo:RdfDocument` typing.
- Existing Bob `20`/`21` import-backed page-main rungs should continue to build on the corrected `19` output.

## Testing

- Unit-test non-RDF integrate planning for an image/binary source such as `favicon.ico`.
- Unit-test first payload weave for a non-RDF working file, including historical `ArtifactManifestation` and `LocatedFile` typing without `sflo:RdfDocument`.
- Regenerate and validate Alice Bio conformance manifests for `14` through `19`.
- Run focused weave tests that read the regenerated `a.14` through `a.19` branches.
- Verify `/alice` rendered output still includes the governed authored main region and sidebar after the reordered `16`/`17` pair.
- Verify `favicon/index.html`, `favicon/_history001/index.html`, and `favicon/_history001/_s0001/ico/index.html` are generated after `19`.
- Verify the root `favicon.ico` bytes match the fixture asset byte-for-byte, and the historical located file matches the same bytes.
- Verify existing pages are not broadly regenerated by the favicon rung unless the command intentionally targets them.

## Non-Goals

- Do not implement mesh-wide ResourcePage layout/profile config here; that belongs to [[wa.task.2026.2026-05-24_2304-honor-mesh-config]].
- Do not add Carol custom page behavior here; that belongs to [[wa.completed.2026.2026-05-25_0849-carol]].
- Do not remove `sflo:targetLocalRelativePath` from the ontology or runtime.
- Do not add a rich media-type ontology slice unless the first favicon integration cannot be represented correctly without it.
- Do not make every mesh asset automatically governed just because `favicon.ico` becomes governed in this fixture.
- Do not redesign Semantic Site styling or browser chrome beyond using the existing favicon detection behavior.

## Implementation Plan

- [x] Update `scripts/fixture-ladder.ts` so Alice Bio rung `14` imports `alice/bio` and integrates `sidebar`, rung `15` weaves them, rung `16` defines Alice's artifact-backed custom page, rung `17` weaves Alice's page, rung `18` integrates `favicon.ico`, and rung `19` weaves `favicon`.
- [x] Add fixture assets for the reordered rungs, including `mesh-content/sidebar.md`, `alice/_knop/_page/page.ttl`, `alice/_knop/_assets/alice.css`, and `favicon.ico`; the Alice bio Markdown should come through `weave import` rather than a manual file drop.
- [x] Add or update integrate support for non-RDF payload artifacts so `favicon.ico` is not typed as `sflo:RdfDocument`.
- [x] Add or update weave support for non-RDF payload histories and manifestations.
- [x] Rewrite Alice Bio conformance manifests `14` through `19` to match the reordered story.
- [x] Update Alice Bio conformance README/story text for the new `14` through `19` sequence.
- [x] Regenerate local Alice Bio fixture branches `a.14` through at least `a.27` so later rungs build on the corrected chain.
- [x] Update Weave fixture-backed tests that read `14` through `19`.
- [x] Run focused core/runtime tests for integrate, weave, ResourcePage rendering, and fixture ladder validation.
- [x] Update [[wa.task.2026.2026-05-23_2230-custom-resourcepage-shared-shell-fixture]] once this task supersedes its remaining Alice `14` through `19` checkbox.

## Implementation Notes

- Landed the reordered `14` through `19` story with `alice/bio`, `sidebar`, artifact-backed Alice page definition, and governed `favicon.ico`.
- Carried the downstream local fixture refs through `a.27-carol-woven` so the ladder remains linear after `20` now starts from `19-favicon-woven`.
- Added non-RDF payload typing and binary snapshot handling for integrate/weave while keeping RDF Turtle payloads typed as `sflo:RdfDocument`.
- Updated the conformance story for the new mesh inventory ordinals introduced by favicon and Carol's later nested `carol/data` weave.
- Validation run: `deno task check`; `deno test --allow-read --allow-env --allow-run=git src/core/integrate/integrate_test.ts`; `deno test --allow-read --allow-env --allow-run=git src/core/weave/weave_test.ts`; `deno test --allow-read --allow-write --allow-run=git,deno --allow-env tests/scripts/fixture_ladder_test.ts`; `deno task fixture:ladder --scenario alice-bio --check-scenario-index`; executed and validated `14-alice-bio-imported` through `27-carol-woven`.
