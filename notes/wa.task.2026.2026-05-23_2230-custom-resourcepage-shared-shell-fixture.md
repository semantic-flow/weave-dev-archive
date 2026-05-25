---
id: 8pctn1ttq5bnkweb5o79jtz
title: 2026 05 23_2230 Custom Resourcepage Shared Shell Fixture
desc: ''
updated: 1779599172163
created: 1779599172163
---

## Goals

- Add a narrow fixture proving a custom identifier ResourcePage can keep the shared Semantic Site shell and look while rendering authored Markdown content.
- Use the next from-scratch Alice Bio fixture regeneration to correct the older Alice/Bob page-source rungs so authored Markdown becomes governed content-panel source material rather than loose mesh files that make an identifier page look like the payload itself.
- Use that regeneration to rename Alice's RDF payload from `alice-bio.ttl` at `alice/bio` to `alice-data.ttl` at `alice/data`, so the RDF data artifact is clearly distinct from authored Markdown biography/page copy.
- Use imported or import-shaped Carol Burnett Markdown content as the first richer custom page source once [[wa.task.2026.2026-05-21_0907-import]] lands.
- Stack generated panels onto authored content explicitly, so the fixture proves custom content and generated ResourcePage panels compose without auto-appending everything.
- Keep the first fixture small enough to review before adding sidecar or branch-published custom page variants.

## Summary

This task follows the ResourcePage config/templating work and likely follows the first `weave import` slice. The desired fixture is not just "add another person." It should prove a workflow: import or otherwise materialize real Markdown content as a governed local artifact, bind a `ResourcePageDefinition` to that content, opt into the shared Semantic Site presentation, then explicitly add one generated panel after the authored region.

Carol Burnett content is a strong candidate because the commit-pinned source note includes YAML frontmatter, headings, images, lists, emphasis, and a Bob relationship. That gives the fixture more coverage than a tiny placeholder while still being human-readable.

The Alice Bio ladder already has related custom page rungs, but they were created before the current content-panel and import/source-binding direction settled. Rungs `14` through `19` add loose Markdown page sources for Alice and later repoint the main region to `alice/page-main`; rungs `20` and `21` claim an import boundary for Bob while actually dropping a loose `bob-page-main.md` and wiring Bob's identifier page to it. In the next from-scratch regeneration, these should be corrected instead of carried forward as recommended examples.

## Discussion

Recommended first content:

```sh
weave import \
  "https://raw.githubusercontent.com/djradon/public-notes/f46d85187ed7781917b73dd7779b756e2d2b7494/user.carol-burnett.md" \
  carol/page-main \
  --working-file carol/page-main.md \
  --expected-digest sha256:5634ffc14165c55ab43c2af38b9d6395e22c8385f54d4a94a7d22d83c99afee7
```

The imported content should become governed local bytes. Page generation should read the local working file, not refetch the remote URL.

Possible fixture story:

- Add a `carol` identifier or custom page path in the Alice Bio fixture ladder.
- Import Carol Markdown into a governed local artifact such as `carol/page-main`.
- Add a `ResourcePageDefinition` for Carol that binds an authored Markdown region to `carol/page-main`.
- Set `sfcfg:hasResourcePagePresentationConfig` on that page definition so the custom page uses the shared Semantic Site shell.
- Opt into the `children` generated panel after authored content.
- In a second rung, add the Bob/Carol semantic reference or related composition coverage once the first shared-shell fixture is stable.

This should not be a whole-site look-and-feel update. A whole-site presentation update would intentionally change broad generated HTML/CSS output and likely force fixture ladder regeneration. This fixture should prove the shared-shell composition path with the smallest useful output change.

### Alice/Bob Regen Correction

The current Alice `14` through `19` fixture sequence proves several useful implementation abilities:

- define a Knop-owned `ResourcePageDefinition`
- render authored Markdown regions
- register and weave a governed `alice/page-main` Markdown payload
- repoint a page region from direct path resolution to artifact-backed resolution

