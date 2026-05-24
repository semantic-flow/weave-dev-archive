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
- Let default pages use synthesized document/panel inputs from runtime context and effective config, without requiring every default page to persist a `_knop/_page/page.ttl`.
- Treat Weave's `/defaults` RDF files as the likely home for application-default ResourcePage presentation selection, not just invisible hard-coded renderer behavior.
- Split implementation into a zero-drift internal panel extraction, a later RDF/config selection slice, and a later custom-page chrome-sharing slice instead of trying to solve templating, config inheritance, and rendering in one pass.

## Summary

[[wa.task.2026.2026-04-08_1545-resource-page-definition-and-sources]] has become the historical home for both page-source semantics and later presentation/template ideas. The page-source parts are still useful, but the default/custom ResourcePage presentation direction is now distinct enough to deserve its own task.

Current default ResourcePages are rendered by TypeScript functions in `src/runtime/weave/pages.ts` with inline CSS behind an internal `ResourcePageTheme` seam. Custom identifier pages use `ResourcePageDefinition` / `ResourcePageRegion` / `ResourcePageSource` for content composition plus optional `_knop/_assets` stylesheets. They do not yet share the same presentation model as generic default pages.

This task should define and then implement the cleaner model: default pages and customized pages should both be explainable as a resource-page document model plus presentation config plus reusable panels. Explicit `_knop/_page/page.ttl` remains the authored override for identifier-page content composition. Generic pages can synthesize default document and panel inputs at render time, but those synthesized inputs should not be described as a `ResourcePageDefinition` in diagnostics because no physical `_knop/_page/page.ttl` exists.

## Discussion

The desired end state is not "templates own everything." Runtime code should continue to compute graph-aware data: child identifiers, RDF facts, references, history groups, raw-source panels, breadcrumbs, source/provenance summaries, and policy-filtered ResourcePage paths. Presentation config and templates should decide how those structured models are arranged and displayed.

Useful conceptual layers:

- `ResourcePageDefinition`: declares authored page content regions and their source bindings for explicit `_knop/_page` customization.
- Internal ResourcePage document model: synthesized or assembled runtime input for rendering a page, including title, canonical IRI, breadcrumbs, classes, summary, chrome metadata, stylesheet selections, and ordered panels.
- `ResourcePagePresentationConfig`: selects outer shell, inner/body layout, stylesheets, panel inclusion/order, and chrome behavior.
- `OuterResourcePageTemplate`: renders the page shell, metadata, global navigation/chrome, and site-level wrapper.
- `InnerResourcePageTemplate`: renders the body/content layout for a page kind or panel arrangement.
- `ResourcePageStylesheet`: carries reusable style assets, including the built-in Semantic Site look and any mesh-specific stylesheet resources.
- ResourcePage panel models: named structured components such as children, properties, references, histories, raw source, provenance, and Semantic Flow metadata.

The default experience should be modeled as built-in config and templates, not as special branches that cannot be reused. A mesh or Knop can later override pieces deliberately, but the default should remain the easiest path and the fallback for incomplete config.

The current major refactor matters here. [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] and related runtime page-generation extraction work are making the code easier to change. This task should wait until the relevant model-builder and page-renderer seams are stable enough to keep the behavioral diff small.

### Synthesized Default ResourcePage Documents

Default generated pages should be described as synthesized ResourcePage documents, not synthesized `ResourcePageDefinition` artifacts. A `ResourcePageDefinition` remains the authored `_knop/_page/page.ttl` source of truth for custom identifier pages. The synthesized default document is the resolved runtime input to rendering: it combines runtime-discovered resource facts, selected built-in panels, effective presentation config, and page chrome data into a `ResourcePageDocumentModel`.

Diagnostics should use language such as "resolved ResourcePage document" or "synthesized default ResourcePage document." They should not imply that a physical `_knop/_page/page.ttl` exists, nor should they call the synthesized runtime object a `ResourcePageDefinition`.

### External Template Contract

