---
id: 22mrvr0xaeygma905chb1ex
title: 2026 08 21_new Releases
desc: ''
updated: 1787335247828
created: 1787335237391
---

You are working in /home/djradon/hub/semantic-flow/weave.

Goal: prepare, validate, and publish SFLO v0.4.0 first, then Weave v0.8.0. Complete the releases only after every gate below passes. This prompt authorizes the scoped commits, pushes, tags, GitHub Releases, npm publication, and SFLO Pages publication required for those two releases. Do not publish unrelated repositories or changes.

Never expose credentials, tokens, private environment values, or other secrets. Use existing authenticated tooling without printing authentication details.

Read before acting:

- AGENTS.md
- documentation/notes/product-vision.md
- documentation/notes/wd.general-guidance.md
- documentation/notes/wd.release-runbook.md
- dependencies/github.com/semantic-flow/sflo/notes/ont.dev.guidance.md
- dependencies/github.com/semantic-flow/sflo/notes/ont.dev.release-runbook.md
- dependencies/github.com/semantic-flow/weave-dev-archive/notes/ont.completed.2026.2026-08-14_1949-content-digest-contract.md
- dependencies/github.com/semantic-flow/weave-dev-archive/notes/wa.review.2026-08-21_1022-content-digest-contract-claude.md
- dependencies/github.com/semantic-flow/weave-dev-archive/notes/ont.completed.2026.2026-08-21_1048-cross-engine-shacl-conformance.md
- dependencies/github.com/semantic-flow/semantic-flow-framework/notes/sf.spec.2026-08-21-content-digest.md
- dependencies/github.com/semantic-flow/sflo/notes/ont.disc.2026-08-21_0923-response-to-stagecraft-requirements.md
- documentation/notes/release-notes.v0.7.0.md

Current known state—verify rather than trusting blindly:

- Weave HEAD includes:
  - c0daa57 fix(provenance): separate expected and observed digests
  - 2ec7606 fix(provenance): address content-digest review findings
- Weave still reports version 0.7.0 and needs v0.8.0 release preparation.
- SFLO includes:
  - 936bf8b6 initial digest contract
  - 9fd8fc54 downstream digest acceptance documentation
  - e809c739 executable/tightened SHACL follow-up
- Framework includes:
  - 144db3d initial digest behavior spec
  - 3ae7b0f lifecycle clarification
- The archive contains the task, review disposition, and cross-engine conformance plan.
- SFLO source metadata still declares v0.3.0 and needs v0.4.0 preparation.
- Stagecraft confirmed the digest design and asked that the changed Weave `integrate` behavior ship in a named release.
- Stagecraft currently pins `@semantic-flow/weave` 0.6.0.
- All digest implementation tests were green at handoff:
  - Weave `deno task ci`: 834 tests.
  - SFLO: 30 Deno guardrails, 11 PySHACL fixtures, release validation.
- Some commits may be ahead of origin. Verify every repository’s status, divergence, tags, and remotes.

Important dirty-worktree rule:

The Weave root may contain user edits in:

- documentation/notes/roadmap.md
- documentation/notes/wd.todo.md

Those edits belong to the user. Do not stash, reset, overwrite, reformat, stage, or include them in release commits without explicit approval. Prefer a clean temporary release worktree based on the current Weave HEAD if they remain dirty.

Create or refine release task notes before making release-specific changes. Treat this as a release-management/Jimbo session for task-note creation and closure, following the archive’s standing closure/rename/maintenance rules.

Phase 1: establish clean release candidates

1. Inspect status, branches, origin divergence, tags, and recent logs for:
   - Weave
   - SFLO
   - Semantic Flow Framework
   - weave-dev-archive
2. Verify that all intended digest commits are present.
3. Push already-approved source/spec/task commits using the repository’s established workflow.
4. Confirm remote CI for the pushed commits.
5. Preserve unrelated user changes.
6. If a clean release worktree is needed, create it from the exact intended HEAD; do not mutate the dirty working tree.

Phase 2: complete cross-engine SFLO conformance

Implement and complete:

dependencies/github.com/semantic-flow/weave-dev-archive/notes/ont.completed.2026.2026-08-21_1048-cross-engine-shacl-conformance.md

Requirements:

