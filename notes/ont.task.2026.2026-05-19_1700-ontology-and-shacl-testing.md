---
id: wuvsndhlzer74txfmjr0bs9
title: 2026 05 19_1700 Ontology and Shacl Testing
desc: ''
updated: 1779235278771
created: 1779235278771
---

## Goals

- Add code-based ontology and SHACL testing to the `semantic-flow/sflo` repository.
- Use Deno if practical, so the ontology repo can share the same lightweight tooling posture as Weave without introducing a Node project.
- Move ontology-owned guardrails out of Weave CI.
- Keep Weave responsible only for Weave-owned defaults and runtime behavior.
- Make the `sflo` release workflow safer before automating ontology releases.

## Summary

Weave currently has integration tests that read active ontology files from `dependencies/github.com/semantic-flow/sflo`. That was useful while ontology/runtime behavior was moving quickly, but it is the wrong long-term ownership boundary. `sflo` should test its own ontology and SHACL files. Weave should test how it consumes packaged/default configuration and how its runtime behaves.

This task sets up a first Deno test harness in `sflo`, ports the ontology guardrails there, adds initial SHACL-oriented checks, and then removes the cross-repo smell from Weave.

## Discussion

### Current State

The `sflo` repo currently has Turtle ontology files, Dendron notes, release notes, and runbooks, but no code-based test harness.

Important active files:

- `semantic-flow-core-ontology.ttl`
- `semantic-flow-core-shacl.ttl`
- `semantic-flow-config-ontology.ttl`
- `semantic-flow-job-ontology.ttl`
- `semantic-flow-prov-ontology.ttl`

Weave currently carries tests in `tests/integration/ontology_guardrails_test.ts` that check some of these files by path. That forced Weave CI to check out `semantic-flow/sflo` just to run Weave's own test suite. The checkout fix is pragmatic for the current PR, but the dependency is a smell.

### Desired Ownership Boundary

`sflo` should own tests for:

- ontology Turtle parseability.
- SHACL Turtle parseability.
- duplicate RDF triples in ontology/SHACL files.
- canonical namespace usage.
- retired ontology/config names.
- config-term IRI shape.
- SHACL shapes that should validate representative conforming and non-conforming example graphs.

Weave should own tests for:

- `defaults/application.ttl` and `defaults/config-resolution.ttl` parsing.
- Weave runtime behavior that consumes those defaults.
- Weave-owned retired config fragments in Weave source/defaults.
- integration tests against checked-out fixtures when those fixtures are part of Weave's own conformance workflow.

### Deno Harness Shape

Add a minimal Deno project to `sflo`:

```json
{
  "tasks": {
    "fmt": "deno fmt deno.json tests",
    "fmt:check": "deno fmt --check deno.json tests",
    "lint": "deno lint tests",
    "check": "deno check tests/**/*.ts",
    "test": "deno test --allow-read tests",
    "ci": "deno task fmt:check && deno task lint && deno task check && deno task test"
  },
  "imports": {
    "@std/assert": "jsr:@std/assert@1",
    "@std/path": "jsr:@std/path@1",
    "n3": "npm:n3@2.0.3"
  },
  "fmt": {
    "proseWrap": "preserve",
    "exclude": ["notes/**", "**/*.md"]
  }
}
```

The first tests can use `n3` only. If SHACL validation is added in the first pass, prefer a Deno-compatible npm package such as `rdf-validate-shacl` only after a small spike proves it works cleanly under Deno. If that package is awkward, keep the first pass to RDF/shape graph guardrails and add SHACL validation as the second slice.

### First Test Slices

Start with tests that are cheap, deterministic, and already valuable:

- parse every active `.ttl` ontology and SHACL file with `n3`.
- reject duplicate RDF triples in every active RDF file.
- assert the config ontology imports core terms through `https://semantic-flow.github.io/sflo/ontology/`.
- assert the config ontology uses `@base <https://semantic-flow.github.io/sflo/config/> .`.
- assert the slashless config ontology IRI remains `<https://semantic-flow.github.io/sflo/config>`.
- assert active `sfcfg:` terms use flat namespace-local names, except explicit release/content URL paths if those remain intentional.
- assert retired config fragments such as `artifactResolutionMode_current`, `artifactResolutionMode_pinned`, `meshRootPathBase`, and old boolean policy switches do not reappear in active ontology/config files.

Then add SHACL-focused tests:

