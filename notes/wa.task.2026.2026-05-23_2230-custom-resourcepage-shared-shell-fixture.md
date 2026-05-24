---
id: 8pctn1ttq5bnkweb5o79jtz
title: 2026 05 23_2230 Custom Resourcepage Shared Shell Fixture
desc: ''
updated: 1779599172163
created: 1779599172163
---

## Goals

- Add a narrow fixture proving a custom identifier ResourcePage can keep the shared Semantic Site shell and look while rendering authored Markdown content.
- Use imported or import-shaped Carol Burnett Markdown content as the first richer custom page source once [[wa.task.2026.2026-05-21_0907-import]] lands.
- Stack generated panels onto authored content explicitly, so the fixture proves custom content and generated ResourcePage panels compose without auto-appending everything.
- Keep the first fixture small enough to review before adding sidecar or branch-published custom page variants.

## Summary

This task follows the ResourcePage config/templating work and likely follows the first `weave import` slice. The desired fixture is not just "add another person." It should prove a workflow: import or otherwise materialize real Markdown content as a governed local artifact, bind a `ResourcePageDefinition` to that content, opt into the shared Semantic Site presentation, then explicitly add one generated panel after the authored region.

Carol Burnett content is a strong candidate because the commit-pinned source note includes YAML frontmatter, headings, images, lists, emphasis, and a Bob relationship. That gives the fixture more coverage than a tiny placeholder while still being human-readable.

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
- Opt into one generated panel, likely references or children, after authored content.
- Later add a Bob/Carol reference or reusable panel set once the first fixture is stable.

This should not be a whole-site look-and-feel update. A whole-site presentation update would intentionally change broad generated HTML/CSS output and likely force fixture ladder regeneration. This fixture should prove the shared-shell composition path with the smallest useful output change.

### Sequencing

`weave import` first is reasonable because it gives the fixture a real acquisition story and avoids manually sneaking external Markdown into the ladder. Fixture-helper generalization is still valuable, but it should be behavior-preserving. It should not require a full ladder regeneration unless the helper rewrite accidentally changes generated output.

Recommended sequence:

- Land a behavior-preserving fixture-helper generalization if it looks small and mechanical.
- Land `weave import` with a GitHub-backed Carol e2e/import fixture and lower-level local tests for failure cases.
- Add this custom ResourcePage fixture using imported Carol content.
- Only then consider broader fixture ladder variants or a whole-site look-and-feel update.

## Open Issues

- Should fixture-helper generalization happen immediately before this fixture, or immediately after the first narrow Carol fixture proves the user workflow?
- Should the first generated panel be `references`, `children`, or source/provenance? Recommendation: use the smallest panel that proves composition without introducing unrelated data requirements.
- Should the Carol Markdown remote images render as normal Markdown image links, be escaped/source-rendered, or be policy-controlled? Recommendation: treat them as authored Markdown links for now; Weave should not fetch those images during generation.
- Should the fixture introduce an explicit Bob/Carol semantic reference in the same rung, or reserve that for a second rung?

## Decisions

- Use the Carol Burnett Markdown note as the preferred first rich content candidate after import lands.
- Allow the import-backed custom ResourcePage fixture to depend on live GitHub for initial acquisition, using the commit-pinned Carol URL plus expected digest.
- Do not turn this into a whole-site presentation update.
- Keep generated panels explicit; the custom page should not receive default generated panels automatically.

## Contract Changes

- The fixture may add a new imported payload artifact, a Carol identifier/page definition, and expected generated ResourcePage output.
- If import lands first, the fixture should use the imported governed local working file as the authored page source.
- No new ontology vocabulary should be required for the first custom ResourcePage fixture beyond the ResourcePage config/templating vocabulary already added.

## Testing

- Add a fixture/e2e path proving Carol content can be imported from GitHub into a governed local working file.
- Add ResourcePage generation coverage proving the custom Carol page uses the shared shell and renders authored Markdown content.
- Add coverage proving the selected generated panel appears after authored content.
- Verify later generation does not fetch the remote Carol URL again.
- Avoid full ladder regeneration unless generated output intentionally changes for affected rungs.

## Non-Goals

- Do not implement reusable panel sets in this first custom fixture.
- Do not add raw authored HTML trust/sanitization behavior.
- Do not redesign the whole Semantic Site look and feel.
- Do not make remote image fetching part of page generation.
- Do not require sidecar or branch-published topology in the first custom page fixture.

## Implementation Plan

- [ ] Decide whether to land behavior-preserving fixture-helper generalization first.
- [ ] Land enough `weave import` behavior to materialize Carol Markdown into a governed local working file.
- [ ] Add the Carol custom page definition and authored Markdown binding.
- [ ] Opt the page into the shared Semantic Site presentation config.
- [ ] Opt the page into one generated panel after authored content.
- [ ] Add focused tests and regenerate only affected fixture expectations.
