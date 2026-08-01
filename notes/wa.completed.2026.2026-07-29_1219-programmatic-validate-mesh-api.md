---
id: watask20260729validatemeshapi
title: 'Programmatic validateMesh API + weave/weave-lib consumption-model ruling'
desc: 'Deliver the API surface Stagecraft needs: a stable validateMesh export in src/api and @semantic-flow/weave-lib returning structured findings, and settle/document the weave vs weave-lib consumption model raised in the 2026-07-29 consumer follow-up (§1). Candidate v0.6.0 headline; cut 2026-07-29 12:19 by Jimbo from wd.consumer-feedback.0.5.1.2026-07-29_1213. Codex r1 corrections folded 2026-07-29.'
---

## Goals

- Deliver programmatic mesh validation as a stable API: `validateMesh` implemented in `src/api`, re-exported from `src/mod.ts`, and shipped in `@semantic-flow/weave-lib`, returning structured findings (severity, stable code, message, path/target attribution) instead of formatted CLI text. This is the adoption-deciding ask deferred from v0.5.1 ([[wd.consumer-feedback.0.5.1]] §8) and boarded in `documentation/notes/wd.todo.md` as the v0.6.0 candidate headline.
- Rule and document the `@semantic-flow/weave` vs `@semantic-flow/weave-lib` consumption model so the 2026-07-29 §1 questions ([[wd.consumer-feedback.0.5.1.2026-07-29_1213]]) get one durable documented answer, not a reply-thread answer that evaporates.
- Match `versionPayloads` rigor: exact request/result shapes, stable machine discriminants, additive-evolution rules, and a green `src/api/fs_purity_test.ts` over the enlarged API graph.

## Summary

Stagecraft consumes Weave as a CLI-only validation consumer (`weave validate mesh --mesh-root …` from a package script) moving from `0.3.0` to `0.5.1`, and their 2026-07-29 follow-up asks what the intended consumption model is before they commit. The obliging answer has two parts: settle the packaging questions now (cheap — most of it already shipped as documented fact in v0.5.1), and deliver the programmatic validation surface as the v0.6.0 headline so their CI gate can consume structured findings rather than exit codes and formatted text. This note owns both. The bounded-memory engine work that whole-mesh validation needs at their scale is deliberately NOT here — it is the sibling note [[wa.completed.2026.2026-07-29_1220-whole-mesh-validate-bounded-memory]]; neither slice blocks the other, but the scale acceptance bench there must run through both the CLI and `validateMesh`.

This note was adversarially reviewed by Codex on 2026-07-29 (r1 below); its corrections are folded into the body, and its three blockers are now named deliverables of the spec review this note schedules as its first implementation item.

## Consumption-model ruling (proposed; ratify at spec review, then it becomes documented contract)

Their three §1 questions, answered:

1. **Alongside, not replacement.** `weave-lib` is the programmatic surface for in-process consumers; the CLI remains a first-class supported consumption path indefinitely. CLI consumers (including Stagecraft today) should stay on `@semantic-flow/weave`.
2. **The CLI will not become a consumer-visible wrapper over `weave-lib`.** Both are thin surfaces over the same source tree, built from the same commit. Whether CLI routing internally shares API entry points is an implementation detail, never a consumption contract; no `weave-lib` runtime dependency will appear in the CLI package.
3. **Lockstep, yes — and already documented.** Both generated package READMEs state that the packages share one version line, built from the same commit and released together (`scripts/release/npm.ts:210` for the CLI wrapper, `scripts/build-npm-lib.ts:137-138` for the lib). `weave-lib` "starting at 0.5.0" is not a versioning split: `weave-lib@0.5.0` is an orphaned lib-only pre-release artifact of the one-time first publish (deprecation pending — [[wd.consumer-feedback.0.5.1]] Open Issues); `0.5.1` is the first lockstep release of both.

Documentation deliverable: a short "which package do I consume?" section in [[wu.api-reference]] carrying rulings 1–3, so the Stagecraft reply can quote a durable location.

## §2 disposition (reply input; no new task — Codex F12)

The consumer's §2 (`sflo:hasResourcePage` on extracted-term Knops) is already settled by landed contract and boarded work; it belongs in the reply, not in either new task note:

