---
id: y0v7549n960uv2bovv6hzv6
title: 2026 05 20_2016 Resourcepages Show Latest State
desc: ''
updated: 1779333417586
created: 1779333417586
---

## Goals

- Current ResourcePages for governed `sflo:RdfDocument` artifacts should present the latest woven state bytes as the primary raw RDF panel when a latest historical state exists.
- Keep working-file information available as source metadata/context, but do not use mutable working bytes as the source for semantic panels once the artifact has a settled current/latest state.
- When the latest state has more than one manifestation, select a preferred manifestation deterministically, using the current working file extension as a hint for the author's preferred serialization.
- Preserve exact historical behavior: ResourcePages for an ArtifactHistory, HistoricalState, ArtifactManifestation, or LocatedFile should continue to show the bytes for that exact historical component.
- Verify the normal invariant that a never-woven payload/governed resource should not have a generated current ResourcePage. Do not add a broad "unwoven working file" display mode as part of this task.

## Summary

ResourcePages currently surface raw RDF for `RdfDocument` pages from the current `workingLocalRelativePath` in several current-artifact cases. That is useful while developing, but it is the wrong primary source for a published/current page after weave has created a latest HistoricalState: the page can show mutable, unversioned bytes that are not yet part of the mesh's settled state.

This task changes current RdfDocument ResourcePage semantic-panel source selection so that the current artifact page prefers the latest state manifestation. The working file can remain visible as a locator/property, while the historical/manifestation pages remain exact-state views.

## Discussion

The page model already has most of the pieces:

- `src/runtime/weave/weave.ts` builds raw source panels for generated pages.
- `addPayloadRawSourcePanels(...)` currently adds `Current working file` to the public designator page and `Historical manifestation file` to the manifestation page.
- Support-artifact raw panels are similarly assembled from working-file locators.
- `src/runtime/weave/pages.ts` uses those raw panels for RDF facts and display, and currently shows `Working File` in Semantic Flow metadata when `workingLocalRelativePath` is present.

The behavior we want is a sharper separation between mutable source and published state:

- The working file says where current authoring happens.
- The latest HistoricalState says what has been woven into the mesh.
- The current ResourcePage is a publication surface, so it should describe/display the latest woven state when one exists.

Manifestation selection needs a small rule because a state can have multiple manifestations. The current working file extension is a reasonable preference hint because it tells us the authoring serialization. For example, a `workingLocalRelativePath` ending in `.ttl` should prefer a Turtle manifestation of the latest state when one exists. That hint should not become authority over the graph: if there is no matching manifestation, choose a deterministic fallback rather than reading the working file.

Never-woven payloads should not need a display fallback. If a payload artifact has not been woven, it normally should not have a generated current ResourcePage yet. If the runtime finds a current ResourcePage for a historical-state-managed RdfDocument without a latest state, that is more likely an inventory/generation-policy inconsistency than a reason to show mutable working bytes in semantic panels. Current-only support artifacts may need role-specific handling if their configured policy intentionally has no historical state.

## Open Issues

- None blocking this implementation slice.
- Future SFLO support for explicit canonical manifestations is tracked in [[ont.todo]].
- Future Weave UX follow-ups for user-visible findings and optional working-file metadata suppression are tracked in [[wd.todo]].

## Decisions

- For historical-state-managed `sflo:RdfDocument` pages, latest woven state beats working file for primary raw RDF display.
- Cover all current artifact pages whose role/config causes ResourcePage generation, including supporting artifacts as well as payload public identifier pages.
- `workingLocalRelativePath` remains useful metadata and a manifestation-preference hint. It does not need to be hidden by default.
- Working-file extension matching is only a preference rule for choosing among latest-state manifestations.
- Current implementation should assume there is no reliable content-type metadata for manifestation choice; choose by extension, then deterministic fallback.
- If a historical-state-managed current RdfDocument ResourcePage has no latest state, skip the raw RDF-derived panels rather than deriving semantic panels from the working file. Current-only support artifacts can keep using their current files until their role/config gives them a settled state to prefer.
- Missing-latest-state warnings/findings should be added through the broader `generate`/`validate` findings surface rather than as a one-off warning path in this slice.
- Working-file bytes should stay out of current published ResourcePage semantic panels once a settled state exists. Working-file locator metadata can remain visible unless a later publication option suppresses it.
- Publication-mode hiding of working-file locator metadata is desirable eventually, but not part of this implementation slice.
- A `canonicalManifestation` or similar predicate is desirable eventually; until SFLO grows that shape, Weave should continue using extension preference plus deterministic fallback.
- No floating named branch or mutable source behavior belongs in this page-rendering rule.
- Do not add a generic never-woven fallback in this task. Treat unexpected current ResourcePages without latest state as an invariant to inspect.

## Contract Changes

- Current RdfDocument ResourcePages should render raw RDF from the latest HistoricalState preferred manifestation when available.
- Current published ResourcePages may continue to display a `Working File` metadata row/link for governed RdfDocuments. Page title, summary, RDF classes, child classification, properties, blank-node rows, and raw RDF display should be derived from the chosen settled source panel where possible.
- ArtifactManifestation and LocatedFile ResourcePages remain exact-resource views and should continue to display their own located bytes.
- Existing ResourcePageDefinition/custom page behavior should not change except where those pages consume the same raw-source model for Semantic Flow metadata.

## Testing

- Add a focused runtime/page generation test where a payload has a latest historical Turtle manifestation, then the working file is changed without weaving. Running `generate` should keep the current ResourcePage raw RDF and RDF-derived facts aligned with the historical manifestation, not the changed working file.
- Add selector coverage for manifestation preference: matching working extension wins; no matching extension falls back deterministically.
- Keep coverage proving the historical manifestation page still shows the manifestation bytes directly.
- Add or update support-artifact tests if the implementation slice covers KnopInventory, ReferenceCatalog, ResourcePageDefinition, MeshInventory, MeshMetadata, or MeshConfig current pages.
- Run focused `tests/integration/validate_version_generate_test.ts`, `tests/integration/weave_test.ts`, and `src/runtime/weave/pages_test.ts`; run broader `deno task test` if shared page-model behavior changes.

## Non-Goals

- Do not change version planning, history naming, or manifestation naming policy.
- Do not introduce a persistent page/source cache.
- Do not add a generic page mode for unversioned working files.
- Do not make renderer prose/layout assertions broader than needed; tests should target source selection and semantic RDF-derived facts.
- Do not change exact historical pages to point at current/latest state.

## Implementation Plan

- [x] Trace current raw-source panel assembly for payloads and support artifacts, especially `addPayloadRawSourcePanels`, `addSupportArtifactRawSourcePanels`, and mesh support panel collection.
- [x] Add a small resolver for the preferred latest-state RdfDocument source panel: latest state -> manifestations -> located file, with working-file extension as a preference hint and deterministic fallback.
- [x] Replace current-artifact raw source panel selection so current RdfDocument pages use the preferred latest-state panel when one exists.
- [x] Keep the working-file locator available as metadata and as the manifestation preference hint.
- [d] Implement no-latest-state behavior as a user-visible finding. Deferred to [[wd.todo]] so it can be designed with the broader `generate`/`validate` findings surface instead of adding a one-off warning path here.
- [x] Add tests for stale working file vs latest woven state on generated current ResourcePages.
- [x] Add selector coverage for multiple manifestation preference once the resolver shape is clear.
- [x] Run focused page-generation tests and broaden to the full suite if the page model or shared generation path changes.
