---
id: exploring-additive-natur-kitrpm
title: 'exploring additive nature of meshes'
desc: ''
created: 1779078947746
updated: 1779078947746
participants: [djradon, codex.gpt-5.5]
conversationEventKinds: [message.assistant, message.user, tool.call, tool.result]
---

# djradon_2026-05-17_2126_03

exploring additive nature of meshes

In general, weave should only "add new", not modify old "settled" states unless a "regeneration mode" is being used. Unless source content has changed, or the ArtifactHistory is retargeted, or config has changed, we should regenerate ResourcePages unless explicit. 

This could be a challenge. 

I'm working on "ci/cd" and automated releasing of the sflo ontology itself, and if weave has already been run for a particular source, it either shouldn't be re-run or re-running shouldn't change anything. 

But the exceptions... if an external, accessible thing is the source, we could check it for changes. But that should be a modality too. 

There was a recent regression, see the conversation "extraction appended a reference catalog to the current inventory, then first weave re-derived the woven inventory from a template and forgot to carry that catalog forward." -- it's not clear to me the inventory is append-only, and using a template seems weird too. 

Give me an full analysis: can I re-run weave in an automated release action, and if so, which commands should I be using?

# gpt-5.5_2026-05-17_2126_14

I’ll trace this from the project’s own docs first, then the CLI and generation paths, so the answer is grounded in how weave currently behaves rather than how we wish it behaved. After that I’ll map the safe automation commands and the spots where the model is still leaky.

The worktree is clean, which makes this a nice read-only investigation. I’ve found the required guidance files and I’m going to anchor the analysis in those before looking at the command implementation.

The docs already make one important distinction: `weave` is validate/version/generate, while `weave generate` is supposed to regenerate current ResourcePages without adding historical states. I’m going into the implementation now to see whether that promise holds for automated release use.

A quick warning flag: the public CLI text says `prepare gh-pages` can materialize a source file, version it, generate pages, and optionally commit, but the docs do not state that this is idempotent. I’m checking tests and internals now because automation turns that missing sentence into a real release risk.

The deployment path has a deliberate byte-comparison gate: a materialized source is only payload-updated and woven when the source bytes differ, and a second identical run is tested as no-op. That supports automated publishing, but it is specifically in `prepare gh-pages`; the lower-level `weave` command is more subtle.

I’ve found the source of the “template” smell too: there are still planner/render helpers that rebuild inventory-shaped Turtle blocks, but current `prepare gh-pages` adds the source registry after weaving and has tests proving it carries that forward on reruns and source updates. I’m going to check the referenced archive note so I can separate past regression from current behavior.

The release automation shape is clearer now: `prepare gh-pages` is the safe high-level path for branch publication, but it is currently one source binding per invocation. That means a multi-artifact release needs either a commit between invocations or a controlled “dirty publish root” phase followed by one explicit publication commit.

One wrinkle I’ll call out in the answer: the SFLO example note currently uses `releases` plural, while your earlier preference in the conversation moved toward `release` singular. That choice is part of the release identity, so automation should not casually switch it after a state has been published.

Short answer: **yes, but don’t make CI run naked `weave` as the release action.** Use `weave prepare gh-pages` as the release/materialization command, and reserve plain `weave` for explicit “settle pending extracted terms” or local workflows.

**What Is Additive Now**

Weave is mostly additive for historical payload states: `prepare gh-pages` only calls `weave` when source bytes differ, and the version path fails if a requested state already exists instead of overwriting it. See [gh_pages.ts](../../../../../src/runtime/deploy/gh_pages.ts:740) and [weave.ts](../../../../../src/core/weave/weave.ts:7486).

But the **current inventory is not append-only**. It is a mutable current index. Historical state files are the append-ish record. ResourcePages are also mutable generated views. The recent `_references` regression was exactly this boundary: first weave re-rendered a current inventory and failed to preserve carried support data. That is now covered by regression tests in [extract_test.ts](../../../../../tests/integration/extract_test.ts:406), but the broader “graph-preserving inventory upserts” work is still a real TODO in [wd.todo.md](../../../../../documentation/notes/wd.todo.md:17).

**Important Caveats**

`weave generate` is explicitly a regeneration command. It writes ResourcePages from settled current state, and pages include a generated date. In CI, set `WEAVE_GENERATED_AT` to a stable value, ideally the source commit date, or reruns may update HTML just because time passed. The date source is [weave.ts](../../../../../src/runtime/weave/weave.ts:2096).

`prepare gh-pages` does not fetch remote URLs itself. It reads a local `--source-root`, records repo/ref/commit/digest, and compares bytes. For release automation, the “external thing changed?” modality should be GitHub Actions checkout/tag resolution before Weave runs.