- Extraction is deliberately non-publication-bearing: it emits no page claims; the following weave owns page materialization (`sf.spec.2026-04-05-extract-behavior`, reaffirmed by the r1/r2 adjudication in [[wa.task.2026.2026-07-21_1603-extractor-defect-pair]], archive `origin/main`).
- No, extraction did not gain those claims in `0.4.0`/`0.5.1`, and it will not — downstream synthesis is not the intended long-term pattern either. The intended pattern is extract→weave→generate, which was not viable for their ~1,700-term mesh (nested-source root-Knop planner defect + scale cost, per the extractor R2 diagnosis).
- Their workaround remains necessary until `wd.todo` item "Make extracted-term weave viable for thousand-term nested-source meshes" (TODO 23) proves the supported sequence viable. No extraction behavior change rides this note or its sibling.

## Claim verification (measured 2026-07-29, this repo at `c962ab2`)

- Registry facts as they cite them (`bin: { weave: bin/weave.js }`, `main: ./script/api/mod.js`, version histories) are consistent with our packaging scripts; their registry observations are consumer-reported, not independently re-queried.
- **Validate today is CLI-only as a stable surface.** `executeValidate` (`src/runtime/weave/weave.ts:117-215`) is exported from the runtime façade — technically reachable from a pinned checkout via `./src/mod.ts` — but it is not part of `src/api`, not in `weave-lib` (dnt builds from `src/api/mod.ts` only: `scripts/build-npm-lib.ts:66`), and returns loosely-typed findings `{ severity: "error"; message: string }` (`src/runtime/weave/weave.ts:82-92`) with no stable codes and no per-file/per-target attribution. Today every caught `WeaveInputError`/`WeaveRuntimeError` collapses to one message-only finding while other errors escape (`src/runtime/weave/weave.ts:196-211`, Codex F3). "Structured findings" therefore needs a designed contract, not an export of what exists.
- `validateVersionPlanRdf` is already imported by `src/api/version_payloads.ts` — the RDF-parse check is already inside the fs-pure API closure. The additional graph `validateMesh` pulls in (candidate loading, mesh state, publication presets) must keep `src/api/fs_purity_test.ts` green.
- **Validation coverage is narrower than the word "validate" suggests (Codex F4).** Mesh-scope validation today is a dry run of the recursive version planner plus publication-readiness checks: it parses the *planned* RDF outputs (`src/runtime/weave/version_execution.ts:1107-1111`), not a traversal-parse of every existing mesh file; a settled mesh takes the support-page branch (`version_execution.ts:440-459`) and reports no versioned designators. The portable spec frames comprehensive mesh integrity as something validation "should grow over time" into. The v1 API must state exactly this coverage rather than imply whole-mesh integrity checking.
- **Scale caveat, verified:** untargeted `validate mesh` accumulates planner state on pending-heavy meshes (mechanism and citations in the sibling note). The v1 API contract must not promise whole-mesh viability at 10⁴-file scale ahead of that fix; it must simply avoid any shape that would preclude it (findings are compact result data; no retained plan artifacts in the result).
- **Source-capability divergence risk between CLI and lib (Codex F1).** Pending repository-backed candidates reach the lazy git resolver from `loadPayloadWorkingArtifact` (`src/runtime/weave/artifact_loaders.ts:76-94` → `src/runtime/operational/repository_source.ts` → `repository_source_git.ts`); under Deno that can spawn `git`, while the dnt/Node build feature-detects `Deno.Command` and degrades to no-match. The fs-purity guard deliberately treats that statically-visible dynamic import as the sanctioned boundary, so it stays green while `validateMesh`-under-Node could silently diverge from CLI behavior on such meshes. An unconditional "one findings pipeline / CLI parity" promise is therefore not honest without a capability ruling (below).

## Contract sketch (PROPOSED — a starting shape for spec review, not a ratified contract)

