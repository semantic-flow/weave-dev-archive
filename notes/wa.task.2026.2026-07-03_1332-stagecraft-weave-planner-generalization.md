---
id: stagecraft-weave-planner-generalization-20260703
title: 2026 07 03 1332 Stagecraft Weave Planner Generalization
desc: ''
created: 1783110720000
---

## Goals

- Turn the Stagecraft fixture-ladder failure around "The current local weave slice only supports the settled second payload weave shape..." into a focused Weave epic.
- Remove fixture-shaped payload-weave blockers that prevent ordinary application data ladders from advancing through full `weave`.
- Keep `weave generate` in its proper role: rendering ResourcePages from already-settled mesh state, not papering over a planner that cannot create or advance the needed histories.
- Replace "current local slice only supports..." diagnostics with condition-specific failures that tell an operator which RDF fact, inventory progression, target selection, or source resolution is invalid.
- Use Stagecraft as the first application-shaped pressure test for payload-history progression, source provenance, generated inspection pages, and conformance-ladder acceptance.
- Avoid a broad speculative rewrite. Generalize the seams Stagecraft and existing fixture ladders prove are real, then consolidate helpers only where repetition is visible.

## Summary

Stagecraft is exposing the same weakness that earlier Alice, Fantasy Rules, SFLO, and URPX work exposed in smaller pieces: parts of the Weave planner still validate exact carried fixture shapes instead of deriving valid progression from RDF facts. The current reported failure names the second-payload weave shape, but the broader smell is the same family as the first-payload and extracted-term blockers tracked in [[wa.task.2026.2026-05-04-refactor-planFirstPayloadWeave]] and [[wa.task.2026.2026-04-13_0910-weave-shape-generalization-for-later-carried-states]].

The right response is not to tell Stagecraft to run `weave generate` unless the target checkout already has the payload histories and ResourcePage claims it needs. If the rung is expected to create or advance payload histories, update current/latest state, and then render `index.html` pages, the correct operation is full `weave`. A failure in that path means Weave's planner is too narrow for the model it claims to support.

This epic should produce an implementation sequence that starts from a reproducible Stagecraft ladder case, adds a failing Weave-side test, and then generalizes payload weave planning from "does the current Turtle match the one accepted fixture rung?" to "can the current RDF state support the requested transition coherently?" The desired endpoint is boring: Stagecraft-shaped payload data can be integrated, versioned, and inspected without forcing hand-authored rung-specific workarounds.

## Discussion

The current code still contains several assertions whose error messages describe local implementation history rather than semantic validity. The Stagecraft failure most likely comes from `assertCurrentKnopInventoryShapeForSecondPayloadWeave`, which expects a settled second-payload pattern for one designator path. That made sense when the code was being carried one fixture rung at a time; it is not a durable rule for application data.

There are two separate questions that should not be blurred:

- Is the current mesh state semantically and operationally valid enough to advance?
- Does it happen to have the exact subject blocks, history names, next ordinals, and page links from an earlier fixture rung?

Only the first question should block `weave`. The second question is useful for old golden fixtures but should not be the planner's acceptance criterion.

Stagecraft also changes the prioritization. Ontology publication ladders can sometimes be rerun with staged rituals because they are relatively controlled. Application persistence will run into ordinary sequences: integrate one payload, update it, add another payload, update both, keep some support artifacts current-only, and expect inspection pages to follow. If those workflows have to know about "first payload" or "second payload" implementation slices, the abstraction has leaked.

The existing backlog already points in the right direction:

- [[wa.task.2026.2026-05-04-refactor-planFirstPayloadWeave]] tracks first-payload and extracted-term blockers.
- [[wa.task.2026.2026-04-13_0910-weave-shape-generalization-for-later-carried-states]] tracks later carried-state shape generalization.
- [[wa.task.2026.2026-05-17-append-onlyish-inventory]] describes the inventory behavior Weave probably needs once it stops rewriting exact fixture shapes.
- [[wa.task.2026.2026-06-30_1108-stagecraft-driven-semantic-flow-requirements]] makes Stagecraft the concrete consumer pressure.

This note ties those together around a user-visible blocker. It should not delete or rename the older tasks. It should act as the epic that orders the slices and gives the first implementation pass a narrow repro.

## Stagecraft Reproducer Captured

First concrete case, from the Stagecraft temporal vocabulary rung:

- fixture repo: `/home/djradon/hub/spectacular-voyage/stagecraft/dependencies/github.com/spectacular-voyage/test-inn-ambush`
- records side: `a.11-temporal-vocabulary-records` after `230a30ee0820`
- intended woven side: `a.12-temporal-vocabulary-woven` at `3899d326c10d`
- changed designators:
  - `projections/contracts/inn-ambush-contract-context`, currently `_history001/_s0003`, desired `_s0004`, with `sflo:nextStateOrdinal "4"`
  - `projections/contracts/inn-ambush-contract-shapes`, currently `_history001/_s0003`, desired `_s0004`, with `sflo:nextStateOrdinal "4"`
  - `world/states/inn-ambush-plan-b-state`, currently `_history001/_s0002`, desired `_s0003`, with `sflo:nextStateOrdinal "3"`