Also: pick `release` vs `releases` once. Current CLI docs show singular `release`, while the SFLO example note currently says plural. After anything is published, do not casually switch the ArtifactHistory segment; that creates a different history.

**Recommended CI Shape**

Tag/commit SFLO first, then publish from that exact tag:

```sh
export WEAVE_CLI="$WEAVE_ROOT/src/main.ts"
export SFLO_VERSION="v0.1.0"
export SFLO_SOURCE_REPO="https://github.com/semantic-flow/sflo.git"
export SFLO_SOURCE_COMMIT="$(git -C "$SFLO_SRC" rev-parse "$SFLO_VERSION^{commit}")"
export WEAVE_GENERATED_AT="$(git -C "$SFLO_SRC" show -s --format=%cI "$SFLO_SOURCE_COMMIT")"
export WEAVE_LOG_DIR="${RUNNER_TEMP:-/tmp}/weave-logs"
```

For each released source artifact:

```sh
deno run -A "$WEAVE_CLI" prepare gh-pages \
  --source-root "$SFLO_SRC" \
  --publish-root "$SFLO_PUB" \
  --mesh-base 'https://semantic-flow.github.io/sflo/' \
  --source-path semantic-flow-core-ontology.ttl \
  --target-path semantic-flow-core-ontology.ttl \
  --designator-path ontology \
  --payload-history-segment release \
  --payload-state-segment "$SFLO_VERSION" \
  --payload-manifestation-segment ttl \
  --source-repository-url "$SFLO_SOURCE_REPO" \
  --source-ref "$SFLO_VERSION" \
  --source-commit "$SFLO_SOURCE_COMMIT"
```

Repeat for:

```text
semantic-flow-config-ontology.ttl -> config
semantic-flow-core-shacl.ttl      -> ontology/shacl
```

If you want one final publication commit for multiple artifacts, the current command is awkward because it expects a clean publish root by default. Either commit after each `prepare gh-pages`, or deliberately run subsequent commands with `--allow-dirty-publish-root` and then make one explicit git commit yourself. I’d prefer one explicit final commit in CI, with a clean-status check before starting.

For term pages:

```sh
deno run -A "$WEAVE_CLI" extract --all-terms \
  --mesh-root "$SFLO_PUB" \
  --source ontology \
  --add-source-references \
  --reference-role canonical \
  --accept-preview

deno run -A "$WEAVE_CLI" --mesh-root "$SFLO_PUB"
```

But extraction is **not a backfill mechanism**. Existing terms are skipped. If you need to repair source references broadly, use a clean publication regeneration mode/worktree, not “rerun extract and hope.”

# djradon_2026-05-17_2140_24

where do cli docs show singular release? if it's in an IRI as an Artifact History, it should be releases.

Inventory should change over settled facts, and the "current pointers" are supposed to go in meta. Why couldn't inventory be append only?

# gpt-5.5_2026-05-17_2140_29

I’ll pin down the exact doc/test locations first, then answer the ontology-design side separately. The singular/plural question is small on the surface, but it touches the artifact history identity model, so I want the references crisp.

# gpt-5.5_2026-05-17_2140_41

The quick find turned up one singular example in the user CLI doc, plus a couple of gh-pages tests that encoded the same singular path. Most of the core versioning docs/tests already use `releases`, which makes the inconsistency narrower than I remembered.

# gpt-5.5_2026-05-17_2142_12

You’re right to push on this.

The singular CLI doc example is here: [wu.cli-reference.md](../../../../../documentation/notes/wu.cli-reference.md:214)

```sh
--payload-history-segment release
```

That should be `releases` if it is naming an `sflo:ArtifactHistory` IRI. Most other CLI examples already use plural, e.g. [wu.cli-reference.md](../../../../../documentation/notes/wu.cli-reference.md:45) and [wu.cli-reference.md](../../../../../documentation/notes/wu.cli-reference.md:100). The same singular leak exists in the gh-pages tests: [deploy_gh_pages_test.ts](../../../../../tests/integration/deploy_gh_pages_test.ts:483) and [deploy_gh_pages_cli_test.ts](../../../../../tests/e2e/deploy_gh_pages_cli_test.ts:196). My previous command example should use:

```sh
--payload-history-segment releases
```

On inventory: I agree with your model more than with my earlier phrasing. There is no ontological reason inventory cannot be append-only over settled facts. In fact, the docs already say MeshInventory progression moved out to `_mesh/_meta`: [wd.codebase-overview.md](../../../../../documentation/notes/wd.codebase-overview.md:17). The current code reflects that for MeshInventory history membership: inventory renders `sflo:hasHistoricalState`, while `_mesh/_meta` carries `sflo:latestHistoricalState` / `sflo:nextStateOrdinal` progression: [weave.ts](../../../../../src/core/weave/weave.ts:5320), [weave.ts](../../../../../src/core/weave/weave.ts:3164).

