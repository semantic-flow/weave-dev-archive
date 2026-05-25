---
id: 551jgelsrwdpe00fr8xdu7z
title: 2026 03 24 Release Runbook
desc: ''
updated: 1779680284652
created: 1779073532998
---

## Goals

- Define an SFLO ontology release runbook that humans can actually follow.
- Record v0.1.0 release notes for the first tagged ontology release.
- Specify two GitHub Actions workflows: one to re-generate ResourcePages from the current mesh, and one to run SFLO release validation.
- Keep weave validation distinct from SFLO release validation.
- Avoid re-weaving or versioning ontology payloads as a side effect of ResourcePage regeneration.
- Keep the task open until detached publication source binding plus publication-branch update behavior is far enough along for automated publication to be boring.
- Move job and prov ontology publication under the existing `/sflo/` project Pages surface, alongside core, config, and core SHACL.

## Summary

The first durable output is [[ont.dev.release-runbook]], plus [[ont.release-notes.v0.1.0]] for the first tagged release. Those notes remain useful as a manual source-release guide, but the workflow shape should now be tuned for two ordinary maintenance actions rather than a heavyweight release ceremony. There are no external consumers yet, so GitHub Release publication can stay manual/optional while the branch-published Pages workflow becomes boring.

There should be two authored actions:

- Re-generate Resource Pages: optionally run `weave validate`, then regenerate ResourcePages from the current mesh state. This action does not run payload weaving, does not version payloads, and does not decide whether ontology source files have changed.
- Release: run SFLO release validation over source Turtle, release notes, version metadata, and publication expectations. This action does not generate pages by default, but can expose a checkbox/input to generate after release validation passes.

Because the first action re-renders existing mesh state instead of appending payload states, it is not blocked on append-onlyish inventory work. Repository-backed working-source integration for a detached publication mesh is implemented enough to use for SFLO `gh-pages` publication: `integrate --source-repository-current` can read from a separate source checkout, store a floating repository source locator, and avoid host-local path leakage. Automated payload publication should still get a dry-run/reporting step before it commits output, but the model no longer needs to wait for a special branch-publishing command. See [[wa.completed.2026.2026-05-18_0627-remove-prepare]] and [[wa.completed.2026.2026-05-17_2206-prepare-symmetry]].

## Discussion

### What Is In Place

- [[ont.dev.release-runbook]] documents source release checks, Turtle validation, version metadata checks, tagging, Pages publication, and post-publish checks.
- [[ont.release-notes.v0.1.0]] documents the first tagged ontology release and the current Pages publication boundary.
- The Weave CLI docs now distinguish in-place sidecar meshes from detached publication roots in [[wu.cli-reference]].
- Weave has separate commands for validation, generation, and versioning.
- `weave prepare gh-pages` has been removed rather than deprecated; detached publication roots should use composed mesh operations.
- The first sflo-owned Deno guardrail harness now runs RDF parseability, duplicate-triple, config namespace, retired-term, and selected SHACL source-binding checks, and [[ont.dev.release-runbook]] now includes `deno task ci` as a release preflight command.
- SFLO now has `deno task release:validate`, which checks active source Turtle release metadata, namespace policy, release notes, version URLs, and optional tag-at-HEAD consistency with `--require-tag`.
- SFLO now has `deno task release:set-version`, which rewrites deterministic release metadata in the active Turtle files for an explicit version and issue date without creating tags, commits, release notes, or Pages output.
- The SFLO source CI workflow now delegates to `deno task ci`, so GitHub CI runs the same format, lint, type-check, test, and release-validation preflight used locally.
- SFLO now has manual GitHub Actions workflows for ResourcePage regeneration and release validation. These are authored workflows with `workflow_dispatch`; the generated GitHub Pages `pages-build-deployment` workflow itself remains GitHub-owned and does not expose useful manual inputs.
- Weave now supports detached source integration well enough for SFLO publication replays: `weave integrate --source-repository-current` records `sflo:hasRepositorySourceFloatingLocator` with `sflo:sourceRepositoryUrl` and `sflo:sourceRepositoryPathFromRoot`, while intentionally omitting local checkout paths, refs, commits, and digests for the floating working-source case.
- The SFLO dogfooding sequence in Weave's [[wu.cli-reference.examples.sflo]] shows source-lane payload integrations from a source checkout into a detached publication checkout, followed by explicit history/state intent and targeted weaving.
- The job and prov ontologies should move under the existing `/sflo/` project Pages surface, using paths parallel to config rather than the earlier top-level `/ontology/job/` and `/ontology/prov/` experiment.

