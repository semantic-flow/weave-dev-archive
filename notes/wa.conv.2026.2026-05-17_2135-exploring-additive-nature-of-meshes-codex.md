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