The catch is implementation, not principle. Some current code still treats inventory files as canonical rewritten projections, and some current/latest/next-style predicates still live in Knop/payload inventory rendering paths rather than a clean meta/progression surface. There is even a TODO for replacing subject-level canonical rewrites with graph-preserving updates: [wd.todo.md](../../../../../documentation/notes/wd.todo.md:17).

So my corrected answer is:

Inventory *should* be append-only or append-mostly for settled membership facts. Current pointers belong in meta. The remaining challenge is separating three things the implementation still partially conflates: durable inventory facts, mutable progression facts, and generated/current projection facts such as ResourcePage materialization. Once those are cleanly split, append-only inventory becomes the sane default, not a weird special case.

# djradon_2026-05-17_2200_20

I don't really think line 17 in wd.todo is relevant any more. There should be no rewriting. Can you fill out [wa.task.2026.2026-05-17-append-onlyish-inventory.md](wa.task.2026.2026-05-17-append-onlyish-inventory.md) including required code and documenetation changes?

# gpt-5.5_2026-05-17_2200_31

I’ll turn that into a proper task note, and I’ll also check the adjacent append-only note plus the current TODO wording so the new task supersedes the stale “graph-preserving rewrites” framing cleanly.

# gpt-5.5_2026-05-17_2200_42

The target note is basically a stub right now. The “append-only-inventory” tab path doesn’t exist under this checkout’s archive dependency, so I’ll treat the `append-onlyish` note as the canonical task and fold the stronger “no rewriting” position into it.

# gpt-5.5_2026-05-17_2201_32

I have enough context now. I’m going to make this task note explicit that “graph-preserving rewrite” is still the wrong shape: the write primitive should be append/no-op/fail-closed, with repair/regeneration as a separate mode.

# gpt-5.5_2026-05-17_2203_18

The task note is now filled out with the stronger invariant. I’m going to replace the stale TODO line too, because leaving “graph-preserving rewrites” in the backlog would keep pointing future work at the wrong abstraction.

# gpt-5.5_2026-05-17_2204_03

Done. I filled out [wa.task.2026.2026-05-17-append-onlyish-inventory.md](wa.task.2026.2026-05-17-append-onlyish-inventory.md:9) with the stronger model: inventory writes are append/no-op/fail-closed, not graph-preserving rewrites.

I also replaced the stale TODO line in [wd.todo.md](../../../../../documentation/notes/wd.todo.md:17) so the backlog now points at append-onlyish inventory semantics instead of “better rewriting.”

Included in the task note:
- settled inventory facts vs current/progression facts
- explicit repair/regeneration/retraction exceptions
- CI/CD rerun contract
- required code changes
- required documentation changes
- tests and implementation checklist
- the `release` -> `releases` cleanup as part of the same semantic correction

Verification: ran `git diff --check` for both changed docs. No lint/tests run since this was documentation-only.

# djradon_2026-05-17_2227_33

"A multi-payload release therefore uses one `prepare gh-pages` run per payload." -- is this for new payloads only? Because once you integrated something, weave should know how to get it without preparing again?

# gpt-5.5_2026-05-17_2227_44

That sentence sounds over-broad to me too. I’ll look at the surrounding note and the current `prepare gh-pages` behavior, then I’ll tighten the wording if it’s misleading.

# gpt-5.5_2026-05-17_2227_55

You’re right: “per payload” collapses two cases. `prepare gh-pages` is per source materialization/update in a detached publication root; once a payload is already integrated and the publication-root bytes are current, ordinary `weave` operations should be enough. I’m going to patch that note so it says exactly that.

# gpt-5.5_2026-05-17_2228_34

Yes, exactly. That sentence was too broad.

`prepare gh-pages --source-path` is needed for a detached publication root when Weave must copy/refresh bytes from `--source-root`, compare them to the materialized publication file, and update source provenance in `_knop/_sources`.

Once a payload is already integrated in the publication root and its working bytes are current, `weave --mesh-root <publishRoot>` or targeted `weave version --mesh-root <publishRoot>` should be able to operate from the mesh-local state. If it can’t, that’s a runtime reader/design bug, not a reason to keep preparing forever.

I patched the task note here: [wa.task.2026.2026-05-17_2206-prepare-symmetry.md](wa.task.2026.2026-05-17_2206-prepare-symmetry.md:18)

Important nuance: `prepare` is not only for brand-new payloads. It is for new or refreshed external source payloads. If the source checkout changed and the publication root needs new bytes, `weave` alone should not magically fetch them from the source repo; `prepare` is currently that source-sync operation.

Verification: ran `git diff --check` for the edited task note.

# djradon_2026-05-17_2231_53

