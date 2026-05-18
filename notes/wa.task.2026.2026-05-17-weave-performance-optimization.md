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
- Introduce a per-command planning context with cached parsed RDF graphs, mesh inventory state, knop inventories, source registries, reference catalogs, and source payload descriptions. The overlay can still invalidate or replace entries for files it changes.
- Keep the recursive application model if dependency ordering needs it, but avoid re-reading immutable inputs between candidates.
- Add lightweight timing logs around mesh state load, candidate discovery, version planning, RDF validation, writes, and page generation so the SFLO publication run tells us where the time is actually going.
- Consider a regression fixture based on the SFLO all-terms publication shape once the command is stable enough, even if it is a perf smoke rather than a strict timing gate.

## Acceptance Notes

- Progress output should be visible by default for `weave`.
- `--silent` should suppress progress output without hiding the final summary.
- The performance pass should be careful not to weaken the overlay behavior that lets later candidates see inventory updates from earlier candidates.
