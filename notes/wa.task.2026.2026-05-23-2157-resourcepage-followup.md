---
id: v4k9uoaie8c3yn07hwji974
title: 2026 05 23 2157 Resourcepage Followup
desc: ''
updated: 1779598665624
created: 1779598665624
---

## Goals

- Capture the ResourcePage design backlog left after [[wa.task.2026.2026-05-22_2253-resourcepage-config-and-templating]] without pretending it is all one implementation slice.
- Preserve the ideas around reusable panel/config sets, low-impact panels, RDF-described default template/style artifacts, external templates, and richer fixture coverage.
- Identify useful next steps for fixture ladders that prove custom ResourcePage config and custom identifier pages can keep the Semantic Site look and feel while changing content or panel composition.
- Keep this follow-up distinct from [[wa.task.2026.2026-05-22_2308-fixture-helper-generalization]] and [[wa.task.2026.2026-05-21_0907-import]], while making the likely sequencing clear.

## Summary

The initial ResourcePage config/templating work landed the shared model: default pages and custom identifier pages now flow through a resolved ResourcePage document model, built-in presentation config, selected code-backed panels, target/data gating, custom-page shared shell opt-in, explicit generated panel opt-in, and a type-only external template contract.

This note is the holding area for the next wave. The next wave should not start by rewriting the renderer again. It should first produce better fixture coverage and examples for custom configuration, custom identifier pages, and maybe reusable panel sets. After that, we can decide which presentation capabilities are worth implementing rather than merely modeling.

## Discussion

### Current Baseline

The current baseline is intentionally conservative:

- The built-in Semantic Site presentation remains the default look and feel.
- Application defaults select known built-in presentation, stylesheet, and panel-selection identities.
- Runtime owns ResourcePage document/panel assembly, source resolution, config resolution, and access policy.
- Templates receive resolved document data rather than reading RDF, files, network resources, mesh inventory, or config directly.
- Custom identifier pages keep legacy behavior unless they explicitly name the shared ResourcePage presentation config.
- Custom identifier pages only receive generated panels when they explicitly opt into specific generated panel selections.
- Authored regions render before generated panels when both are present and no more specific ordering exists.
- Generated page output did not require fixture ladder regeneration in the first wave.

That is a good base. It is also not the end state. The first wave proved the seam; the next wave should exercise it with examples that matter.

### Deferred Work Sets

The backlog naturally falls into a few sets.

Presentation artifact set:

- Persist built-in templates and stylesheets as RDF-described defaults artifacts once the model is stable.
- Add `defaults/default-stylesheet.ttl` as the metadata artifact for the built-in stylesheet.
- Keep `defaults/stylesheet.css` as the working file for the actual stylesheet artifact.
- Decide how built-in outer/inner template metadata artifacts should be represented in `defaults/`.
- Move from embedded CSS toward linked stylesheet output only when we are ready for intentional generated HTML fixture diffs.

Panel/config set:

- Add reusable ResourcePage panel sets or selection bundles so a page or presentation config can opt into a named group such as "basic facts", "source and provenance", or "debug metadata" without repeating individual selection IRIs.
- Decide whether panel sets are first-class `sfcfg:ResourcePagePanelSet` resources, lightweight selection-list resources, or just named `ResourcePagePresentationConfig` fragments.
- Keep `ResourcePageDefinition` focused on authored regions and source bindings. If page definitions refer to panel sets, they should do so through explicit generated-panel composition properties, not by turning the definition itself into a presentation policy object.
- Add replace/remove controls for inherited stylesheet and panel selections once additive inheritance becomes too coarse.
- Tighten merge semantics beyond the first-pass rule: nearest-wins templates, additive stylesheets/panels, nearer duplicate stylesheet IRIs, nearer duplicate selection IRIs, and deterministic `panelOrder` ties.

Panel presentation mode set:

- Sketch low-impact generated panels that appear as footer icons, menu entries, badges, drawers, or popovers rather than full page sections.
- Decide whether low-impact mode is a property of the panel selection, the panel identity, the template/layout, or a later user preference.
- Keep first implementations accessible without client-side framework assumptions.
- Do not let low-impact panels become hidden data loss; generated output should still be inspectable and linkable.

Template set:

- Keep the external template contract as `ResourcePageTemplateRenderRequest` to `ResourcePageTemplateRenderResult`.
- Prefer declarative slot/layout selection before executable template loading.
- If executable templates arrive later, define loader boundaries, trust model, deterministic execution rules, and diagnostics before implementation.
- Decide whether templates can return only `pageHtml`, only named fragments, or a constrained combination based on outer/inner role.
- Expand slots only when a real template needs more than `head`, `shell`, `masthead`, `body`, `panels`, and `footer`.

Content and trust set:

- Continue treating authored Markdown as the safe custom content path.
- Defer raw authored HTML until there is an explicit trust and sanitizer model.
- Decide whether page-source content type dispatch belongs on `ResourcePageSource`, authored-content panel metadata, or a broader content-kind model.

Large-mesh ergonomics set:

- Decide display limits, pagination, virtual scrolling, or grouping controls for large children/reference/property panels.
- Keep this out of the presentation core until there is a real fixture or mesh with enough scale to justify it.

Diagnostics and validation set:

- Improve diagnostics for unsupported presentation config IRIs, unknown panel identities, skipped target-gated panels, and malformed generated-panel opt-in.
- Make config/SHACL validation messages useful enough that a mesh author can fix a custom page without reading runtime code.
- Consider a debug mode that explains the resolved ResourcePage document, selected presentation config, selected panel set, and target/data gating results.

### Fixture And Example Ideas

The next fixture work should get creative without exploding the ladder surface.

Useful fixture directions:

- Custom identifier page that opts into the shared Semantic Site shell and keeps the rest of the look and feel while adding one authored Markdown region.
- Custom identifier page that opts into exactly one generated panel, such as children or references, after authored regions.
- Custom identifier page that opts into a source/provenance panel and proves target/data gating keeps unrelated panels out.
- Mesh-level custom presentation config that changes panel selection/order but keeps the built-in templates and stylesheet.
- Knop-local or branch-local config that inherits application defaults and adds/removes a small panel selection.
- Reusable panel set fixture once the model exists, such as `<resource-page-panel-set/basic-identifier>` or `<resource-page-panel-set/source-audit>`.
- Branch-published mesh fixture where custom identifier pages and config are published without leaking local source checkout paths.
- Sidecar mesh fixture where custom page source Markdown lives outside the mesh root through an allowed, portable locator.

Recommended fixture ladder shape:

- Start with one narrow whole-mesh Alice Bio ladder step for custom ResourcePage config and generated-panel opt-in. This keeps the page output reviewable and avoids topology noise.
- Add one sidecar or branch-published variant only after the narrow case is stable.
- Use fixture-helper generalization before adding many more hand-rendered expectation helpers.
- Avoid regenerating the existing ladder just to prove documentation-only or zero-drift config changes.

### Relationship To Other Tasks

[[wa.task.2026.2026-05-22_2308-fixture-helper-generalization]] should probably come before broad new fixture ladder expansion. The current fixture helpers still carry Alice Bio assumptions, and adding several ResourcePage variants before generalizing them may make the old fixture machinery harder to unwind.

[[wa.task.2026.2026-05-21_0907-import]] is a later, larger workflow task. It becomes especially relevant when custom page Markdown or ResourcePage sources should be acquired into governed local working files instead of being directly integrated from an external checkout or URL. It should not block a first custom ResourcePage fixture, but it may be the right follow-up once we want imported Markdown/page-source examples.

[[wa.task.2026.2026-05-19_2349-branch-based-workingfile-fix]] remains relevant for branch-published ResourcePage fixtures. If a fixture reads custom page content or RDF sources from a source checkout while publishing to another branch, the durable source locator must not degrade back into host-local or sibling-worktree paths.

