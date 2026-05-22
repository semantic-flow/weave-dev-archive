---
id: weave-core-candidate-mod-rpqyxl
title: 'weave core candidate model extraction'
desc: ''
created: 1779479956445
updated: 1779479956445
participants: [codex.gpt-5.5]
conversationEventKinds: [message.assistant, message.user, tool.call, tool.result]
---

# User_2026-05-22_1250_14

weave core candidate model extraction

Please review our next task: [[wa.task.2026.2026-05-21_1037-core-weave-first-extraction-slice]] and suggest any impovements, and surface any open issues.

# gpt-5.5_2026-05-22_1250_22

I’ll read the required guidance first, then the task note itself and nearby context so the review is anchored in the repo’s own shape rather than abstract extraction advice.

The task note is already nicely constrained: it is explicitly type/model extraction, with public façade compatibility and no semantic output change. I’m going to check the linked parent task and the current `weave.ts` shape now, because the useful review is likely hiding in import edges and stale checklist items.

One concrete thing already surfaced: the handoff line count is still accurate at 8,597, and the public exports targeted by the note are exactly clustered near the top of `weave.ts`. I’m checking the dependency direction now so I can tell you where the task should be tightened before somebody starts moving files.

I reviewed the task against the current code and docs. Net: it is a good next slice and I’d keep it active. The scope is appropriately boring: the targeted exported models are still clustered at the top of [weave.ts](../../../../../src/core/weave/weave.ts:168), the line count is still `8,597`, and the destination split matches the core-layer guidance in [wd.codebase-overview.md](../../../../../documentation/notes/wd.codebase-overview.md:29).

**Suggested Improvements**

1. Add an explicit dependency-shape note before implementation:
   - `requests.ts` imports only target types from `../targeting.ts`.
   - `candidates.ts` imports `RepositorySourceFloatingLocator` from `resource_page_models.ts`.
   - `planning_models.ts` imports `PlannedFile`, request/candidate types, policy types, and `ResourcePageModel`.
   - `weave.ts` imports these as types and re-exports them as the façade.

2. Tighten the direct-import policy. The task currently says runtime imports should not need broad changes, but I’d push a little here: update touched runtime type imports to the new focused modules. For example [request_normalization.ts](../../../../../src/runtime/weave/request_normalization.ts:9), [artifact_loaders.ts](../../../../../src/runtime/weave/artifact_loaders.ts:4), and [candidate_loader.ts](../../../../../src/runtime/weave/candidate_loader.ts:10) should stop importing moved type names through the big façade. Keep `detectPendingWeaveSlice` from `weave.ts` until the later slice.

3. Make the import-graph audit concrete. The note says to run one, but not how. I’d add acceptance criteria like:
   - no new `src/core` -> `src/runtime` edges
   - no local cycles among `src/` modules
   - `src/core/weave/weave.ts` remains the only public façade for existing imports
   - moved type-only runtime imports prefer the extracted modules

4. Move the already-completed ResourcePage checklist item out of the implementation checklist. It is correctly marked done in [the task note](wa.task.2026.2026-05-21_1037-core-weave-first-extraction-slice.md:103), but leaving it in the active plan adds noise.

**Open Issues**

- `WeaveSlice` is the only slightly awkward boundary. It is used by `detectPendingWeaveSlice`, which is explicitly not moving in this slice. Moving `WeaveSlice` to `planning_models.ts` is fine, but call it transitional and avoid using that as an excuse to move slice detection early.

- `RepositorySourceFloatingLocator` living in `resource_page_models.ts` still feels semantically sideways once `candidates.ts` imports it. I agree with the task’s decision to leave it there for now, but I’d keep a future cleanup note for a neutral source/artifact model module.

- `WeaveRequest` and `VersionRequest` are structurally identical right now. Do not DRY them into one alias in this slice; they are separate operation contracts even if the current fields match.

- `PlanWeaveInput.request: VersionRequest` may look odd to a future implementer, since `WeaveRequest` also exists. I’d add a one-line note saying this is existing planner API shape and not part of this cleanup.

I did not edit the task note; this was a review pass only.

# User_2026-05-22_1255_26

As you go, if you encounter any opportunities for performance optimization or find any bugs, record them under "Orthogonal Opportunies". For WeaveSlice and RepositorySourceFloatingLocator, feel free to change the task ordering or move things around how you see fit. Agreed on other points. Refine our task and let me know if you are ready to begin implementation.

# gpt-5.5_2026-05-22_1255_40

I’ll fold the review into the task note itself now: concrete module dependency shape, runtime direct-import expectations, a place for orthogonal findings, and clearer treatment of `WeaveSlice` / `RepositorySourceFloatingLocator`.

There is already a modified `documentation/notes/wd.codebase-overview.md` in the worktree that I did not touch, so I’ll leave that alone and keep this patch scoped to the archive task note. I’m choosing to pull `RepositorySourceFloatingLocator` into a neutral source model module in this slice; that removes a small semantic wrinkle before it calcifies.

I’ve made the first pass. I’m doing a quick read-through now for stale wording because the task went from three modules to five, and I don’t want the note quietly contradicting itself.

Refined the task note here: [wa.task.2026.2026-05-21_1037-core-weave-first-extraction-slice.md](wa.task.2026.2026-05-21_1037-core-weave-first-extraction-slice.md:33).

Key changes:
- Added `source_models.ts` for `RepositorySourceFloatingLocator`.
- Split `WeaveSlice` into a tiny `slices.ts` instead of `planning_models.ts`.
- Added the expected dependency shape and import-graph acceptance checks.
- Tightened runtime direct-import guidance for moved type names.
- Filled in `Orthogonal Opportunities` as the place to record bugs/perf findings without widening this slice.
- Updated the implementation plan to match the revised ordering.

I did not run tests since this was documentation-only. I also left existing unrelated worktree changes alone, including `documentation/notes/wd.codebase-overview.md` and other archive-note changes.

I’m ready to begin implementation.

# User_2026-05-22_1259_48

begin