### Action: Re-generate Resource Pages

The ResourcePage action should operate on the current mesh and published/publication worktree as they already exist. It is for cases such as improved page templates, styling, renderer fixes, identifier-page metadata, or repaired generated output.

GitHub `workflow_dispatch` gives us `boolean`, `choice`, `number`, `environment`, and `string` input types. That is enough for a small, explicit manual surface; avoid inventing JSON blobs until a workflow actually needs structured batches.

The ResourcePage action should start with these inputs:

- `run_weave_validate`: `boolean`, default `true`.
- `publish_mode`: `choice`, default `artifact`, options `artifact`, `commit-to-gh-pages`. A later PR mode is nice, but not required for the first SFLO-only workflow.
- `publication_branch`: `string`, default `gh-pages`.
- `mesh_root`: `string`, default `.` for workflows checked out directly on `gh-pages`; use a separate checkout path only when the workflow checks out source and publication worktrees side by side.
- `ref_to_publish`: `string`, default the configured publication branch.

The operation should be conceptually:

- optionally run `weave validate`;
- run ResourcePage generation for the configured mesh/publication root;
- report whether generated files changed;
- optionally commit directly to `gh-pages` when `publish_mode` says so and the generated diff is intentional.

This action should not call top-level `weave`, should not call `weave version`, and should not infer "expected semantic change" from source-file dirtiness. If working ontology files do not match the latest woven payload states, that may be reported by a future informational check, but it should not block ordinary page regeneration by default.

### Action: Release

The release action is about SFLO source/release correctness, not page rendering by default. It should run SFLO release validation first, then optionally generate pages after validation passes if the caller checks an input such as `generate_after_validation`.

The Release action should start with these inputs:

- `version`: `string`, required for release-candidate and tag validation runs.
- `require_tag`: `boolean`, default `false`; set `true` only after the tag exists locally/remotely and should point at `HEAD`.
- `generate_after_validation`: `boolean`, default `false`.
- `publish_mode`: `choice`, default `artifact`, options `artifact`, `commit-to-gh-pages`.
- `publication_branch`: `string`, default `gh-pages`.
- `mesh_root`: `string`, default `.` for the publication checkout.
- `weave_version`: `string`, default `latest`.
- `create_github_release`: `boolean`, default `false`; useful later, but optional while there are no external consumers.

SFLO release validation should include source and release-policy checks such as:

- Turtle syntax/shape validation for active ontology source files.
- Consistent release metadata, including `owl:versionInfo`, `dcterms:hasVersion`, release `dcterms:issued`, `owl:versionIRI`, `schema:contentUrl`, and `dcat:downloadURL` where applicable.
- Release notes presence and consistency with the version being released.
- Expected source payloads are present and use the intended public IRIs.
- Job/prov ontology namespace and metadata use the `/sflo/` project Pages surface once the Turtle metadata is updated.
- Optional post-deploy fetch/validation of published Pages Turtle when a publish step exists; this is useful but not a blocker for the first workflows.

The release action should not generate pages by default. That keeps release validation usable as a fast, policy-focused check and avoids surprising publication diffs during tag or release-candidate workflows.

### Validation Split

There are two different validations here:

- Weave validation checks the mesh and generated/current Semantic Flow surfaces: RDF graph integrity, local-path leakage, source binding consistency, host-preset readiness, and publication-root invariants that have precise enough hooks to check.
- SFLO release validation checks the ontology release itself: Turtle validity, version metadata, release notes, namespace policy, expected source payloads, and tag/version consistency.

### Detached Source Integration Status

Detached source integration is done enough for the SFLO `gh-pages` path. The important supported shape is:

- source checkout and publication checkout may be separate local worktrees
- `integrate --source-repository-current` reads local bytes from the source checkout under approved local access policy
- public mesh facts record repository identity and repository-root path, not the developer's local path
- release payload states are created only by explicit `weave`, `weave version`, or targeted history/state commands
- generated pages can be committed to `gh-pages` after review

