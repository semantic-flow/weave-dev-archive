---
id: ryy4rs3d3mnyyx618iwd894
title: 2026 05 19_0200 Coderabbi
desc: ''
updated: 1779181228381
created: 1779181225668
---

Verify each finding against current code. Fix only still-valid issues, skip the rest with a brief reason, keep changes minimal, and validate.

Status legend: `[x]` handled, `[c]` canceled as wrongheaded, `[d]` deferred.

## Inline Comments

- [x] `AGENTS.md` duplicate rename rule and typos. Removed the duplicate rule, fixed "no '.md' extension" and "wikilinks", and clarified that durable developer notes belong under `documentation/notes/wd.*`.
- [x] `documentation/notes/wd.general-guidance.md` task-note location conflict. Refined rather than applying the review literally: Kato/workflow task notes remain in `dependencies/github.com/semantic-flow/weave-dev-archive/notes`, while durable developer guidance remains in `documentation/notes/wd.*`.
- [x] `src/core/payload/version_intent.ts` duplicate current-history triples. Deduplicated identical `sflo:currentArtifactHistory` values before ambiguity checking and added a regression test.
- [x] `src/core/weave/weave.ts` first-payload history-intent guard. Changed payload history blocking to require a declared `ArtifactHistory`, and narrowed the older KnopInventory history guard so payload history intent does not block first-payload weaving.
- [x] `src/core/weave/weave.ts` reference-catalog working file resolution. Added literal `sflo:workingLocalRelativePath` fallback after `sflo:hasWorkingLocatedFile`, with a regression test using the literal form.
- [x] `src/runtime/integrate/integrate.ts` source digest verification. Always computes the sha256 digest, rejects mismatched requested digests, and records the verified computed digest.
- [x] `src/runtime/integrate/integrate.ts` workspace-root grants. Routes workspace-root grants to host-local access config instead of trying to write a mesh-carried `pathPrefix ".."` rule, with a CLI regression test.

## Nitpick Comments

- [x] `src/cli/run.ts` source-binding option help. Updated the integrate option help strings to state the repository metadata coupling. There is only one option block in the current file.
- [x] `src/core/integrate/integrate.ts` duplicated Turtle escaping. Extracted `escapeTurtleString` to `src/core/rdf/turtle.ts` and imported it from core integrate/extract.
- [x] `src/core/mesh/create_test.ts` auto GitHub Pages config assertion. Added an assertion that `_mesh/_config/config.ttl` includes the `githubPages` publication profile triple.

## Validation

- [x] Focused tests: `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env src/core/payload/version_intent_test.ts src/core/weave/weave_test.ts src/core/mesh/create_test.ts tests/integration/integrate_test.ts tests/e2e/integrate_cli_test.ts`
- [x] Focused retry after assertion fix: `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env tests/e2e/integrate_cli_test.ts`
- [x] `deno task fmt:check`
- [x] `deno task lint`
- [x] `deno task check`
- [x] `git diff --check`
