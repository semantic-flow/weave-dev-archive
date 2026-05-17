---
id: weave-all-terms-extracti-rmme1u
title: 'weave all-terms extraction creates references'
desc: ''
created: 1779032222615
updated: 1779032222615
participants: [djradon, codex.gpt-5.5]
conversationEventKinds: [message.assistant, message.user, tool.call, tool.result]
---

# djradon_2026-05-16_1802_24

weave all-terms extraction creates references

Let's implement [wd.task.2026.2026-05-16_1748-add-source-references.md](../../../../../documentation/notes/wd.task.2026.2026-05-16_1748-add-source-references.md) 

Regarding open issues:
- only for terms newly extracted
- I think pinned reference mode is good. In general references should pin, so for consistency, yes. I could be convinced otherwise if you disagree.

# gpt-5.5_2026-05-16_1802_29

I’ll read the product vision, developer guidance, and the task note first, then trace the extraction path before touching code. The two decisions you gave are enough to move without a clarification loop.

I’ve got the guidance and task note. Your “newly extracted only” choice narrows the behavior nicely; I’m going to update the task decisions as part of the work, and I’ll treat pinned reference state as the intended consistency direction unless the existing reference model makes that awkward.

I found the core shape: all-terms extraction builds new Knop inventories in memory, and `knop add-reference` already knows how to render the ReferenceCatalog plus inventory registration. The important implementation wrinkle is that references for new terms must update those just-created inventory files in the same staged mutation, rather than treating them like pre-existing files.

I’m going to make three small code edits: extend the reference planner so it can render an optional `referenceTargetState`, teach all-terms extraction to attach those reference plans to newly created terms, and expose the new CLI flags with validation that `--reference-role` only matters when `--add-source-references` is present.

The main implementation is now in place. I’m going to format and run the focused tests next; I expect the first pass may flush out a couple of type-shape nits because the result object grew a few explicit reference-reporting fields.

The first test command was my bad: these fixture tests shell out to `git`, so Deno needs `--allow-run=git,deno` and the test tmp preload. I’m rerunning with the same permissions the repo’s `deno task test` uses.

Focused coverage is green now: 30 tests passed across the reference planner, runtime extraction, and CLI extraction path. I’m moving to repo-wide `check` and `lint`, which should catch the wider TypeScript and style surface.

Implemented `extract --all-terms --add-source-references --reference-role <role>`.

What changed:
- Added all-terms source-reference request/result fields in [extract.ts](../../../../../src/runtime/extract/extract.ts:109).
- Reused the existing `knop add-reference` planner, including optional `sflo:referenceTargetState` support in [add_reference.ts](../../../../../src/core/knop/add_reference.ts:63).
- Threaded CLI flags and validation through [run.ts](../../../../../src/cli/run.ts:378).
- Added runtime and CLI coverage for creation-scoped references, rerun idempotence, and pinned `--source-state` references.
- Updated task note, CLI reference, codebase overview, and decision log.

Behavior note: I kept `--source` references broad/current, and only add `sflo:referenceTargetState` when the user explicitly uses `--source-state`. That matches the source-resolution contract without silently pinning a current-mode extraction to whichever state happened to be current.

Validation passed:
- focused extract/reference/e2e tests
- `deno task fmt:check`
- `deno task check`
- `deno task lint`

Suggested commit message:

```text
feat(extract): create source references during all-terms extraction

- add opt-in --add-source-references and --reference-role support for extract --all-terms
- create ReferenceCatalog/ReferenceLink artifacts only for newly extracted terms
- pin source references with sflo:referenceTargetState when extraction uses --source-state
- report source-reference paths in runtime results and clarify CLI summary output
- cover runtime idempotence, pinned references, reference planner rendering, and CLI behavior
- update CLI docs, task note, codebase overview, and decision log
```

I left the unrelated untracked `documentation/notes/wd.task.2026.2026-05-16-deploy-profile.md` alone.
