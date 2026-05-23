---
id: k7m2q9z6v1x4f8r0s3p5t2n
title: 2026 05 22_2253 ResourcePage Config And Templating
desc: ''
updated: 1779509580000
created: 1779509580000
---

## Goals

- Converge default generated ResourcePages and customized `_knop/_page` identifier pages onto one presentation pipeline.
- Preserve or intentionally port the existing Semantic Site look and feel as the built-in default ResourcePage presentation.
- Make current generated sections such as children, properties, references, histories, raw source, provenance, and Semantic Flow metadata reusable panel models.
- Keep `ResourcePagePresentationConfig`, `OuterResourcePageTemplate`, `InnerResourcePageTemplate`, and `ResourcePageStylesheet` as presentation/chrome concerns, separate from content-source resolution.
- Let default pages use synthesized definitions/panels from runtime context and effective config, without requiring every default page to persist a `_knop/_page/page.ttl`.

## Summary

[[wa.task.2026.2026-04-08_1545-resource-page-definition-and-sources]] has become the historical home for both page-source semantics and later presentation/template ideas. The page-source parts are still useful, but the default/custom ResourcePage presentation direction is now distinct enough to deserve its own task.

Current default ResourcePages are rendered by TypeScript functions in `src/runtime/weave/pages.ts` with inline CSS behind an internal `ResourcePageTheme` seam. Custom identifier pages use `ResourcePageDefinition` / `ResourcePageRegion` / `ResourcePageSource` for content composition plus optional `_knop/_assets` stylesheets. They do not yet share the same presentation model as generic default pages.

This task should define and then implement the cleaner model: default pages and customized pages should both be explainable as a resource-page model plus presentation config plus reusable panels. Explicit `_knop/_page/page.ttl` remains the authored override for identifier-page content composition, but generic pages can synthesize equivalent default inputs at render time.

## Discussion

The desired end state is not "templates own everything." Runtime code should continue to compute graph-aware data: child identifiers, RDF facts, references, history groups, raw-source panels, breadcrumbs, source/provenance summaries, and policy-filtered ResourcePage paths. Presentation config and templates should decide how those structured models are arranged and displayed.

Useful conceptual layers:

- `ResourcePageDefinition`: declares or synthesizes page content regions and their source bindings.
- `ResourcePagePresentationConfig`: selects outer shell, inner/body layout, stylesheets, panel inclusion/order, and chrome behavior.
- `OuterResourcePageTemplate`: renders the page shell, metadata, global navigation/chrome, and site-level wrapper.
- `InnerResourcePageTemplate`: renders the body/content layout for a page kind or panel arrangement.
- `ResourcePageStylesheet`: carries reusable style assets, including the built-in Semantic Site look and any mesh-specific stylesheet resources.
- ResourcePage panel models: named structured components such as children, properties, references, histories, raw source, provenance, and Semantic Flow metadata.

The default experience should be modeled as built-in config and templates, not as special branches that cannot be reused. A mesh or Knop can later override pieces deliberately, but the default should remain the easiest path and the fallback for incomplete config.

The current major refactor matters here. [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] and related runtime page-generation extraction work are making the code easier to change. This task should wait until the relevant model-builder and page-renderer seams are stable enough to keep the behavioral diff small.

## Open Issues

- Which current generated sections should become first-class panel model types, and which should stay local renderer details?
- Should panel inclusion/order use config ontology terms, page-definition terms, or a small presentation-only panel vocabulary?
- Should the built-in Semantic Site stylesheet become a `ResourcePageStylesheet` artifact, an embedded default, or both?
- How should synthesized default page definitions be described in diagnostics without implying that a physical `_knop/_page/page.ttl` exists?
- Should custom identifier pages be able to opt back into default panels such as children, references, and histories, or is that a later composition feature?
- How should `ResourcePagePresentationConfig` inherit across mesh, Knop, artifact role, and page kind?
- How much generated HTML drift is acceptable when porting the current default look into the shared pipeline?

## Decisions

- Default and customized ResourcePages should share a presentation pipeline.
- Default pages may use synthesized definitions and panel sets; they do not need persisted `_knop/_page/page.ttl` files.
- The existing Semantic Site look and feel is the default presentation baseline unless a later design task explicitly replaces it.
- Runtime code computes graph-aware page and panel data. Templates render structured inputs and must not own RDF discovery, source resolution, or navigation computation.
- Page-content composition stays separate from template/chrome policy. `ResourcePagePresentationConfig` is adjacent to `ResourcePageDefinition`, not a replacement for source resolution.

## Contract Changes

- No immediate CLI or ontology behavior change is required just to create this task.
- Later implementation may make default generated pages and custom identifier pages share internal model types and presentation config resolution.
- Later implementation may add or refine config ontology terms for panel inclusion/order, built-in templates, and stylesheet selection.
- Generated HTML should remain stable until an implementation slice explicitly records expected presentation diffs.

## Testing

- Preserve current generated HTML output while extracting reusable panel models unless a slice explicitly updates expected output.
- Add focused renderer/model tests for synthesized default page definitions and panel ordering.
- Add tests proving custom identifier pages can keep the default Semantic Site chrome when desired.
- Add tests proving templates receive structured panel inputs and do not need to parse RDF or local files.
- Run focused `src/runtime/weave/pages_test.ts`, runtime page-generation integration tests, and any fixture-stable generated-page comparisons after implementation slices.

## Non-Goals

- Do not combine this with the current move-only core weave refactor.
- Do not require every default ResourcePage to persist a `_knop/_page/page.ttl`.
- Do not make templates responsible for RDF graph discovery, history resolution, source lookup, or local/remote access policy.
- Do not add a client-side app framework for ResourcePages.
- Do not replace page-source resolution work from [[wa.task.2026.2026-04-08_1545-resource-page-definition-and-sources]].
- Do not implement remote `targetAccessUrl` or `workingAccessUrl` fetching as part of presentation templating.

## Implementation Plan

- [ ] Audit current config ontology terms for `ResourcePagePresentationConfig`, `OuterResourcePageTemplate`, `InnerResourcePageTemplate`, and `ResourcePageStylesheet`, and list any missing panel-selection terms.
- [ ] Inventory current default generated-page sections in `src/runtime/weave/pages.ts` and map them to candidate panel model names.
- [ ] Decide which default page inputs are synthesized runtime defaults versus persisted mesh/config artifacts.
- [ ] Define a first internal panel model shape that can represent children, properties, references, histories, raw source, provenance, and Semantic Flow metadata.
- [ ] Refactor renderer internals so default pages consume panel models without changing generated output.
- [ ] Define how custom identifier pages can use the same outer shell and stylesheet baseline as default pages.
- [ ] Add or update tests for default/custom shared presentation behavior.
- [ ] Only after the model is stable, decide whether to persist built-in templates/stylesheets as RDF-described artifacts or keep them as embedded defaults with RDF identities.