The awkward part is the story it tells. `14` initially drops `alice/alice.md` and `mesh-content/sidebar.md` as loose mesh-local Markdown page sources. That makes a custom `/alice` page behave like a small one-off Markdown page. For Semantic Flow, the better story is that `/alice` remains Alice's identifier ResourcePage, and Alice's authored bio or narrative appears as an authored content panel sourced from a governed artifact such as `alice/page-main`.

During a from-scratch regen, prefer this shape:

- rename the RDF data payload from `alice-bio.ttl` / `alice/bio` to `alice-data.ttl` / `alice/data`
- introduce or integrate `alice/page-main` before the custom Alice page definition uses it
- make `alice/_knop/_page/page.ttl` source its main authored region from `sflo:hasTargetArtifact <alice/page-main>` rather than from a loose `targetLocalRelativePath "alice/alice.md"`
- opt the page definition into the shared Semantic Site presentation config
- explicitly select the `children` generated panel after authored content
- avoid treating the custom page as if it replaced the identifier page's generated semantics wholesale

Alice's main page Markdown should be concise and explicit about identifier semantics. The fixture copy should say, in substance: `https://semantic-flow.github.io/mesh-alice-bio/alice` is an IRI which identifies Alice, the person. It demonstrates the [Semantic Flow framework](https://semantic-flow.github.io/sflo). Keep the wording short enough that the authored region does not read like a replacement for the generated identifier-page panels.

The shared sidebar should become a governed content artifact and should be reused anywhere this ladder uses the shared authored shell. Re-examine the sidebar copy during regeneration rather than carrying the current quick-links list forward unchanged. One caveat: this should not quietly imply automatic sidebar injection into every generated ResourcePage unless the shared shell/template behavior is intentionally expanded; for this task, reuse means the custom page definitions that opt into an authored sidebar should point at the same governed sidebar artifact.

Bob `20` and `21` have the same issue plus a naming problem. They are not honest `weave import` examples today: they do not create `bob/page-main`, do not create Bob page-main Knop support, and do not record import provenance. The import task [[wa.task.2026.2026-05-21_0907-import]] already captures the replacement: make Bob `20/21` a real import of Bob Markdown into `bob/page-main`, then use that governed content as Bob's authored ResourcePage panel source.

This task owns the page-composition side of that correction. The import task owns the acquisition/provenance side.

### Sequencing

`weave import` first is reasonable because it gives the fixture a real acquisition story and avoids manually sneaking external Markdown into the ladder. Fixture-helper generalization is still valuable, but it should be behavior-preserving. It should not require a full ladder regeneration unless the helper rewrite accidentally changes generated output.

Recommended sequence:

- Land a behavior-preserving fixture-helper generalization if it looks small and mechanical.
- Land `weave import` with a GitHub-backed Carol e2e/import fixture and lower-level local tests for failure cases.
- In the next from-scratch Alice Bio fixture regeneration, correct Alice `14` through `19` and Bob `20` through `21` so custom identifier pages use governed authored content panels instead of loose Markdown page sources.
- Add this custom ResourcePage fixture using imported Carol content, or defer Carol until after Alice/Bob have been corrected if the regen is already carrying enough page-source churn.
- Only then consider broader fixture ladder variants or a whole-site look-and-feel update.

## Open Issues

- Resolved: fixture-helper generalization has landed before this fixture/regeneration work.
- Resolved: use `children` as the first generated panel after authored content.
- Resolved: treat Carol Markdown remote images as normal authored Markdown links for now; Weave should not fetch those images during generation.
- Resolved: reserve the explicit Bob/Carol semantic reference or richer composition proof for a second rung.
- Resolved: Alice's shared sidebar should become a governed content artifact, but its content should be re-examined before regeneration.
- Should Alice `14` through `19` be renumbered around the corrected `alice/page-main` sequence, or should a new `b.*` ladder preserve old `a.*` numbers as historical comparison?
- Confirm whether `alice/data` should replace every current `alice/bio` reference in examples, tests, and docs during the same regeneration, or whether any compatibility note should mention the historical `alice/bio` path.

