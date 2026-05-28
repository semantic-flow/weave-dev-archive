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

The first confirmed bottleneck was fixture setup, not Deno itself. Alice Bio, Sidecar Fantasy Rules, and Branch Fantasy Rules helpers repeatedly materialized the same fixture refs by running one `git ls-tree` and then one `git show` per file. The immutable snapshot cache fixed enough of that cost to cut the full analytics run from about 8m 32s to about 4m 35s.

The next target is a 2-minute full-suite run. The remaining slow suites are still e2e CLI-heavy, so the next likely costs are CLI subprocess startup, repeated workspace setup, and tests that still use full real fixtures when a tiny realistic mesh would preserve the same signal.

The e2e tests are mostly using real dependency fixtures, not an internal minimal mesh:

- `mesh-alice-bio` for most Alice-path integration/e2e coverage.
- `mesh-sidecar-fantasy-rules` for docs-rooted sidecar coverage.
- `mesh-branch-fantasy-rules` for branch-published fixture and import coverage.

Raw `deno test --parallel` is not ready as a switch-flip optimization. Deno's `--parallel` runs test modules in parallel and exposes only job-count control through CPU count / `DENO_JOBS`; it does not let us declare dependency groups like "these tests share a fixture cache" or "these tests mutate process env." We saw failures when trying it, which is useful evidence: first make fixture setup faster and env/log behavior more explicit, then re-test parallelism on a deliberately safe bucket.

## Baseline

Codecov Test Analytics is now the canonical rolling history for test timing: https://app.codecov.io/gh/semantic-flow/weave/tests

Baseline summary captured from the normalized Codecov JUnit artifact at `/tmp/semantic-flow-coverage/codecov-junit.xml` after individual test rows started populating in Codecov:

- Captured artifact timestamp: `2026-05-28T02:07:02.049Z`
- Tests: 606
- Suites: 64
- Failures/errors/skips: 0 / 0 / 0
- Reported total test time: 512.486s (about 8m 32s)

Slowest suites in this baseline:

- `./tests/e2e/weave_cli_test.ts`: 35 tests, 209.578s
- `./tests/e2e/integrate_cli_test.ts`: 12 tests, 98.547s
- `./tests/e2e/extract_cli_test.ts`: 9 tests, 53.875s
- `./tests/e2e/payload_update_cli_test.ts`: 4 tests, 33.008s
- `./tests/e2e/knop_add_reference_cli_test.ts`: 3 tests, 24.155s
- `./tests/e2e/knop_create_cli_test.ts`: 4 tests, 19.013s
- `./tests/e2e/mesh_create_cli_test.ts`: 4 tests, 18.911s
- `./tests/integration/weave_test.ts`: 47 tests, 11.144s

Slowest individual tests in this baseline:

- `weave integrate requires a designator path before logging or execution` (`./tests/e2e/integrate_cli_test.ts`): 14.451s
- `weave payload update rejects conflicting designator paths before logging or execution` (`./tests/e2e/payload_update_cli_test.ts`): 12.695s
- `weave knop add-reference matches the manifest-scoped alice-bio referenced fixture as a black-box CLI run` (`./tests/e2e/knop_add_reference_cli_test.ts`): 11.376s
- `weave reports progress by default and --silent suppresses progress` (`./tests/e2e/weave_cli_test.ts`): 11.169s
- `weave integrate rejects partial repository-backed source metadata before execution` (`./tests/e2e/integrate_cli_test.ts`): 10.767s

The main signal is that e2e CLI tests dominate the baseline. That does not automatically mean "make e2e smaller"; several are valuable black-box acceptance checks. It does mean that optimizing repeated fixture materialization and CLI subprocess setup should come before micro-optimizing core unit tests.

## Discussion

### Fixture Materialization

The original first target was `tests/support/mesh_alice_bio_fixture.ts` and sibling fixture helpers. They used to:

1. Resolve a fixture ref.
2. List every file in the Git tree.
3. For each file, create its parent directory and read content through `git show`.
4. Write the file into a fresh temp workspace.

