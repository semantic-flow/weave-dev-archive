---
id: weave-core-candidate-mod-rpqyxl
title: weave core candidate model extraction
desc: ''
updated: 1779479956445
created: 1779479956445
participants:
  - codex.gpt-5.5
conversationEventKinds:
  - message.assistant
  - message.user
  - tool.call
  - tool.result
---

# User_2026-05-22_1250_14

weave core candidate model extraction

Please review our next task: [[wa.completed.2026.2026-05-21_1037-core-weave-first-extraction-slice]] and suggest any impovements, and surface any open issues.

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

# gpt-5.5_2026-05-22_1300_02

I’m starting the implementation slice now. First I’ll take the promised baseline: re-read the task guidance, capture the current export/line-count shape, run the import graph audit, then run the pre-edit checks before touching source files.

# gpt-5.5_2026-05-22_1300_21

Baseline docs/export capture is done; the exported type cluster is unchanged and `weave.ts` is still 8,597 lines. I’m running the graph audit and pre-edit checks next, which may take a bit.

# gpt-5.5_2026-05-22_1300_34

The pre-edit baseline is clean: `deno task check` passed, the focused core weave test passed, and the graph audit found 13 local `src` modules, 21 local edges, 0 core-to-runtime edges, and 0 cycles from the current root. I’m moving the type definitions now, keeping the runtime behavior imports in place.

# gpt-5.5_2026-05-22_1301_53

The first extraction compiles. I’m doing one more pass over imports now: a few runtime modules still import policy or behavior from the façade, which is fine, but moved type-only names should use the new focused modules.

# gpt-5.5_2026-05-22_1302_10

The new modules are in place and `deno task check` passed after the move. I’m formatting and then I’ll run the post-change graph audit plus the focused core and integration tests.

# gpt-5.5_2026-05-22_1302_27

Lint found one leftover type import in `weave.ts` that is only re-exported now, not used by implementation. I’m cleaning that up and then I’ll rerun the full validation set.

# gpt-5.5_2026-05-22_1302_47

Formatting, lint, and type-check are clean now. I’m running the post-change graph audit and the focused test set from the task note.

# gpt-5.5_2026-05-22_1303_09

All focused tests passed, and the post-change graph audit still shows 0 core-to-runtime edges and 0 local cycles. I’m checking the docs/diff now so I can update the task note accurately and avoid trampling the existing `wd.codebase-overview.md` edits.

# User_2026-05-22_1357_18

can I move it to completed? What's next?

# gpt-5.5_2026-05-22_1357_37

Yes. This slice is complete and safe to move to `completed` if you want. It met the task criteria: models extracted, façade re-exports preserved, runtime type imports cleaned up, graph audit clean, and `fmt` / `lint` / `check` / focused tests all passed. No orthogonal bugs or perf opportunities were found.

Next, I’d do the parent task’s next low-risk slice: extract pure RDF query/assertion helpers and Turtle block helpers out of `src/core/weave/weave.ts`. I would not jump straight into `detectPendingWeaveSlice` yet; it will be cleaner after the parser/query/block helpers have homes.

Suggested next task title:

`Core Weave RDF And Turtle Helper Extraction`

Scope:
- move pure RDF parse/query/fact helpers
- move pure Turtle block editing helpers
- preserve `weave.ts` planner behavior
- no generated RDF/Page changes
- run the same graph audit plus core/integration tests

After that, slice classification extraction should be much easier.

# User_2026-05-22_1359_53

OK, fill out wa.task.2026.2026-05-22_1358-core-weave-rdf-and-turtle-helper-extraction and update the parent task.

# gpt-5.5_2026-05-22_1400_02

I’ll open the new task stub and the parent note, then patch only those archive notes. I’ll also check the archive status because it looks like you may have already renamed 1037 to completed.

# gpt-5.5_2026-05-22_1400_09

I’ve got the stub and parent. I’m going to base the new task on the actual helper clusters still in `weave.ts`, especially the pure `parseWeaveShapeQuads` / fact-resolution helpers and Turtle block editing helpers, and keep renderers/planner state out of scope.

# gpt-5.5_2026-05-22_1400_30

The helper inventory confirms the next slice should stay pretty leaf-level: RDF parsing/query/resolution helpers around `parseWeaveShapeQuads`, plus block helpers around `splitTurtleBlocks` / `replaceSubjectBlock` / `upsertSubjectBlockAfter`. I’m going to write the task with a deliberate “do not move shape assertions/renderers yet” boundary.

# gpt-5.5_2026-05-22_1401_00

The new task is filled in. I’m updating the parent now so its extraction order points to this task and reflects the post-1037 state rather than the earlier 8.7k-line snapshot.

# gpt-5.5_2026-05-22_1401_30

Filled out the new task note and updated the parent.