The first external-template boundary should be a data contract, not a general plugin execution API. Runtime remains responsible for RDF reads, file reads, mesh inventory access, effective-config resolution, source lookup, navigation computation, and access policy. A template request contains only the resolved `ResourcePageDocumentModel` plus the selected template descriptor (`iri` and `outer`/`inner` role). A template result is either complete page HTML or named HTML fragments.

The initial fragment slots are intentionally coarse: `head`, `shell`, `masthead`, `body`, `panels`, and `footer`. These slots are enough to describe the current Semantic Site page shape while avoiding a premature low-level component API. Later slices may add narrower panel or chrome slots when a real alternate template needs them.

The core TypeScript contract lives in `src/core/weave/resource_page_template_contract.ts` and deliberately exports serializable request/result shapes rather than a renderer function type. A later loader may adapt executable code, declarative layout files, or another template engine to this boundary, but the stable cross-boundary shape should stay document-in/result-out.

Implementation should proceed in deliberately separate slices:

- First slice: extract internal `ResourcePagePanelModel` and ResourcePage document-model assembly from the current default renderer, with no ontology change and no generated HTML drift. This should convert the existing children, properties, blank nodes, references, history, raw source, Semantic Flow metadata, ReferenceCatalog links, and Knop artifact lists into panel-shaped inputs while leaving current shell/header/footer behavior intact.
- Second slice: define and parse the minimal config ontology needed for application-default presentation selection and built-in code-backed panel selection. This is the earliest slice that should update `defaults/application.ttl`.
- Third slice: let custom identifier pages use the same outer Semantic Site shell, stylesheet baseline, and optional default panels through explicit composition. Do not auto-append default panels to every custom page.
- Later slices: external template loading, richer config inheritance behavior, arbitrary custom panel renderers, and HTML trust/sanitization policy.

Settled design direction:

- Application defaults should enter through `defaults/application.ttl`, alongside the existing history/page-generation defaults, but only in the same slice that parses and uses the selected presentation identity. Do not add ignored decorative RDF triples that imply behavior runtime does not yet honor.
- Add `sfcfg:hasDefaultResourcePagePresentationConfig` with domain `sfcfg:Config` for application, mesh, and Knop config scopes. Keep the existing `sfcfg:hasResourcePagePresentationConfig` domain on `sflo:ResourcePageDefinition`; it remains the adjacent override for explicit `_knop/_page` definitions.
- The first RDF model should only select code-backed panels. It must not define arbitrary panel renderers, query languages, external template execution, or third-party plugin behavior.
- Use `ResourcePagePanel` / "panel" as the durable vocabulary term unless ontology review turns up a real conflict. Keep it distinct from `ResourcePageRegion`: regions are authored content composition slots in `_knop/_page`; panels are generated or code-backed structured components such as children, references, histories, raw source, provenance, metadata, or authored Markdown content; sections are rendered HTML output.
- Panel selection needs targeting, not just ordering. The first targeting model should support page kind, RDF class, artifact role, and data availability. Prefer direct target references such as `sflo:DigitalArtifact` or `sfcfg:artifactRole_payload`; introduce named target-profile individuals only for reusable combinations, not as the only targeting mechanism.
- Model authored Markdown before authored HTML. A single authored-content panel can dispatch by supported media type, but the first supported rendered content type should be safe Markdown. Raw authored HTML should be deferred or escaped/rendered as source until there is an explicit trust and sanitizer model.
- Prefer hash identifiers or other stable named identifiers over blank nodes for small presentation selections when practical. Named selection nodes are easier to reuse, override, inspect, and dereference than blank nodes, even when they are too small to deserve a full Knop.

Possible default shape to evaluate:

```ttl
<application>
  sfcfg:hasDefaultResourcePagePresentationConfig
    <resource-page-presentation/semantic-site-default> .

<resource-page-presentation/semantic-site-default>
  a sfcfg:ResourcePagePresentationConfig ;
  sfcfg:hasOuterResourcePageTemplate
    <resource-page-template/semantic-site/outer> ;
  sfcfg:hasInnerResourcePageTemplate
    <resource-page-template/semantic-site/inner> ;
  sfcfg:hasResourcePageStylesheet
    <resource-page-stylesheet/semantic-site> ;
  sfcfg:hasResourcePagePanelSelection
    <resource-page-presentation/semantic-site-default#children-panel>,
    <resource-page-presentation/semantic-site-default#current-working-source-panel> .

<resource-page-presentation/semantic-site-default#children-panel>
  a sfcfg:ResourcePagePanelSelection ;
  sfcfg:hasResourcePagePanel sfcfg:resourcePagePanel_children ;
  sfcfg:panelOrder "10"^^xsd:nonNegativeInteger ;
  sfcfg:hasPanelInclusionPolicy sfcfg:panelInclusionPolicy_auto .

<resource-page-presentation/semantic-site-default#current-working-source-panel>
  a sfcfg:ResourcePagePanelSelection ;
  sfcfg:hasResourcePagePanel sfcfg:resourcePagePanel_currentWorkingSource ;
  sfcfg:panelOrder "80"^^xsd:nonNegativeInteger ;
  sfcfg:hasPanelInclusionPolicy sfcfg:panelInclusionPolicy_auto ;
  sfcfg:hasPanelTargetClass sflo:DigitalArtifact ;
  sfcfg:hasPanelDataRequirement sfcfg:panelDataRequirement_currentWorkingSource .
```

The exact property names should be refined with the ontology open, but this shows the desired split: defaults select stable built-in template/panel identities; code implements the renderers; targeting decides whether a selected panel applies to a given page model. If named target profiles are added later, they should be shorthand for inspectable target constraints rather than opaque magic values.

## Open Issues

- What exact merge controls are needed for additive panel selections versus nearest-wins template/style selection? Initial likely rule: outer/inner templates are nearest-wins; stylesheets and panels are ordered/additive until replace/remove vocabulary exists.
- What exact replace/remove vocabulary should panel and stylesheet selection use once additive inherited defaults become insufficient?
- What exact low-impact panel presentation modes should be supported later, such as footer icons, menus, or popovers?

## Decisions

