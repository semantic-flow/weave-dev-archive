---
id: conformance-ladder-appro-a6g87u
title: 'Conformance ladder approach'
desc: ''
created: 1783108018328
updated: 1783108018328
participants: [djradon, codex.gpt-5.5]
conversationEventKinds: [message.assistant, message.user, tool.call, tool.result]
---

# djradon_2026-07-03_1101_28

Conformance ladder approach

While working on Stagecraft, Claude came up with some insight on the "main" branch of the ladder: 

"## Main Update Convention

When a new rung is accepted, update `main` by merging the woven rung, then commit conformance manifests and scenario-index updates on top:

```sh
git merge a.0N-<rung-name>-woven
# then add/commit conformance/*.jsonld changes on main
```

Do not re-apply rung content to `main` as fresh linear commits. Merging makes the accepted rung an ancestor of `main`, so provenance is visible in history and the "accepted state + harness" invariant is guaranteed at merge time instead of re-verified by hand.

Before pushing, verify the invariant — `main` may differ from the latest woven rung only under `conformance/`:

```sh
git diff a.0N-<rung-name>-woven main --name-only | grep -v '^conformance/' && echo "DRIFT: main diverged from the accepted rung" || echo "ok: only conformance/ differs"
```

Known exception: the a.03/a.04 recommendation rung landed on `main` as linear commits and the invariant was verified manually; see the close notes in [[sc.task.2026-07-03_0852-plan-b-recommendation-fixture-ladder]]. Resume the merge convention from the next rung.

## Weave And ResourcePages

`weave generate` renders ResourcePages from an already-settled mesh state. It may create no files if the needed histories or page-bearing resources do not exist yet.

Full `weave` runs the composed local flow:

```text
validate
version
generate ResourcePages
```

When the expected result is a fully woven fixture rung with payload histories and `index.html` ResourcePages, run full `weave`, not only `weave generate`.

A rung named `*-woven` should normally include the generated ResourcePages and the RDF links that claim them, such as `sflo:hasResourcePage`.

## Accord Checks

Accord is the transition checker. It does not execute Weave.

Use Accord to answer:

```text
Did this branch transition add, update, or remove the expected paths?
Do representative RDF facts exist in the target ref?
Did generated support files land where the contract expects?
```

Do not expect Accord to prove exact bytes for newly added files unless Accord grows that capability. For added RDF files, pair path expectations with `hasAskAssertion` checks for the semantic facts that matter.

Keep `conformance/` off rung refs unless there is a concrete reason to include it. If conformance files appear on rung refs, either cover them explicitly or ignore them with `conformance/**`; do not let harness files accidentally become the thing being tested.

## Stagecraft Fixture Checks

Stagecraft CTest fixture checks validate the checkout that Stagecraft reads. They do not validate every fixture ladder transition.

Use them to answer:

```text
Does Stagecraft still parse and accept the current woven fixture checkout?
Do JSON, Turtle, JSON-LD, SHACL, and C++ contract checks still pass against the accepted fixture state?
```

Use Accord to validate branch transitions. Use Stagecraft checks to validate Stagecraft's interpretation of the accepted current fixture data.

## Useful Commands

Run Accord transition checks from the Stagecraft repo:

```sh
deno run -A /home/djradon/hub/semantic-flow/weave/dependencies/github.com/spectacular-voyage/accord/src/main.ts check dependencies/github.com/spectacular-voyage/test-inn-ambush/conformance/01-plan-b-fixture-records.jsonld --fixture-repo-path dependencies/github.com/spectacular-voyage/test-inn-ambush
deno run -A /home/djradon/hub/semantic-flow/weave/dependencies/github.com/spectacular-voyage/accord/src/main.ts check dependencies/github.com/spectacular-voyage/test-inn-ambush/conformance/02-plan-b-woven.jsonld --fixture-repo-path dependencies/github.com/spectacular-voyage/test-inn-ambush
```

Run the Inn Ambush fixture subset:

```sh
ctest --test-dir build/experiments/inn-ambush --output-on-failure -R 'inn-ambush-(jsonld-context-json|projection-notification-json|chat-contribution-json|participant-projection-request-json|authority-operation-request-json|authority-journal-turtle|rdf-export-turtle|contract-shapes-turtle|contract-shacl)'
```

Check ResourcePages in the accepted fixture checkout:

```sh
find dependencies/github.com/spectacular-voyage/test-inn-ambush -path '*/.git' -prune -o -name 'index.html' -print
```
"

--- if that makes sense for us here, can you adopt that recommendation into [wd.testing.md](../../../../../documentation/notes/wd.testing.md) or [wd.testing.fixture-ladder-regeneration.md](../../../../../documentation/notes/wd.testing.fixture-ladder-regeneration.md) ?

# gpt-5.5_2026-07-03_1101_38

I’ll fold this into the repo’s existing testing guidance rather than inventing a new note. First I’m reading the local guidance and the two candidate docs so the wording lands in the right place.

The recommendation fits best in the fixture ladder playbook, especially because the current “merge to main” section uses `reset --hard`/force-push language that fights the provenance invariant. I’m also going to add a small generic Accord note in `wd.testing.md`; the Stagecraft-specific bits don’t belong here verbatim, but the boundary between transition checks and checkout interpretation does.

I found the local shape: Weave keeps transition manifests in `semantic-flow-framework`, while the live fixture repos are mostly branch-state fixtures. So I’m adapting the convention rather than copying the Stagecraft wording whole cloth: merge accepted rungs into `main`, keep harness files out of rung refs, and use Accord for transitions.

The first patch missed because the existing final section has slightly different fencing than I expected. I’m checking the exact tail and then I’ll make the same edit against the real text.

I also spotted that the existing tail wasn’t fenced as a shell block, so I’ll fix that while replacing the force-push recipe.

The generic testing note now captures the Accord boundary: transition checker, not executor; semantic facts for new RDF; and harness files kept out of rung refs. Next I’m updating the operational playbook.

The playbook now has the ResourcePages and Accord guidance in the workflow itself. I’m replacing the old `reset --hard` main recipe with merge-based provenance guidance and an invariant check.

One small cleanup: the first pass had push commands before the “before pushing” invariant, which is the sort of tiny ordering bug docs love to smuggle in. I’m reordering that section so verification comes before publication.

The docs edit is in place. I’m doing a quick review pass now for wording, markdown structure, and repo conventions before I call it done.

The diff looks directionally right: the force-push recipe is gone, the merge convention is now explicit, and Weave’s separate conformance-manifest repo is called out. I’m doing one last sanity check on formatting and repo status.

Only the two documentation notes are modified. I’m running a lightweight format check on those markdown files; no code test suite is needed for this particular change.

Deno’s default formatter target selection ignored the markdown paths, so I’m retrying with an explicit markdown extension. If that still doesn’t apply cleanly, I’ll note the limitation rather than pretending it checked something.

The explicit markdown format check still found no Deno-supported target files, so I’m using `git diff --check` for whitespace sanity. That’s the useful check available here without bringing in another formatter.

Whitespace checks passed. I’m grabbing the final line numbers now so the wrap-up can point you straight at the edits.

Adopted the recommendation in both docs, adapted for Weave’s current fixture layout.

Changed:
- [wd.testing.md](../../../../../documentation/notes/wd.testing.md:91): clarified Accord as a transition checker, not an executor, and added guidance for RDF semantic assertions, `conformance/`, and downstream fixture checks.
- [wd.testing.fixture-ladder-regeneration.md](../../../../../documentation/notes/wd.testing.fixture-ladder-regeneration.md:31): documented that Weave’s conformance manifests live in `semantic-flow-framework`, not generated rung refs.
- [wd.testing.fixture-ladder-regeneration.md](../../../../../documentation/notes/wd.testing.fixture-ladder-regeneration.md:60): added the `weave` vs `weave generate` / ResourcePages convention.
- [wd.testing.fixture-ladder-regeneration.md](../../../../../documentation/notes/wd.testing.fixture-ladder-regeneration.md:344): replaced the old `reset --hard`/force-push main recipe with merge-based accepted-rung guidance and a drift check.

