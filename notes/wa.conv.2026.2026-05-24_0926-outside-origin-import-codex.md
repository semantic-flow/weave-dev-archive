---
id: outside-origin-import-weuvqb
title: 'outside-origin import'
desc: ''
created: 1779640008050
updated: 1779640008050
participants: [codex.gpt-5.5]
conversationEventKinds: [message.assistant, message.user, tool.call, tool.result]
---

# User_2026-05-24_0713_15

outside-origin import

We're going to work on [[wa.task.2026.2026-05-21_0907-import]] . Somehow we missed filling out a [wu.cli-reference.import.md](../../../../../documentation/notes/wu.cli-reference.import.md) earlier, but we'll do that too.

I'm kinda surprised about this "`weave import` should also own the localize-from-integrated workflow: after an artifact has been integrated from an external/local-floating source, the user should be able to copy the current bytes into a regular governed working file and make that file the active working locator." -- it feels like import should both copy the file to a specified location (I guess --working-file could be the flag for that) and integrate is at a specified designator path. So we shouldn't say "after an artifact has been integrated", maybe just "after an artifact has been copied to its working file location, a knop gets created under the specified designator path" or something like that.

# gpt-5.5_2026-05-24_0713_23

I’ll ground this in the repo guidance first, then I’ll read the import task and CLI note together so the wording and implementation intent stay aligned.

The task note already has the tension you spotted: it describes import as both “after integrate” localization and as first-class acquisition. I’m going to check the surrounding CLI/user docs before editing so the fix lands in the same vocabulary as the rest of the vault.

I agree with your pushback. I’d keep “from an already integrated artifact” as an allowed source acquisition mode, but I would stop making it sound like import happens only after integrate; the primary contract is copy bytes to `--working-file`, then register/update the designator/Knop around that local file.

I’m going to make two documentation edits now: first the task note so the decision is “copy bytes to `--working-file`, then create/update the Knop at the target designator”; second the import CLI note, with a visible “planned contract” status so it doesn’t masquerade as a shipped command.

# User_2026-05-24_0715_23

it doesn't make sense to me to keep from an already integrated artifact unless you're using import to copy an existing artifact. Seems like a rare use case.

# gpt-5.5_2026-05-24_0715_33

Yes, that’s the cleaner boundary. I’m going to demote the “existing artifact as source” idea out of the first import contract rather than keeping a rare path in the main shape; if it ever matters, it can be a later “copy existing artifact” command or mode.

That wording sweep now leaves the rare “copy existing artifact” idea as explicitly deferred, not part of import’s first contract. I’m doing a quick whitespace/status check and one small cleanup in the general CLI reference where it still used the mushy “integration/import” phrase.

You were right. I narrowed the task so first-slice `weave import` is just outside-origin acquisition: read/fetch explicit source bytes, copy them to `--working-file`, then create/update the Knop/payload artifact at the target designator.

Changed:
- [wa.task.2026.2026-05-21_0907-import.md](wa.task.2026.2026-05-21_0907-import.md:20): removed the “from integrated/localize” surface from the first contract and made “copy existing artifact/current source” an explicit deferred non-goal.
- [wu.cli-reference.import.md](../../../../../documentation/notes/wu.cli-reference.import.md:9): filled out the planned CLI reference, marked it as not shipped yet, and kept it focused on local path, `file:`, and bounded HTTP(S) imports.
- [wu.cli-reference.md](../../../../../documentation/notes/wu.cli-reference.md:45): replaced the ambiguous “integration/import” phrase with `integrate` plus future import.

Verification: `git diff --check` passed in both `weave` and `weave-dev-archive`. I didn’t run the linter since this was docs-only.

Suggested commit messages:

```text
docs: clarify planned import CLI contract

- Fill the planned import CLI reference with usage, examples, source rules, and topology notes
- Mark weave import as not yet shipped
- Distinguish import from integrate and payload update
```

```text
docs: narrow import task to outside-origin acquisition

- Define import as copying explicit source bytes to --working-file before Knop registration
- Defer copying from existing current-source bindings as a separate rare workflow
- Remove localize-from-integrated planning and test requirements
```

# User_2026-05-24_0725_06

Is it weird that @20-bob-page-imported-source.jsonld doesn't actually include the raw github url? Maybe that's a part of what we're building here. I'm not really clear on what import functionality is already in place, but "add a `weave import` CLI command." is clearly wrong?

