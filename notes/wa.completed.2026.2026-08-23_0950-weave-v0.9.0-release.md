---
id: 5dd241ac-014d-46de-9d74-5589ef298b72
title: Weave v0.9.0 Release
desc: 'Publish additive Knop creation and FoundingReferentData across CLI, native artifacts, and weave-lib'
created: 1787503800000
---

## Parent Plan

[[wa.completed-plan.2026.2026-08-23_0950-founding-capability-releases]]

## Status

Completed 2026-08-23. Weave/`weave-lib` v0.9.0 were rehearsed, published from the identical commit, and independently verified.

## Goals

- Publish Weave and `@semantic-flow/weave-lib` v0.9.0 from the reviewed Stagecraft initialization implementation.
- Prove the new library API exists and behaves correctly in the packed off-tree Node artifact.
- Rehearse every native/npm/GitHub artifact before irreversible publication and verify the identical release commit afterward.

## Summary

The release adds a new consumer-visible capability and is therefore a minor release. It includes the measured additive `knop.create` path, optional exact-byte FoundingReferentData initialization, initial settlement and later correction through the ruled `weave version` arm, equivalent `versionFoundingReferentData` API, validation/preservation behavior, executable Accord transitions, and final fail-closed/API/rollback/path-policy hardening.

`@semantic-flow/weave-lib` is generated from `src/api/mod.ts`, which exports the new function and types, but the current packed-library smoke does not exercise them. Release preparation must extend the off-tree Node/source parity smoke through at least initial founding settlement and correction, compare results and workspace bytes, and verify the exact named exports from the packed artifact.

## Discussion

The release candidate depends on published SFLO v0.5.0 for the vocabulary it emits. Framework commits `e6d8bdd`/`f391813` carry the portable contract and Accord sequence and must be on canonical `main` before publication.

Release notes must be honest about the N=552 observations: current no-founding 2.59 s / 230,216 KiB peak versus founding plus first settlement 4.62 s / 246,144 KiB peak on the recorded host. These single observations select the accepted singular release path but are not a stable performance guarantee.

## Open Issues

- Confirm whether the current Stagecraft checkout has already wired the new API. Do not claim an end-to-end Stagecraft founding press if only the direct Weave/packed-library path exists.

## Decisions

- Version is 0.9.0; CLI and library remain one version line.
- G3 selected singular accepted; no batch path enters this release.
- The packed Node smoke is a publication gate, not optional coverage.
- No release workflow publication occurs until SFLO v0.5.0 source and Pages are verified.
- Any change after the all-platform rehearsal requires a new rehearsal.

## Contract Changes

- `weave knop create <D> --founding-data <path>` admission-copies validated exact bytes and creates the optional working artifact atomically.
- `weave version <D> --artifact-role founding-referent-data [--source <path>]` settles or corrects founding data without pages.
- `versionFoundingReferentData({ meshRoot, designatorPath, bytes? })` exposes the equivalent programmatic surface; `versionPayloads` remains payload-only.
- Publication validation reports unsettled working founding data and fails closed on missing/malformed registered KnopInventory; immutable snapshot digest mismatches remain errors.
- Atomic rollback preserves pre-existing directories and the public barrel withholds failure-injection seams.

## Testing

- Focused source/API/CLI/validation/rollback/library smoke tests, `deno task fmt`, and `deno task ci`.
- `deno task build:npm-lib` and `deno task smoke:npm-lib` with packed Node/source founding lifecycle parity and exact named-export assertions.
- Local native build/package, wrapper/platform npm dry-runs, installed CLI smoke, library npm dry-run, package-content/version inspection, and whitespace checks.
- Canonical CI at the release commit.
- Release Manual dry-run/draft rehearsal across four platforms and both npm surfaces.
- Post-publish tag, GitHub Release, npm dist-tag/package, archive checksum, installed CLI JSON, installed library, direct founding lifecycle, and available Stagecraft consumer checks.

## Non-Goals

- Batch Knop creation.
- Founding-data ResourcePages, remote fetch, root founding data, or standalone update command.
- Unrelated backlog/refactoring work beyond a release-blocking defect with a fail-on-old test.
- Publishing before SFLO v0.5.0.

## Implementation Plan

- [x] Push reviewed source/docs commits and require canonical CI green.
- [x] Extend the packed npm-library smoke and README for `versionFoundingReferentData` parity.
- [x] Inventory the v0.8.0..candidate delta and write `release-notes.v0.9.0` plus the release receipt skeleton.
- [x] Bump the shared version with `deno task bump:version -- --version 0.9.0` and inspect all generated metadata.
- [x] Run full source, packaging, npm dry-run, off-tree consumer, and whitespace gates.
- [x] Commit/push the release candidate and require canonical CI green.
- [x] Run and inspect the all-platform dry-run/draft Release Manual rehearsal.
- [x] Publish from the identical commit and verify all public artifacts and consumers.
- [x] Record receipts and close the task.

## Implementation Receipt

- Library parity: `398c6f8` added the generated-package documentation and off-tree Node/source settlement/correction byte-parity gate.
- Release commit/tag: `727c4b22c5307e9a0715de6e201a970b0b548e6c` / `v0.9.0`.
- Source gates: 887/887 local CI; canonical CI `32654946339`; CodeQL `32654946260`.
- Rehearsal: Release Manual `32655150477`, success with four native builds/install smokes, six npm dry-runs, enhanced library founding smoke, and inspected draft/eight assets.
- Publication: Release Manual `32655411689`, success from the identical commit; published `2026-08-23T17:41:56Z`.
- npm: wrapper, four platforms, and library all resolve `latest=0.9.0`.
- Installed CLI: `{"version":"0.9.0","commit":"727c4b22c5307e9a0715de6e201a970b0b548e6c","built":"2026-08-23T17:37:14Z"}`.
- Published library: exact four-symbol export set; state 1 and state 2 founding lifecycle/digests passed with state-1 preservation and zero founding page.
- Full package, archive, checksum, dependency, and consumer-boundary evidence is in [[release-receipt.v0.9.0]].
