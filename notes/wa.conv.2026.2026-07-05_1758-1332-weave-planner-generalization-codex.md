---
id: 1332-weave-planner-gener-d2gevt
title: '1332 weave planner generalization'
desc: ''
created: 1783299489283
updated: 1783299489283
participants: [djradon, codex.gpt-5.5]
conversationEventKinds: [message.assistant, message.user, tool.call, tool.result]
---

# djradon_2026-07-05_1757_36

1332 weave planner generalization 

Kim — new Weave task, first slice of the planner-generalization epic. Working repo: /home/djradon/hub/semantic-flow/weave. Read dependencies/github.com/semantic-flow/weave-dev-archive/notes/wa.task.2026.2026-07-03_1332-stagecraft-weave-planner-generalization.md fully before writing code — especially "Stagecraft Reproducer Captured", the Decisions, and Contract Changes.

The defect: the planner can weave a first and second payload state because those paths are carried fixture shapes — resolveFirstPayloadVersionLayout / resolveSecondPayloadVersionLayout are the only layout resolvers in src/core/weave/payload_version_layout.ts, ~13 renderer templates in payload_renderers.ts and knop_inventory_renderers.ts hardcode sflo:nextStateOrdinal "2", and shape_assertions.ts gates like assertCurrentKnopInventoryShapeForSecondPayloadWeave require the exact settled shape. Third and subsequent states have no code path and die with "settled second payload weave shape" errors.

Scope for this slice — single-target later-ordinal payload advancement:
1. Failing test first. Reproduce the rejection with a small Weave-native fixture shaped like the repro (later-ordinal payload, current _history001/_sNNNN, support artifacts intentionally current-only with no support histories), without carrying the full Stagecraft fixture repo. The captured repro (test-inn-ambush, a.11-temporal-vocabulary-records → a.12-temporal-vocabulary-woven, projections/contracts/inn-ambush-contract-context _s0003→_s0004) is the reference shape.
2. Trace candidate discovery through detectPendingWeaveSlice classification (note: current-only support artifacts prevent later-payload classification without hints — fix classification, don't require hints) to the shape assertion.
3. Replace the second-payload fixture gate on this path with a fact-driven read model: derive existing payload history, latest state, next _sNNNN naming, support-artifact policy, and KnopInventory/MeshInventory progression from the current RDF facts (sflo:latestHistoricalState, sflo:nextStateOrdinal) for any coherent ordinal. Generalize the layout resolver and the inventory rendering this path uses so ordinals are computed, not templated.
4. Condition-specific diagnostics: failures must name the missing or conflicting fact (missing current history, missing/conflicting latest state, unsupported history policy, missing working file, invalid target selection, impossible inventory progression) — never a "settled ... shape" message on this path.

Hard constraints: existing Alice/Fantasy fixture ladders stay green (intentional, semantically equivalent output normalization only); weave generate stays render-only and must not create or advance histories; no Stagecraft-specific vocabulary or branches; weave --target designatorPath=x must not silently advance unrelated pending payloads.

Deliberately out of this slice — record decisions in the epic note: multi-target advancement in one transaction (validate the rung via sequential single-target invocations; multi-target is the immediate next slice), append-onlyish inventory writes (land this as a narrower read-model change; record how much of wa.task.2026.2026-05-17-append-onlyish-inventory the clean fix actually needs), and the remaining first-payload/extracted-term gates (owned by wa.task.2026.2026-05-04-refactor-planFirstPayloadWeave).

Tests beyond the repro: later-ordinal advancement in a mesh with extra unrelated Knops/support artifacts; a regression proving prior _history*/_s*/... payload files stay byte-identical with only expected new state paths, inventory facts, and pages added; at least one missing-fact and one conflicting-fact diagnostic test that previously collapsed into the settled-shape error. Gate: deno task test and deno task lint. Tick the epic-note checkboxes you complete, record decisions there, and update wd.todo/wd.decision-log if the planner contract changes beyond removing the bug.

# gpt-5.5_2026-07-05_1757_44

Kim here. I’ll start by reading the repo guidance and the epic note end to end, then I’ll trace the current planner path before touching code.

I’ve got the key epic text in memory now. I also noticed `documentation/notes/wd.todo.md` is already modified before I start, so I’ll treat that as user-owned unless I need to make a surgical update there later.

The epic note is 183 lines, so the first read covered the full file. I’m going to review the local AGENTS file and ontology summary next, then I’ll map the planner code path around classification, layout resolution, renderers, and shape assertions.

The current code already has a generalized-looking renderer layer in places, but the second-payload entry point still imports the fixture resolver/assertion directly. I’m going to read those modules in chunks and then pin the smallest path to widen.