# gpt-5.5_2026-05-24_0725_17

I’ll check the carried fixture and the actual CLI/runtime surface before answering, because this is exactly the kind of spot where old fixture language can make the task note lie by implication.

The manifest itself confirms your suspicion: it says “from raw GitHub URL” in prose, but the assertions only require `targetLocalRelativePath "bob-page-main.md"` and explicitly assert no `targetAccessUrl`. I’m checking the target fixture state too, because the URL could still be in the actual Turtle even if the manifest doesn’t assert it.

That fixture ref is not present as a local branch/tag in the checked-out `mesh-alice-bio` repo, so I’m going to look at the conformance manifests and any generated fixture material we do have locally rather than pretending the branch is checked out.

Interesting wrinkle: the remote branch for the carried fixture does contain the raw URL in inventory Turtle, but the conformance manifest you pointed at does not assert it. The local `.assets` overlay also looks like a narrower page-source overlay, so I’m checking whether any production command can create the URL-bearing import shape today.

I’m going to tighten the task note to say what the repo actually shows: import exists as a conceptual/fixture operation, not as a runnable command. I’ll also add the Bob URL assertion gap as part of the work, because otherwise the fixture can keep claiming “raw GitHub import” without proving the provenance survived.

# User_2026-05-24_0726_28

Sorry the relevant fixture branches have the "a.*" prefix

# gpt-5.5_2026-05-24_0726_48

Ah, thanks. I’ll check the `a.*` refs directly and then I’ll adjust the task wording if my note should name that branch family instead of vaguely saying “remote fixture branch.”

You’re right to point at `a.*`: the relevant scenario index binds `20` to `a.20-bob-page-imported-source`. That branch does not look the same as the non-`a` remote ref, so I’m correcting my note to avoid treating the non-`a` branch as authoritative for this task.

The `a.20` branch is even narrower than the non-`a` branch: it has `bob-page-main.md` and `bob/_knop/_page/page.ttl`, but no `bob/page-main` Knop/payload artifact and no raw GitHub URL provenance. I’m updating the task note so we don’t accidentally treat that carried fixture as the command contract.

Yep, you’re right on both points.

The relevant fixture lane is `a.20-bob-page-imported-source`, and that branch does **not** include the raw GitHub URL. It proves a narrower thing: `bob-page-main.md` exists, and `bob/_knop/_page/page.ttl` points at it with `sflo:targetLocalRelativePath "bob-page-main.md"`. It does not create `bob/page-main` as a payload artifact, and it does not record import provenance. So if the manifest prose says “from the raw GitHub URL,” the fixture is under-asserting that claim.

Also, “add a `weave import` CLI command” was sloppy. There is no supported `weave import` command today; import exists as a concept/fixture operation. I updated the task note to say “implement the first runnable `weave import` command,” and added the `a.*` fixture nuance so Bob doesn’t accidentally define the real import contract.

Verification: `git diff --check` passes in both repos.

# User_2026-05-24_0735_03

I wonder how we did the import in a.20. Guess it doesn't matter.

Regarding open issues:

- agree, we can add --working-file inference later
- the --working-file path might be permanently okay with just mesh-root-relative: people can just get their own bytes into a sidecar-adacent folder or non-mesh branch. I mean, thinking about it now, import doesn't do that much additional work over like a wget + weave import. I almost want to take it out. I guess it can record the original location and fingerprint, and infer the proper location. What's you're take.

# gpt-5.5_2026-05-24_0735_22

I think the healthy pushback is: if import does not record origin/fingerprint and does not create/update the governed artifact boundary, it is too close to `curl > file && weave integrate` to deserve a command. I’m going to encode that in the task note: keep `--working-file` mesh-root-relative, treat inference as future ergonomics, and make provenance/digest part of the minimum useful command.

I’m editing this as a decision rather than leaving it in “open issues”: `--working-file` stays mesh-root-relative as the normal contract, and import earns its keep only by doing acquisition bookkeeping that a bare download will not do.

My take: I would **not** take `import` out, but I’d make it earn its place. If it’s only “download/copy bytes, then register them,” it is too close to `curl > file && weave integrate`. The durable value is: explicit acquisition boundary, governed in-mesh placement, origin/provenance recording, observed digest calculation, optional `--expected-digest` verification, and later path inference.

I updated the notes that way:

- [wa.task.2026.2026-05-21_0907-import.md](wa.task.2026.2026-05-21_0907-import.md:24): says import must record origin and fingerprint, otherwise it is not much better than shell download plus `integrate`.
- [wa.task.2026.2026-05-21_0907-import.md](wa.task.2026.2026-05-21_0907-import.md:73): makes `--working-file` mesh-root-relative as the normal contract; outside-mesh staging is not import’s job.
- [wu.cli-reference.import.md](../../../../../documentation/notes/wu.cli-reference.import.md:15): mirrors that planned user-facing contract.

So: yes to `--working-file` inference later, yes to mesh-root-relative as the permanent baseline, and yes to keeping import only if provenance/fingerprint are part of the minimum useful slice.

Verification: `git diff --check` passes in both repos.

# User_2026-05-24_0736_40

I guess import is extremely useful in the case where we're using an API instead of the CLI.

# gpt-5.5_2026-05-24_0736_49

Yes, exactly. The API case is the strongest argument for keeping import: one call can do acquisition, digesting, governed placement, and registration atomically, without making every client reinvent the sequence. I’ll add that framing to the task and planned CLI note.

I found the right place to add it. I’m going to make the core/API benefit explicit in both notes so it doesn’t read like this is only a convenience wrapper for terminal users.

Yep, that’s the strongest version of the argument. I added it explicitly:

- [wa.task.2026.2026-05-21_0907-import.md](wa.task.2026.2026-05-21_0907-import.md:85) now says the API case makes import more important, because daemon/web/automation callers should get acquisition, placement, digesting, registration, and provenance as one operation.
- [wu.cli-reference.import.md](../../../../../documentation/notes/wu.cli-reference.import.md:19) mirrors that below the CLI.

Verification: `git diff --check` passes in both repos.

# User_2026-05-24_0747_40

So back to open issues:
- support fetch immediately, I guess from an arbitrary URL which is different from where import is allowed to do?
- I like --replace-working, seems the most explicit
- update both
- We should definitely not assert RdfDocument unless the target is RDF. Can we avoid that without a type-model cleanup? What would that cleanup look like? 
- Sure on --expected-digest
- use source registry, make sure the task note reflects our latest thinking there. 
- yes, the markdown, and I wouldn't mind taking the image mentioned in the markdown notes... it makes me think eventually import could have some scraper-like features, where content referenced in the target is also imported.

# gpt-5.5_2026-05-24_0747_59

I’ll turn those into decisions in the task note, but I want to quickly inspect the current payload typing and SFLO vocabulary first. The RdfDocument point is a real semantic bug waiting to happen, not a bikeshed.

The ontology already supports selective `RdfDocument`: it is a content-kind classification, not mandatory payload type. So yes, we can avoid asserting it for Markdown in the import slice without doing a full content-kind cleanup. I’m going to convert your bullets into decisions and leave the larger cleanup sketched as future work.

I’m going to make `HTTP(S) now`, `--replace-working`, `update both`, `source registry`, `--expected-digest`, and “no Markdown as RdfDocument” into decisions. For the referenced images, I’ll mark recursive asset import as a future scraper-like mode, not part of the first byte-for-byte import.

I’m also going to correct an earlier implication in the note: once we choose source registry, the remote URL should not be written as the payload’s `workingAccessUrl` in the normal import case, because the active working locator is the local copied file.

One small cleanup: now that HTTP(S), `--replace-working`, and `--expected-digest` are decisions, I’m removing the old “if HTTP lands” flavor and making the first CLI shape match the decided flags.

Updated both notes with the latest shape of the import contract.

The task note now reflects:

- first runnable `weave import`, not “add a command” as if nothing exists conceptually
- explicit HTTP(S) fetch in scope for import only
- `--replace-working`
- `--expected-digest sha256:...`
- source registry as the provenance home
- mesh-root-relative `--working-file`
- no false `sflo:RdfDocument` assertion for Markdown/images/non-RDF
- Carol Markdown as the first strong whole-mesh fixture candidate
- linked images/assets as a future scraper-like import mode, not first slice

I also filled the planned CLI reference at [wu.cli-reference.import.md](../../../../../documentation/notes/wu.cli-reference.import.md:1), and lightly adjusted the main CLI reference at [wu.cli-reference.md](../../../../../documentation/notes/wu.cli-reference.md:45).