This was simple and correct, but expensive. The first implementation pass added an immutable snapshot cache under `/tmp/semantic-flow-fixture-snapshots`, keyed by fixture label and resolved commit SHA. Each fixture ref resolves to a commit, that commit materializes once into the temp snapshot root, and each test still receives a private mutable workspace copied from the snapshot.

`git archive` plus `tar` is not the right next bet. It could reduce cold snapshot construction time, but the cache already pays that extraction cost once per resolved commit and the remaining benchmark is dominated by e2e CLI suites. Keep the test subprocess permission surface narrow for now; revisit bulk export only if cold-cache data says snapshot construction is still material.

Possible later improvements:

- Use hardlinks or copy-on-write clones from immutable snapshots when the platform supports it, falling back to normal copy.
- Add a targeted counter for snapshot builds and materializations if Codecov cannot explain the next bottleneck.

First-pass local smoke checks after adding the snapshot cache:

- `deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git --allow-env tests/support/fixture_snapshot_test.ts`: passed
- `deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env tests/integration/import_test.ts`: passed in 42ms
- `deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env --filter "weave integrate requires a designator path before logging or execution" tests/e2e/integrate_cli_test.ts`: passed in about 2s on a warm local run
- `deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env --filter "weave knop add-reference matches the manifest-scoped alice-bio referenced fixture as a black-box CLI run" tests/e2e/knop_add_reference_cli_test.ts`: passed in about 5s on a warm local run

These are smoke checks, not the final benchmark. Use Codecov and a fresh CI/local baseline to judge the actual suite-level improvement.

Full benchmark after the snapshot cache:

- Command: `deno task test:analytics`
- Captured run: `2026-05-27 23:38` America/Los_Angeles, with normalized JUnit at `/tmp/semantic-flow-coverage/codecov-junit.xml`
- Result: 631 tests, 66 suites, 0 failures/errors/skips
- Deno runner wall-clock: 4m 34s
- Reported total test time: 274.967s (about 4m 35s)
- Change from baseline: 512.486s to 274.967s, a reduction of 237.519s / 46.3%, despite the suite growing from 606 to 631 tests

Slowest suites after the snapshot cache:

- `./tests/e2e/weave_cli_test.ts`: 35 tests, 116.413s, down from 209.578s
- `./tests/e2e/integrate_cli_test.ts`: 12 tests, 46.456s, down from 98.547s
- `./tests/e2e/extract_cli_test.ts`: 9 tests, 31.753s, down from 53.875s
- `./tests/e2e/knop_create_cli_test.ts`: 4 tests, 10.532s, down from 19.013s
- `./tests/e2e/mesh_create_cli_test.ts`: 4 tests, 9.385s, down from 18.911s
- `./tests/e2e/payload_update_cli_test.ts`: 4 tests, 9.231s, down from 33.008s
- `./tests/e2e/knop_add_reference_cli_test.ts`: 3 tests, 9.153s, down from 24.155s
- `./tests/e2e/import_cli_test.ts`: 2 tests, 7.297s
- `./tests/integration/weave_test.ts`: 47 tests, 5.300s, down from 11.144s

The snapshot cache had a large suite-level impact and should stay. The next optimization should target the remaining e2e CLI subprocess/workspace overhead rather than reworking fixture extraction again immediately.

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

### Timing And Fixture Instrumentation

Codecov Test Analytics now covers the main per-suite and per-test timing baseline, including slowest tests. Do not add `WEAVE_TEST_TIMING=1` just to duplicate that view locally.

Local opt-in instrumentation may still be useful, but it should answer questions Codecov cannot:

- Count fixture materializations by repo/ref.
- Count fixture file reads by repo/ref/path.
- Print total time spent in fixture helper subprocesses.
- Count CLI subprocess launches by test helper, if that turns out to be the real bottleneck after fixture caching.

If added, keep this output opt-in so normal test output stays quiet. The name can still be `WEAVE_TEST_TIMING=1` if the diagnostics stay broad, but a fixture-specific knob such as `WEAVE_FIXTURE_TIMING=1` may be clearer once the implementation is scoped.

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