- Default and customized ResourcePages should share a presentation pipeline.
- Default pages may use synthesized `ResourcePageDocumentModel` values and panel sets; they do not need persisted `_knop/_page/page.ttl` files.
- The existing Semantic Site look and feel is the default presentation baseline unless a later design task explicitly replaces it.
- Runtime code computes graph-aware page and panel data. Templates render structured inputs and must not own RDF discovery, source resolution, or navigation computation.
- Page-content composition stays separate from template/chrome policy. `ResourcePagePresentationConfig` is adjacent to `ResourcePageDefinition`, not a replacement for source resolution.
- Panel renderer implementation stays in code for the first slice. RDF selection should choose from stable built-in panel identities rather than define arbitrary renderer behavior.
- Prefer named/hash-identified presentation selection nodes over blank nodes when those nodes may be reused, overridden, debugged, or dereferenced.
- The first implementation slice should preserve generated HTML exactly and should not change ontology, default RDF, CLI behavior, or fixture expectations.
- Use config ontology terms for panel inclusion/order/targeting. Do not put panel ordering on `ResourcePageDefinition` or `ResourcePageRegion`; those remain authored content composition terms.
- Use `ResourcePagePanel` / "panel" as the working durable term while preserving a clear distinction from `ResourcePageRegion` and rendered HTML sections.
- Prefer targeting by direct RDF classes, artifact roles, page kinds, and data availability over opaque target individuals. Named target individuals may be added later as reusable profiles.
- Defer raw authored HTML rendering. Render safe Markdown first; escape or source-render HTML until an explicit trusted/sanitized HTML policy exists.
- Persist built-in Semantic Site templates and stylesheets as RDF-described defaults artifacts once the model is stable. The stylesheet metadata artifact should live at `defaults/default-stylesheet.ttl`, and the actual stylesheet artifact should use `defaults/stylesheet.css` as its working file.
- Custom identifier pages may opt into default panels later, but they should not receive generated default panels automatically.
- Add `sfcfg:hasDefaultResourcePagePresentationConfig` with domain `sfcfg:Config`; do not broaden or replace `sfcfg:hasResourcePagePresentationConfig`.
- Core should own presentation-neutral `ResourcePageDocumentModel`, `ResourcePagePanelModel`, and panel payload model types. Runtime should own model assembly because runtime reads files, resolves effective config, loads mesh state, and enforces local/remote access policy. HTML-specific adapter rows, CSS classes, rendered HTML strings, syntax highlighting, and renderer implementation details should stay out of core.
- Treat the current `ResourcePageRenderInput` as a renderer-local adapter, not the durable public model. It may be retired if `ResourcePageDocumentModel` and panel rendering make it unnecessary.
- Use `defaults/` IRIs for built-in Semantic Site presentation, template, stylesheet, and panel-selection resources. Do not mint duplicate config-ontology individuals for the same default resources merely to have vocabulary-local IRIs.
- When custom identifier pages opt into default generated panels and no explicit order is provided, authored regions should render before full generated panels.
- Future panel selection should leave room for low-impact panel presentation modes such as footer icons, menus, or popovers, but the first shared-pipeline slice should render ordinary full panels only.
- Diagnostics should describe computed inputs as a `ResourcePageDocumentModel`, resolved ResourcePage document, or synthesized default ResourcePage document, not as a `ResourcePageDefinition`.
- External templates should receive a resolved ResourcePage document model and return page HTML or named HTML fragments. Templates must not read RDF, local files, remote URLs, mesh inventory, or config sources directly; they must render structured panel inputs and chrome metadata supplied by runtime. The first external-template mechanism should prefer declarative slot/layout selection over arbitrary executable template code.
- The stable external-template TypeScript boundary is `ResourcePageTemplateRenderRequest` to `ResourcePageTemplateRenderResult`: a request carries the resolved `ResourcePageDocumentModel` and template descriptor only; a result is either `pageHtml` or `fragments` for named slots.
- Slice 2 should define a deterministic first-pass merge rule before `defaults/application.ttl` names the default presentation profile: outer and inner templates are nearest-wins; stylesheet lists are additive with duplicate stylesheet IRIs keeping the nearer occurrence; panel selections are additive by selection IRI, with a nearer selection replacing a farther selection with the same IRI; if two different selections name the same panel for the same effective target/mode, the nearer selection wins unless a later explicit duplicate policy allows both; `panelOrder` ties break by config precedence and then selection IRI for deterministic output.
- Unknown selected panel identities should produce diagnostics during config/shape validation, and generation must not silently drop them. SHACL validation should catch invalid configured panels first; if an unknown panel identity reaches runtime, runtime should fail closed with a clear diagnostic.
- Custom identifier page opt-in to generated panels should be expressed through `ResourcePagePresentationConfig` panel selections or reusable panel-set references, not a boolean flag on `ResourcePageDefinition`. Authored `ResourcePageRegion` content should behave like authored-content panels during composition.
- The built-in Semantic Site stylesheet should appear as a real defaults asset in the slice that parses and honors the default presentation profile. Use a defaults IRI such as `<default-stylesheet>` for the stylesheet artifact, describe that artifact in `defaults/default-stylesheet.ttl`, and point its working located file at `defaults/stylesheet.css`; the CSS file itself is not a Turtle config file.
- Custom identifier pages opt into the shared Semantic Site shell by putting `sfcfg:hasResourcePagePresentationConfig` on the authored `ResourcePageDefinition`. In the first implementation, only the known built-in Semantic Site profile is supported; unsupported presentation config IRIs fail closed. Page definitions without this property keep the legacy custom-page rendering path.
- Runtime panel selection must honor page-kind targets, RDF-class targets, artifact-role targets, and data requirements before rendering a selected panel. `sfcfg:hasPanelTargetClass sflo:DigitalArtifact` may match either an explicit `sflo:DigitalArtifact` class or a known runtime artifact role, because the first runtime pass is not a general RDFS reasoner.
- SHACL should validate the structure of `ResourcePagePresentationConfig` and `ResourcePagePanelSelection`: required template links, stylesheet/panel-selection presence, exactly one selected panel, exactly one panel order and inclusion policy, known target value classes, and at least one data requirement.
- Custom identifier pages opt into generated/code-backed panels by putting `sfcfg:hasGeneratedResourcePagePanelSelection` on the authored `ResourcePageDefinition`, pointing at selection nodes from the active `ResourcePagePresentationConfig`. The first implementation supports explicit selection references only, not reusable panel sets. Authored regions render before selected generated panels.
- Add a concise user-facing `wu.resource-pages` note covering ResourcePage purposes, page types, composition, and customization.

