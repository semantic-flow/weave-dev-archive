---
id: z9nq1y4u5p7r3m8v2c6a0bxf
title: 2026 05 24_2058 Fixture Ladder Accord File Operations
desc: ''
updated: 1779668400000
created: 1779668400000
---

## Goals

- Make Weave's fixture ladder planner consume Accord replay metadata for manual file operations instead of hard-coding file-operation transitions in `scripts/fixture-ladder.ts`.
- Let manifests say where fixture bytes come from, including checked-in `.assets` paths, source refs, URLs, digests, and derivation notes.
- Keep branch ladders disposable by making the durable regeneration contract live in Accord manifests and scenario indexes.
- Preserve the `.assets`-on-seed-rung pattern, where `.assets/**` may be present in the fixture repo but ignored by product-output checks unless explicitly expected.
- Add enough branch-prefix control to generate a new `b.*` ladder without cloning or editing the scenario definition.

## Summary

The current Weave fixture ladder is only partly Accord-driven. Command-backed transitions are hydrated from Accord replay profiles, but manual file-operation transitions such as Alice Bio `14` and `24` are still declared in TypeScript. This means the durable manifest knows what should change, while the script privately knows where source bytes come from and how to copy them.

Accord already has much of the needed model from [[ac.task.2026.2026-05-14-generalized-replay-and-provenance]]: `ReplayProfile`, `hasFileOperation`, `FileOperation`, `targetPath`, `operationKind`, and `hasSourceProvenance`. Weave should consume that model before adding more fixture-specific ladder logic.

The `.assets` convention from [[ac.task.2026.2026-05-03-ignorePaths]] remains useful. For generated branch ladders, the seed rung can carry `.assets/**` as harness source material, while manifests use `ignorePaths` or runner guardrails so those files do not count as product output. A future `b.00` or source-seed rung should be generated from committed `main` assets rather than hand-copied through TypeScript declarations.

## Current Gap

- `scripts/fixture-ladder.ts` owns the Alice Bio file-operation source list for rungs like `14-alice-page-customized`, `18-alice-page-artifact-source`, `20-bob-page-imported-source`, and `24-root-page-customized`.
- The Accord manifests validate those outputs, but they do not currently drive Weave's file-copy plan.
- The planner has no CLI option for branch prefix override, so generating a new `b.*` ladder requires changing scenario data or duplicating the scenario.
- The scenario index is checked for topology, but the planner still primarily uses its local TypeScript scenario constants.

## Proposed Shape

Use Accord replay metadata for manual transitions:

```json
"hasReplayProfile": {
  "type": "ReplayProfile",
  "hasFileOperation": [
    {
      "type": "FileOperation",
      "operationKind": "copyFile",
      "targetPath": "alice/_knop/_page/page.ttl",
      "hasSourceProvenance": {
        "type": "SourceProvenance",
        "sourceKind": "fixtureAsset",
        "sourcePath": "14-alice-page-customized/alice/_knop/_page/page.ttl",
        "contentDigest": "sha256:..."
      }
    }
  ]
}
```

Weave's fixture ladder planner should derive file-operation sources from `hasReplayProfile.hasFileOperation` when present. Hand-declared TypeScript file-operation sources can remain only as a temporary fallback while the manifest corpus is migrated.

For source-seed rungs, use the same `FileOperation` shape to copy source files from `.assets`, including `.assets/**` itself when the scenario deliberately carries harness sources in the fixture repo. Manifests that compare whole trees should ignore `.assets/**` unless the case explicitly lists those files.

## Contract Changes

- Weave fixture manifests should add `hasReplayProfile.hasFileOperation` for manual file-operation rungs.
- Weave fixture ladder planning should accept a branch-prefix override, such as `--branch-prefix b.`, without changing scenario constants.
- Weave fixture ladder planning should eventually load scenario topology from the fixture-owned scenario index rather than duplicating all ordered steps in TypeScript.
- Accord may only need small follow-up validation or documentation if existing `FileOperation` terms are insufficient; the first implementation should assume the current Accord replay model is the source contract.

## Testing

- Add fixture-ladder planner tests proving a file-operation transition can be hydrated entirely from Accord `hasFileOperation`.
- Add execution tests proving copied bytes come from the declared `SourceProvenance.sourcePath`.
- Add digest verification for `contentDigest` when present.
- Add a branch-prefix override test for generating `b.*` plans.
- Add an Alice Bio manifest migration test where rung `14` no longer needs a hard-coded source list in `scripts/fixture-ladder.ts`.
- Add a seed-rung test proving `.assets/**` can be carried as harness source material while product-output validation ignores it.

## Non-Goals

- Do not make Accord execute fixture ladders in this slice.
- Do not turn Weave's fixture ladder script into a general workflow engine.
- Do not require every Accord manifest to include replay metadata.
- Do not make `.assets/` a mandatory Accord-reserved directory.
- Do not solve Bob/Carol page composition in the same pass; this task only removes private file-operation knowledge from the ladder machinery.

## Implementation Plan

- [x] Add `hasReplayProfile.hasFileOperation` entries to Alice Bio file-operation manifests.
- [x] Teach Weave's fixture ladder hydration to derive file-operation sources from Accord `FileOperation` nodes.
- [x] Verify `SourceProvenance.sourcePath` resolves under the configured asset root, or a declared source ref/path when that shape is used.
- [x] Verify `SourceProvenance.contentDigest` when present.
- [x] Add `--branch-prefix` to fixture ladder planning and execution.
- [x] Add a source-seed path for carrying committed `.assets/**` into the next generated ladder when the scenario asks for harness assets in the fixture repo.
- [x] Reduce or remove hard-coded file-operation source declarations from `scripts/fixture-ladder.ts`.
- [ ] Update [[wa.task.2026.2026-05-23_2230-custom-resourcepage-shared-shell-fixture]] once Alice `14` through `19` can be regenerated from manifest-declared file operations.

## Implementation Notes

- 2026-05-24: Weave fixture ladder hydration now uses `hasReplayProfile.hasFileOperation` when a file-operation transition manifest declares it. The TypeScript scenario source lists remain as fallback data for unmigrated manifests.
- 2026-05-24: Alice Bio file-operation manifests now declare `.assets` source paths for `01`, `14`, `18`, `20`, and `24`. Local fixture assets intentionally omit `contentDigest` so authored asset changes do not require manifest churn; digest verification remains available when a manifest needs pinned bytes.
- 2026-05-24: `fixture:ladder` accepts `--branch-prefix` for planning, materialization, and execution, while scenario-index write/check remains pinned to the canonical checked-in topology.
- 2026-05-24: `fixture:ladder --seed-source-ref main --branch-prefix b.` prepares the first `b.00-blank-slate` rung by filtering the source ref down to `.assets`, `.gitignore`, and `README.md`. Generated support files such as `.nojekyll` stay out of `00` so later rungs can create them in order. If a durable `assets` branch becomes useful later, use it as the seed source ref without changing the ladder flow.