Observed behavior:

- Direct `weave version --target 'designatorPath=...,stateSegment=_s0004'` rejected the targets as not currently weaveable.
- `weave set next-state ...` followed by `weave version` or `weave --validate-after` reached `The current local weave slice only supports the settled second payload weave shape for projections/contracts/inn-ambush-contract-context`.
- Passing `--history-tracking-policy currentOnly` did flow through the diagnostic config as `payload currentOnly`, `knopInventory currentOnly`, and `meshInventory currentOnly`; the later assertion still required the old second-payload fixture shape.
- `weave generate` could render pages only after inventories were manually made coherent. That confirms the current note's boundary: `generate` is not the operation that creates or advances payload histories.

The important diagnosis is narrower than "Stagecraft needs a special lane." This is later-ordinal payload advancement in an already woven payload whose support artifacts are intentionally current-only. The support artifacts do not have their own support histories, so `detectPendingWeaveSlice` does not classify the slice as a later payload weave unless hints are supplied. Once hinted, `assertCurrentKnopInventoryShapeForSecondPayloadWeave` still requires `sflo:nextStateOrdinal "2"` and the exact carried second-payload shape.

The first Weave fix should therefore support appending `_sNNNN` from the current `sflo:latestHistoricalState` and `sflo:nextStateOrdinal` for any coherent ordinal, not only the second payload state. It should also support this Stagecraft-style multi-target operation when the selected payloads are coherent and independent. Diagnostics should name the missing or conflicting RDF fact, current-only policy mismatch, or target-selection issue instead of falling through to the settled-shape message.

## Proposed Task Breakdown

### Task 1: Capture The Stagecraft Reproducer

- [ ] Record the exact Stagecraft repo, branch, fixture repo path, source ref, target ref, command, and failing designator path.
- [ ] Preserve the failing checkout or add a small Stagecraft-shaped Weave fixture that reproduces the same second-payload weave rejection.
- [ ] Identify whether the rung needs full `weave` or only `weave generate`; if it needs new or advanced histories, keep full `weave` as the expected operation.
- [ ] Write a failing test in Weave before changing planner behavior.

### Task 2: Inventory Fixture-Shaped Payload Gates

- [ ] List every payload-weave path that still throws "current local weave slice only supports..." for a settled shape.
- [ ] Classify each gate as a true invariant, a malformed-state diagnostic, or fixture-shaped implementation debt.
- [ ] For the Stagecraft blocker, write down the exact facts that should be required: payload artifact type, current artifact history, latest historical state, KnopInventory relationship, MeshInventory progression, ResourcePage eligibility, and working-source resolution.
- [ ] Replace one broad fixture-shaped assertion with a smaller read model that reports missing or conflicting facts.

### Task 3: Generalize Later Payload Weave Progression

- [ ] Generalize the second-payload weave path so it derives existing payload history, latest state, next state naming, support-artifact policy, and KnopInventory progression from RDF rather than a carried Turtle shape.
- [ ] Support later payload revisions in meshes that have additional unrelated Knops, support artifacts, source registries, or generated pages.
- [ ] Ensure target selection can weave the requested payload without requiring unrelated pending candidates to match the same slice.
- [ ] Preserve current successful Alice/Fantasy fixture behavior except for intentional, semantically equivalent output normalization.

### Task 4: Unify Payload-Weave Diagnostics And Candidate Selection

- [ ] Replace "settled first/second payload weave shape" messages with condition-specific diagnostics.
- [ ] Revisit the one-candidate limit in `planWeave` only where a concrete Stagecraft or existing ladder case needs multi-target payload advancement.
- [ ] Keep exact targets narrow: `weave --target designatorPath=x` should not silently advance unrelated pending payloads.
- [ ] Keep untargeted behavior deterministic if multiple selected candidates become supported.

### Task 5: Align Inventory Progression With Append-Onlyish Behavior

- [ ] Decide how much of [[wa.task.2026.2026-05-17-append-onlyish-inventory]] must land before the Stagecraft blocker can be fixed cleanly.
- [ ] Prefer appending/no-oping settled inventory facts and failing on conflicts over regenerating exact subject blocks.
- [ ] Keep current/latest/next progression explicit and auditable.
- [ ] Avoid inventing a Stagecraft-only inventory path.

### Task 6: Add Acceptance Coverage

- [ ] Add or update an Accord manifest that proves the Stagecraft-shaped transition adds or updates the expected paths.
- [ ] Pair new RDF paths with `hasAskAssertion` checks for the important semantic facts rather than relying only on byte comparison.
- [ ] Keep `conformance/` off rung refs unless the fixture explicitly needs to be self-contained.
- [ ] Add Weave integration coverage that exercises the same transition locally.

### Task 7: Document The Operational Rule

