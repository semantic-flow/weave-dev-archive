---
id: esauypshatgw8vzineusj3p
title: 2026 05 19_1536 Latest State Conformance
desc: ''
updated: 1779230229380
created: 1779230229380
---

## Goals

- Add a small conformance fixture that proves `artifactResolutionMode_latestState` resolves settled payload state rather than mutable working bytes.
- Keep this as a future fixture-ladder addition, not a reason to reopen the just-completed ladder regeneration.
- Exercise the already-implemented payload-backed `ResourcePageSource` behavior, not the broader future shared `ArtifactResolutionTarget` resolver.
- Avoid renumbering the current Alice Bio ladder unless a later regeneration pass is already doing broader sequence work.

## Summary

Latest-state page-source resolution is now implemented and covered by focused Weave integration tests, but it is externally visible Semantic Flow behavior and deserves one conformance-level example.

The fixture should demonstrate the difference between:

- `working`: a page source reads the target payload's current editable working file.
- `latestState`: a page source reads the target payload's latest settled `HistoricalState`, even when the working file contains newer draft bytes.

The smallest useful conformance story is in the Alice Bio ladder. Alice already has a governed Markdown payload at `alice/page-main`, and Alice's page definition already has an artifact-backed page-source rung. A new fixture can mutate the `alice/page-main` working file to a visible draft, switch Alice's main page source to `artifactResolutionMode_latestState`, weave Alice's page, and assert that the rendered page uses the settled `alice/page-main` state rather than the draft working file.

## Discussion

### Why Add A Fixture

This behavior is not just an implementation detail. It is part of the Semantic Flow API contract for `ResourcePageSource` resolution:

- users can publish pages from stable mesh history while local working files continue to change.
- a portable implementation should be able to demonstrate that page generation does not confuse mutable working bytes with settled history.
- the visible rendered page is a better conformance signal than only checking RDF triples.

### Why Not Add A New Ladder

A new ladder would be too much ceremony for one page-source behavior. Alice Bio already has the necessary setup:

- `16-alice-page-main-integrated` creates the governed `alice/page-main` payload.
- `17-alice-page-main-integrated-woven` gives `alice/page-main` a settled state.
- `18-alice-page-artifact-source` repoints Alice's page source at that governed artifact in working mode.
- `19-alice-page-artifact-source-woven` proves the working-mode behavior.

The latest-state fixture should reuse that context.

### Where To Add It

Prefer appending two transitions after the current Alice Bio ladder:

- `26-alice-page-latest-state-source`
- `27-alice-page-latest-state-source-woven`

Appending avoids renumbering the existing `20` through `25` Bob/root-page rungs. If a future regeneration pass is already renumbering Alice for a broader reason, it would also be reasonable to place the pair immediately after `19-alice-page-artifact-source-woven`, because that is the most local conceptual neighborhood.

### Fixture Shape

`26-alice-page-latest-state-source` should be a file transition from `25-root-page-customized-woven`:

- update `alice-page-main.md` with a visible draft string that is not present in the settled `alice/page-main` state, for example `DRAFT latest-state should not render`.
- update `alice/_knop/_page/page.ttl` so `#main-source` targets `<https://semantic-flow.github.io/mesh-alice-bio/alice/page-main>` with `sflo:hasArtifactResolutionMode sflo:artifactResolutionMode_latestState`.
- keep the sidebar source unchanged.
- do not update mesh inventory; the page definition already exists.
- expect public page output to remain unchanged until weave runs.

`27-alice-page-latest-state-source-woven` should run `weave --history-tracking-policy versioned --target designatorPath=alice` from `26-alice-page-latest-state-source`:

- page definition history advances because the page source mode changed.
- Alice Knop inventory history advances.
- `alice/index.html` is updated only as needed for the new page-definition state.
- rendered `alice/index.html` must include the settled `alice/page-main` content.
- rendered `alice/index.html` must not include the draft marker from the working `alice-page-main.md`.

## Open Issues

- Do Accord text expectations currently support "contains" and "does not contain" assertions for rendered HTML, or do we need to add that expectation type before this fixture is useful?
- Should the fixture also assert the observed settled state IRI used by the page-source resolver, or is rendered output plus source-mode RDF enough for conformance?
- Should this transition be appended as `26`/`27`, or inserted after `19` during a broader future renumbering pass?

## Decisions

- Add the fixture during the next fixture-ladder regeneration pass that is already happening.
- Do not reopen the just-completed ladder regeneration solely for latest-state conformance.
- Use Alice Bio rather than sidecar or branch fantasy rules because Alice already has page-source fixtures and a governed Markdown payload.
- Prefer append-only transition numbering unless a future pass is already renumbering.
- Keep latest-across-all-histories, requested-history latest-state, exact-state redundancy, and ambiguous no-history failure in unit/integration tests for now.

## Contract Changes

- No new ontology vocabulary is expected.
- No new CLI surface is expected.
- The conformance manifests should make existing page-source behavior observable through Accord cases.
- If Accord lacks rendered-text assertions, add a minimal expectation shape for text contains / does-not-contain before relying on the fixture for semantic proof.
- Keep [[sf.spec.2026-04-11-identifier-page-customization-and-root-lifecycle]] as the behavior spec source of truth for page-source latest-state semantics.

## Testing

- Add or update fixture-ladder script tests so the new Alice transitions are included in plan order and asset materialization.
- Add Accord manifest expectations for the non-woven transition:
  - `alice-page-main.md` updated.
  - `alice/_knop/_page/page.ttl` updated.
  - page source has `hasTargetArtifact <.../alice/page-main>`.
  - page source has `hasArtifactResolutionMode artifactResolutionMode_latestState`.
  - public `alice/index.html` unchanged before weave.
- Add Accord manifest expectations for the woven transition:
  - page definition state advances.
  - Alice Knop inventory state advances.
  - `alice/index.html` does not contain the draft marker.
  - `alice/index.html` contains content known to come from the settled `alice/page-main` state.
- Run focused fixture-ladder tests before any full ladder regeneration.
- Regenerate the Alice Bio ladder only as part of the broader fixture regeneration pass.

## Non-Goals

- Do not implement the shared `ArtifactResolutionTarget` resolver in this task.
- Do not add latest-state source-registry or extraction-source conformance here.
- Do not test remote repository refs or live network behavior.
- Do not copy source bytes into the mesh.
- Do not use this fixture to reopen the just-completed ladder regeneration.
- Do not add a separate latest-state fixture repository.

## Implementation Plan

- [ ] Confirm whether Accord can assert rendered text contains / does-not-contain against `alice/index.html`.
- [ ] Add fixture assets for `26-alice-page-latest-state-source`, including the draft `alice-page-main.md` and updated `alice/_knop/_page/page.ttl`.
- [ ] Add `26-alice-page-latest-state-source.jsonld` manifest.
- [ ] Add `27-alice-page-latest-state-source-woven.jsonld` manifest.
- [ ] Extend `ALICE_BIO_FIXTURE_SCENARIO` in `scripts/fixture-ladder.ts` with the two transitions.
- [ ] Update focused fixture-ladder tests for plan order, materialized assets, and replay expectations.
- [ ] Run focused fixture-ladder tests.
- [ ] Regenerate the Alice Bio ladder during the next broader fixture regeneration pass.
- [ ] Update [[wa.task.2026.2026-05-19_0022-lateststate-improvement]] when the conformance fixture lands.