What is still future work is automation polish, not a missing semantic command family:

- manifest-driven bulk `integrate`
- a dry-run/reporting command that clearly reports no-op, update, and conflict cases before committing publication output
- first-class workflow support for opening a publication PR
- exact/latest-state source-policy follow-up if a future release wants to replay pinned source states rather than a checked-out working source tree

### Source-Vs-Latest-Woven Drift

Source-vs-latest-woven drift means the current source bytes named by a source binding differ from the latest materialized payload state in the publication mesh. For example, `main:semantic-flow-core-ontology.ttl` might have new terms or release metadata, while `gh-pages:ontology/releases/v0.1.0/ttl/semantic-flow-core-ontology.ttl` and the current `ontology/semantic-flow-core-ontology.ttl` payload still reflect the last woven state. ResourcePage regeneration alone should not repair that drift, because generation re-renders pages from the mesh state that already exists. The repair is an explicit payload publication step: set the intended history/state if needed, weave/version the ontology payload, then generate pages from the new mesh state.

Treat this as informational for now. It can become a warning when Weave can cheaply compare floating source bindings against the latest materialized payload bytes without making normal page regeneration noisy.

### What Is Not Settled

The release runbook still has to describe current commands rather than the ideal release command. `prepare gh-pages` is gone, so the manual runbook now documents validation and ResourcePage generation for an existing publication mesh, plus explicit mesh creation when a local publication worktree is new.

The durable detached-payload publication path is now shaped as explicit `integrate`, `weave set history`, `weave set next-state`, `weave version`, validation, and generation. The first implemented source policy is repository-backed working-source integration. Latest-state and exact source-policy support can wait until a workflow needs settled source-state resolution instead of checked-out release bytes.

An unconditional top-level `weave` is not safe as a release-action habit. Regenerating ResourcePages should be generation, not payload weaving. Creating a new payload state should be an explicit versioning operation, controlled by the payload-focused `weave set history`, `weave set next-state`, and `weave version` surface described in [[wa.completed.2026.2026-05-18_0627-remove-prepare]].

The GitHub Actions work should therefore be staged:

- source validation and metadata checks can be automated first;
- ResourcePage regeneration can be automated once the exact command path is stable;
- automatic payload publication commits should wait for release validation plus dry-run/reporting, but no longer need to wait for a special detached-source integration primitive;
- append-onlyish inventory is not a blocker for the ResourcePage regeneration action because that action should not append payload states.

## Open Issues

- How much dry-run/reporting belongs inside Weave versus inside the GitHub Actions wrapper?
- Should `publish_mode=pull-request` be implemented immediately, or is direct `gh-pages` commit enough for SFLO while there are no external consumers?
- Should the release action eventually create/update a GitHub Release body from `notes/ont.release-notes.v*.md`, or should GitHub Releases stay optional?
- Should post-deploy Pages fetch/validation become part of the release workflow once Pages deployment timing is predictable enough?
- Should source-vs-latest-woven drift be reported as an informational release check first, then promoted to a warning only after the comparison is proven quiet?

## Decisions

- Keep this task open; do not rename it to a completed note yet.
- Treat [[ont.dev.release-runbook]] as the current manual release guide, not the final CI contract.
- Model CI as two actions: Re-generate Resource Pages, and Release.
- The ResourcePage action regenerates pages from current mesh state and does not re-weave or version payloads.
- The ResourcePage action should offer a `weave validate` checkbox/input, defaulting to enabled unless the final workflow has a better reason to skip it.
- ResourcePage and Release workflows should use ordinary `workflow_dispatch` inputs. Use booleans for gates, choices for publish mode, and strings for version/ref/path values.
- The release action runs SFLO release validation and does not generate pages by default.
- The release action may offer a checkbox/input to generate pages after release validation passes.
- The release action may optionally create or update a GitHub Release from `notes/ont.release-notes.v*.md`, but that remains opt-in while there are no external consumers.
- Published job and prov ontology IRIs should move under the existing `/sflo/` Pages mesh, parallel to config.
- Keep weave validation and SFLO release validation separate in docs, workflow names, and failure messages.
- Do not add an unconditional top-level `weave` step to release automation.
- Do not use or document `weave prepare gh-pages`; it has been removed rather than deprecated.
- Treat detached payload publication as ordinary source-binding `integrate`, not as a special branch-publishing command.
- Treat current detached source integration as implemented enough for reviewed `gh-pages` commits.
- Use plural `releases` for release ArtifactHistory paths.
- Do not treat append-onlyish inventory as a blocker for ResourcePage regeneration; do require dry-run/reporting before unattended payload publication commits.
- Leave published Pages fetch validation for later.
- Treat source-vs-latest-woven drift as informational for now.