## Contract Changes

- No immediate CLI or ontology behavior change is required just to create this task.
- The first implementation slice should make default generated pages use shared core panel/document model types with runtime-owned assembly and without changing generated HTML.
- Later implementation may make custom identifier pages share the same outer shell and presentation config resolution.
- Later implementation may refine config ontology terms for panel inclusion/order, built-in templates, stylesheet selection, and panel targeting.
- Core exposes a type-only external-template contract for resolved document/template requests and page-HTML/fragments results. The contract does not load templates, execute template code, or grant RDF/filesystem/network access.
- Built-in templates and stylesheets should be RDF-described defaults artifacts, not merely embedded renderer code with untracked identities.
- Likely ontology additions include `sfcfg:hasDefaultResourcePagePresentationConfig`, code-backed ResourcePage panel identities, panel selection/order/inclusion terms, direct RDF-class/artifact-role target terms, and data-availability target terms.
- Weave's default application config should begin naming a built-in ResourcePage presentation profile in `defaults/application.ttl` only once runtime parses and honors that profile.
- Generated HTML should remain stable until an implementation slice explicitly records expected presentation diffs.
- Introducing default presentation/profile RDF should happen together with validation/runtime support for those identities, including known-panel validation and deterministic merge behavior.

## Testing

- Preserve current generated HTML output while extracting reusable panel models unless a slice explicitly updates expected output.
- Add focused renderer/model tests for synthesized default `ResourcePageDocumentModel` values and panel ordering.
- Add config parser tests for application-default presentation selection once `/defaults/application.ttl` begins naming a default presentation profile.
- Add tests proving selected panels are target-gated, such as current working source panels appearing for DigitalArtifact pages but not unrelated identifier-only pages.
- Add tests for Markdown source rendering if a Markdown/authored-content panel lands in the first implementation wave.
- Add tests proving custom identifier pages can keep the default Semantic Site chrome when desired.
- Add tests proving templates receive structured panel inputs and do not need to parse RDF or local files.
- Run focused `src/runtime/weave/pages_test.ts`, runtime page-generation integration tests, and any fixture-stable generated-page comparisons after implementation slices.

## Non-Goals

- Do not combine this with the current move-only core weave refactor.
- Do not require every default ResourcePage to persist a `_knop/_page/page.ttl`.
- Do not describe synthesized default document-model inputs as `ResourcePageDefinition` artifacts in diagnostics.
- Do not promote `ResourcePageRenderInput` wholesale to core; it mixes renderer-adapter details with document data.
- Do not make templates responsible for RDF graph discovery, history resolution, source lookup, or local/remote access policy.
- Do not add a client-side app framework for ResourcePages.
- Do not replace page-source resolution work from [[wa.task.2026.2026-04-08_1545-resource-page-definition-and-sources]].
- Do not implement remote `targetAccessUrl` or `workingAccessUrl` fetching as part of presentation templating.
- Do not implement a general third-party panel/plugin system in the first slice.
- Do not allow arbitrary authored HTML execution or raw injection without an explicit safety model.
- Do not add ignored presentation-profile triples to `defaults/application.ttl` before runtime parses and honors them.
- Do not auto-append default generated panels to custom identifier pages without explicit composition policy.
- Do not add pagination, virtual scrolling, display limits, or large-mesh children-panel ergonomics in the first internal panel extraction. Preserve current output; panel display-limit policy belongs in a later presentation-policy slice.

