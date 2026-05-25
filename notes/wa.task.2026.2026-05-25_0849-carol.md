---
id: txhpili1zs1uvetruuc9tz9
title: 2026 05 25_0849 Carol
desc: ''
updated: 1779724205333
created: 1779724205333
---

## Goals

- Append Alice Bio fixture rungs `26-carol` and `27-carol-woven` without rewriting the earlier `a.*` carried ladder.
- Materialize a richer Carol Burnett Turtle dataset as governed payload artifact `/carol/data`.
- Extract only the new `/carol` identifier from `/carol/data`, leaving existing `/alice`, `/bob`, and `/carol/data` surfaces alone.
- Import the commit-pinned Carol Burnett Markdown note into governed payload artifact `/carol/bio`.
- Weave `/carol/data` in `26-carol`, then weave the extracted identifier and imported bio in follow-up `27-carol-woven`.
- Record `https://djradon.github.io/ns/dave-richardson` as the creator of the Alice and Carol dataset artifacts.
- Use the rung as a practical stretch for nested payload paths, all-terms extraction scope, and future ResourcePage panel composition.

## Summary

`26-carol` should be a command-backed fixture rung. It first materializes fixture-authored `carol-data.ttl`, integrates it at `carol/data`, weaves that nested data artifact, runs `extract --all-terms --source carol/data --accept-preview`, and then imports the commit-pinned Carol Markdown note into `carol/bio`.

`27-carol-woven` should be the woven partner rung for the extraction result and imported content. It versions `carol/bio`, versions the extracted `carol` Knop support surface, generates Carol-facing ResourcePages, and proves the extraction source remains current-tracking against the already-woven `carol/data`.

The Carol dataset deliberately includes both fixture-local relationships and real-world biographical facts. Carol knows `/alice` and `/bob`; `/alice` gets `foaf:name "Alice Ghostley"`; `/bob` gets a Bob Newhart name; `/carol/data` names Dave Richardson as dataset creator; and `/carol` is filled out with common FOAF, Dublin Core, Schema.org, and OWL/Wikidata-style links based on Wikipedia/Britannica cross-checking.

## Discussion

This rung is doing a little more than "add another person," on purpose. It exercises the nested designator `carol/data` before `/carol` exists as a Knop, then uses extraction to mint `/carol` from that data. That should be legal: a nested payload artifact can be created by command replay, and Weave should create the needed directories and Knop support rather than requiring an intermediary `/carol` resource first.

The nested data artifact needs its first weave before extraction. Weaving extracted `/carol` expects the source payload to have current woven history; weaving `/carol/data` after extraction currently tries to refresh `/carol` before the Carol ResourcePage exists. So `26-carol` intentionally weaves `/carol/data` between integration and extraction, and `27-carol-woven` targets `/carol` plus `/carol/bio`.

The extraction scope is the interesting edge. The data graph mentions `/alice`, `/bob`, `/carol`, and `/carol/data`, but only `/carol` should be missing when extraction runs. That gives us a tight test of "extract missing mesh terms only" without dragging external Wikipedia, Wikidata, IMDb, or Britannica IRIs into the mesh.

The import step uses:

```sh
weave import \
  "https://raw.githubusercontent.com/djradon/public-notes/f46d85187ed7781917b73dd7779b756e2d2b7494/user.carol-burnett.md" \
  carol/bio \
  --working-file carol-bio.md \
  --expected-digest sha256:5634ffc14165c55ab43c2af38b9d6395e22c8385f54d4a94a7d22d83c99afee7
```

The working file is intentionally `carol-bio.md` at the mesh root for this first rung, matching the existing `alice-page-main.md` and `bob-page-main.md` fixture style. A later ResourcePage/panel rung can decide whether it wants to move Carol authored content under `carol/` or keep the current governed working-file convention.

## Open Issues

- Should the later Carol ResourcePage rung target `/carol/bio`, or should the content panel use a more page-specific `carol/page-main` artifact? Recommendation: keep `/carol/bio` for this data/content import rung, then add a page-source alias or new artifact only if the ResourcePage definition wants a different semantic role.
- Should the fixture eventually cover the three Carol Markdown versions and `--replace-working`? Recommendation: yes, but keep that as a separate import-versioning slice so this rung stays about nested data, extraction scope, and first Carol content import.
- Should imported Markdown image links become first-class imported assets? Recommendation: later. This rung should treat them as authored Markdown links and should not recursively fetch assets.

