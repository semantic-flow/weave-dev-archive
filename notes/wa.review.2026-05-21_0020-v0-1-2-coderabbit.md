---
id: kmx2yrg3ii0ohnnwe5b0nxf
title: 2026 05 21_0020 V0 1 2 Coderabbit
desc: ''
updated: 1779348044795
created: 1779348042614
---

Verify each finding against current code. Fix only still-valid issues, skip the rest with a brief reason, keep changes minimal, and validate.

## Inline Comments

- [x] In `documentation/notes/wd.general-guidance.md`, fix the `live-server` command typo for the `mesh-branch-fantasy-rules` mount. The current command uses `--mount=/mesh-branch-fantasy-rules://home/...`; it should use normal live-server mount syntax, `--mount=/mesh-branch-fantasy-rules:/home/...`, matching the other `--mount=<name>:<path>` entries.

- [x] In `src/core/weave/weave_test.ts`, strengthen `planWeave accepts extracted terms from floating repository source payloads`. The current test mutates `input.currentMeshInventoryTurtle` but leaves `input.weaveableKnops[0].referenceTargetSourcePayloadArtifact` modeled as the original working-path source payload, so it mostly proves the mesh-inventory path-only fallback. After the extracted-source model can carry a `RepositorySourceFloatingLocator`, update this test or add a helper variant of `createExtractedBobWeaveInput()` so the source payload artifact and mesh inventory agree on the same floating locator before calling `planWeave`.

- [x] In `src/core/weave/weave.ts`, thread payload source metadata into first-payload created page models. `buildFirstPayloadWeavePages()` and `identifierPage()` currently preserve `workingLocalRelativePath` only, so freshly woven identifier pages can omit `workingAccessUrl` and `repositorySourceFloatingLocator` until a later generate pass reparses inventory. Extend the helper options to include `workingAccessUrl` and `repositorySourceFloatingLocator`, pass them from `payloadArtifact`, and update focused page-model assertions.

- [x] In `src/core/weave/weave.ts`, make extracted-source validation compare complete floating repository locators instead of only `sourceRepositoryPathFromRoot`. Salvaged scope: add `repositorySourceFloatingLocator?: RepositorySourceFloatingLocator` to `ReferenceTargetSourcePayloadArtifact`, populate it in runtime loading from the source payload inventory, and update `assertCurrentMeshInventoryShapeForFirstExtractedKnopWeave()` so working-path sources still use `hasCurrentWorkingFileLocator()` while floating sources require matching `sourceRepositoryUrl` and normalized path. Also wrap path normalization failures in `hasRepositorySourceFloatingLocatorPathFact` or its replacement so malformed RDF becomes a `WeaveInputError`.

- [x] In `src/runtime/operational/local_path_policy.ts`, tighten `collectRepositorySourceCandidateRoots()` without breaking host-local grants. The useful part of the review is about mesh-owned rules: if `rule.source === "mesh"`, only add the resolved rule root when it remains inside `policy.workspaceRoot`, matching the trust boundary used by `resolveAllowedLocalPath()`. Do not apply that workspace restriction to host-local rules, because detached source checkouts intentionally rely on local `~/.sf-local-access.ttl` grants outside the publication workspace.

## Outside Diff Comments

- [x] In `src/runtime/weave/weave.ts`, narrow the floating-locator raw-source fix to the remaining current/canonical page-source panel paths. `loadPayloadWorkingArtifact()` already resolves `payloadArtifact.repositorySourceFloatingLocator` for `currentPayloadTurtle`, so do not redo that part. The still-actionable paths are `addCanonicalSourceRawSourcePanels()` and `addExtractionSourceRawSourcePanels()` when they render a current or working source panel: if `sourcePayloadArtifact.repositorySourceFloatingLocator` is present, resolve with `resolveRepositorySourceFloatingLocalPath()` instead of `resolveAllowedLocalPath()`, preserving the existing `NotFound` and `LocalPathAccessError` behavior.

- [c] In `src/runtime/weave/weave.ts`, do not move the entire `executeWeave()` startup sequence into the existing try/catch just to force `weave.failed` logging. If logger resolution fails, there may be no logger available for failure logging; if timing creation failed, there is no timing instance to finish. Expanding the catch boundary here would mostly add complicated partial-initialization code around rare startup failures. Leave this out unless a concrete failing case appears.

## Nitpick Comments

- [x] In `documentation/notes/release-notes.v0.1.2.md`, populate the release notes before release. Keep this human-authored and factual: summarize the post-`v0.1.1` changes, call out floating repository source locators, ResourcePage source-state behavior, timing/caching, targeted generation fixes, validation, and compatibility notes. Do not invent PR or issue numbers if the release notes are not using them.

- [x] In `src/runtime/weave/pages.ts`, render the repository URL for floating source metadata. The current `Repository Source` row only shows `repositoryPathFromRoot`, which is ambiguous across repositories. Add a separate `Repository URL` row or include both URL and path in the existing metadata rows, using `page.repositorySourceFloatingLocator.repositoryUrl` and `repositoryPathFromRoot`.

## Validation

- [x] `deno fmt --check deno.json scripts src tests`
- [x] `deno lint scripts src tests`
- [x] `deno check scripts/**/*.ts src/**/*.ts tests/**/*.ts`
- [x] `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env src tests`