## Implementation Plan

- [x] Inventory current default generated-page sections in `src/runtime/weave/pages.ts` and map them to first-slice panel model names: children, properties, blank nodes, references, histories, raw source, Semantic Flow metadata, ReferenceCatalog links, and Knop artifact lists.
- [x] Classify shell/header/chrome fields that should not become panels in the first slice: title, canonical IRI, breadcrumbs, RDF classes, summary, favicon, built-in CSS, stylesheet links, generated footer, and canonical slash-normalization script.
- [x] Define first core `ResourcePageDocumentModel` and `ResourcePagePanelModel` shapes that can represent the current default page inputs without exposing template execution, HTML adapter rows, CSS classes, or ontology parser details.
- [x] Keep runtime-owned assembly for `ResourcePageDocumentModel`, including filesystem reads, effective config resolution, mesh-state loading, and local/remote access policy checks.
- [x] Refactor renderer internals so default pages consume panel models without changing generated output.
- [x] Retire `ResourcePageRenderInput` if the new document model can feed panel rendering directly; otherwise keep it as a private renderer adapter only.
- [x] Add focused renderer/model tests for default document-model assembly, panel ordering, and zero-drift output.
- [x] Audit current config ontology terms for `ResourcePagePresentationConfig`, `OuterResourcePageTemplate`, `InnerResourcePageTemplate`, and `ResourcePageStylesheet`, and list missing panel-selection terms after the internal model exists.
- [x] Draft minimal ontology additions: `sfcfg:hasDefaultResourcePagePresentationConfig`, `sfcfg:ResourcePagePanel`, `sfcfg:ResourcePagePanelSelection`, panel selection/order/inclusion properties, direct RDF class/artifact-role target properties, and panel data-availability terms.
- [x] Define deterministic first-pass merge behavior for templates, stylesheets, panel selections, duplicate panels, and `panelOrder` ties before defaults begin selecting built-in panels.
- [x] Define validation/runtime behavior for unknown panel identities: SHACL/config diagnostics first, runtime fail-closed if an unknown selected panel reaches generation.
- [x] Add config parser support for the application-default ResourcePage presentation profile and only then update `defaults/application.ttl` to name the built-in Semantic Site profile.
- [x] Add the built-in Semantic Site stylesheet as a real defaults asset such as `defaults/stylesheet.css` in the same slice that parses and honors the default presentation profile.
- [x] Define first built-in panel targeting rules for page kind and data availability.
- [x] Add tests proving selected panels are target-gated by page kind.
- [x] Add richer RDF class/artifact-role target enforcement, including a rule that current working source panels apply only when the page represents a `DigitalArtifact` with current working source data.
- [x] Define authored-content panel behavior for safe Markdown and explicitly defer raw authored HTML rendering until a trust/sanitizer policy exists.
- [x] Define how custom identifier pages can explicitly use the same outer shell and stylesheet baseline as default pages.
- [x] Define custom identifier page opt-in through `ResourcePagePresentationConfig` panel selections or reusable panel-set references, without changing existing custom pages by default; when no explicit order is provided, authored regions should come before full generated panels.
- [ ] Sketch future low-impact panel presentation modes such as footer icons, menus, or popovers without implementing them in the first shared-pipeline slice.
- [x] Define the minimal external-template contract: templates receive resolved document and panel inputs, return page HTML or named fragments, and cannot read RDF/files/network/config directly.
- [x] Add or update tests for default/custom shared presentation behavior once custom pages enter the shared shell.
- [x] Decide whether to persist built-in templates/stylesheets as RDF-described artifacts or keep them as embedded defaults with RDF identities: persist them as RDF-described defaults artifacts once the model is stable.