Checks:
- `git diff --check -- documentation/notes/wd.testing.md documentation/notes/wd.testing.fixture-ladder-regeneration.md` passed.
- `deno fmt --check ...` did not apply because Deno reported `No target files found` for these markdown paths.

Suggested Weave commit message:

```text
docs: clarify fixture ladder conformance flow

- document Accord as a transition checker rather than a Weave executor
- record ResourcePage expectations for woven fixture rungs
- replace fixture main reset guidance with merge-based accepted-rung provenance
```

# djradon_2026-07-03_1330_19

With my stagecraft ladder, we're seeing "The current local weave slice only supports the settled second payload weave shape...; generate" -- that sounds like a weakness we've run into before.

# gpt-5.5_2026-07-03_1330_27

That wording does smell like the old “fixture-carried slice instead of general operation” problem. I’ll trace where that message comes from so we can talk about the actual constraint, not just the vibe.

# gpt-5.5_2026-07-03_1330_32

Yep, the phrase is coming from the core weave planner, not Accord or Stagecraft. I’m opening the planner/shape check around that branch now; the useful bit is whether it’s rejecting a genuinely unsupported transition shape or just a too-specific fixture assertion.

# gpt-5.5_2026-07-03_1330_46

Yeah, I agree. That is the same family of weakness.

That message comes from a fixture-shaped assertion in the weave planner: [shape_assertions.ts](../../../../../src/core/weave/shape_assertions.ts:966). It is not a Stagecraft semantic failure so much as Weave saying: “I only know how to advance this exact settled fixture shape.” There are several of these still around, plus the broader single-candidate limitation in [weave.ts](../../../../../src/core/weave/weave.ts:224).

So I would not treat `weave generate` as the real fix unless the rung truly only needs page rendering from an already-versioned state. If Stagecraft is trying to create or advance payload histories, then the right command is full `weave`, and the failure means the planner needs to generalize beyond the old Alice/Fantasy carried shapes.

This is already recognized as backlog in [wd.todo.md](../../../../../documentation/notes/wd.todo.md:33), but Stagecraft makes it less abstract. I’d sharpen the diagnosis to: Weave still has too many “accepted fixture shape” gates where it should have condition-specific planning over RDF facts: current payload state, history policy, inventory append, ResourcePage eligibility, and support-artifact progression. Stagecraft is poking that bruise directly.

# djradon_2026-07-03_1332_05

OK, can you formulate some tasks (or an epic) to address? Let me know if you need some troubleshooting/how-to-reproduce detail.

# gpt-5.5_2026-07-03_1332_19

I can frame the epic without blocking on repro details, and I’ll leave a tight “need from Stagecraft” section so the task can accept your failure case later. I’m going to look at the existing task-note style and the prior planner-generalization notes, then I’ll write one durable task note and lightly wire it into the backlog.

# gpt-5.5_2026-07-03_1332_24

I found the right home for this: the weave-dev archive has both the older first-payload refactor task and a shape-generalization note. Stagecraft deserves a new epic that ties those threads together instead of hiding this as one more line in `wd.todo`.

# gpt-5.5_2026-07-03_1332_33

The older notes are very candid about the fixture-shaped origin, which is good. I’m going to create a new epic note that names Stagecraft as the forcing case, then update `wd.todo` so it shows up as an active P1 item instead of being buried under older Alice/URPX language.

# gpt-5.5_2026-07-03_1332_48

I’ve got the structure: one epic note with slices underneath, not a giant “rewrite weave” blob. The key push-back I’m encoding is that `generate` is only a legitimate workaround for already-settled histories; when Stagecraft needs new payload states, the planner must generalize.

# gpt-5.5_2026-07-03_1333_43

The epic note is drafted and the Weave backlog now points at it as an active Stagecraft-triggered blocker. I’m reviewing the diff and checking which repo owns the new note, since it lives in the archive dependency rather than the Weave docs tree.

# gpt-5.5_2026-07-03_1333_51