How does that compare with integrate, when bytes are outside the mesh but not the branch. Does weave alone magically fetch in the sidecar case? I think it's dependent on pinning and policy? I still sense some inappropriate asymmetry.

# gpt-5.5_2026-05-17_2231_59

I agree there’s an asymmetry smell here. I’m going to compare the actual `integrate` and `prepare gh-pages` mechanics, especially what gets recorded as the current working-byte locator and what `weave` later trusts.

# gpt-5.5_2026-05-17_2232_33

The sidecar case does “live” current-byte resolution from the recorded locator; the detached publication case materializes a copy and treats the external repo binding mostly as provenance/current-sync input for `prepare`. That’s the heart of the asymmetry, and your pinning instinct is right.

# gpt-5.5_2026-05-17_2232_38

I’m going to update the prepare-symmetry task again. The current note says “asymmetry is real,” but it doesn’t name the most important split: sidecar `integrate` can leave a live `workingLocalRelativePath`, while `prepare` creates a publication-local materialized copy plus provenance.

# gpt-5.5_2026-05-17_2233_12

Yes, your read is right.

For sidecar `integrate`, if bytes are outside `--mesh-root` but still inside the workspace/branch, Weave can record a live `sflo:workingLocalRelativePath` with a local-path grant. Later `weave` does read that current file again under policy. So it “fetches” in the narrow local sense: it resolves an allowed local working locator and reads bytes from disk. It does not fetch remote repo bytes.

For `prepare gh-pages`, the detached publication root is different: `prepare` copies source bytes into the publication root, integrates that copied file, and records repo/ref/commit/digest in `_knop/_sources`. Ordinary `weave` then reads the publication-local current file. It does not currently dereference `_knop/_sources` to refresh from the source checkout.

So the asymmetry is real:

- sidecar current mode: live local locator, policy-gated, `weave` can see source changes
- detached publication mode: materialized copy, portable, `prepare` syncs source changes
- pinned mode: read the historical state snapshot, not the live/current locator

I updated the note to name this as “live locator versus materialized copy,” and to frame it as an explicit modality split rather than a random command difference: [wa.task.2026.2026-05-17_2206-prepare-symmetry.md](wa.task.2026.2026-05-17_2206-prepare-symmetry.md:18)

Verification: `git diff --check` passed for the edited note.

# djradon_2026-05-17_2244_48

So, "Should `prepare` grow a manifest/batch input that lists all payloads, target designators, release segments, and source repository metadata?" -- i.e., it seems like we should replace (parts of) prepare with separate-repo support for integrate?

# gpt-5.5_2026-05-17_2250_09

That’s the cleaner decomposition, I think. I’m going to amend the task note so the open question stops centering “batch prepare” as the main abstraction and instead points at shared separate-repo/materialized integrate support that `prepare gh-pages` can wrap.

# djradon_2026-05-17_2250_09

i.e., it shouldn't matter whether to source is intra-repo or inter-repo.

# gpt-5.5_2026-05-17_2250_20

Yes. The repo boundary is the wrong abstraction; the real choice is source locator and materialization policy. I’m going to tune the note so it says intra-repo vs inter-repo should not determine behavior by itself.

# gpt-5.5_2026-05-17_2250_48

Yes. Same-repo vs separate-repo should be metadata and safety context, not the semantic split.

The better decomposition is:

- `integrate` or a shared source-sync layer handles source locators and materialization modes.
- `prepare gh-pages` becomes a publication preset/wrapper: bootstrap detached root, `.nojekyll`/`CNAME`, clean-worktree checks, local-path leakage validation, maybe commit.
- A manifest/batch form should target that shared source-sync/integrate layer, not be a special `prepare`-only concept.

So the behavior is driven by mode:

- live local current locator
- materialized current copy
- pinned historical state
- repository ref/commit source
- future remote access URL

I updated the note to say that directly: [wa.task.2026.2026-05-17_2206-prepare-symmetry.md](wa.task.2026.2026-05-17_2206-prepare-symmetry.md:11)

Also replaced the “batch prepare” open issue with “separate-root / separate-repository materialized integrate or source-sync support reused by `prepare gh-pages`.”

Verification: `git diff --check` passed.

# djradon_2026-05-17_2252_33

could bootstrap detached root be folded into mesh create? or it's about something different?

# gpt-5.5_2026-05-17_2252_44

I think it can be split: the semantic part of “bootstrap detached root” smells like `mesh create`, while the publication-host housekeeping is a different layer. I’ll quickly check the current mesh-create and prepare bootstrap surfaces so I don’t blur them.

# gpt-5.5_2026-05-17_2252_55

