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

# User_2026-05-27_1904_00

We're basically finished [wa.task.2026.2026-05-24_1748-shared-artifact-resolution-runtime-service.md](wa.task.2026.2026-05-24_1748-shared-artifact-resolution-runtime-service.md) ... ended up doing resolver cleanup.

I don't understand what's left in [wa.task.2026.2026-05-27_1246-config-source-discovery-and-resolution.md](wa.task.2026.2026-05-27_1246-config-source-discovery-and-resolution.md) . "Complete config-source discovery and resolution" Do we need to revisit that?

# gpt-5.5_2026-05-27_1904_16

I’ll sanity-check the task note against the current code before answering. My hunch is that the todo wording may now be stale if `config_sources.ts` is already wired into effective config loading, but I want to verify the actual hooks and remaining unchecked items.

# gpt-5.5_2026-05-27_1904_25

There are already local changes from the current work, including the shared resolver note and the config-source note, so I’ll treat those as your work and only read them. I’m checking whether the config-source checklist still maps to actual missing code, or whether it’s now just stale backlog language.

# gpt-5.5_2026-05-27_1905_02

The current code says the mesh-local config-source slice is basically there: discovery is wired into `loadWeaveEffectiveConfig`, it resolves through the shared artifact resolver, and there are unit plus integration tests for the main behavior. The remaining question is mostly backlog wording and whether Knop-level config-source discovery is a separate future task.

# gpt-5.5_2026-05-27_1905_29

Short answer: yes, revisit it, but mostly to **close/narrow it**, not to do another implementation push.

What’s already done:

- `loadWeaveEffectiveConfig` now discovers mesh-local config sources and feeds resolved inputs into compilation: [effective_config.ts](../../../../../src/runtime/config/effective_config.ts:986)
- `discoverMeshLocalConfigSources` resolves attachments recursively through the shared artifact resolver, records trace entries, detects cycles, and compiles returned Turtle as config input: [config_sources.ts](../../../../../src/runtime/config/config_sources.ts:76)
- tests cover mesh-local source policy, chaining, metadata discovery, unsupported subjects, unsupported inheritable sources, cycles, digest mismatch, remote denial, and path denial: [effective_config_test.ts](../../../../../src/runtime/config/effective_config_test.ts:302)
- integration coverage proves existing runtime commands honor a policy supplied through `sfcfg:hasConfigSource`: [validate_version_generate_test.ts](../../../../../tests/integration/validate_version_generate_test.ts:730)

So the `wd.todo` item “Complete config-source discovery and resolution” is now stale. The task note’s implementation checklist is basically all checked except:

- pre-implementation verification, which is awkwardly still unchecked after implementation: [task note](wa.task.2026.2026-05-27_1246-config-source-discovery-and-resolution.md:157)
- CLI/user docs, intentionally delayed until the authoring pattern is stable: [task note](wa.task.2026.2026-05-27_1246-config-source-discovery-and-resolution.md:169)

The real residual thing is **not** “complete config-source discovery” generally. It’s either:

```md
- [ ] Close/narrow [[wa.task.2026.2026-05-27_1246-config-source-discovery-and-resolution]]: mesh-local config-source discovery is implemented; decide whether to document the authoring pattern now or defer docs, and spin Knop-local/inheritable config-source discovery into a later task if still desired.
```

Also, the task note’s Testing section still talks about the same reusable source participating through mesh, Knop-local, and Knop-inheritable attachments, but the Decisions/Non-Goals explicitly leave Knop discovery out of the first slice. Tiny stale-note smell there. I’d trim that into a future task rather than keep it as part of this one.

# User_2026-05-27_1910_29

OK, so we need a new task for knop config inheritance?

# gpt-5.5_2026-05-27_1910_50

Yes, but I’d name/scope it carefully:

**Knop Config-Source Discovery And Inheritance**, not just “Knop config inheritance.”

Reason: the current task intentionally landed the mesh-local slice and rejects Knop attachment subjects for now. What’s missing is the next runtime layer:

- discover `sflo:Knop sfcfg:hasConfigSource ?source` as Knop-local config
- discover `sflo:Knop sfcfg:hasInheritableConfigSource ?source` as descendant-default config
- resolve those sources through the shared artifact resolver
- project inherited config down the Knop tree deterministically
- fail closed on cycles, unsupported source coordinates, ambiguous attachment roles, and inheritance conflicts
- prove precedence against application defaults, mesh-local config, command overrides, and existing inline Knop config if any

I’d keep it separate from the now-finished mesh-local config-source task. That makes the current task closeable, and the new one can focus on ancestry/projection semantics without muddying the resolver cleanup.

# User_2026-05-27_1913_53

And the "first mover" "sflo:Knop sfcfg:hasConfigSource ?source" will be in metadata right?

How are we going to project inherited config, like there will be an explicit mapping for every knop? I think we need Oxigraph.

# gpt-5.5_2026-05-27_1914_09

I’m going to peek at the existing inheritance helper before answering, because there may already be a small projection model hiding in plain sight. Then I’ll separate “where authored config-source attachments live” from “what the resolver projects at runtime.”

# gpt-5.5_2026-05-27_1914_42

Yes, I think the first-mover attachment belongs in **current Knop metadata**, not inventory and not `_sources`.

Something like:

```turtle
<alice/_knop> a sflo:Knop ;
  sfcfg:hasConfigSource [
    a sfcfg:ConfigSource ;
    sflo:targetLocalRelativePath "alice/_knop/_config/config.ttl"
  ] ;
  sfcfg:hasInheritableConfigSource [
    a sfcfg:ConfigSource ;
    sflo:targetLocalRelativePath "alice/_knop/_config/descendant-defaults.ttl"
  ] .
```

Why metadata:

- `_knop/_meta/meta.ttl` is the natural small current support graph for facts about the Knop itself.
- `_knop/_inventory` should stay about settled membership/resource structure, not config attachment control.
- `_knop/_sources` is provenance/source bindings, not operational configuration.
- `_knop/_config/config.ttl` can be the target config bytes, but it should not be required to bootstrap its own discovery.

On projection: I don’t think we should persist an explicit mapping for every Knop. The projection should be runtime-derived, with trace output if needed.

Shape:

1. For a target Knop, build its ancestor scope path: root, `alice`, `alice/data`, etc.
2. Read config attachments from each scope’s current metadata/config seed.
3. Resolve `hasConfigSource` as local config for that exact scope.
4. Resolve `hasInheritableConfigSource` as an outbound offer to descendants.
5. Project inherited offers down the path using the existing helper in [inheritance.ts](../../../../../src/runtime/config/inheritance.ts:25).
6. Compile the effective config from defaults, mesh config, inherited Knop config, local Knop config, command overrides.

I’m with you that Oxigraph is attractive here, but I’d push back on making it a prerequisite. Oxigraph would help query named graphs, ancestry, attachment subjects, and provenance. It will not by itself solve layer ordering, source-order tie breaks, inheritance stop/block policy, or fail-closed conflict behavior. Those semantics still need explicit TypeScript.

So my recommendation: new task should implement Knop config inheritance with a small in-memory scope/attachment index first. Add an Oxigraph spike only if the graph collection/query code starts getting gross or if we need cross-mesh/cache-invalidation queries.