- Which e2e CLI tests still need a full curated fixture, and which can move to a tiny realistic mesh without losing regression signal?
- How much of the remaining e2e time is CLI subprocess startup, workspace copying, or actual command behavior?
- Can snapshot-to-workspace copies use hardlinks or copy-on-write safely enough to matter?
- Which narrower test task aliases are useful once the fast/slow bucket boundaries are clear?
- Which safe bucket, if any, can use `deno test --parallel` after CLI env/log isolation is explicit?

## Decisions

- Use Deno's native JUnit XML output as the source for Codecov Test Analytics rather than adding a separate test-report formatter.
- Keep local Codecov Test Analytics upload explicit via a dedicated task; the default `deno task test` and `deno task test:coverage` should not upload anything from a developer machine.
- Keep all coverage-producing output outside the repository at `/tmp/semantic-flow-coverage`, including CI/local raw coverage, JUnit XML, and LCOV.
- Recreate `/tmp/semantic-flow-coverage` in coverage-producing tasks so reboot-cleared temp storage does not need manual setup.
- Normalize Deno's JUnit XML for Codecov Test Analytics before upload because the raw Deno report is accepted for aggregate run time but does not currently populate individual test rows.
- Use the main Codecov action with `report_type: test_results` in CI instead of introducing the deprecated `codecov/test-results-action`.
- Do not add raw `deno test --parallel` back as a normal task yet. The first attempt exposed hidden coupling and did not produce a trustworthy faster suite.
- Optimize fixture materialization before broad scheduling changes.
- Target a 2-minute full-suite analytics run as the next performance bar.
- Keep immutable fixture snapshots in `/tmp/semantic-flow-fixture-snapshots`; reboot-cleared temp storage is fine for this cache.
- Use resolved commit SHA as the correctness key for fixture snapshots. Human fixture labels are diagnostic and path-grouping context.
- Do not add `tar` or another bulk export path now; the snapshot cache made extraction a smaller problem than e2e CLI subprocess/workspace overhead.
- Keep the full `deno task test` as the merge-confidence command while introducing narrower developer-loop tasks.
- Keep e2e black-box coverage, but trim unnecessary real-fixture use where a tiny realistic mesh fixture would test the same behavior.
- Do not classify acceptance coverage in this optimization pass. Preserve existing black-box/fixture coverage while trimming only cases that clearly do not need curated real fixtures.
- Prefer Codecov Test Analytics for broad timing. Add local targeted timing only when it answers a question Codecov cannot, such as setup-vs-command attribution inside one slow e2e suite.

## Contract Changes

- No user-facing Weave behavior should change.
- Add developer-facing test task aliases in `deno.json` once buckets are chosen.
- Do not add a local slowest-test timing harness while Codecov Test Analytics is working; add opt-in local fixture/subprocess diagnostics only if Codecov cannot explain the next bottleneck.
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
- [x] Save a Codecov-backed timing baseline summary in this note.
- [c] Do not add `WEAVE_TEST_TIMING=1` just to print slowest tests; Codecov Test Analytics now covers that.
- [d] Add opt-in fixture/subprocess instrumentation only if Codecov cannot attribute the next bottleneck.
- [c] Record a separate serial baseline for `deno task test`, `tests/e2e`, `tests/integration`, and `src` tests; Codecov JUnit timing is the canonical baseline now.
- [x] Add an immutable per-ref fixture snapshot cache keyed by fixture repo + resolved commit.
- [x] Convert Alice Bio, Sidecar Fantasy Rules, and Branch Fantasy Rules fixture materializers to copy from snapshots.
- [x] Benchmark the snapshot cache against the baseline.
- [ ] Add a tiny mesh fixture builder for tests that do not need the real Alice Bio or Sidecar fixture ladder.
- [ ] Migrate obvious validation/logging/host-policy tests from heavy real fixtures to the tiny mesh builder.
- [ ] Split test task aliases in `deno.json` after the bucket boundaries are clear.
- [ ] Make black-box CLI test helpers pass explicit env maps where they currently rely on ambient test-process env.
- [ ] Retry `deno test --parallel` on only the safe bucket and document the result here.