The code already points that way: `prepare gh-pages` calls `executeMeshCreate` for the actual `_mesh` bootstrap, then adds publication-root concerns around it. I’m going to note that distinction explicitly so “bootstrap detached root” doesn’t sound like a separate mesh primitive.

# gpt-5.5_2026-05-17_2253_18

Yes, the mesh part should be folded into `mesh create`, and in current code it basically already is.

`prepare gh-pages` calls `executeMeshCreate` when the publication root does not yet have `_mesh/_meta/meta.ttl` and `_mesh/_inventory/inventory.ttl`. So the SemanticMesh bootstrap is not a separate concept.

What is different is the publication wrapper around it:

- source root and publish root must be distinct
- publication worktree cleanliness checks
- stale generated-output checks
- Pages controls like `.nojekyll` / `CNAME`
- local path leakage validation
- optional local commit
- source materialization/provenance updates

So I’d phrase it as: `mesh create` owns “make this root a mesh.” `prepare gh-pages` should own “make this root safe and suitable as a portable publication target.”

I updated the task note with that split, and added the related open issue that `.nojekyll` may be too host-specific to live in `mesh create` by default: [wa.task.2026.2026-05-17_2206-prepare-symmetry.md](wa.task.2026.2026-05-17_2206-prepare-symmetry.md:17)

Verification: `git diff --check` passed.

# djradon_2026-05-17_2259_56

I'm thinking we aim to get rid of prepare gh-pages, unless there is some real branch-only work that need to be done.

"Pages controls like .nojekyll / CNAME" should happen for sidecar meshes too.
"source root and publish root must be distinct" - seems similar to "workspace root and mesh root", 
"publication worktree cleanliness checks" I'm not sure why this is important, but if it is, it could be done generically. Probably just warn if the worktree is dirty. But I can imagine situation where you were manually updating the publich worktree/mesh root, e.g. dropping a favicon... no real need to commit first.
"stale generated-output checks" -- shouldn't those happen in sidecar and whole-repo scenarios too?
"local path leakage validation" -- i'm not sure what this is
"optional local commit" -- this should be true of regular weave too?
"source materialization/provenance updates" -- also doesn't seem unique to branch-based meshes

# gpt-5.5_2026-05-17_2305_12

I’m with you directionally: most of that list sounds like generic mesh/output-root behavior, not branch-only behavior. I’ll look at the repository-options note and the current validation code so I can separate “actually gh-pages-specific” from “accidentally parked under prepare.”

# gpt-5.5_2026-05-17_2305_18

I found the current “local path leakage” check: it scans publication files for the absolute source/publish root strings. That’s really a portability/privacy validation, not gh-pages-specific. I’ll capture this as another design correction in the task note, and likely add a short clarification to repository-options since you’re looking there.

# gpt-5.5_2026-05-17_2305_46

I’m going to make the task note more decisive: “prepare gh-pages” should be a temporary preset or compatibility wrapper unless we discover true branch-only behavior. Most existing responsibilities should move to mesh creation, source sync/materialization, publication validation, or generic git commit support.

# gpt-5.5_2026-05-17_2306_08

I’m also going to nudge `wu.repository-options.md`, because the branch-published section currently makes this sound more special than it needs to be. The topology note should say the same primitives should work across whole-repo, sidecar, and branch-published outputs.

# gpt-5.5_2026-05-17_2306_53

I think your instinct is right: `prepare gh-pages` should probably dissolve unless we find real branch-only work hiding in it.

I updated the task note to make that the target: [wa.task.2026.2026-05-17_2206-prepare-symmetry.md](wa.task.2026.2026-05-17_2206-prepare-symmetry.md:11)

I also updated [wu.repository-options.md](../../../../../documentation/notes/wu.repository-options.md:28) to say branch-published meshes should not require a separate semantic command family.

My read of your list:

- `.nojekyll` / `CNAME`: yes, generic publication-host controls.
- source root vs publish root distinctness: generic source/output boundary, not branch-only.
- dirty worktree checks: git safety policy; probably warn by default, fail only for automated commit/publication modes.
- stale generated output: generic publication validation.
- local path leakage: portability/privacy validation. It currently means “public RDF must not contain absolute local checkout paths or `../` traversal.”
- optional local commit: generic mesh-root git operation.
- source materialization/provenance: generic source-sync/integrate behavior.

So the decomposition becomes: `mesh create`, source sync/materialized integrate, `weave`, publication validation/controls, optional git commit. `prepare gh-pages` can become a thin compatibility alias or go away.

Verification: `git diff --check` passed for the edited notes.

# djradon_2026-05-17_2310_41

nojekyll and cname are part of a "github pages preset", there could be a different one for gitlab or vercel, etc. i.e., let's make github-specific stuff modular.

So we have a bunch of design tasks. Should we just design into a new task note? or an informal design doc (to be created)?

# gpt-5.5_2026-05-17_2311_09

