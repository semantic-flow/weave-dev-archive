---
id: 0v10gnsy8u7my9vg0b6aro8
title: 2026 05 26_2237 Testing Optimization
desc: ''
updated: 1779860295055
created: 1779860295055
---

## Goals

- Make the full Weave test suite materially faster without weakening the semantic, filesystem, and CLI regression signal.
- Preserve hermetic tests: no ambient `HOME`, `WEAVE_SETTINGS`, XDG, or log-directory leakage.
- Give developers cheaper targeted loops for common changes: core logic, runtime logic, scripts, integration, and black-box CLI.
- Identify and remove fixture-materialization overhead before trying broad parallel execution again.
- Keep acceptance-style coverage for real mesh behavior, but stop using heavy mesh fixtures for tests that only need a tiny mesh shape.

## Summary

The slowest-looking path is fixture setup, not Deno itself. The current Alice Bio and Sidecar Fantasy Rules helpers materialize fixture branches by running one `git ls-tree` and then one `git show` per file. Many integration and e2e tests repeat that over the same handful of refs. That means the suite pays a lot of subprocess overhead before Weave does useful work.

The e2e tests are mostly using real dependency fixtures, not an internal minimal mesh:

- `mesh-alice-bio` for most Alice-path integration/e2e coverage.
- `mesh-sidecar-fantasy-rules` for docs-rooted sidecar coverage.
- `mesh-branch-fantasy-rules` for branch-published fixture and import coverage.

Raw `deno test --parallel` is not ready as a switch-flip optimization. Deno's `--parallel` runs test modules in parallel and exposes only job-count control through CPU count / `DENO_JOBS`; it does not let us declare dependency groups like "these tests share a fixture cache" or "these tests mutate process env." We saw failures when trying it, which is useful evidence: first make fixture setup faster and env/log behavior more explicit, then re-test parallelism on a deliberately safe bucket.

## Discussion

### Fixture Materialization

The obvious first target is `tests/support/mesh_alice_bio_fixture.ts` and sibling fixture helpers. They currently:

1. Resolve a fixture ref.
2. List every file in the Git tree.
3. For each file, create its parent directory and read content through `git show`.
4. Write the file into a fresh temp workspace.

This is simple and correct, but expensive. Possible improvements, roughly from safest to bolder:

- Cache `read*BranchFile(ref, path)` results by resolved commit and path. This helps both unit/core tests that read expected fixture files and materialization helpers that read the same files repeatedly.
- Cache `list*BranchFiles(ref)` by resolved commit. This is tiny but removes repeated `git ls-tree` calls.
- Build an immutable per-ref snapshot cache once per test process, then copy from that snapshot into each temp workspace. Tests still get private mutable workspaces, but Git extraction happens once per ref.
- Use a bulk Git export path instead of per-file `git show`. `git archive` is the cleanest mental model, but extraction needs either a Deno tar implementation or allowing `tar` in the test task. Another candidate is a carefully reviewed Git work-tree checkout into a scratch directory, but that has more foot-gun potential around the fixture repo's index.
- Use hardlinks or copy-on-write clones from immutable snapshots when the platform supports it, falling back to normal copy. This can be a later optimization once correctness is measured.

My bias: start with in-process content/list caching because it is small and low risk, then add immutable snapshot caches if the baseline still hurts.

### Narrow Fixtures

Some e2e tests need the real fixture ladder because the behavior is meaningful only against settled mesh output and conformance manifests. Others only need a mesh root with `_mesh/_meta`, `_mesh/_inventory`, a config file, and one or two content files. Those should not pay the Alice Bio fixture cost.

Good candidates for narrow synthetic fixture builders:

- CLI argument validation that should fail before execution.
- Logging placement and `WEAVE_LOG_DIR` override tests.
- Host-local access policy tests where the source path is the focus, not Alice Bio semantics.
- Mesh-root/workspace-root rejection tests.
- Publication profile or default config tests that only inspect a small generated file set.

The builder should live in `tests/support/`, use the same production helpers where practical, and produce a realistic but tiny mesh rather than a bag of unrelated files.

### Test Task Shape

The current `deno task test` is a good merge-confidence command, but it is too blunt for active development. Add stable task aliases without changing coverage:

- `test:unit`: narrow `src/**/*_test.ts` tests that do not spawn the CLI or materialize large fixtures.
- `test:runtime`: runtime tests that may touch filesystem and env, but avoid black-box CLI.
- `test:integration`: `tests/integration`.
- `test:e2e`: `tests/e2e`.
- `test:scripts`: `tests/scripts`.
- `test:fast`: unit + selected runtime tests that should stay comfortably under a minute.

The exact buckets should be measured rather than guessed. Some `src/core/*_test.ts` files read fixture files heavily and may not be "unit" in cost even if they are co-located with core code.

### Timing Instrumentation

Before changing too much, add a cheap opt-in timing harness:

```bash
WEAVE_TEST_TIMING=1 deno task test
```

The existing preload already wraps `Deno.test` for temp cleanup. It can also record per-test duration and print a slowest-test summary when the env var is set. This avoids scraping Deno's pretty reporter and gives us repeatable before/after data.

Also useful:

- Count fixture materializations by repo/ref.
- Count fixture file reads by repo/ref/path.
- Print total time spent in fixture helper subprocesses when `WEAVE_TEST_TIMING=1`.
- Keep timing output opt-in so normal test output stays quiet.

### Parallelism

Parallelism should be a second pass, not the foundation. The suite currently has patterns that are awkward under raw module parallelism:

- Process-global environment mutation, even though the preload isolates common variables per test.
- CLI subprocesses that inherit env unless every helper passes a complete explicit `env`.
- Default logs and settings that are mesh-identifier based and can collide if env isolation is incomplete.
- Shared fixture repos used read-only, which is mostly safe, but shared extraction caches need explicit locking if test modules run in parallel.

Once the fast/slow buckets are clear, try parallelism only on a known-safe bucket, probably unit/core tests first. Keep e2e serial until CLI env/log behavior is explicit enough that parallel failures would be surprising.

### CLI Env Explicitness

For black-box CLI tests, prefer command helpers that pass an explicit env map to `Deno.Command` rather than relying on ambient `Deno.env` state. That makes each subprocess reproducible and reduces the surface area that raw module parallelism can race over.

This is also friendlier for debugging: the failing command can print the env knobs that mattered without dumping the user's real shell environment.

## Open Issues

- What is the target full-suite time? A useful first bar might be under 4 minutes locally, then under 2 minutes for a fast developer loop.
- Should bulk fixture extraction add a `tar` runtime dependency to the test task, use a Deno-native tar reader, or avoid archive extraction entirely?
- Where should immutable fixture snapshots live when `WEAVE_KEEP_TEST_TMP=1` is set?
- Should snapshot cache keys use the human fixture ref, resolved commit SHA, or both? Resolved commit SHA is the correctness key; the human ref is useful in diagnostics.
- Which tests are acceptance coverage and should remain on real mesh fixtures even if they are slow?
- How much timing output belongs in `WEAVE_TEST_TIMING=1` before it becomes noise?

## Decisions

- Use Deno's native JUnit XML output as the source for Codecov Test Analytics rather than adding a separate test-report formatter.
- Keep local Codecov Test Analytics upload explicit via a dedicated task; the default `deno task test` and `deno task test:coverage` should not upload anything from a developer machine.
- Keep all coverage-producing output outside the repository at `/tmp/semantic-flow-coverage`, including CI/local raw coverage, JUnit XML, and LCOV.
- Recreate `/tmp/semantic-flow-coverage` in coverage-producing tasks so reboot-cleared temp storage does not need manual setup.
- Normalize Deno's JUnit XML for Codecov Test Analytics before upload because the raw Deno report is accepted for aggregate run time but does not currently populate individual test rows.
- Use the main Codecov action with `report_type: test_results` in CI instead of introducing the deprecated `codecov/test-results-action`.
- Do not add raw `deno test --parallel` back as a normal task yet. The first attempt exposed hidden coupling and did not produce a trustworthy faster suite.
- Optimize fixture materialization before broad scheduling changes.
- Keep the full `deno task test` as the merge-confidence command while introducing narrower developer-loop tasks.
- Keep e2e black-box coverage, but trim unnecessary real-fixture use where a tiny realistic mesh fixture would test the same behavior.
- Prefer opt-in timing instrumentation over always-on noisy output.

## Contract Changes

- No user-facing Weave behavior should change.
- Add developer-facing test task aliases in `deno.json` once buckets are chosen.
- Add `WEAVE_TEST_TIMING=1` as an opt-in test harness diagnostic.
- If a bulk extraction strategy needs more subprocesses, update test task permissions deliberately rather than broadening `--allow-run` casually.

## Testing

- Run timing baseline before and after each optimization so the changes are evidence-driven.
- Add focused tests for any new fixture cache/materializer helper, including cache invalidation by resolved commit SHA.
- Verify materialized fixture trees match the previous per-file materializer output.
- Run representative integration/e2e fixture tests after materializer changes.
- Keep a final `deno task test` pass before considering this task done.
- Re-try a narrow parallel bucket only after env/log explicitness and fixture cache locking are in place.

## Non-Goals

- Do not reduce semantic coverage just to lower the stopwatch number.
- Do not rewrite the fixture ladder or Accord model in this task.
- Do not make e2e tests depend on generated state that is harder to inspect than the current fixture repos.
- Do not make test output noisy by default.
- Do not add broad `--allow-run` or `--allow-all` permissions for convenience.

## Implementation Plan

- [x] Generate JUnit XML during coverage test runs for Codecov Test Analytics.
- [x] Upload CI test results to Codecov with the existing Codecov action and OIDC authentication.
- [x] Add an explicit local Codecov Test Analytics upload path that requires `CODECOV_TOKEN`.
- [x] Add focused script tests for the local Codecov Test Analytics parser and token gate.
- [x] Move coverage-producing output out of the repo workspace to `/tmp/semantic-flow-coverage`.
- [x] Recreate `/tmp/semantic-flow-coverage` before coverage-producing tasks.
- [x] Normalize Deno JUnit XML into Codecov's expected shape before test analytics upload.
- [ ] Add `WEAVE_TEST_TIMING=1` support to the test preload wrapper and print the slowest tests plus fixture helper totals.
- [ ] Record a serial baseline for `deno task test`, `tests/e2e`, `tests/integration`, and `src` tests.
- [ ] Cache resolved fixture refs, branch file lists, and branch file contents by fixture repo + resolved commit + path.
- [ ] Benchmark the content/list cache against the baseline.
- [ ] Add an immutable per-ref fixture snapshot cache if per-file Git subprocess overhead remains significant.
- [ ] Add a tiny mesh fixture builder for tests that do not need the real Alice Bio or Sidecar fixture ladder.
- [ ] Migrate obvious validation/logging/host-policy tests from heavy real fixtures to the tiny mesh builder.
- [ ] Split test task aliases in `deno.json` after the bucket boundaries are clear.
- [ ] Make black-box CLI test helpers pass explicit env maps where they currently rely on ambient test-process env.
- [ ] Retry `deno test --parallel` on only the safe bucket and document the result here.