## Decisions

- Append `26-carol` after `25-root-page-customized-woven`; do not insert Carol earlier in the existing `a.*` ladder.
- Append `27-carol-woven` after `26-carol` as the woven partner rung.
- Use `carol/data` for the Turtle dataset and `carol/bio` for the imported Markdown payload.
- Use `foaf:name`, not `foaf:Name`, for Alice and Carol names.
- Use `https://djradon.github.io/ns/dave-richardson` as the `dcterms:creator` for the Alice and Carol dataset artifacts.
- Keep external real-world references as external IRIs or literals so `extract --all-terms` only sees `/carol` as the new mesh-scoped term.
- Allow fixture replay to fetch the commit-pinned Carol Markdown from raw GitHub with an expected digest.

## Contract Changes

- The Alice Bio scenario index gains `26-carol`, from `a.25-root-page-customized-woven` to `a.26-carol`, and `27-carol-woven`, from `a.26-carol` to `a.27-carol-woven`.
- Fixture command replay for Weave now grants scoped `--allow-net=raw.githubusercontent.com` so HTTP(S) import manifests can execute through the ladder runner.
- The `26-carol` manifest records a four-command replay sequence: integrate data, weave `carol/data`, extract all terms from that data, import Carol Markdown.
- The `27-carol-woven` manifest records a targeted weave over `carol` and `carol/bio`.
- Weave's first-payload mesh inventory renderer now tolerates a nested payload whose parent identifier has not been minted yet by anchoring generated payload pages under `_mesh/index.html` instead of requiring the missing parent page.
- First-payload weave shape checks now accept non-RDF `PayloadArtifact` + `DigitalArtifact` inputs, which lets imported Markdown payloads like `carol/bio` receive history and ResourcePages.

## Testing

- Validate `carol-data.ttl` parses as Turtle.
- Validate the checked-in Alice Bio scenario index matches the generated fixture topology.
- Run the focused fixture ladder tests after updating the scenario definition and index.
- Run `deno task lint` because the fixture runner changed.
- Dry-run replay of `26-carol` should show all four commands exiting 0 and `extract --all-terms` creating exactly `carol`. Accord comparison will still fail until the `a.26-carol` target branch exists.
- Once `a.26-carol` exists, replay `27-carol-woven` and verify Carol pages, histories, and current extraction-source resolution.
- Future SF/weave-pushing tests:
  - replay `26-carol` end to end once an `a.26-carol` branch is desired
  - assert extraction creates `/carol` but not `/carol/data`, `/alice`, `/bob`, or external Carol biography IRIs
  - add a later ResourcePage rung that renders imported Carol Markdown beside generated identifier panels
  - add a later `--replace-working` Carol import sequence over the same governed working file

## Non-Goals

- Do not regenerate or rewrite earlier Alice/Bob/root fixture rungs in this slice.
- Do not create the Carol custom ResourcePage definition yet.
- Do not recursively import images or other linked assets from the Carol Markdown.
- Do not solve Markdown interpretation beyond using the imported Markdown as realistic future panel input.

## Implementation Plan

- [x] Fill `26-carol/carol-data.ttl` with Carol Burnett data and local Alice/Bob relationships.
- [x] Add `26-carol.jsonld` with integrate, weave `carol/data`, extract, and import replay commands.
- [x] Append `26-carol` to the Alice Bio fixture scenario.
- [x] Regenerate the Alice Bio `scenario-index.jsonld`.
- [x] Update focused fixture-ladder tests for the added rung and asset materialization.
- [x] Dry-run replay `26-carol` far enough to prove the command sequence succeeds and extraction scopes to `carol`.
- [x] Add `27-carol-woven` as the woven partner manifest and scenario rung.
- [x] Update Alice and Carol dataset creator triples to `https://djradon.github.io/ns/dave-richardson`.
- [x] Fix nested first-payload weaving for parentless paths such as `carol/data`.
- [x] Fix first-payload weaving for non-RDF imported Markdown payloads such as `carol/bio`.
- [ ] Replay the rung and commit the resulting `a.26-carol` fixture branch when ready.
- [ ] Replay the woven rung and commit the resulting `a.27-carol-woven` fixture branch when ready.