Updated:
- [wa.task.2026.2026-05-22_1358-core-weave-rdf-and-turtle-helper-extraction.md](wa.task.2026.2026-05-22_1358-core-weave-rdf-and-turtle-helper-extraction.md:1)
- [wa.task.2026.2026-05-21_0849_careful-extraction-refactor.md](wa.task.2026.2026-05-21_0849_careful-extraction-refactor.md:25)

The new task is scoped to two helper modules:
- `src/core/weave/rdf_helpers.ts`
- `src/core/weave/turtle_blocks.ts`

I also tightened the non-goals so this slice does not drift into slice classification, shape assertions, source-locator rendering, payload layout, or large renderers.

No tests run; this was documentation/task grooming only.

# User_2026-05-22_1402_50

tell me about core vs shared

# gpt-5.5_2026-05-22_1403_08

In this repo, I’d treat **core** as a layer, and **shared** as a reuse property.

**Core** means portable Semantic Flow semantics:
- no filesystem reads/writes
- no git/process/logging/runtime config
- no CLI assumptions
- no host-local recovery behavior
- inputs are already-loaded strings/typed requests/models
- outputs are plans, models, validation errors, RDF strings, planned files

So `src/core/**` is allowed to be reused by runtime, CLI, daemon, or future web/API code because it describes semantic behavior without knowing where bytes came from.

**Shared** just means “used by more than one place.” That is weaker. Something can be shared but not core.

Examples:
- `src/core/targeting.ts`: core and shared. It owns portable target semantics.
- `src/core/weave/requests.ts`: core and shared. Runtime/CLI can use these request contracts.
- `src/runtime/weave/artifact_loaders.ts`: shared inside runtime, but not core. It reads files and applies operational path policy.
- `src/runtime/weave/page_contexts.ts`: shared runtime page-generation machinery, but not core because it loads workspace state and tolerates runtime conditions.

The trap is creating a vague `shared/` bucket. I’d push against that. “Shared” often becomes a junk drawer unless the sharing boundary is named by domain and constraints. Better names are:
- `core/weave/rdf_helpers.ts` for portable RDF helpers
- `core/weave/turtle_blocks.ts` for portable Turtle block editing
- `runtime/weave/planning_context.ts` for runtime staged-read/cache behavior
- `runtime/weave/artifact_loaders.ts` for runtime file-backed artifact loading

For the current task, the helpers are **core** if they only parse/query RDF strings or edit Turtle text blocks. They are not core if they resolve local paths, inspect the filesystem, consult effective config, log, or depend on workspace policy.

So: core is the semantic, portable layer. Shared is just who happens to use it. Keep naming around the layer and domain, not the fact that multiple callers import it.

# User_2026-05-22_1404_06

OK, ready to resolve those open issues?

# gpt-5.5_2026-05-22_1404_15

Yes. I’ll resolve them in the task note now, keeping the answers conservative so the implementation slice stays leaf-helper-only and does not quietly absorb source-locator or renderer work.

# gpt-5.5_2026-05-22_1404_27

I’m resolving these as implementation boundaries, not just opinions: the task will now say exactly what moves, what stays, and why. In particular I’m going to keep source-locator helpers and SFLO-specific semantic shorthands out of this slice unless the move itself forces them.

# User_2026-05-22_1404_47

convservative is fine, just make your your ideas aren't lost.

# gpt-5.5_2026-05-22_1404_53

Good call. I’ll add a compact deferred-ideas section to the task note so the conservative implementation boundary doesn’t erase the next seams we noticed.

# gpt-5.5_2026-05-22_1405_05

Done. I added a **Deferred Follow-Up Ideas** section to the RDF/Turtle task note so the conservative boundary keeps the implementation clean without losing the next seams:

- source/artifact locator helper slice
- SFLO-specific semantic RDF convenience helper slice
- shape assertion slice
- slice-classification slice
- render helper slice
- narrowly scoped constants cleanup if duplication later justifies it

No tests run; note-only update.

# User_2026-05-22_1405_17

ok, implement here or in a new conversation?

# gpt-5.5_2026-05-22_1405_19

Implement here is fine. We still have the relevant context loaded: parent task, new RDF/Turtle task, the completed 1037 slice, and the current code shape.

Only reason to start a new conversation would be if you want a cleaner Kato record centered solely on implementation. Practically, I’m ready to begin here.

# User_2026-05-22_1410_06

Claude says "Open Issue Resolutions say this helper's dependency on SAFE_DESIGNATOR_SEGMENT_PATTERN "does not pull planner state." True, but the analysis should also confirm whether ../designator_segments.ts is already in weave.ts's import graph (no new dep) or whether rdf_helpers.ts would introduce a new transitive dependency path for callers who don't currently reach designator_segments.ts through rdf_helpers.ts."

# gpt-5.5_2026-05-22_1410_20

Good catch from Claude. I’ll add that dependency-shape note explicitly, because it matters for whether `rdf_helpers.ts` is truly generic RDF or “core weave RDF plus designator-aware resolution.”

# gpt-5.5_2026-05-22_1410_36

Claude is right. I updated the task note to make that dependency analysis explicit.