I’d split this into a stable design note plus implementation task notes. The GitHub Pages preset idea cuts across CLI shape, config, source-sync, publication validation, and repository topology docs, so a task note alone will get crowded fast. I’ll capture the modular preset point in the current task note first, then I’ll suggest the design-doc shape.

# gpt-5.5_2026-05-17_2311_31

I’m also going to adjust the repository-options wording from generic “publication-host controls” to “host presets,” since that’s the better concept and leaves room for GitLab/Vercel/etc.

# gpt-5.5_2026-05-17_2311_54

I’d create a design doc, not just keep expanding the task note.

The task note is now useful as the “why this needs design” tracker, but the actual design wants a stable developer note, probably something like:

`documentation/notes/wd.publication-pipeline.md`

or

`documentation/notes/wd.design.publication-primitives.md`

Shape I’d use:

- **Problem:** `prepare gh-pages` bundles unrelated concerns.
- **Principle:** repo topology is not semantic behavior; locator/materialization policy is.
- **Primitives:**
  - `mesh create`
  - source sync / materialized integrate
  - `weave` / `version` / `generate`
  - publication validation
  - publication-host presets
  - optional git output action
- **Presets:**
  - GitHub Pages: `.nojekyll`, `CNAME`, maybe branch/worktree defaults
  - GitLab Pages
  - Vercel/Netlify/static host
- **Migration:** thin or remove `prepare gh-pages`.

I updated the current task note to make GitHub-specific behavior modular: [wa.task.2026.2026-05-17_2206-prepare-symmetry.md](wa.task.2026.2026-05-17_2206-prepare-symmetry.md:30)

I also updated repository options so branch-published meshes don’t sound like a different semantic model: [wu.repository-options.md](../../../../../documentation/notes/wu.repository-options.md:28)

Verification: `git diff --check` passed.

# djradon_2026-05-18_0634_37

Instead of a new design document, I created a new task [wa.task.2026.2026-05-18_0627-remove-prepare.md](wa.task.2026.2026-05-18_0627-remove-prepare.md) . Please review the sf.spec.* notes in the sff repo, and updated those and the @sf.api. We don't really want to repeat ourselves, so maybe @sf.api should be more of an overview, with a sentence or two of description and then a link to a corresponding spec. So [wa.task.2026.2026-05-18_0627-remove-prepare.md](wa.task.2026.2026-05-18_0627-remove-prepare.md) is really an API documentation refinement task too.

# gpt-5.5_2026-05-18_0634_54

I’ll treat this as a docs/API cleanup task, not just a note tweak. First I’m going to locate the framework repo and read the new task note plus the existing `sf.spec.*` / `sf.api` shape so the edits land in the right layer.

# gpt-5.5_2026-05-18_0635_00

The framework checkout is under `weave/dependencies/github.com/semantic-flow/semantic-flow-framework`. The new task note is still a stub, so I’ll fill it with the API-doc refinement scope while updating the framework notes.

# gpt-5.5_2026-05-18_0635_11

There are existing deletions in the framework checkout, apparently moved/copied into the archive. I’ll leave those alone and only touch `sf.api.md` plus relevant `sf.spec.*` notes.

# gpt-5.5_2026-05-18_0636_20

I’ll treat this as an SFF API-doc cleanup plus a concrete task-note refinement. I’m going to read the spec surface for the actual behavior boundaries first, then edit the API note so it becomes a map instead of another copy of the specs.

# gpt-5.5_2026-05-18_0636_25

The new task note is intentionally blank, so I’ll use it as the place to capture the design decisions we’ve already been converging on. The first thing I see from the repo guidance is that this belongs in `sf.spec.*` because it changes externally visible operation boundaries.

# gpt-5.5_2026-05-18_0636_49

The specs already contain the right source-resolution ideas, especially in the page customization spec. The mismatch is mostly that `mesh.create` still owns GitHub Pages files, `integrate` is described too narrowly as workspace-local only, and `sf.api` repeats operational detail that should just point at behavior specs.

# gpt-5.5_2026-05-18_0637_29

I’m going to make the publication/source-sync boundary its own spec instead of stuffing that logic into `sf.api`. That gives `sf.api` a clean link target and avoids making `mesh.create`, `integrate`, or `weave` carry host-specific branch publishing details.

# gpt-5.5_2026-05-18_0638_32

Now I’m trimming `sf.api` down to the thing it wants to be: a public map of job kinds and boundaries, with the detailed behavior delegated to the specs. This should make future API doc changes less repetitive and less likely to diverge.

# gpt-5.5_2026-05-18_0639_51

I’m filling the task note as an implementation-bearing plan, not just a doc todo. It’ll capture the code split we’re implying: host presets, source sync/materialized integrate, generic publication validation, and finally thinning/removing the old `prepare gh-pages` wrapper.