On the `RdfDocument` question: yes, we can avoid asserting it without a full type-model cleanup. The first slice can simply use conservative content detection and only assert `sflo:RdfDocument` when the imported target is actually RDF. The larger cleanup would be threading a real content-kind/media-kind model through import, integrate, payload rendering, inventories, versioning, and page generation so Markdown/images/generic blobs have first-class representation instead of being squeezed through RDF-ish assumptions.

Verification: `git diff --check` passes in both the main weave repo and the weave-dev-archive dependency repo.

# User_2026-05-24_0751_42

obviously expected-digest is optional, right?

Regarding "The current carried Bob fixture uses older namespace vocabulary in some historical branches. " -- are those the non-a.* branches? Probably we should clean those up (except for main)... they are a source of confusion with little value even as historical reference. What do you think?

# gpt-5.5_2026-05-24_0751_49

I’ll tighten the docs so `--expected-digest` is unmistakably optional, then I’ll check exactly how the historical Bob branch wording is framed before giving you a concrete cleanup recommendation.

Yep, globally optional. I do think the Carol fixture should require it, but that is a test-contract choice rather than a CLI requirement; I’m going to make that distinction explicit so it doesn’t read like the command demands it every time.

The local framework checkout only has `main` fetched, so I’m checking the remote refs rather than trusting the sparse local view. That should tell us whether the confusing historical branch names are actually still hanging around upstream.

I found two spots where the wording was technically right but not loud enough: the general CLI surface and the implementation-plan checklist. I’m patching those, and I’ll add a small fixture-hygiene note for the non-`a.*` Bob refs as a separate cleanup, not part of import itself.

Yes, `--expected-digest` is optional. I made that explicit in [wu.cli-reference.import.md](../../../../../documentation/notes/wu.cli-reference.import.md:31) and in the task decisions at [wa.task.2026.2026-05-21_0907-import.md](wa.task.2026.2026-05-21_0907-import.md:109). Import should always compute/report the observed digest; `--expected-digest` just turns that into an assertion.

On the Bob fixture branches: yes, I meant the stale historical refs outside the current `a.*` scenario lane. I agree with you: they are now mostly confusion, and their “historical reference” value is weak if they contain old vocabulary and misleading import shapes. I added a note recommending a separate Semantic Flow Framework fixture-hygiene pass to prune stale non-`a.*` refs after confirming no active scenario index or docs still point at them: [wa.task.2026.2026-05-21_0907-import.md](wa.task.2026.2026-05-21_0907-import.md:91).

Verification: `git diff --check` passes in both affected repos.

# User_2026-05-24_0803_13

Sorry the relevant fixture branches have the "a.*" prefix

Sorry the relevant fixture branches have the "a.*" prefix

I created [[sf.task.2026.2026-05-24_0754-framework-hygiene]] - can you include the pruning and anything else you noticed.

# gpt-5.5_2026-05-24_0820_55

I’ll open the new hygiene task and the nearby conventions, then I’ll fold in the stale fixture-ref pruning plus the related cleanup observations from the import thread.

The hygiene note is currently just a shell, and the `hygiene3` tab doesn’t exist on disk. I’m going to seed the real task note with concrete framework cleanup items, keeping the import-specific implementation details in the Weave task.

I found the canonical branch prefix in `scenario-index.jsonld`; the individual manifests still use unprefixed `fromRef`/`toRef`, which is probably part of why the old refs keep tripping us. I’m checking the actual fixture repo branches before writing the pruning checklist.

The actual fixture repos confirm the shape: Alice Bio and sidecar have duplicate unprefixed remote branches plus the canonical `a.*` ladder, while branch-fantasy is already mostly clean with `a.*`, `main`, and `gh-pages`. I’m going to make the task specific about preserving canonical refs and pruning only stale duplicate fixture branches after an audit.

I’ve got enough context now. I’m filling the task note with a scoped cleanup plan: stale branch pruning, manifest/ref terminology, Bob import wording, and a small audit of framework API/spec notes that mention import before the command exists.

The task note is populated now. I’m doing a quick pass for markdown hygiene and to make sure I didn’t introduce stale path-style internal links.

Updated [sf.task.2026.2026-05-24_0754-framework-hygiene.md](sf.task.2026.2026-05-24_0754-framework-hygiene.md:8) with the pruning work and the related cleanup I noticed.

I included:

- prune stale unprefixed fixture branches after audit, preserving `main`, `gh-pages`, and all active `a.*` refs
- clarify that `scenario-index.jsonld` lane bindings are authoritative for actual fixture refs
- treat unprefixed manifest `fromRef`/`toRef` as logical scenario names unless resolved by the scenario index
- fix the Bob `20/21` confusion: current fixture proves local page-source behavior, not real import provenance
- audit framework API/spec notes so conceptual `import` wording does not imply a shipped Weave command
- note that `mesh-sidecar-fantasy-rules` has an incomplete stale unprefixed ladder, while `mesh-branch-fantasy-rules` is already mostly clean

`git diff --check` passes for the archive repo.

# User_2026-05-24_0825_08

How did you fix the 20/21 confusion? I'm leaning towards complete fixture regen again soon to test the fixture generalization.

# gpt-5.5_2026-05-24_0825_24

I didn’t fix the actual `20/21` fixture files yet. I fixed the confusion in the hygiene task note by making it explicit that:

- the active refs are `a.20-bob-page-imported-source` / `a.21-bob-page-imported-source-woven`
- the unprefixed `20/21` names in manifests are logical step names, not the canonical branch refs
- the current `20/21` fixture proves local page-source behavior, not real `weave import` provenance
- the manifest prose should be rewritten because it still implies raw GitHub URL import provenance that the assertions don’t verify

So: documentation/planning cleanup only, not fixture correction.

I think complete fixture regen soon is a good call, especially if you want to test fixture generalization. I’d use that as the moment to stop papering over Bob `20/21`: regenerate from the canonical `a.*` lane, rewrite the `20/21` descriptions to match what they actually assert, and prune the stale unprefixed branches after the regen proves the helper can walk the full ladder cleanly. That’s cleaner than hand-polishing one confusing rung now and then regenerating over it later.

# User_2026-05-24_0829_00

So we should rewrite 20 so that it actually gets a remote URL? There isn't really a "local import" use case in a whole-repo mesh.

# gpt-5.5_2026-05-24_0829_16

I think you’re right to separate “local file copied into a whole-repo mesh” from the fixture’s real purpose. I’ll tweak the hygiene task so it says `20` should either become a real remote-origin import fixture or stop calling itself import; that makes the regen path less mushy.

I updated the hygiene note with that sharper fork. I’m doing the small validation pass now.

Yes, if `20` keeps the “imported-source” role, it should actually carry the remote URL provenance. Otherwise it should stop calling itself import.

I updated the hygiene task with that fork: Bob `20/21` should either regenerate as a real remote-origin import pair with source-registry URL/digest provenance, or be renamed/reworded as local page-source coverage. See [sf.task.2026.2026-05-24_0754-framework-hygiene.md](sf.task.2026.2026-05-24_0754-framework-hygiene.md:38).

I agree with your instinct: a local-copy import fixture in a whole-repo mesh is pretty weak. It can exist as a mechanics test somewhere, but the meaningful whole-repo import story is “acquire outside bytes, record origin/fingerprint, then render from governed local bytes.” That’s exactly what `20` was gesturing at, so fixture regen is a good moment to make it honest. `git diff --check` still passes for the archive repo.

# User_2026-05-24_0835_01

Maybe our complete fixture regen can move to a b.* series. We can then compare with a.* to check the generalization.

Do you want the raw bob URL? 

And then do we even need the carol import described in [[wa.task.2026.2026-05-21_0907-import]] ? Maybe we make that a test for the ResourcePage panel inclusion, like the References panel. Or we could do the --replace-working stuff there, since there are actually three versions of that page

# gpt-5.5_2026-05-24_0835_19

I like the `b.*` idea: it gives us a clean regenerated lane without destroying the `a.*` comparison target. I’m going to check the current import task wording and recover whatever Bob URL is already embedded in old fixture state so we can decide whether we need you to hand me anything.

I found the legacy floating Bob URL in the old fixture state and in the import task. For the regenerated lane, I’d want a commit-pinned Bob raw URL plus digest; the floating `refs/heads/main` URL is fine as a clue, but too squishy for a conformance fixture.

I’m going to update the import task to make Bob the likely whole-mesh remote-origin import fixture if we regenerate into `b.*`, and demote Carol from “first import fixture” to a better fit: ResourcePage panel inclusion or `--replace-working`/multi-version behavior.

I’ve updated the import task. I’m also going to add the `b.*` lane idea to the framework hygiene note, since that belongs with fixture pruning/generalization rather than the import implementation alone.

The notes now say `b.*` should be the regeneration/comparison lane, with Bob `b.20/b.21` as the honest remote-origin import candidate. I’m running the markdown diff checks and then I’ll summarize the tradeoff.

