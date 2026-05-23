---
id: vl5eedqdfuxiiqr4g98sezk
title: 2026 05 22 Refactor Coderabbit
desc: ''
updated: 1779517924396
created: 1779517921639
---

Verify each finding against current code. Fix only still-valid issues, skip the
rest with a brief reason, keep changes minimal, and validate.

## Triage

- [ ] `src/core/weave/knop_inventory_renderers.ts`: preserve non-default ReferenceCatalog working file paths in subsequent page-definition KnopInventory rendering. Directionally right, but the current renderer only receives `hasReferenceCatalog`, not the original `ReferenceCatalogWorkingArtifact`; implementing this cleanly requires threading the current ReferenceCatalog working path into the page-definition planning/render boundary. Defer to a focused follow-up rather than guessing `references.ttl` replacements locally.
- [ ] `src/core/weave/mesh_inventory_renderers.ts`: remove or gate the Alice-specific legacy fallback fragments. Directionally right and belongs with [[wa.task.2026.2026-05-22_2308-fixture-helper-generalization]] because changing it now would alter legacy fixture-preservation output.
- [ ] `src/core/weave/mesh_inventory_renderers.ts`: decide whether current MeshInventory output should duplicate progression triples already preserved in MeshMetadata. A trial implementation changed settled fixture output, so this should be a semantic/output decision rather than a review-fix patch.
- [x] `src/core/weave/payload_renderers.ts`: keep all existing KnopInventory artifact histories linked when using the multi-history renderer, instead of linking only the selected current history.
- [x] `src/core/weave/payload_renderers.ts`: route single named payload histories through the generalized multi-history renderer instead of the ordinal single-history fallback.
- [x] `src/core/weave/reference_catalog_links.ts`: restrict current-link subject discovery to true `sflo:ReferenceLink` subjects or subjects referenced by `sflo:hasReferenceLink`, so unrelated catalog fragments are ignored.
- [c] `src/core/weave/resource_page_history_groups.ts`: cancel the suggested recency sort as written. `ResourcePageHistoryGroupModel` has no `timestamp`, `updatedAt`, or `recency` field, and the collector already sorts groups with current-history/history-ordinal/latest-state heuristics before merge. Changing merged order without a real recency field would be speculative.
- [x] `src/core/weave/turtle_blocks.ts`: render type-only subject blocks without an empty predicate semicolon.
- [x] `src/runtime/weave/artifact_loaders.ts`: resolve historical snapshot paths through `resolveAllowedLocalPath` before reading, instead of raw `join(workspaceRoot, snapshotPath)`.
- [x] `src/runtime/weave/page_generation.ts`: reject invalid `WEAVE_GENERATED_AT` values with a clear `WeaveInputError` rather than passing an `Invalid Date` to page rendering.
- [x] `src/core/weave/source_locator_renderers.ts`: normalize the repository floating-locator blank-node template by lifting JSON stringification out of the interpolation.
- [c] `src/core/weave/working_file_paths.ts`: cancel the safe-predicate suggestion. Returning `false` for invalid paths would make renderers emit `sflo:workingLocalRelativePath` for malformed input and weaken the current fail-closed behavior.

## Validation

- `deno task fmt`
- `deno task lint`
- `deno task check`
- `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env src/core/weave/turtle_blocks_test.ts src/core/weave/reference_catalog_links_test.ts src/core/weave/weave_test.ts tests/integration/validate_version_generate_test.ts tests/integration/weave_test.ts`