# gpt-5.5_2026-05-18_0641_11

The task note now has the decisions and implementation backlog, including the API-doc work already done. I’m doing a cleanup pass now for whitespace and any contradictory wording, especially around the “mesh create no longer owns `.nojekyll`” decision.

# gpt-5.5_2026-05-18_0642_04

Implemented the API-doc/spec refinement.

I added a new SFF behavior spec for the missing boundary: [sf.spec.2026-05-18-publication-source-sync.md](../../semantic-flow-framework/notes/sf.spec.2026-05-18-publication-source-sync.md). It defines `prepare gh-pages` as a transitional wrapper at most, and splits the real work into source sync/materialized integrate, publication-host presets, generic validation, and optional git output.

I also updated:
- [sf.api.md](../../semantic-flow-framework/notes/sf.api.md) into a concise overview with links to specs.
- `mesh.create`, `integrate`, `weave`, `extract`, and page-customization specs to remove the branch-only/GitHub-specific asymmetry.
- [wa.task.2026.2026-05-18_0627-remove-prepare.md](wa.task.2026.2026-05-18_0627-remove-prepare.md) with goals, decisions, contract changes, testing, non-goals, and an implementation checklist.

Key decisions captured:
- Core `mesh.create` should not create `.nojekyll` or `CNAME`.
- GitHub Pages behavior belongs to a modular publication-host preset.
- `integrate` remains the semantic operation for source-lane payloads, intra-repo or inter-repo.
- `weave` should not fetch, sync, materialize, apply host presets, commit, or push.

Verification:
- `git -C dependencies/github.com/semantic-flow/semantic-flow-framework diff --check` passed.
- Targeted whitespace check on the edited/new files passed.
- I did not run code lint/tests because this was documentation/spec-only work. A full archive-wide `diff --check` still reports unrelated trailing whitespace in an existing conversation note, which I left untouched.

# djradon_2026-05-18_0659_52

Does "materialized integration" mean bringing the working file into the mesh? If so, that's import. integration leaves the source where it is.

# gpt-5.5_2026-05-18_0700_09

Yes, that phrase is doing damage. I’m going to tighten the docs so `integrate` means “bind an existing source in place,” and `import`/source sync is the thing that creates a governed copy inside the mesh or publication tree.

# gpt-5.5_2026-05-18_0705_33

I’m treating this as a real correction, not just wording. The docs should now say: copy into the mesh equals `import` or source-sync/import; `integrate` can bind mesh-local or policy-approved external bytes, but it does not move them.

# gpt-5.5_2026-05-18_0707_34

Yes, exactly. “Materialized integration” was a bad phrase because it smuggled copying into `integrate`.

I updated the docs to make the boundary explicit:

- [sf.spec.2026-05-18-publication-source-sync.md](../../semantic-flow-framework/notes/sf.spec.2026-05-18-publication-source-sync.md:27): `integrate` binds available bytes without moving them; `import`/source sync creates governed local copies.
- [sf.spec.2026-04-04-integrate-behavior.md](../../semantic-flow-framework/notes/sf.spec.2026-04-04-integrate-behavior.md:37): `integrate` leaves source bytes where they are.
- [sf.api.md](../../semantic-flow-framework/notes/sf.api.md:43): API overview now says copied bytes are `import`/source sync, not integration.
- [wa.task.2026.2026-05-18_0627-remove-prepare.md](wa.task.2026.2026-05-18_0627-remove-prepare.md:85): decision recorded directly.

I also scrubbed the misleading “materialized integration” wording and ran targeted `diff --check`; it passes.

# djradon_2026-05-18_0717_13

Why "Core `mesh.create` must not create `.nojekyll` or `CNAME`." -- it makes sense to me to allow providing a profile at create time  and create the mesh as necessary for publishing.

# gpt-5.5_2026-05-18_0722_31

Good catch. The real line I meant was “core `mesh.create` must not infer host files by itself,” not “a create-time workflow can never apply a publishing profile.” I’ll soften that across the task/spec notes so an explicit `--publication-profile github-pages` at create time is allowed.

# gpt-5.5_2026-05-18_0722_43

I found the overcorrection in exactly the place you selected. I’m changing it from “must not create” to “must not create implicitly,” and adding the create-time profile path as the intended ergonomic shape.

# gpt-5.5_2026-05-18_0723_29

You’re right. The better rule is:

`mesh.create` should not create `.nojekyll` or `CNAME` implicitly, or just because `meshBase` looks like GitHub Pages. But a create-time publishing profile absolutely makes sense.

I updated the wording so the intended shape is explicit composition:

- Core mesh bootstrap stays portable.
- `mesh.create --publication-profile github-pages` or API equivalent may create/validate `.nojekyll` / `CNAME` in the same user-facing operation.
- Those files remain host-preset output, not mesh inventory facts.

Updated:
- [wa.task.2026.2026-05-18_0627-remove-prepare.md](wa.task.2026.2026-05-18_0627-remove-prepare.md:81)
- [sf.spec.2026-04-03-mesh-create.md](../../semantic-flow-framework/notes/sf.spec.2026-04-03-mesh-create.md:44)
- [sf.spec.2026-05-18-publication-source-sync.md](../../semantic-flow-framework/notes/sf.spec.2026-05-18-publication-source-sync.md:73)
- [sf.api.md](../../semantic-flow-framework/notes/sf.api.md:67)

`diff --check` passes for the touched SFF docs and task note.

# djradon_2026-05-18_0831_02

So we could almost infer the publishing profile from the mesh base. Do we try for that (by default) and allow override?

# gpt-5.5_2026-05-18_0831_14

I think the safe design is “detect a likely profile, but make the applied profile explicit in the plan/result.” I’m going to capture that nuance in the task/spec notes: `auto` can exist, but it should resolve to `github-pages`/`none` visibly rather than silently smuggling host behavior into mesh creation.

# gpt-5.5_2026-05-18_0832_23

Yes, with a conservative `auto` mode.

I’d make the default something like `publicationProfile=auto`, but with two guardrails:

- Only infer from strong signals. `*.github.io` can resolve to `github-pages`; an arbitrary custom domain should resolve to `none` unless repo/CI metadata also proves GitHub Pages.
- Always report the resolved profile in the plan/result, and allow `--publication-profile none|github-pages|auto`.

I updated the notes/specs with that shape:
- `auto` is allowed and can be defaulted.
- It is visible and overrideable.
- Host files are still preset output, not hidden core mesh behavior.

`diff --check` passes for the touched docs.

# djradon_2026-05-18_0847_49

perhaps we persist the publication-profile in mesh config? So we might need a new property in the config ontology.

# gpt-5.5_2026-05-18_0848_01

That feels right to me: if `auto` resolves once, the mesh should remember the resolved publication profile so later runs don’t keep re-guessing. I’ll check the current config ontology shape and then add this as a contract/task item rather than leaving it as a hand-wave.

# gpt-5.5_2026-05-18_0848_13

There’s also an older `sf.config` note still saying GitHub Pages should be included by default, so I’m going to pull that into alignment too. I’ll check the TTL before deciding whether to add actual ontology terms now or just mark them as required contract work.

# gpt-5.5_2026-05-18_0848_52

I agree enough to make it concrete: `auto` should be a request-time resolution mode, but the mesh should persist the resolved profile. I’m going to add the config vocabulary term and update the specs/task so repeated runs read `_mesh/_config/config.ttl` before trying inference again.

# gpt-5.5_2026-05-18_0849_55

I added the concrete config vocabulary terms, not just a TODO. Next I’m doing a quick consistency pass: the older `sf.config` note now needs to agree that `auto` resolves to a persisted concrete profile, and the Turtle should at least parse if we have a parser available.

# gpt-5.5_2026-05-18_0850_24

Yes. I made that concrete.

I added config ontology vocabulary in [semantic-flow-config-ontology.ttl](../../sflo/semantic-flow-config-ontology.ttl:69):

- `sfcfg:PublicationProfile`
- `sfcfg:publicationProfile_none`
- `sfcfg:publicationProfile_githubPages`
- `sfcfg:hasPublicationProfile`

And tightened the rule: `auto` is request-time behavior, not what gets persisted. The mesh config should store the resolved concrete profile, e.g. `sfcfg:publicationProfile_githubPages` or `sfcfg:publicationProfile_none`.

I also updated:
- [sf.config.md](../../semantic-flow-framework/notes/sf.config.md:79)
- [sf.spec.2026-04-03-mesh-create.md](../../semantic-flow-framework/notes/sf.spec.2026-04-03-mesh-create.md:48)
- [sf.spec.2026-05-18-publication-source-sync.md](../../semantic-flow-framework/notes/sf.spec.2026-05-18-publication-source-sync.md:77)
- [wa.task.2026.2026-05-18_0627-remove-prepare.md](wa.task.2026.2026-05-18_0627-remove-prepare.md:48)

Validation:
- `riot --validate semantic-flow-config-ontology.ttl` passed.
- `diff --check` passed for touched files.

Suggested commit message:

```text
docs: persist resolved publication profile in mesh config

- add config ontology terms for PublicationProfile and hasPublicationProfile
- clarify auto publication profile resolves to concrete persisted mesh config
- update mesh-create and publication-source-sync specs for profile persistence
- update remove-prepare task with config ontology and testing requirements
```