---
id: a4de6c6f-5731-4c81-9c74-09bb8bde2b92
title: SFLO v0.6.0 Release
desc: 'Publish the resolutionSpec and named repository-locator ontology, SHACL contract, and Pages surface'
created: 1790640492000
---

## Parent Plan

None. Dave directly requested this bounded SFLO release; the fixture-repository branch divergence is a read-only audit, not a release child with its own deliverable.

## Goals

- Publish the reviewed `resolutionSpec` vocabulary and LocatedFile validation contract from canonical SFLO `main`.
- Publish the preceding unreleased named `RepositorySourceFloatingLocator` clarification and SHACL constraint in the same source/Pages release.
- Produce immutable source-tag, cross-engine SHACL, Pages payload, and dereferenceability receipts.
- Explain the stale local `main` divergence in `mesh-sidecar-fantasy-rules` and `mesh-alice-bio` without overwriting recoverable local history.

## Summary

SFLO v0.5.0 is the current source and Pages release. Canonical `main` adds stable IRI identity for persisted repository floating locators and introduces `resolutionSpec`, which lets a `LocatedFile` designate at most one `ArtifactResolutionSpec` that targets that same file. These are public vocabulary and validation-contract changes, so the established pre-1.0 release convention calls for v0.6.0 rather than a patch-number increment.

The local fixture `main` branches are clean but stale: May fixture-ladder tips were assigned directly to local `main`, while canonical remotes were regenerated and merged in August from different histories. Their apparent ahead counts are old ladder commits, not unpushed current work. The local SFLO Pages worktree is similarly attached to the abandoned May history; publication will use a fresh worktree rooted at canonical `origin/gh-pages` rather than rewriting that recoverable local ref.

## Discussion

The release delta is not only the two `resolutionSpec` commits. Commit `a119eec6` changed persisted `RepositorySourceFloatingLocator` values from blank-node-or-IRI to named IRI and added positive/negative fixtures. Release notes and receipts must name both behaviors.

The source release remains one version line across all five active Turtle files. The `/sflo/` Pages publication covers core, config, and core SHACL; job and provenance remain tagged source only under the existing publication boundary.

## Resolved Questions

- The canonical Pages branch advanced successfully from a fresh worktree rooted at `origin/gh-pages`; the stale local Pages worktree was not rewritten.
- A private Stagecraft adapter receipt was not requested as an explicit gate. The release uses the public three-engine matrix required by the runbook and makes no private-consumer claim.

## Decisions

- Release version is 0.6.0, issued 2026-09-28.
- Treat “patch” as a small focused release, not a SemVer patch increment: new public vocabulary and tightened validation follow the repository's minor-release precedent.
- Preserve stale local fixture and Pages refs; do not hard-reset them during this task.
- Do not include unrelated ontology or fixture changes.

## Contract Changes

- Persisted `RepositorySourceFloatingLocator` values must be named IRIs so repeated registration can distinguish a semantic no-op from conflicting coordinates.
- Add `sflo:resolutionSpec` from `LocatedFile` to `ArtifactResolutionSpec`.
- A LocatedFile may designate at most one resolution spec, and a designated spec must target that same LocatedFile exactly once.
- Repository-backed resolution specs continue to use the existing target-artifact, repository-source, and target-local-relative-path contract and receive the existing mode warning when no standardized exact pin mode applies.

## Testing

- `deno task fmt`, `deno task ci`, and `deno task conformance:jena`, followed by normalized three-engine receipt comparison.
- Riot syntax validation for all five active Turtle files.
- `deno task release:validate -- --version 0.6.0` before tagging and `--require-tag` after tagging.
- `git diff --check` and source/tag byte identity checks.
- Pages mesh/publication validation, regeneration, Turtle syntax census, immutable payload byte identity, current/release-page links, and direct dereference checks for the new property and shape.

## Non-Goals

- Rewriting or deleting the stale local fixture branches.
- Regenerating the Alice or sidecar fixture ladders.
- Publishing job or provenance ontology Pages under `/sflo/`.
- Changing the already reviewed `resolutionSpec` model.

## Implementation Plan

- [x] Audit canonical SFLO/Pages state and the two reported fixture divergences.
- [x] Write v0.6.0 release notes and set deterministic release metadata.
- [x] Run source, SHACL, cross-engine, syntax, release, and whitespace gates.
- [x] Commit, push, tag, and verify immutable source bytes and canonical CI.
- [x] Publish v0.6.0 core/config/SHACL Pages states and generated vocabulary pages from canonical `gh-pages`.
- [x] Record release receipts and leave affected worktrees clean.

## Implementation Receipt

- Source release commit and peeled `v0.6.0` tag: `f18e1c07623d8cfb57a0c898b26aba65fc9553df`; annotated tag object: `fbc8239a4dfc72f57a512b3edf47caecffb816fd`.
- Canonical source CI run `36502184700` passed. Release validation with `--require-tag`, all five raw-tag byte comparisons, Riot parsing, and `git diff --check` passed.
- PySHACL 0.40.0, public `shacl-engine` 1.1.2, and Apache Jena SHACL 6.2.0 agreed on all 19 portable cases. The comparison tooling was made result-order-insensitive after the first run exposed serialization-order differences rather than graph disagreements.
- Pages commit `c7c45bbb2d4f3e9fb722e779eb8541a4262a1755` deployed successfully in run `36503094700`.
- The deterministic Pages build produced 376 ResourcePages, 1,517 Turtle files, and 3,430 publication files. Repeat generation reported 0 created and 0 updated; mesh and publication validation reported 0 findings; all Turtle parsed under Riot.
- Live core, config, and SHACL v0.6.0 payloads matched the tagged source byte-for-byte. The current artifact pages plus `resolutionSpec` and `LocatedFileResolutionSpecShape` pages returned 200 and referenced v0.6.0.
- Full receipt: [[ont.report.2026-09-28-v0.6.0-release]]. Release notes: [[ont.release-notes.v0.6.0]].

## Fixture Divergence Audit

- `mesh-sidecar-fantasy-rules` is clean at local `629a1364808a`, ahead 17 and behind 51 relative to canonical `ce91ac2c8918`. The reflog records `reset: moving to a.17-all-remaining-terms-woven` on 2026-05-19; canonical `main` was later regenerated and merged in August.
- `mesh-alice-bio` is clean at local `1f36c41d042e`, ahead 25 and behind 138 relative to canonical `0dec826a368d`. The reflog records `reset: moving to a.25-root-page-customized-woven` on 2026-05-19; canonical `main` was later regenerated through additional ladder steps and merged in August.
- The apparent ahead commits were obsolete local fixture-ladder histories, not current uncommitted work. They were initially left untouched, then discarded under Dave's explicit authorization on 2026-09-28 by resetting both local `main` branches to the fetched `origin/main` commits. Both repositories now report 0 ahead and 0 behind; no archive refs were retained.

## Status

Implementation is complete and pushed. This task is ready for Jimbo to perform the planning-seat closure rename and maintenance-log update.
