---
id: rsr9rzb3pqxb0t49stqoyh1
title: 2026 05 17 Weave Performance Optimization
desc: ''
updated: 1779040273228
created: 1779040273228
---

## Context

Running `weave --mesh-root "$SFLO_PUB"` over the SFLO publication mesh can take long enough that silence feels like a hang. First UX fix: emit progress for each completed knop by default, with a `--silent` flag for quiet automation.

## Observations

The root `weave` flow had been validating by calling the version planner, then calling `executeVersion`, which called the version planner again before writing files. That meant expensive planning work was duplicated before page generation even started. The first fix removes that redundant pre-plan while preserving root `weave`'s pre-write RDF validation and input-error behavior.

The recursive version planner also reloads staged mesh state and reloads weaveable knop candidates for each remaining designator path. That preserves the current overlay semantics, but it likely means repeated RDF parsing, inventory lookup, and source/reference resolution across the same files.

## Ideas

- Keep version planning single-pass in root `weave`; do not reintroduce a separate validate-then-version planning sequence.
- Introduce a per-command, in-memory planning context with cached parsed RDF graphs, mesh inventory state, knop inventories, source registries, reference catalogs, and source payload descriptions. This should not be a persisted cache; it is only a way to avoid rereading and reparsing stable inputs during one command invocation. The overlay can still invalidate or replace entries for files it changes.
- Keep the recursive application model if dependency ordering needs it, but avoid re-reading immutable inputs between candidates.
- Add lightweight timing logs around mesh metadata/config load, mesh inventory load/parse, candidate discovery, per-candidate planning, RDF validation, writes, and page generation so the SFLO publication run tells us where the time is actually going.
- Consider a regression fixture based on the SFLO all-terms publication shape once the command is stable enough, even if it is a perf smoke rather than a strict timing gate.
- Include `validate mesh` in the performance pass. The SFLO all-terms replay showed that pre-weave validation over hundreds of pending extracted terms can feel hung because validation has to dry-run/plan much of the same work. Either validation needs progress output too, or it needs to share the same planning/cache context that weave uses.

## Proposed Sequence

- [ ] Add opt-in timing instrumentation with an environment variable first, probably `WEAVE_TIMING=1`, before doing deeper optimization.
- [ ] Re-run against the regenerated SFLO `gh-pages` worktree, or a copied fixture, and capture which phases dominate. Record runs in `timings/weave-performance.csv` in the weave-dev-archive repo.
- [ ] Add a command-scoped in-memory file/RDF parse cache with planned-output overlay invalidation for files written during the command.
- [ ] Cache enough candidate discovery state that recursive planning does not rescan and reparse the whole mesh from scratch for each remaining candidate.
- [ ] Add `--verbose` for commands that may have lots of update/progress details. Default output should remain reassuring but not noisy, while `--silent` should stay suitable for automation.
- [ ] Add progress output to expensive validation phases, especially `validate mesh` when it is dry-running many pending extracted-term weaves. Detailed per-candidate validation chatter can live behind `--verbose`.
- [ ] Prefer a perf smoke or instrumentation-oriented regression test over strict wall-clock assertions. Timing gates are likely to be flaky; a better early check is something like “unchanged files are not reread/reparsed repeatedly during a batch weave.”

## Decisions

- Timing should start as environment-controlled instrumentation, not a new required CLI surface. `WEAVE_TIMING=1` keeps existing command examples copy/pasteable while profiling them. A later `--timing` flag can be added if this becomes normal user-facing behavior.
- Do timings first, then establish the planning context shape from evidence. The codebase is probably overdue for a larger refactor because several runtime/core files have grown large, but this task should start by showing where the time goes.
- The parse/read cache is a fresh slate for every command invocation. “Invalidation” only means intra-command overlay behavior: if the command has planned a new version of a file, later reads during the same invocation must use that planned content instead of an earlier cached parse of the on-disk file.
- Add a `--verbose` mode for commands that may have many progress/update details. Keep default progress concise; keep `--silent` quiet except for final summaries.
- Keep the SFLO all-terms workload as a local-only profiling recipe for now. We already have checked-in fixture ladders, and the regenerated SFLO mesh is large enough that adding it as another fixture should be a separate decision.
- Record timing runs in weave-dev-archive under `timings/`. Start with `timings/weave-performance.csv`; add helper output later if manual recording becomes annoying.

## Acceptance Notes

- Progress output should be visible by default for `weave`.
- `--silent` should suppress progress output without hiding the final summary.
- The performance pass should be careful not to weaken the overlay behavior that lets later candidates see inventory updates from earlier candidates.
- Any parse/read cache introduced here should be in-memory and command-scoped only.
- `validate mesh` should either show meaningful progress during large pending-weave validation or benefit from the same planning/cache improvements as `weave`.