One small useful thing before I close this out: I’m computing the digest of the current carried Bob bytes, since that gives us a known content fingerprint even before we pin the source URL to a commit.

Yes to `b.*`. I updated both task notes so the next full fixture regen can write a regenerated `b.*` lane, compare it against `a.*`, and only later decide whether `b.*` becomes canonical.

On the raw Bob URL: I found the legacy floating one already:

```text
https://raw.githubusercontent.com/djradon/public-notes/refs/heads/main/user.bob-newhart.md
```

For the fixture, though, I’d rather have a commit-pinned raw URL plus expected digest. The current carried `a.20` Bob bytes hash to:

```text
sha256:0d41e556b20f1ab794cec26c62ef3baedac1e25ab33461880b8a58d174457f2a
```

So if you have the intended pinned Bob URL, yes, that’s useful.

I also changed [wa.task.2026.2026-05-21_0907-import.md](wa.task.2026.2026-05-21_0907-import.md:47) so Carol is no longer framed as the first whole-mesh import fixture if Bob `b.20/b.21` becomes the honest remote-origin import case. Carol now reads as better suited for ResourcePage panel inclusion, linked-asset exploration, or `--replace-working` over multiple versions.

`git diff --check` passes in both the archive repo and main weave repo.

# User_2026-05-24_0848_36

'''''''';;;;;;;;;;;'

# gpt-5.5_2026-05-24_0848_42

Tiny keyboard storm detected. I’m here.

# User_2026-05-24_0851_55

I think we can leave bob as importing a non-pinned "main" asset with no provided sha hash (but it can compute the hash for record keeping if we support that.) And carol will actually turn into 6 steps, an import and weave pair for each of the versions.

# gpt-5.5_2026-05-24_0852_15

That makes sense, with one caveat: Bob as floating `main` should be treated as “record observed provenance” rather than “deterministic replay evidence.” I’ll update the import and hygiene notes so Bob proves no-expected-digest import, while Carol becomes the richer three-version `import`/`weave` ladder.

I’m with you. Bob can specifically exercise the “floating URL, no expected digest, but record observed digest” path, while Carol is the richer repeated replacement/versioning story. I’ll adjust the notes to make that split explicit.

The import task now has Bob as the floating-source/no-expected-digest fixture and Carol as the three-version replacement ladder. I’m mirroring the same distinction into the framework hygiene task so the regen plan stays aligned.

Both notes are aligned now. I’m doing the final `diff --check` pass and grabbing the exact edited snippets so I can close this cleanly.

Agreed. I updated the notes to reflect that split.

Bob is now framed as the floating-source import case: `b.20` imports from the raw `main` URL with no `--expected-digest`, but still records the observed digest and source URL in the source registry. I left the caveat in the task that this is intentionally weaker than a pinned replay contract, but useful because it tests a real user shape. See [wa.task import note](wa.task.2026.2026-05-21_0907-import.md:48).

Carol is now the richer replacement/versioning path: six fixture steps total, as three import/weave pairs over the same `carol/page-main` working file, with the second and third imports using `--replace-working`. I also kept Carol as a good hook for ResourcePage panel inclusion or linked-asset exploration later.

I mirrored the same plan into the framework hygiene note: [sf.task framework hygiene](sf.task.2026.2026-05-24_0754-framework-hygiene.md:39).

Verification: `git diff --check` passes in both the archive repo and the main weave repo.

# User_2026-05-24_0913_42

We'll do that regen after [[wa.task.2026.2026-05-21_0907-import]] lands, so settle the fixture-based testing requirement as deferred?

# gpt-5.5_2026-05-24_0913_54

Yes, that seems right: implementation can land with focused unit/runtime/e2e coverage, while the full `b.*` fixture regen becomes a follow-on acceptance/regression pass. I’ll mark that split explicitly in the import task and keep the hygiene note aligned.

I found the exact places. I’m going to move those fixture-ladder items out of “this must land with import” and mark them as deferred follow-on acceptance coverage after the command lands.

The import task now has the fixture-ladder work marked as deferred. I’m making the framework hygiene task say the same thing: `b.*` comparison is post-import, not a prerequisite for landing the command.

Done. I’m running the final diff checks and checking the deferred checklist markers rendered correctly.

Yes. I settled the fixture-ladder requirement as deferred, not a landing gate for `weave import`.