- Keep one SFLO-owned fixture corpus and expectation manifest.
- Run the same cases through:
  - PySHACL
  - `shacl-engine` over a minimal RDF/JS dataset
  - Stagecraft's exact `shacl-engine`/Oxigraph adapter
  - pinned Apache Jena SHACL
- Riot is syntax preflight only; it is not the Jena SHACL execution gate.
- Do not adopt Oxigraph in Weave or SFLO runtime code.
- Use the shipped ontology and shapes unchanged.
- Standardize graph assembly, inference policy, warning treatment, and network-off behavior.
- Compare normalized semantic receipts, not serialized report bytes.
- Record engine versions, commands, SFLO commit, fixture results, and any adjudicated differences in an `ont.report.*` note.
- Any unexpected conformance/severity disagreement blocks SFLO publication.
- If the private Stagecraft adapter cannot be called directly, produce the exact command/fixture handoff and obtain a returned receipt before continuing.

Phase 3: prepare and release SFLO v0.4.0

Inventory every change since v0.3.0, not only the digest work. Expected content includes at least:

- `extractedFrom`
- clarified DigitalArtifact/standalone LocatedFile identity wording
- `ContentDigestMethod`
- `contentDigestMethod_sha256`
- `contentDigestMethodToken`
- `ContentDigestBearer`
- ArtifactManifestation and LocatedFile bearer semantics
- exact lowercase SHA-256 grammar
- same-method uniqueness
- manifestation/file digest agreement
- explicit-bearer warning
- repository locators carrying none of the digest properties
- observed-digest property targeting
- runtime-only expected/observed lifecycle semantics
- executable PySHACL CI and cross-engine receipts

Then:

1. Create `ont.release-notes.v0.4.0`.
2. Name every breaking or changed semantic/validation behavior.
3. Use the release scripts and actual release date to update metadata.
4. Run the complete SFLO gate:
   - formatting
   - lint
   - type checks
   - Deno guardrails
   - PySHACL fixtures
   - cross-engine conformance
   - release validation
   - Turtle/Riot syntax checks
   - diff/whitespace checks
5. Commit the source release with a detailed semantic message.
6. Tag and push according to `ont.dev.release-runbook`.
7. Verify immutable raw tag URLs.
8. Publish/regenerate the SFLO Pages mesh using the established source/publication workflow.
9. Verify current and v0.4.0 Pages payloads, ResourcePages, and byte identity.
10. Record exact source/tag/Pages SHAs and verification receipts.

Phase 4: prepare Weave v0.8.0

Inventory all behavior since v0.7.0. Expected release content includes at least:

- untargeted extracted-term first-payload batching and its measured memory improvement
- condition-specific planner diagnostics
- `malformed-knop-metadata` finding code
- batch and sequential MeshInventory history-index correction
- bounded ResourcePage render→write→discard generation, command-shared extracted source contents, and the corrected candidate-retention estimator; name the faithful N=1,700 improvement (weave 3.79 GiB → 802 MiB, standalone generate 3.74 GiB → 674 MiB) and retire the v0.7.0 near-ceiling Known Limitation rather than carrying it forward
- digest/provenance behavior:
  - repository locators no longer receive `hasContentDigest`
  - computed integration digests remain observations
  - only caller-supplied digests become expectations
  - repository-backed observations record the concrete local path read
  - expected digest input is canonical lowercase SHA-256 only
  - active source-registry parsing rejects legacy locator digests with actionable migration guidance
  - extract hashes exact bytes before decoding
  - extraction timestamps are `xsd:dateTime`
  - valid LocatedFile digest claims remain accepted
- the ANSI-only test cleanup, which is not itself a release feature

Create `release-notes.v0.8.0` and explicitly describe the consumer-visible `integrate` RDF change. Include upgrade guidance for consumers moving directly from 0.6.0, since they also inherit all v0.7.0 behavior.

Use a minor release, not v0.7.1. The post-v0.7 capability and consumer-visible diagnostic changes already justify v0.8.0.

Phase 5: Stagecraft release-candidate smoke

Before publishing Weave:

1. Build/package the v0.8.0 candidate without publishing.
2. Use a temporary Stagecraft worktree or disposable dependency override.
3. Exercise the real press flow that shells out through `persistence.ts` to:
   - `weave mesh create`
   - `weave integrate`
