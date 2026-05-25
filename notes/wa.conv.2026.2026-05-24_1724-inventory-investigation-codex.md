---
id: inventory-investigation-2tdk9n
title: 'inventory investigation'
desc: ''
created: 1779668655702
updated: 1779668655702
participants: [codex.gpt-5.5]
conversationEventKinds: [message.assistant, message.user, tool.call, tool.result]
---

# User_2026-05-24_1658_53

inventory investigation

I saw in some of the test fixture inventories the currentArtifactHistory properties. We'd tried to move to keeping inventory append-only, and "current" pointers live in meta. Can you confirm that we completed that shift?

# gpt-5.5_2026-05-24_1659_00

I’ll trace this from the repo guidance first, then search both the ontology and fixture inventories for `currentArtifactHistory` and related “current” pointers so we can separate schema vocabulary from actual inventory usage.

I’ve got the repo’s guidance in hand. Next I’m going broad on the vocabulary and fixtures: first exact `currentArtifactHistory`, then looser searches for inventory/meta current-pointer language.

The first exact search found a lot of code/tests still using the term, which is expected if the property still exists for payload artifacts. The interesting question is narrower: whether `_inventory/inventory.ttl` files still own their own current history, so I’m switching to file-path-qualified searches.

Good signal: the physical `*/_inventory/inventory.ttl` files I found have no `currentArtifactHistory`, `latestHistoricalState`, `nextHistoryOrdinal`, or `hasArtifactHistory` matches. I’m checking the renderer and guardrail code now, because generated strings and tests can lag behind fixture files or deliberately encode failure cases.

The source picture is more nuanced than the fixture-file scan: mesh-level guardrails explicitly reject stale MeshInventory progression in `_mesh/_inventory/inventory.ttl`, but some renderer functions still serialize support-artifact history facts inside Knop inventory documents. I’m reading the recent task notes and sampling actual generated files so we can tell whether that is intentional carry-over, dead legacy code, or an incomplete migration.

That sample answered part of it: `_mesh/_inventory` has been cleaned up, but `alice/bio/_knop/_inventory/inventory.ttl` still has `currentArtifactHistory` for the payload and Knop support artifacts. I’m counting how widespread that is across the checked-in fixture meshes and then I’ll give you the clean verdict.

One correction from the broader count: the fixtures still have current/progression facts even in current inventory files, not just historical snapshots. The most important distinction is “about the inventory artifact itself” versus “about other artifacts recorded in the inventory”; the former has partly moved to meta, the latter has not.

The counted fixture state is not fully shifted: among the three checked-in fixture meshes I scanned, 18 current `*/_inventory/inventory.ttl` files still contain `sflo:currentArtifactHistory`. I’m checking the task/decision notes now so I can separate “unfinished TODO” from “we changed our mind.”

The task note says this is still open work, not a completed migration: it explicitly calls MeshInventory “halfway there” and lists moving KnopInventory/payload/support progression into metadata as required work. I’m checking current code paths one more layer down so the final answer can name where the remaining behavior comes from.

**Verdict: not fully completed.** The shift is only partially in place.

The good part: MeshInventory’s own mutable progression is partly moved into `_mesh/_meta/meta.ttl`; for example Alice has `<_mesh/_inventory>` current/latest/next facts in [_mesh/_meta/meta.ttl](../../mesh-alice-bio/_mesh/_meta/meta.ttl:11).

But current inventory files still contain mutable progression facts. I found `sflo:currentArtifactHistory` in 18 current `*/_inventory/inventory.ttl` files across the checked-in fixture meshes. Example: [alice/bio/_knop/_inventory/inventory.ttl](../../mesh-alice-bio/alice/bio/_knop/_inventory/inventory.ttl:12) has current pointers for the payload, Knop metadata, and Knop inventory. Even `_mesh/_inventory/inventory.ttl` still carries current/latest facts for other support artifacts, e.g. [_mesh/_inventory/inventory.ttl](../../mesh-alice-bio/_mesh/_inventory/inventory.ttl:29).

The task note says the same thing: MeshInventory is “halfway there,” and remaining work includes moving KnopInventory, payload, and support history progression into metadata: [wa.task.2026.2026-05-17-append-onlyish-inventory.md](wa.task.2026.2026-05-17-append-onlyish-inventory.md:21). It is still also listed in the backlog: [wd.todo.md](../../../../../documentation/notes/wd.todo.md:16).

So I would not call the migration complete. The fixture current pointers you noticed are real, not just stale tests. No code changes made; this was static inspection only.

# User_2026-05-24_1724_06

One of the movitations is that "current" might be a matter of opinion, e.g. people working in different ArtifactHistories. 

But what I really want to do is solve invalidation for RDF. "current" ages out rapidly, and it'd be nice if there was a way to check what's current, or whether a current assertion is still valid. Or at the very least, the ability to assert "this data is at least pseudo-immutable" vs "this data changes all the time".