## Contract Changes

- Manual release validation should include `riot --validate` over all active Turtle files.
- Release metadata should align `owl:versionInfo`, `dcterms:hasVersion`, release `dcterms:issued`, `owl:versionIRI`, `schema:contentUrl`, and `dcat:downloadURL`.
- Job/prov release metadata should use `/sflo/job/` and `/sflo/prov/` or equivalent `/sflo/`-rooted paths rather than the earlier top-level `/ontology/job/` and `/ontology/prov/` surfaces.
- The ResourcePage regeneration workflow must call generation, not payload weaving/versioning.
- The release workflow must call SFLO release validation, not page generation, unless the caller explicitly enables post-validation generation.
- Publication automation should report no-op, update, and conflict cases before committing generated output.
- Repeated ResourcePage generation over unchanged renderer/config/current mesh state should be a no-op or identical-output operation.

## Testing

- For documentation-only work, run `git diff --check`.
- For the current Deno harness slice, run `deno task ci` in sflo.
- To prepare source Turtle metadata for a chosen release, run `deno task release:set-version -- --version <version> --issued <yyyy-mm-dd>`, then validate it.
- For source release validation only, run `deno task release:validate -- --version <version>`; after the tag exists locally, add `--require-tag`.
- [x] Before closing this task, add release-validation coverage for active Turtle files and release metadata.
- Before closing this task, add ResourcePage-action coverage that proves the action runs validation/generation without appending payload states.
- Before closing this task, add coverage for the `run_weave_validate` input.
- Before closing this task, add coverage for the release action's `generate_after_validation` input.
- Defer release-action coverage for post-publish fetch and Turtle validation of Pages release payloads until published Pages validation becomes part of the workflow.

## Non-Goals

- Do not solve all GitHub Pages publication topology questions in the runbook note itself.
- Do not move or rename this task note until the human explicitly asks.
- Do not add compatibility shims for pre-v1 generated mesh shapes just to make release automation easier.
- Do not make the SFLO release action push payload publication changes without dry-run/reporting.
- Do not make ResourcePage regeneration imply payload versioning.

## Implementation Plan

- [x] Add the durable manual release runbook in [[ont.dev.release-runbook]].
- [x] Add v0.1.0 release notes in [[ont.release-notes.v0.1.0]].
- [x] Document the current Pages publication boundary for v0.1.0.
- [x] Remove `prepare gh-pages` from Weave CLI docs and describe detached publication roots as composed mesh operations.
- [x] Decide the public topology for job/prov ontology IRIs: publish them under the existing `/sflo/` project Pages mesh, parallel to config.
- [x] Finish or unblock [[wa.completed.2026.2026-05-18_0627-remove-prepare]] and [[wa.completed.2026.2026-05-17_2206-prepare-symmetry]] enough that branch-published meshes use the same conceptual operations as sidecar meshes.
- [x] Update [[ont.dev.release-runbook]] to remove `prepare gh-pages`, document validation/generation for existing publication meshes, and call out the detached-source integration path.
- [x] Add an SFLO release-validation script or command covering source Turtle, release metadata, release notes, namespace policy, and tag/version consistency.
- [x] Add an SFLO release metadata setter for deterministic Turtle version, release IRI, tag URL, payload URL, and issued-date updates.
- [x] Add a GitHub Actions workflow for "Re-generate Resource Pages" with a `run_weave_validate` input.
- [x] Add a GitHub Actions workflow for "Release" with a `generate_after_validation` input.
- [ ] Add a publication dry-run/reporting step before unattended publication commits.
- [ ] Add a manually approved publication path that commits/pushes or opens a PR only when the dry-run is non-empty and expected.
- [ ] Update [[ont.dev.release-runbook]] after the first successful manual action run confirms the exact Pages publication path.