4. Verify:
   - command success and exit codes
   - expected created/updated paths
   - repository locator contains no digest property
   - computed digest appears on `ArtifactResolutionObservation`
   - expectation appears only when Stagecraft supplies one
   - observed spec includes the concrete path
   - Stagecraft validation accepts the resulting RDF
5. Restore/discard the temporary pin without committing Stagecraft changes unless separately requested.
6. Record the exact Stagecraft commit, candidate artifact, command, and result as a release receipt.

A failed Stagecraft smoke blocks Weave publication.

Phase 6: publish Weave v0.8.0

After the SFLO release and candidate smoke pass:

1. Bump Weave/package versions to 0.8.0 using the repository scripts.
2. Finalize release notes and developer/user documentation.
3. Run `deno task fmt` and full `deno task ci`.
4. Run packaging and off-tree npm smoke tests required by the release runbook.
5. Confirm package contents and version output.
6. Commit the release preparation.
7. Push, tag, and publish through the established GitHub/npm release workflow.
8. Verify:
   - git tag
   - GitHub Release
   - `@semantic-flow/weave@0.8.0`
   - platform packages
   - `@semantic-flow/weave-lib@0.8.0`
   - checksums and downloadable archives
   - installed CLI `weave --version --json`
9. Confirm Stagecraft can pin the published 0.8.0 package.

Final documentation and closure:

- Update release notes, roadmap/todo only where authorized and appropriate.
- Update the SFLO response note with actual release versions/dates.
- Record release SHAs, tags, CI runs, package versions, Pages commits, and cross-engine receipts.
- Close/rename release task notes only under the planning-seat rules.
- Update affected wikilinks and the monthly maintenance note when closing tasks.
- Leave every repository clean.
- Do not push unrelated user edits.
- Do not stop after “release prepared”; continue through publication and post-publish verification unless a real gate fails or external authority is missing.

At any blocking failure:

- stop before the irreversible publication step;
- preserve evidence;
- explain the exact failed gate;
- fix only in scope;
- rerun the full affected gate;
- never weaken a test or conformance expectation merely to publish.

Final report must include:

- released versions and dates
- source commits and tags
- GitHub/npm/Pages URLs or identifiers
- CI and cross-engine results
- Stagecraft smoke receipt
- breaking/changed behavior summary
- any deferred work
- clean-worktree status for every repository

## Completion — 2026-08-21

- SFLO `v0.4.0` published at source/tag commit `e9c03c2b`; immutable raw-tag bytes verified for all five Turtle files.
- SFLO Pages published at `72d18379`; deployment run `32525587690` green; 371 Knops and 1,491 Turtle files; core/config/SHACL release payloads byte-identical to the source tag.
- Four-engine SHACL receipts agreed across 11 cases under PySHACL 0.40.0, public `shacl-engine` 1.1.2, Stagecraft's `shacl-engine` 1.1.2/Oxigraph 0.5.9 adapter, and Apache Jena SHACL 6.2.0. [[ont.report.2026-08-21-v0.4.0-shacl-conformance]] and [[ont.report.2026-08-21-v0.4.0-release]] carry the details.
- Weave `v0.8.0` published at source/tag commit `e33561d`; Release Manual rehearsal `32527933066` and publication `32528319429` green.
- `@semantic-flow/weave`, four platform packages, and `@semantic-flow/weave-lib` published at `0.8.0` with `latest`; all four archive checksums verified; installed CLI reported the exact release commit/build timestamp.
- Stagecraft candidate receipt `sha256:bc1670e885ebdf2cd084733ac90f5d6381ea930759724dcb41a4d4c91c485e02` passed the real `persistence.ts` press flow and exact repository-backed digest RDF checks at Stagecraft `b83fcf6e`. A fresh registry pin then passed the same real persistence test on published `0.8.0`.
- [[release-receipt.v0.8.0]] records package, workflow, GitHub, npm, checksum, installed-consumer, and Stagecraft receipts.
- Two release-blocking Weave defects found by SFLO Pages dogfooding landed with fail-on-old tests: late `sfcfg:` prefix placement (`8e29b3e`) and repeated support-subject preservation (`55b4f00`).
- No unrelated Stagecraft or user-authored Weave documentation changes were included by this release session.