- [ ] Update developer docs if the implementation changes the boundary between `weave`, `weave version`, and `weave generate`.
- [ ] Update user docs only if the externally visible CLI behavior or recommended Stagecraft workflow changes.
- [ ] Add a decision-log entry if this generalization changes the planner contract beyond removing a bug.

## Open Issues

- For the first slice, the triggering rung and designators are captured above. Are there smaller Weave-native fixtures that reproduce the same current-only support plus later-ordinal payload shape without carrying the full Stagecraft fixture repo?
- For the first slice, the failing rung attempts a multi-target payload advancement, with generated pages as a consequence. Should multi-target selected payload advancement land in the same fix as the later-ordinal read model, or be split immediately after the single-target fix?
- For the first slice, support artifacts are intentionally current-only while payload histories advance. Are there any existing callers that assume support histories must exist before later payload advancement is permitted?
- Stagecraft needs byte-stable existing `_history*/_s*/...` payload states immediately under its new immutability convention. Should Weave grow a generic regression helper that compares old historical payload files across refs and allows only newly added states plus inventory/page progression?
- Should the first implementation slice generalize only second-payload weave, or should it also remove the one-candidate planner limit if the repro has multiple selected candidates?
- Is the append-onlyish inventory task a prerequisite for the clean fix, or can the Stagecraft blocker land as a narrower read-model change first?

## Decisions

- `weave generate` is not a substitute for full `weave` when the expected rung needs payload histories or other versioned support state to be created or advanced.
- The exact settled second-payload fixture shape is not a model invariant.
- Stagecraft should drive a concrete generalization slice, not a Stagecraft-specific workaround.
- The implementation should start from a failing reproducible test and widen the planner only as far as that test and existing fixture behavior justify.
- Failure messages should describe the invalid state, not the implementation slice that failed to recognize it.

## Contract Changes

- A payload that already has a current history should be weaveable again when its working artifact needs a new state and the current RDF facts are coherent, even if the surrounding mesh has grown beyond the original carried fixture shape.
- `weave` should reject malformed or ambiguous current state with specific errors: missing current history, missing latest state, conflicting latest states, unsupported history policy, missing working file, invalid target selection, or impossible inventory progression.
- Existing `weave generate` behavior remains page-rendering-only and should not create new payload histories.
- Existing single-payload fixture ladders remain valid.

## Testing

- Add a Stagecraft-shaped failing test before implementation.
- Add core planner tests for later payload weave against a mesh state that has extra unrelated Knops/support artifacts compared with the old Alice fixture.
- Add coverage for a later-ordinal payload whose support artifacts are current-only and have no support-history pages of their own.
- Add coverage for a selected multi-target payload operation where all targets are coherent and independent.
- Add a regression that proves prior `_history*/_s*/...` payload files are byte-identical after the operation and only expected new state paths, inventory facts, and pages are added.
- Add runtime/integration coverage for the full operation: prepare checkout, run `weave`, verify created/updated histories, verify ResourcePage links, and verify generated `index.html` paths.
- Add diagnostic tests for at least one missing-fact and one conflicting-fact failure that previously collapsed into the settled-shape error.
- Run `deno task test` for the implementation slice and `deno task lint` after significant code changes.
- If Accord manifests are updated, run the relevant Accord transition checks against the fixture repo.

## Non-Goals

- Do not rewrite all of `src/core/weave` into a generic transaction engine in one pass.
- Do not add Stagecraft-specific vocabulary or runtime branches unless the generic model truly cannot express the workflow.
- Do not move `conformance/` files onto fixture rung refs as part of this planner fix.
- Do not rename existing `wa.task.*` notes or completed notes.
- Do not preserve fixture-shaped errors just because they are familiar; they are useful clues, not acceptable operator-facing diagnostics.

## Troubleshooting Inputs Needed For The First Slice

- The exact Stagecraft command that fails.
- The fixture repo path and current checked-out branch/ref when it fails.
- The intended source and target rung refs.
- The full error text including the designator path.
- `git status --short --branch` for the Stagecraft repo and fixture repo at failure time.
- The relevant current KnopInventory, MeshInventory, and payload history Turtle files for the failing designator path, or a tarball/branch that lets Weave reproduce them.
- Whether the intended rung should create a new payload state, only refresh pages, or both.

## Implementation Plan

- [ ] Add the Stagecraft repro details to this note.
- [ ] Create a focused failing Weave test that reproduces the settled second-payload weave rejection without requiring the full Stagecraft app test suite.
- [ ] Trace the failing planner path from candidate discovery through slice classification and the shape assertion.
- [ ] Replace the Stagecraft-blocking fixture assertion with RDF fact validation and condition-specific diagnostics.
- [ ] Preserve existing carried fixture tests.
- [ ] Add integration/Accord coverage for the accepted Stagecraft-shaped transition.
- [ ] Decide whether the next slice should tackle multi-candidate planning, append-onlyish inventory writes, or another shape-specific assertion exposed by the same ladder.
- [ ] Update [[wd.todo]], [[wd.decision-log]], and [[wd.codebase-overview]] when the implementation lands if the public or developer contract changes.
