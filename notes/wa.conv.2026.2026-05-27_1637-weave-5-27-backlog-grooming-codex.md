---
id: weave-527-backlog-groomi-3db3x3
title: 'weave: 5/27 backlog grooming'
desc: ''
created: 1779925040101
updated: 1779925040101
participants: [codex.gpt-5.5]
conversationEventKinds: [message.assistant, message.user, tool.call, tool.result]
---

# User_2026-05-27_1636_41

weave: 5/27 backlog grooming

Audit all `wa.task.*` notes in `weave-dev-archive/notes` and update `weave/documentation/notes/wd.todo.md` as the consolidated current backlog.

For each task note:
- decide whether it appears completed, obsolete/superseded, still active, or needs revision
- do not rename task notes to completed unless I explicitly approve
- update stale task notes only when the correction is clear
- capture the resulting prioritized backlog in `wd.todo.md`
- call out task notes that need human decision before editing

Note that [[wa.task.2026.2026-05-24_1748-shared-artifact-resolution-runtime-service]] is currently in progress, as we speak.

# gpt-5.5_2026-05-27_1636_53

I’ll treat this as a documentation/backlog pass rather than a code change. First I’m going to read the repo guidance and product vision, then inventory the task notes so I can separate clear edits from human-decision cases.

There are 36 `wa.task.*` notes in the archive path. I’m reading the required guidance and current backlog now so the grooming lands in the repo’s own vocabulary instead of inventing a new taxonomy.

# gpt-5.5_2026-05-27_1637_29

The task notes mix long planning notes with checklist state, and a few already have all items checked/deferred. I’m going to summarize each note by title and checkbox status, then spot-check code/docs where the note’s state looks stale.

# gpt-5.5_2026-05-27_1639_10

A few notes look plainly complete or deferred by newer notes, but several old ones are vague enough that I don’t want to “correct” them by guessing. I’m checking current code/docs for the likely stale cases: config loading, temp dirs, markdown rendering, artifact resolution, and fixture rungs.

# gpt-5.5_2026-05-27_1640_52

One correction is unambiguous: the Carol fixture branches exist locally (`a.26-carol` and `a.27-carol-woven`), while the Carol task note still has those replay/commit items unchecked. I’m marking just those two items; broader stale-looking notes need more judgment, so I’ll call them out instead of rewriting history.

# gpt-5.5_2026-05-27_1641_34

I’m replacing the old flat todo list with a groomed backlog plus an audit index. I’m keeping actionable items as checkboxes, using wikilinks for task notes, and separating “human decision before task-note edits” from the actual implementation backlog.

# gpt-5.5_2026-05-27_1643_19

The backlog file now covers every task note, and I’ve only edited the one task note whose stale checklist was mechanically verifiable. I’m doing a quick diff/whitespace pass next, then I’ll give you the human-decision list and suggested commit messages.

# gpt-5.5_2026-05-27_1643_58

Done. I updated [wd.todo.md](../../../../../documentation/notes/wd.todo.md) into a groomed backlog with P0-P3 priorities, a human-decision section, and an audit index covering all 36 `wa.task.*` notes.

I also made the one clear task-note correction: [wa.task.2026.2026-05-25_0849-carol.md](wa.task.2026.2026-05-25_0849-carol.md) now marks the `a.26-carol` and `a.27-carol-woven` replay/commit items complete. I did not rename any task notes.

Human-decision items are called out in `wd.todo.md`, especially `honor-mesh-config`, `ReferenceLink clarification`, the legacy ResourcePage umbrella, renderer/autolinking/Markdown-it overlap, fingerprint verification, Oxigraph, and configurable test temp.

Verification: `git diff --check` passed in both touched repos. I did not run Deno lint/tests because this was documentation-only. Existing in-progress runtime config-source changes were left untouched.

Suggested commit messages:

```text
docs: consolidate current Weave backlog

- audit archived wa.task notes into wd.todo.md
- prioritize current config/source-resolution, publication/runtime correctness, fixture/renderer, and parked future work
- call out task notes that need human decisions before editing
```

```text
docs: mark Carol fixture replay complete

- mark a.26-carol and a.27-carol-woven replay items complete
- keep the task note in place without renaming it
```