```ts
export interface ValidateMeshRequest {
  meshRoot: string;                     // absolute host path; no cwd default (versionPayloads rule)
  targets?: readonly ValidateTarget[];  // absent or empty = whole mesh
  scope?: "mesh" | "publication";       // default "mesh"; "publication" refuses targets (CLI parity)
}

export interface ValidateTarget {
  designatorPath: string;               // "/" is the public root alias, as in versionPayloads
  recursive?: boolean;
}

export interface ValidateMeshResult {
  meshBase: string;
  scope: "mesh" | "publication";
  findings: readonly MeshValidationFinding[];
  // coverage-count field(s): exact name and semantics are a spec-review deliverable (Codex F4) —
  // "validated N designator paths" must say which N: known, selected, or pending/planned.
}

export interface MeshValidationFinding {
  severity: "error" | "warning";
  code: MeshValidationFindingCode;      // stable string union, versionPayloads-style discriminant
  message: string;                      // diagnostic only; never a machine contract
  path?: string;                        // mesh-root-relative, "/" separators
  designatorPath?: string;              // when attributable to a designator
}
```

Boundary principle (exact table is a spec-review deliverable): mesh invalidity is RESULT data (findings); thrown `WeaveApiError` is reserved for cannot-validate (admission failures, absent/unreadable mesh root, read failures). Two constraints on that table, from Codex r1:

- **No taxonomy overload (F2).** Landed `io-failure` means a physical WRITE failure at `stage === "write"` with partial-write disclosure — it must not be reused for read failures. A read-failure classification is an *additive* taxonomy amendment (new code and/or stage) ratified at spec review with migration guidance, or read failures must map onto existing load-stage meanings (e.g. `malformed-mesh`) where those genuinely apply.
- **Complete registry or no build (F3).** The spec review must deliver the full v1 `MeshValidationFindingCode` registry and a table covering every current refusal family (including the fixture-shaped "only supports …" planner gates — e.g. the settled extracted-knop pre-weave shape refusal Stagecraft's srd mesh hit, per the extractor R2 diagnosis): severity, result-vs-throw treatment, attribution fields, ordering/deduplication, and CLI exit-code behavior. A builder must not invent these semantics.

**Source-capability ruling (proposed arm, Codex F1):** v1 `validateMesh` refuses meshes whose pending candidates require repository-backed/floating source resolution, with a typed, documented limitation (consistent with `versionPayloads`' `unsupported-source` refusal), and the parity law is narrowed accordingly: CLI and API produce identical findings for mesh-local-source meshes; repository-backed meshes are a documented API limitation until a subprocess-free resolution seam exists. The alternative arms (injected resolver seam; broadened lib capability contract) are noted for the spec review but not preferred.

CLI parity law (as narrowed above): `weave validate` and `validateMesh` consume one findings pipeline; the CLI text is a rendering of the structured findings. Drift within the shared capability domain must be structurally impossible, not test-patrolled.

## Decisions (proposed)

- `validateMesh` ships as the v0.6.0 headline; a new public surface is a minor, not a 0.5.x patch.
- Same admission conventions as `versionPayloads`: absolute `meshRoot`, exact shapes, readonly result types, additive evolution only, message text never load-bearing.
- The API mutates nothing and takes no lock; document the same read-coherency caveat as dry-run (an unlocked read can observe a writer mid-operation; pointer to the advisory-lock pattern in [[wu.api-reference]]).
- v1 coverage is documented as what it is: recursive planner/preflight validation plus publication-readiness checks — not a whole-mesh integrity traversal (expanding coverage is future additive work, per the portable spec's "grow over time" framing).
- Contract documentation mirrors the versionPayloads pair: a new `wd.programmatic-validate-api` note owning the exact shapes, plus a portable behavior spec entry (`sf.spec.*`) in the framework notes.
- Result carries no retained plan text, parse artifacts, or per-file contents — findings stay compact so the contract is already compatible with the bounded-memory engine work.

## Contract Changes

- New stable exports from `src/api/mod.ts`, `src/mod.ts`, and `@semantic-flow/weave-lib`: `validateMesh`, request/result/target/finding types, and the finding-code union.
- `WeaveApiError` taxonomy: existing codes/stages reused only where landed semantics genuinely match; any new code or stage (e.g. read-failure classification) is itself a contract change requiring spec-review sign-off and migration guidance.
- No change to `versionPayloads`, CLI flags, CLI output text, mesh formats, or ontology in this slice.
- [[wu.api-reference]] gains the consumption-model section (ruling 1–3 above).
- Generated npm-lib package metadata and README (`scripts/build-npm-lib.ts:76-80`, `:117-145`) currently describe a `versionPayloads`-only library; they gain `validateMesh` (description, example, supported-source and validation-coverage caveats) as an explicit deliverable (Codex F5).

## Testing

- Admission/contract suite mirroring the `versionPayloads` tests: invalid roots, duplicate/invalid targets, scope/target combination rules, exact result shapes.
- Parity fixtures: a settled mesh yields zero findings on both surfaces; defect-seeded fixtures yield the same findings through CLI text and API result (one pipeline, two renderings).
- A repository-backed-source mesh case in the off-tree Node smoke proving the ruled F1 behavior (typed refusal, not silent divergence).
- `src/api/fs_purity_test.ts` green over the enlarged graph.
- Off-tree npm smoke (`deno task smoke:npm-lib`) gains a `validateMesh` leg: settled fixture → zero findings; seeded-defect fixture → expected finding code under Node.
- No new fixture-repo refs; prefer `createTestTmpDir()` synthesis per the extractor-lane precedent.

## Non-Goals

- Bounded-memory whole-mesh validation (sibling note [[wa.completed.2026.2026-07-29_1220-whole-mesh-validate-bounded-memory]] owns the engine).
- Expanding validation coverage beyond today's planner/preflight + publication-readiness semantics (documented as-is in v1; growth is future additive work).
- A CLI `--json` output flag (may become a cheap follow-up once the findings model exists; not this slice).
- JSR publishing; SHACL/semantic validation expansion; locking mechanisms; changes to publication-readiness semantics; extraction behavior changes (§2 is dispositioned above, owned by TODO 23).
- A streaming/AsyncIterable findings variant — deferred, but the v1 shape must not preclude adding one additively.

## Implementation Plan

- [x] Spec review r1 with these named deliverables (Codex blockers F1–F3 + F4): complete v1 finding-code registry + refusal-family table; read-failure taxonomy ruling; source-capability ruling; coverage definition and count semantics. Ruled in the 2026-07-30 PM session (ruling record above); PM GO given 2026-07-30 ("GO when you're ready") with the contract transcribed to `wd.programmatic-validate-api` before build.
- [x] Extract a structured findings model at the runtime `executeValidate` boundary; CLI rendering becomes a view of it. (Build receipts below.)
- [x] `src/api/validate_mesh.ts` admission + orchestration; barrel and root exports.
- [x] Contract, parity, fs-purity, and repository-source-refusal tests; dnt build; off-tree smoke legs.
- [x] Docs: `wd.programmatic-validate-api` (pre-build), `scripts/build-npm-lib.ts` package description/README (Codex F5), [[wu.api-reference]] validateMesh section + consumption-model ruling, [[wu.cli-reference.validate]] pointer, `wd.todo` line swap (all in lane commit `fe3a0d7`), and the portable entry `sf.spec.2026-07-30-programmatic-validate-api` (framework vault, uncommitted for the runner).
- [x] Board the release: v0.6.0 headline boarded 2026-07-31 as the `release-notes.v0.6.0` DRAFT stub (lane commit `8f9d742`, includes the named CLI behavioral change per the changelog rule); `wd.todo` line swapped (`fe3a0d7`, updated `8f9d742`).
- [ ] Feed rulings 1–3, the §2 disposition, and the delivery plan into the pending Stagecraft reply (deferred 2026-07-30 at PM direction).

## Build receipts — v1 validateMesh (2026-07-30)

Implementation seat: Codex (`codex exec`, workspace-write, `model_reasoning_effort=high`) against the ratified contract note; reviewer completed the two sandbox-blocked steps. Branch `lane/validate-mesh-api` off `c962ab2` (main), three path-scoped commits, no push:

- `295c530` `feat(validate): classify runtime findings with stable codes` — finding-code slots on `WeaveInputError`/`WeaveRuntimeError` tagged at family emission sites; `InventoryResolutionError` replaces the plain inventory `Error`s; the escaping config/policy families (`EffectiveConfigError`, `ConfigSourceDiscoveryError`, `ConfigInheritanceError`, `OperationalConfigError`, `ResourcePagePolicyError`) are caught and classified. Strict mode (API) rethrows untagged domain refusals raw; the CLI path keeps its exact text/exit behavior. Strict improvement noted: malformed inventory/config now render as CLI findings instead of crashing.
- `4d89b5a` `feat(api): add validateMesh with structured findings` — the contract surface exactly as ratified (14-code registry, coverage counts, optional `meshBase`, additive `read-failure` on the shared union, pre-resolution `unsupported-source` refusal via a threaded `mesh-local-only` source capability, raw propagation). Includes the reviewer's dnt-shim fix (below).
- `310bc40` `feat(lib): package validateMesh in weave-lib` — npm description/README with example + source-capability and planner-coverage caveats; off-tree smoke gains settled and seeded-defect `validateMesh` legs.

Gates: `deno task fmt` / `lint` / `check` green; full `env -u NO_COLOR deno task ci` **733 passed / 0 failed — re-earned independently by the reviewer**; `deno task build:npm-lib` + `deno task smoke:npm-lib` green (reviewer-run; Codex's sandbox had no network for `@deno/shim-deno`): "2 payloads versioned and validateMesh returned settled/defect contract results under Node".

### Adversarial review of the build (Jimbo, 2026-07-30) — ACCEPTED with one defect found and fixed

Verified against the contract: exact request/result shapes and code spellings match; admission (exact keys, absolute root, `/` alias, duplicate refusal); thrown taxonomy (`read-failure` families B1/N/E5, `malformed-mesh` reserved to the B2 no-mesh-surface precondition, `unknown-target`, pre-git `unsupported-source`); `meshBase` absent exactly with `malformed-mesh-metadata`; `WeaveApiErrorCode` extension is the single additive union line; **no message-string classification anywhere on the new paths**; strict-vs-CLI untagged handling honors the propagate-raw ruling (the `unsupported-mesh-shape` fallback fires only on the CLI path, where codes are never rendered).

- **Defect found (reviewer-fixed in `4d89b5a`):** `isDenoReadError` referenced `Deno.errors.NotADirectory`/`IsADirectory`, which the dnt Node shim's `errors` type omits — `build:npm-lib` failed with two TS2339 diagnostics. Fixed with a feature-tolerant lookup for exactly those two classes; focused suites (14/14) and the full smoke re-run green. Codex could not have caught this: its sandbox blocked the dnt build at network install.
- Honest-handoff quality: Codex reported the two environment blocks plainly and fabricated nothing (no invented SHAs, no fake smoke results).

Remaining for close after the docs commit (`fe3a0d7`): the landing plan below, then v0.6.0 release boarding and the Stagecraft reply.

### Landing plan (framed 2026-07-31 — PM decides D1; Jimbo executes D2 on that decision)

Two unpushed lanes exist off main `c962ab2`: this note's `lane/validate-mesh-api` (`fe3a0d7`, 4 commits) and the sibling's `lane/validate-memory-baseline` (`5a994c4`, 3 commits). They **collide on `src/runtime/weave/weave.ts` and `src/runtime/weave/version_execution.ts`** (measured: the only two files in both diffs), so landing order matters.

- **D1 — how to land (PM decision):** recommended arm — push `lane/validate-mesh-api` and open a PR to main, per the house pattern (v0.5.1 landed via PR #23, the arg-separator fix via #24), which buys the CodeRabbit pass and GitHub CI on the exact merge candidate; merge on green. Alternative arm: local merge without a PR (faster, skips the external review pass). Pushing is outward-facing and stays with the PM.
- **D2 — order and reconciliation (execution, once D1 is decided):** land `lane/validate-mesh-api` FIRST (contract-bearing, release headline, large diff), then rebase `lane/validate-memory-baseline` onto the updated main and reconcile — its overlapping changes are small and additive (a `memoryStats` parameter and wiring in the two colliding files), so reconciliation is minutes in that order; the reverse order rebases the large diff onto the small one for no benefit.
- **D3 — release boarding:** after both land, board v0.6.0 with `validateMesh` as the headline; the opt-in `WEAVE_MEMORY_STATS` instrumentation and the pending-heavy generator ride along (dev-facing, inert by default). Merging is not releasing — main routinely carries unreleased work.
- **D4 — sequencing:** the 1220 bounded-memory FIX slice (dedupe/lazy-load candidate source text) cuts only after both lanes are on main, since it builds on the instrumentation and touches the same candidate-loading surface the classification work reshaped.

### LANDED (2026-07-31)

D1 decided push+PR; both lanes are on main. **PR #25** (`lane/validate-mesh-api`) merged as `f9b64ce` after a CodeRabbit round adjudicated adversarially: 2 findings fixed in `8402727` (non-array `request.targets` admission message; `rdf_helpers.parseWeaveShapeQuads` now tags `unsupported-mesh-shape` at the source, covering all raw importers), 1 REFUTED with evidence (its Major proposed rejecting `../` working paths at inventory normalization — that breaks the landed extra-mesh workspace-relative working-file contract, pinned by 4 existing tests; boundary enforcement is the local-path policy layer's job, classified `path-boundary-violation`), 1 skipped per ratified contract (`read-failure` stays validateMesh-only; remapping landed `versionPayloads` codes needs its own amendment). codecov/patch informational-failed at 63.8% (the tagged refusal branches; families have representative coverage). **PR #26** (`lane/validate-memory-baseline`, rebased per D2 — conflicts exactly the two predicted files, memory wiring re-threaded through the classified `executeValidate`) merged as `c0b25b3` after one dnt-shim fix (`929d5e8`: no global `TextEncoder` type under Node type-checking; second instance of the shim defect class after `Deno.errors.NotADirectory`). Gates re-earned on each tip before merge: full ci 733/733 then 737/737, `build:npm-lib` + off-tree Node smoke green. Remaining: D3 release execution (v0.6.0 notes already boarded), D4 fix slice, Stagecraft reply.

### RELEASED (2026-07-31)

**v0.6.0 is live.** D4 landed first (PR #27, the bounded-memory fix — see the sibling note's FIX LANDED receipts), then release prep merged (PR #28, main `57b6b0d`), Release Manual rehearsal (dry-run + draft) came back 13/13 green, and the publish run completed: all six npm packages at `0.6.0` with dist-tag `latest`, tag `v0.6.0` at `57b6b0d`, GitHub Release published with 8 assets targeting the release commit. Post-release verification: fresh `npm install @semantic-flow/weave@0.6.0` self-reports `{"version":"0.6.0","commit":"57b6b0d…"}` via `--version --json`. The Stagecraft reply draft ([[wd.consumer-feedback.0.5.1.reply]] in weave docs) is ready to send — the only remaining act on this epic, and it is the maintainer's.

## Spec review r1 — ruling record (PM session, 2026-07-30, in-chat)

- **F2 read-failure taxonomy: RATIFIED** — additive new code at the existing `load` stage (working name `read-failure`) for I/O-environment problems (permission errors, vanished files); `io-failure` stays write-stage-only; `malformed-mesh` keeps meaning unparsable/invalid content.
- **F4 coverage semantics: RATIFIED** — result carries findings plus a small exact coverage-counts object (working shape: `knownDesignatorPathCount` + `plannedDesignatorPathCount`); no path arrays; documented as planner-coverage, not integrity-coverage.
- **v1 scope: RATIFIED** — mesh-only; `scope: "publication"` is an additive follow-up (matches Stagecraft's actual CI usage).
- **F1 source capability: RATIFIED — typed refusal in v1.** Use-case examination established that the only git surface on the validate path is repository-source *floating-locator* resolution (`src/runtime/operational/repository_source.ts` → lazy `repository_source_git.ts`) — git identifies the enclosing checkout (repo root + remote-URL match against the declared repository identity) before reading a plain local file; it never fetches content from a ref. Under Node this currently degrades to "no checkout", surfacing as a misleading missing-working-payload error. Ruling per the PM's necessary/convenience split: v1 `validateMesh` owns the necessary domain (mesh-local sources) with full CLI equivalence; floating-repository candidates refuse with a stable typed code (versionPayloads precedent); the atomic git operations stay CLI-only. The fullest architecture — one capability-injected checkout-identification seam shared by CLI and lib — is captured per PM requirement as the parked task sketch [[wa.task.2026.2026-07-30_1237-checkout-identification-seam]], strictly additive over the v1 refusal.

### Round 2 rulings (PM session, 2026-07-30 — registry, from the full refusal-family enumeration)

- **F3 finding-code registry: RATIFIED as the working v1 registry (14 codes).** Thrown `WeaveApiError`: `invalid-request` (admit — request/target shape), `read-failure` (load — new, per F2), `unknown-target` (load), `unsupported-source` (load — per F1). Findings, mesh-invalid: `malformed-mesh-metadata`, `malformed-inventory`, `malformed-config`, `missing-artifact`, `path-boundary-violation`, `unresolvable-extraction-source`, `malformed-page-definition`, `progression-conflict`, `naming-policy-violation`, `planned-rdf-invalid`, `plan-conflict`. Findings, weave-limitation: `unsupported-mesh-shape`. Findings, publication (emitted in mesh scope when a profile is configured): `publication-path-leakage`, `publication-profile-unsupported`. Exact spellings and the complete refusal-family → code mapping table are finalized in the `wd.programmatic-validate-api` contract note at build start; granularity is ratified.
- **Malformed mesh content = FINDINGS (validator inversion, RATIFIED).** Unparsable inventory Turtle, config-resolution failures, and bad mesh metadata become findings — deliberately diverging from versionPayloads' thrown `malformed-mesh` (correct for a mutator, wrong for a validator). Named implementation consequence: the API boundary must catch the error classes that today ESCAPE `executeValidate` uncaught (plain `Error` from `runtime/mesh/inventory.ts`, `EffectiveConfigError`, `ConfigSourceDiscoveryError`, `ConfigInheritanceError`, `OperationalConfigError` residues, `ResourcePagePolicyError`) and map them to finding codes; raw IO/permission errors map to thrown `read-failure`. This closes the two contract holes the 2026-07-30 enumeration exposed.
- **Shape gates = one code (RATIFIED).** All fixture-shaped "only supports …" planner/assertion gates surface as `unsupported-mesh-shape` findings; the message carries the specific gate; occurrences shrink as the planner generalizes while the code stays stable.
- **Unexpected errors propagate raw (RATIFIED).** Non-domain errors rethrow unwrapped — a weave defect crashes loudly rather than masquerading as a typed refusal; every typed code means something ruled.
- **Severity (RATIFIED with the registry).** v1 emits only `"error"`; `"warning"` is reserved in the union (the `snapshot-conflict` reserve precedent). CLI exit behavior unchanged: 0 on green, 1 on findings or error.

With these, every r1 blocker deliverable (F1–F4 + registry) is ruled; the note exits BLOCKED-ON-SPEC. Remaining before build: PM GO, and transcribing the registry + family→code table into `wd.programmatic-validate-api`.

## Open Issues

- Should the result carry a derived `valid: boolean` convenience, or is `findings.some(severity === "error")` the only truth? (Leaning: findings-only; a derived boolean invites drift.)
- Whether Stagecraft's CI wants an rc-style severity threshold contract on the CLI too (exit-code semantics doc) — reply-thread question, not a blocker.

## Spec review r1 — Codex (2026-07-29, adversarial, read-only sandbox)

Findings as reported (severities are Codex's proposals): **F1 BLOCKER** — unconditional CLI-parity promise conflicts with the lib's subprocess-free contract on repository-backed sources (fs-purity guard green while behavior diverges under Node). **F2 BLOCKER** — reserving thrown "read `io-failure`" overloads the landed write-stage-only meaning. **F3 BLOCKER** — the finding-code registry and planner-refusal map were left open, which a builder cannot fill without inventing semantics. **F4 HIGH** — "mesh validation" and `validatedDesignatorPathCount` overstated coverage that is actually planner/preflight + publication checks. **F5 MEDIUM** — generated npm-lib package description/README missing from scope. **F12 MEDIUM (cross-note)** — consumer §2 had no owned disposition despite the archive note's pointer.

Disposition (this revision, 2026-07-29): all six folded — F1 became the source-capability ruling (proposed typed-limitation arm) + narrowed parity law + Node smoke case; F2/F3/F4 became named spec-review deliverables and the coverage/count language was corrected throughout; F5 added to Contract Changes and the plan; F12 added as the "§2 disposition" section. Codex's proposed verdict — **BLOCKED-ON-SPEC** — is accepted as the note's correct resting state: build is gated on the spec review r1 this plan schedules first, which must produce the registry, taxonomy, capability, and coverage rulings.