In [wa.task.2026.2026-05-21_0907-import.md](wa.task.2026.2026-05-21_0907-import.md:138), the first implementation now calls for focused unit/runtime/e2e coverage without requiring full `b.*` fixture regen. The Bob `b.20/b.21` and Carol six-step ladder items are marked `[d]` in the implementation plan.

I also aligned [sf.task.2026.2026-05-24_0754-framework-hygiene.md](sf.task.2026.2026-05-24_0754-framework-hygiene.md:88) so the framework hygiene task says `b.*` regen is follow-on acceptance/regression coverage after import lands.

Verification: `git diff --check` passes in both the archive repo and main weave repo.

# User_2026-05-24_0917_21

regarding "keep `--working-file` mesh-root-relative" -- if it's important for API usage, then we could support the sidecar-adjacent and branch based targets. No?

# gpt-5.5_2026-05-24_0917_40

That’s a fair push. I think the right distinction is: keep the first CLI simple with mesh-root-relative `--working-file`, but let the planner/API model a richer destination locator so sidecar-adjacent and branch source-worktree copies are possible under explicit policy. I’ll update the task wording so we don’t accidentally freeze the lower-level API into the CLI’s first-slice constraint.

I found the old wording in both the task and CLI reference. I’m going to keep the user-facing CLI text conservative, but add an API/planner destination model: mesh-local via `--working-file`, plus explicit approved local/worktree destinations for sidecar-adjacent and branch-source layouts.

The task note now has the split I wanted: simple CLI path, richer API destination locator. I’m updating the CLI reference so it doesn’t falsely say outside-mesh targets are impossible, just not part of the first `--working-file` CLI shape.

That’s patched in both places. I’m running the checks and then I’ll give you the short answer with the exact distinction.

Yes. I think the better shape is:

- CLI first slice: keep `--working-file` mesh-root-relative because it is easy to explain and hard to misuse.
- Planner/API: support an explicit destination locator, including mesh-local, approved sidecar-adjacent, and approved branch source-worktree destinations.

I updated the task accordingly. The key bit is now in [wa.task.2026.2026-05-21_0907-import.md](wa.task.2026.2026-05-21_0907-import.md:120): mesh-local imports use `sflo:hasWorkingLocatedFile`; approved outside-mesh local destinations use the existing working-local locator shape instead of forcing everything under the mesh root.

I also updated [wu.cli-reference.import.md](../../../../../documentation/notes/wu.cli-reference.import.md:19) so the planned CLI reference says the first CLI writes under the mesh root, while lower-level API requests may target approved sidecar-adjacent or branch source-worktree destinations.

Verification: `git diff --check` passes in both repos.

# User_2026-05-24_0919_03

I think we can settle "whether import deserves to exist". It does, right?

# gpt-5.5_2026-05-24_0919_12

Yes, I think that one is settled now. Import earns its keep because it is the explicit acquisition/provenance operation, especially for API callers and remote/floating sources; I’ll remove the “whether it deserves to exist” framing and turn it into a decision.

That point is now phrased as a settled decision instead of an open justification. I’m running the usual quick checks.

Settled. I changed the note from “whether import deserves to exist” to a clear decision:

`weave import` deserves to exist as a distinct operation because it couples explicit acquisition with governed placement, origin recording, observed digest calculation, optional verification, artifact registration, and later path inference.

That’s now stated in [wa.task.2026.2026-05-21_0907-import.md](wa.task.2026.2026-05-21_0907-import.md:99), and the decisions section mirrors it at line 118.

`git diff --check` passes in both repos.

# User_2026-05-24_0921_01

OK, are we ready to implement? New conversation, or this one?

# gpt-5.5_2026-05-24_0921_09

We’re ready to implement, but I’d start a new conversation.

This thread did a lot of design settling, and the actual implementation will need fresh attention on code paths, tests, and maybe a few decisions from the task note. A new thread with something like “Implement `weave import` from `wa.task.2026.2026-05-21_0907-import.md`” will give us a cleaner working context.

Before switching, the important state is already in the notes:

- import is a real distinct operation
- first CLI shape is settled
- HTTP(S), `file:`, local paths, observed digest, optional `--expected-digest`, and `--replace-working` are in scope
- CLI `--working-file` is mesh-root-relative
- planner/API should allow richer destination locators
- full `b.*` fixture regen is deferred until after import lands
- Carol six-step replacement ladder is deferred follow-up

So yes: new conversation, same task note as the anchor.