The note now says:
- `designator_segments.ts` is already in `weave.ts`’s direct import graph, so moving `resolveOptionalSegmentHint` does not add a new dependency to the `weave.ts` root graph.
- It does create a module-level dependency for future direct importers of `rdf_helpers.ts`.
- That’s acceptable for this internal core-weave helper slice, but if `rdf_helpers.ts` later becomes a broadly generic RDF utility, split out `resolveOptionalSegmentHint` or keep it near slice/payload naming code.

I also added that as a deferred follow-up idea so the distinction is not lost.

# User_2026-05-22_1410_42

ok, proceed

# gpt-5.5_2026-05-22_1410_53

I’m starting the RDF/Turtle helper extraction now. I’ll take the task’s baseline first, then make the move-only module edits and run the same focused verification set so we can trust that RDF bytes and planner behavior stayed put.

# gpt-5.5_2026-05-22_1411_30

Baseline is clean: `weave.ts` is 8,515 lines, graph audit is 18 local modules / 35 edges / 0 core-to-runtime / 0 cycles, `deno task check` passed, and the core weave tests passed. I’m moving the leaf helpers now, starting with exact copies into focused modules and then deleting the originals from `weave.ts`.

# gpt-5.5_2026-05-22_1414_23

The helper modules are in place and `deno task check` passed after the first move. The moved functions are only defined in `rdf_helpers.ts` and `turtle_blocks.ts` now; source-locator and SFLO semantic shorthands stayed in `weave.ts` as planned.

# gpt-5.5_2026-05-22_1414_43

Format, lint, and check are clean. I’m running the post-change graph audit plus the core and integration tests now.

# gpt-5.5_2026-05-22_1415_06

The post-change audit and tests are clean: graph still has 0 core-to-runtime edges and 0 cycles, core weave tests passed, and the focused integration tests passed. I’m updating the task/overview notes with the actual module layout, line counts, and verification results now.

# gpt-5.5_2026-05-22_1416_09

I’m going to finish the bookkeeping now: update the overview and task notes so Claude’s dependency point, the actual graph result, and the verification trail all live somewhere durable. Then I’ll do a quick status pass and give you the “ready / next” answer with commit-message text.

# gpt-5.5_2026-05-22_1416_20

The code side is already through the important checks. I’m patching the notes in place now so the task tells the same story as the implementation: what moved, what intentionally stayed put, and what the import graph did before and after.

# gpt-5.5_2026-05-22_1417_15

One tiny correction while I’m here: the parent task’s next steps should say the RDF query/resolution layer is done, not imply the shape assertion wrappers moved too. I’m checking the final diff/status for that sort of wording drift now.

# gpt-5.5_2026-05-22_1418_29

Implemented the RDF/Turtle helper extraction here.

Changed:
- Added [rdf_helpers.ts](../../../../../src/core/weave/rdf_helpers.ts) for RDF parsing/query/resolution helpers.
- Added [turtle_blocks.ts](../../../../../src/core/weave/turtle_blocks.ts) for Turtle subject-block editing helpers.
- Updated [weave.ts](../../../../../src/core/weave/weave.ts) to import those helpers while keeping planner behavior in place.
- Updated [wd.codebase-overview.md](../../../../../documentation/notes/wd.codebase-overview.md).
- Filled in the RDF/Turtle task and updated the parent task note.

Key result: `src/core/weave/weave.ts` went from 8,515 lines to 8,026 lines. Post-slice graph audit stayed clean: 0 `core -> runtime` edges and 0 local cycles. Claude’s dependency point is recorded: `rdf_helpers.ts` now imports `designator_segments.ts` via `resolveOptionalSegmentHint`; that does not add a new dependency to the `weave.ts` root graph because `designator_segments.ts` was already directly imported there, but future direct `rdf_helpers.ts` consumers will inherit it.

Verification passed:
- `deno task fmt`
- `deno task lint`
- `deno task check`
- core weave test
- focused validate/version/generate integration test
- focused weave integration test
- import graph/cycle audit
- `git diff --check` in both repos

No orthogonal bugs or performance opportunities surfaced during this slice.

Suggested root commit message:

```text
refactor(core-weave): extract RDF and Turtle helpers

- move core weave RDF parser/query/resolution helpers into rdf_helpers.ts
- move Turtle subject-block editing helpers into turtle_blocks.ts
- keep source-locator helpers, SFLO semantic shorthands, shape assertions, slice classification, and renderer behavior in weave.ts
- update the codebase overview with the extracted helper modules
- verify with fmt, lint, check, core weave tests, integration weave tests, and import-cycle audit
```

Suggested archive commit message:

```text
docs(tasks): record core weave RDF helper extraction

- update the RDF/Turtle helper extraction task with results and verification
- update the parent core weave refactor task with completed helper-slice status
```

I’m ready for the next implementation slice. My recommendation: slice classification next, centered on `detectPendingWeaveSlice`, `classifyWeaveSlice`, and related pending-slice policy checks.