- parse `semantic-flow-core-shacl.ttl` as Turtle.
- assert key shape nodes exist for local working source bindings, repository-backed source bindings, resolution-mode guidance, and publication/config vocabulary as those shapes stabilize.
- validate a small conforming graph for a working source binding.
- validate a small non-conforming graph for a contradictory exact/working/latest-state shape.
- distinguish warning and info severities where the SHACL vocabulary now intends that distinction.

### Weave Cleanup

After `sflo` owns the ontology tests:

- remove the `semantic-flow/sflo` checkout from `.github/workflows/ci.yml`.
- split or replace Weave's `tests/integration/ontology_guardrails_test.ts`.
- keep a Weave-local defaults guardrail that only reads:
  - `defaults/application.ttl`
  - `defaults/config-resolution.ttl`
  - any Weave-owned runtime source files that should stay free of retired config fragments.
- keep Weave integration tests that use SFF/fixture repos only where those fixtures are part of Weave's conformance story.

## Open Issues

- Which SHACL validation engine should `sflo` use under Deno? Recommendation: spike `rdf-validate-shacl`; if it is unpleasant, start with RDF graph guardrails and add validation later.
- Should `sflo` tests validate ontology files against their own SHACL shapes, or only validate small curated example graphs? Recommendation: do both eventually, but start with curated examples because self-validation can produce noisy meta-model questions.
- Should release metadata/content URL checks belong in this test harness? Recommendation: yes, but as a later release-specific slice once the v0.1.1 release process is clearer.
- Should Weave keep an optional cross-repo dogfood workflow? Recommendation: maybe, but separate from core Weave CI. It should be named as an integration canary, not disguised as Weave unit/integration testing.

## Decisions

- `sflo` owns ontology and SHACL correctness tests.
- Weave should not require a checkout of `semantic-flow/sflo` for ordinary `deno task ci`.
- Deno is the preferred first harness for `sflo`.
- First-pass tests should be deterministic file/graph guardrails before attempting richer SHACL validation.
- The temporary Weave CI checkout of `semantic-flow/sflo` should be removed once the tests are ported or split.

## Contract Changes

- Add `deno.json` to `sflo`.
- Add `tests/` to `sflo`.
- Add `.github/workflows/ci.yml` to `sflo` if one does not exist.
- Move ontology-owned guardrail expectations from Weave into `sflo`.
- Replace Weave's ontology guardrail test with a Weave defaults/runtime guardrail.
- Update `sflo` release runbook so release candidates run `deno task ci` before publishing/tagging.

## Testing

In `sflo`:

- `deno task fmt:check`
- `deno task lint`
- `deno task check`
- `deno task test`
- `deno task ci`

In Weave after cleanup:

- `deno task fmt:check`
- `deno task lint`
- `deno task check`
- focused Weave defaults guardrail tests.
- full `deno task ci`.

For SHACL validation, add focused tests that assert both positive and negative behavior, not just parser success.

## Non-Goals

- Do not solve full ontology release automation in this task.
- Do not require Weave to fetch or vendor the active `sflo` repo for ordinary CI.
- Do not make `sflo` depend on Weave implementation code.
- Do not add a large build system or Node package setup unless Deno proves unsuitable.
- Do not attempt complete ontology logical consistency checking in the first pass.
- Do not conflate Dendron note validation with ontology/SHACL RDF validation.

## Implementation Plan

- [ ] Add `deno.json` to `semantic-flow/sflo` with Deno-native fmt/lint/check/test/ci tasks.
- [ ] Add `tests/helpers/rdf.ts` or similar utilities for reading repo files, parsing Turtle with `n3`, and normalizing RDF terms.
- [ ] Add RDF parseability tests for all active ontology/SHACL Turtle files.
- [ ] Add duplicate-triple tests for all active RDF files.
- [ ] Port config namespace and flat-term guardrails from Weave into `sflo`.
- [ ] Add retired-term guardrails for old config names, boolean policy switches, `_current`, and `_pinned`.
- [ ] Spike a Deno-compatible SHACL validator.
- [ ] Add first conforming/non-conforming SHACL example tests, or defer with a clear task if the validator spike needs more work.
- [ ] Add or update `sflo` GitHub Actions CI to run `deno task ci`.
- [ ] Update `sflo` release runbook to require the new CI/test command.
- [ ] Split Weave's `ontology_guardrails_test.ts` into a Weave-owned defaults/runtime guardrail.
- [ ] Remove `semantic-flow/sflo` checkout from Weave CI once no Weave test reads active `sflo` files.
- [ ] Run `deno task ci` in both `sflo` and Weave after the split.
- [ ] Provide separate commit messages for `sflo`, Weave, and `weave-dev-archive`.