## Open Issues

- What should first-class reusable panel/config sets be called: `ResourcePagePanelSet`, `ResourcePagePanelSelectionSet`, `ResourcePagePresentationFragment`, or something else?
- Should panel sets be referenced from `ResourcePagePresentationConfig`, `ResourcePageDefinition`, or both through different properties?
- What exact replace/remove vocabulary should panel and stylesheet selection use once additive inherited defaults become insufficient?
- How should low-impact panel presentation modes be expressed, and which mode should be implemented first?
- When should the built-in stylesheet switch from embedded output to linked `defaults/stylesheet.css` output?
- How much fixture ladder coverage is enough before implementing external template loading?

## Decisions

- Do not add more fixture ladder rungs merely because the first ResourcePage config/templating wave landed; generated output was intentionally kept stable.
- Do not start with external executable templates. Build fixture coverage and RDF-described default artifacts first.
- Keep reusable panel/config sets explicit and named when they are introduced; avoid anonymous list shapes that are hard to debug or override.
- Prefer a narrow custom ResourcePage fixture before topology-heavy sidecar or branch-published variants.
- Treat fixture-helper generalization as a likely prerequisite for broad ResourcePage fixture expansion.

## Contract Changes

- Likely future ontology additions include a named reusable panel/config set resource and properties for including, replacing, or removing inherited selections.
- Later generated HTML may change when the built-in stylesheet becomes a linked stylesheet artifact rather than embedded CSS.
- Later custom ResourcePage fixtures may add or change expected generated pages, especially once panel sets or low-impact panel modes are implemented.
- External template loading, if added, should adapt to the existing core template request/result contract rather than replacing it.

## Testing

- Add focused tests for reusable panel/config set parsing and merge behavior when that vocabulary lands.
- Add ResourcePage generation tests proving a custom identifier page can keep the shared shell while adding authored Markdown content.
- Add tests proving explicit generated-panel opt-in preserves authored-region-first ordering.
- Add fixture ladder coverage only when generated output intentionally changes or a topology-specific ResourcePage workflow needs end-to-end proof.
- For branch-published fixtures, validate that generated RDF and HTML do not contain local checkout paths.
- For sidecar fixtures, validate that custom page sources resolve through allowed source policy without becoming public absolute paths.

## Non-Goals

- Do not implement the entire design backlog as one follow-up slice.
- Do not add arbitrary third-party panel renderers before built-in panel sets, merge controls, and diagnostics are solid.
- Do not add raw authored HTML injection without a trust and sanitizer model.
- Do not make low-impact panels depend on a client-side app framework.
- Do not use fixture ladder regeneration as a substitute for a small focused unit or renderer test.
- Do not fold `weave import` into ResourcePage presentation work; import is a later acquisition/localization command.

## Implementation Plan

- [ ] Choose the first follow-up slice: recommended first slice is a narrow custom ResourcePage fixture that keeps the shared look and feel while exercising authored Markdown plus explicit generated-panel opt-in.
- [ ] Decide whether fixture-helper generalization should land before that fixture or immediately after the first narrow fixture.
- [ ] Sketch the initial reusable panel/config set vocabulary and decide whether it belongs in `sfcfg` before implementation.
- [ ] Add a task-note example of a reusable panel set such as "basic identifier" or "source audit".
- [ ] Implement tests for whichever small slice is chosen before adding ladder-wide fixture expectations.
- [ ] If generated HTML changes, regenerate only the affected fixture ladder rungs and document the reason.
- [ ] Revisit [[wa.task.2026.2026-05-22_2308-fixture-helper-generalization]] before expanding custom ResourcePage fixture coverage.
- [ ] Revisit [[wa.task.2026.2026-05-21_0907-import]] when examples need explicit acquisition/localization of custom page Markdown or other page-source bytes.