The new note lives in `weave-dev-archive`, which is its own git repo here. There’s also an unrelated untracked Kato conversation note already in that archive; I’m leaving it alone and only checking the task note plus Weave backlog edit.

# gpt-5.5_2026-07-03_1333_53

Whitespace checks passed for the tracked backlog edit. Because the epic note is untracked in the archive repo, I’m using a no-index diff check against `/dev/null` for that new file too.

# gpt-5.5_2026-07-03_1334_01

The new file passed the no-index whitespace check as well. I’m grabbing the final references now and then I’ll summarize what I added and what repro detail would be useful for the first implementation slice.

# gpt-5.5_2026-07-03_1334_11

Created the epic/task note here:

[wa.task.2026.2026-07-03_1332-stagecraft-weave-planner-generalization.md](wa.task.2026.2026-07-03_1332-stagecraft-weave-planner-generalization.md:1)

It breaks the work into slices:
- capture the Stagecraft reproducer
- inventory the fixture-shaped payload gates
- generalize later payload weave progression
- replace vague “settled shape” errors with fact-specific diagnostics
- align with append-onlyish inventory behavior
- add Accord and Weave integration coverage
- document the `weave` vs `weave generate` operational rule

I also wired it into [wd.todo.md](../../../../../documentation/notes/wd.todo.md:20) as the concrete Stagecraft-driven blocker epic, plus a P1 runtime-correctness item at [wd.todo.md](../../../../../documentation/notes/wd.todo.md:34).

I do not need repro detail to formulate the epic, but the first implementation slice will need:
- exact failing Stagecraft command
- fixture repo path and branch/ref
- source and target rung refs
- full error text with designator path
- relevant `git status --short --branch`
- current KnopInventory, MeshInventory, and payload history Turtle for the failing designator
- whether the rung should create a new payload state, refresh pages, or both

Checks passed:
- `git diff --check -- documentation/notes/wd.todo.md`
- no-index `git diff --check` for the new task note

One repo-status note: the new task note is in the separate `weave-dev-archive` repo. That archive also already has an unrelated untracked Kato conversation note, which I left alone.

# djradon_2026-07-03_1353_45

I guess I don't understand the "public .proto schemas / generated SDK artifacts" question. We wouldn't have to generate anything to consume the API, right? What's the "plain http calls" alternative to?

# gpt-5.5_2026-07-03_1353_52

I’ll ground this in the note you’re looking at, because that phrase can mean a few different things depending on whether we’re talking about Accord as a CLI, a runner protocol, or a future service boundary.

# gpt-5.5_2026-07-03_1354_09

Right: you would not have to generate anything just to consume a normal HTTP/JSON API.

“Plain HTTP calls” is the alternative to:

- generated SDK clients from OpenAPI, like `client.createPlan(...)`
- generated gRPC/protobuf stubs from `.proto`, like `RecommendationServiceClient.GetRecommendation(...)`
- language-specific wrapper packages that hide method/path/body details

For Accord-style runner-neutral tests, I think the better base layer is plain HTTP:

```json
{
  "@type": "HttpInteractionCase",
  "method": "POST",
  "path": "/recommendations",
  "headers": {
    "content-type": "application/json"
  },
  "body": {
    "sceneId": "plan-b"
  },
  "expectedStatus": 200
}
```

A runner can execute that with `fetch`, `curl`, C++, Deno, Python, whatever. No generated SDK required.

The OpenAPI `operationId` would just be a stable cross-reference:

```json
{
  "openapiOperationId": "createRecommendation"
}
```

That says “this case covers that documented operation,” but it does not mean the runner has to call generated OpenAPI client code.

`.proto` only matters if the API is actually gRPC / protobuf / Connect-style, where the wire contract is a proto schema and generated stubs are the normal ergonomic path. If Stagecraft’s API is ordinary REST-ish JSON/JSON-LD over HTTP, then `.proto` is probably beside the point.

So: “plain HTTP calls” is the alternative to coupling conformance cases to a generated client library. It keeps the test spec portable and runner-neutral. OpenAPI/proto can annotate or validate the surface, but Accord should not require codegen to run the cases.