## Progress Notes

- Removed Alice/Bob fixture-specific first-extracted page rendering from Weave core; the Bob extraction rung now uses the generic extracted identifier page path.
- Renamed Alice's RDF payload fixture from `alice-bio.ttl` / `alice/bio` to `alice-data.ttl` / `alice/data` across the ladder assets, conformance manifests, tests, and CLI examples.
- Rewrote Alice's authored main content so it says that `https://semantic-flow.github.io/mesh-alice-bio/alice` is the IRI for Alice, the person, and links to the Semantic Flow framework.
- Integrated a governed reusable `sidebar` content artifact and pointed later Alice/root custom page definitions at it. The early authored Markdown rung remains loose for now, matching the review decision that authored Markdown is acceptable at that rung.
- Regenerated the Alice Bio local fixture branches and conformance expectations through the affected root page rungs.
- Because the `mesh-alice-bio` asset changes are being committed on `main`, remember to seed the next `b.00` source rung by copying the committed `.assets` tree into that rung before generating the `b.*` ladder. Track the broader automation gap in [[wa.task.2026.2026-05-24_2058-fixture-ladder-accord-file-operations]].

## Decisions

- Use the Carol Burnett Markdown note as the preferred first rich content candidate after import lands.
- Allow the import-backed custom ResourcePage fixture to depend on live GitHub for initial acquisition, using the commit-pinned Carol URL plus expected digest.
- Do not turn this into a whole-site presentation update.
- Keep generated panels explicit; the custom page should not receive default generated panels automatically.
- Use the explicit `children` generated panel as the first post-authored panel in the shared-shell fixture.
- Treat authored Markdown links, including remote images in imported Markdown, as normal Markdown links for now.
- Put richer Bob/Carol relationship coverage in a second rung after the first shared-shell composition proof.
- Treat the current Alice `14` through `19` direct Markdown page-source story as historical, not as the recommended modeling pattern.
- Treat Bob `20` and `21` as future honest import/page-composition rungs: import Bob Markdown into governed `bob/page-main`, then render Bob's identifier ResourcePage from that governed content panel.
- A custom identifier page should preserve the identifier-page semantics and compose authored content as panels; it should not make `/alice` or `/bob` masquerade as the Markdown payload artifact itself.
- Fixture-helper generalization has landed, so the next fixture work can use the generalized helper path rather than blocking on helper cleanup.
- The original non-`a.*` numbered fixture branches have been deleted from the `mesh-alice-bio` and `mesh-sidecar-fantasy-rules` remotes before the next from-scratch regeneration. `mesh-branch-fantasy-rules` already only had `a.*`, `main`, and `gh-pages` remote branches.
- Rename the Alice RDF data payload to `alice-data.ttl` and publish it as `alice/data`, distinguishing data from authored Markdown bios/page content.
- Make the shared sidebar a governed reusable content artifact and point custom page definitions at that artifact rather than at a loose mesh-local Markdown helper file.
- Rewrite `alice/page-main` content so it explains that `https://semantic-flow.github.io/mesh-alice-bio/alice` is the IRI for Alice, the person, and links to the Semantic Flow framework at `https://semantic-flow.github.io/sflo`.

## Contract Changes

- The fixture may add a new imported payload artifact, a Carol identifier/page definition, a governed shared-sidebar artifact, and expected generated ResourcePage output.
- If import lands first, the fixture should use the imported governed local working file as the authored page source.
- No new ontology vocabulary should be required for the first custom ResourcePage fixture beyond the ResourcePage config/templating vocabulary already added.
- Alice Bio conformance manifests should stop describing loose Markdown page-source drops as the recommended custom page path once the ladder is regenerated.
- Alice Bio conformance manifests, fixture assets, tests, and docs should reflect `alice-data.ttl` / `alice/data` instead of `alice-bio.ttl` / `alice/bio` for the RDF data artifact.
- Bob import conformance should assert `sflo:ImportSource` provenance and governed `bob/page-main` payload support rather than only a local `bob-page-main.md` file.

## Testing

- Add a fixture/e2e path proving Carol content can be imported from GitHub into a governed local working file.
- Add ResourcePage generation coverage proving the custom Carol page uses the shared shell and renders authored Markdown content.
- Add coverage proving the selected generated panel appears after authored content.
- Verify later generation does not fetch the remote Carol URL again.
- Avoid full ladder regeneration unless generated output intentionally changes for affected rungs.
- During from-scratch Alice Bio regen, update the `14` through `19` manifests so Alice's authored main content region is artifact-backed from governed `alice/page-main`, not direct `targetLocalRelativePath "alice/alice.md"`.
- During from-scratch Alice Bio regen, update the shared sidebar expectations so custom pages use a governed shared-sidebar artifact.
- During from-scratch Alice Bio regen, rename the RDF payload source and designator from `alice-bio.ttl` / `alice/bio` to `alice-data.ttl` / `alice/data`.
- During from-scratch Alice Bio regen, update `20` and `21` so Bob's Markdown is acquired with real `weave import`, produces `bob/page-main` support, records `ImportSource` provenance and observed digest evidence, and renders Bob's identifier page from the governed content panel.
- Add rendered-output expectations, if Accord supports them, proving authored content appears as an authored content panel while selected generated panels still appear on the same identifier page.

## Non-Goals

- Do not implement reusable panel sets in this first custom fixture.
- Do not add raw authored HTML trust/sanitization behavior.
- Do not redesign the whole Semantic Site look and feel.
- Do not make remote image fetching part of page generation.
- Do not require sidecar or branch-published topology in the first custom page fixture.
- Do not preserve Alice/Bob loose Markdown page-source rungs as recommended examples during a full from-scratch regen.
- Do not make `ResourcePageSource targetAccessUrl` a supported rendering path as part of this correction.
- Do not make the governed sidebar imply automatic sidebar injection into all generated ResourcePages unless a separate shell/template task intentionally expands that behavior.

## Implementation Plan

- [x] Decide whether to land behavior-preserving fixture-helper generalization first.
- [x] Delete original non-`a.*` numbered fixture branches from the fixture mesh remotes before the next from-scratch regeneration.
- [x] Land enough `weave import` behavior to materialize Carol Markdown into a governed local working file.
- [x] During the next from-scratch Alice Bio regeneration, rename Alice's RDF payload source and designator from `alice-bio.ttl` / `alice/bio` to `alice-data.ttl` / `alice/data`.
- [ ] Copy the committed `mesh-alice-bio` `.assets` tree from `main` into the next `b.00` source rung before generating `b.*` branches, or replace the manual step with [[wa.task.2026.2026-05-24_2058-fixture-ladder-accord-file-operations]] first.
- [ ] During the next from-scratch Alice Bio regeneration, adjust Alice `14` through `19` so the main authored content panel is backed by governed `alice/page-main` from the start of the custom page story.
- [x] During the next from-scratch Alice Bio regeneration, rewrite `alice/page-main` content to identify Alice's IRI and link to the Semantic Flow framework.
- [x] During the next from-scratch Alice Bio regeneration, make the shared sidebar a governed reusable content artifact and revise its content.
- [ ] During the next from-scratch Alice Bio regeneration, replace Bob `20` and `21` with real import-backed `bob/page-main` rungs and artifact-backed authored content panels.
- [ ] Add the Carol custom page definition and authored Markdown binding.
- [ ] Opt the page into the shared Semantic Site presentation config.
- [ ] Opt the page into the `children` generated panel after authored content.
- [ ] Add a second rung for the Bob/Carol semantic reference or richer composition proof.
- [x] Add focused tests and regenerate only affected fixture expectations.
