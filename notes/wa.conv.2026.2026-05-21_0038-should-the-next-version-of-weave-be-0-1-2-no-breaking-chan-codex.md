---
id: should-the-next-version-t7pfg4
title: 'should the next version of weave be 0.1.2 (no breaking chan…'
desc: ''
created: 1779349097123
updated: 1779349097123
participants: [codex.gpt-5.5]
conversationEventKinds: [message.assistant, message.user, tool.call, tool.result]
---

# User_2026-05-20_2137_35

should the next version of weave be 0.1.2 (no breaking changes) or 0.2.0 (breaking changes)? I'm going to have you write the release notes, so you can bring all the changes so far into your context.

# gpt-5.5_2026-05-20_2137_46

I’ll pull the release context from the repo rather than guessing from memory: guidance docs first, then tags/version, status, and the accumulated changes since the last release.

The repo is on `main` with local changes, and the last release tag is `v0.1.1`. I’m going to compare against that tag and also inspect the uncommitted work, since both matter for release notes and versioning.

There’s already a sizeable post-`0.1.1` body: floating repo locators, integrate/extract work, timing instrumentation, recursive weave discovery caching, plus the local “latest state” page work. I’m checking whether any of that changes user-facing contracts rather than just adding capabilities.

So far I’m not seeing a removed CLI command or rejected old mesh shape in this slice. The one important wrinkle is that generated current pages now prefer the latest historical manifestation over dirty working source when a payload has a latest state, which is a real behavior change even if it looks like a correctness fix.

I’d make it **`0.1.2`**, not `0.2.0`.

My read of the changes since `v0.1.1`: they are meaningful, but mostly additive or corrective:

- added floating repository source locators via `--source-repository-current` / `--source-repository-remote`
- preserved those locators through `weave` and `extract`
- added `WEAVE_TIMING=1` instrumentation and command-scoped caching
- fixed targeted generation child-shape classification
- changed current `RdfDocument` ResourcePages to prefer latest woven state bytes over dirty working bytes
- updated SFLO replay docs, CI guardrails, workspace/docs

The only serious “is this breaking?” wrinkle is the ResourcePage behavior change. It can break tests or expectations that current pages show mutable working file contents. But conceptually it’s a correctness/safety fix: a current published page should reflect the latest woven state, not an unwoven draft. I’d put that under **Changed Behavior**, not use it to justify `0.2.0`.

I’d reserve **`0.2.0`** for something sharper: removed commands/flags, rejected existing mesh shapes, changed default CLI workflow, or a deliberate public RDF contract break. This release feels like **`0.1.2`: compatibility-preserving, with important changed behavior called out plainly**.

One caveat: the local latest-state ResourcePage task is still uncommitted and appears only partly through its checklist. I’d either finish/scope it cleanly before release, or leave it out of `0.1.2` rather than shipping a half-contract.

# User_2026-05-20_2328_16

I forgot to branch after commit 28de5a520708d08b84d3ccfcbbeac76340fedd41 -- I want coderabbit review for all the work since then. What are my options?

# gpt-5.5_2026-05-20_2333_31

I’ll check the local branch/remote shape around that commit so the options are concrete instead of generic Git horoscope.

Your current shape is: `main` and `origin/main` are already at `ddcf499`, and `28de5a5` is behind them. So a normal PR **into `main`** from the current work would be empty. You need a review-only compare shape.

Best option: create a temporary base branch at `28de5a5`, create a temporary review branch at current `main`, and open a PR from review branch into base branch.

```sh
git switch main
git pull --ff-only

git branch review/base-28de5a5 28de5a520708d08b84d3ccfcbbeac76340fedd41
git branch review/weave-since-28de5a5 HEAD

git push origin review/base-28de5a5 review/weave-since-28de5a5
```

Then open a PR:

```sh
gh pr create \
  --base review/base-28de5a5 \
  --head review/weave-since-28de5a5 \
  --title "Review changes since 28de5a5" \
  --body "Temporary review PR for CodeRabbit. Do not merge."
```

That should give CodeRabbit the full diff since `28de5a5`.

Two caveats:

- If you mean “all work since the `v0.1.1` release tag,” the better base is `0241e1f` / `v0.1.1`, not `28de5a5`. `28de5a5` is the merge commit before the tagged release-note tightening commit.
- I would **not** rewrite `main` backward just to make a normal PR. Since `origin/main` already has the work, force-moving it would be noisy and risky for no real gain.

After review, close the temporary PR and delete both review branches.

# User_2026-05-21_0000_07

OK I did another commit. Give me these commands again, but with your recommended commits:

git branch review/base-28de5a5 28de5a520708d08b84d3ccfcbbeac76340fedd41
git branch review/weave-since-28de5a5 HEAD

# gpt-5.5_2026-05-21_0000_16

I’ll re-check the repo tips so the branch names point at the actual new HEAD and the base is the release commit I’d use for a review diff.

I’d base the review on **`v0.1.1`**, not `28de5a5`, so CodeRabbit sees only post-release work.

```sh
git branch review/base-v0.1.1 0241e1fc3bd57eb0d3d6a1ab02a170237ce3857d
git branch review/weave-since-v0.1.1 de074d642041aef538ed66737b986d24cd7e99cf
```

If you already created those local branch names and want to update them:

```sh
git branch -f review/base-v0.1.1 0241e1fc3bd57eb0d3d6a1ab02a170237ce3857d
git branch -f review/weave-since-v0.1.1 de074d642041aef538ed66737b986d24cd7e99cf
```

Then push:

```sh
git push origin review/base-v0.1.1 review/weave-since-v0.1.1
```

If those temp branches already exist on GitHub:

```sh
git push --force-with-lease origin review/base-v0.1.1 review/weave-since-v0.1.1
```

# User_2026-05-21_0023_42

Please review [[wa.review.2026-05-21_0020-v0-1-2-coderabbit]] and give each item a checkbox. An items that are totally wrongheaded or not worth doing, mark as [c]. Any that have problems but should be salvaged, improve them.

# gpt-5.5_2026-05-21_0023_53

I’ll treat this as a documentation cleanup pass: first I’ll read the review note and re-ground in the repo guidance, then I’ll patch the note in place with explicit checkbox statuses and tightened actionable wording.

The review note has eight candidate items. I’m going to verify each against the current `next/v0.1.2` code before labeling it, because a couple of CodeRabbit-style comments often turn out to be “right symptom, wrong fix.”

I found one comment that I’d cancel outright: the suggested broad startup try/catch would not reliably do what it claims when logger/timing setup itself fails, and it risks muddying the startup failure path. Several others are valid but need narrower wording, especially the host-local source grant and raw-source-panel comments.

Updated [[wa.review.2026-05-21_0020-v0-1-2-coderabbit]].

I gave every item a checkbox, marked the broad `executeWeave()` startup try/catch suggestion as `[c]`, and tightened the salvageable comments so they preserve the real issue without breaking intended behavior. Notably, I narrowed the local-path-policy item so host-local detached source grants still work, and narrowed the runtime raw-source item to the remaining canonical/extraction panel paths instead of redoing already-fixed payload loading.

No tests run; this was a markdown triage edit only.

# User_2026-05-21_0752_18

ok, implement the items.

# gpt-5.5_2026-05-21_0752_34

I’ll implement these as a small review-fix batch and keep the review note updated as I go. First I’m checking the current diffs in the touched files so I don’t trample anything already in flight.

# gpt-5.5_2026-05-21_0752_41

The Weave worktree itself is clean, so I can make scoped edits there. I’m going to update core/runtime/tests/docs together, then run focused tests before the broader linter/check gate.

# gpt-5.5_2026-05-21_0754_03

I’ve got the concrete patch shape now. The code changes are small but touch the contract in three places: page-model source metadata, floating-source equality, and host-local path policy.

# gpt-5.5_2026-05-21_0754_48

The first patch was too broad for the exact current file context, so I’m splitting it into smaller patches. Same changes, less drama from the patch matcher.

# gpt-5.5_2026-05-21_0757_53

The first formatting check only found two line-wrap issues in `src/core/weave/weave.ts`. I’ll let Deno format the TypeScript now, then I’ll run the focused checks.

# gpt-5.5_2026-05-21_0758_12

The touched files type-check. I’m running the focused tests now; if those pass I’ll do lint/check more broadly.

# gpt-5.5_2026-05-21_0759_07

Focused tests are green now for the core weave planner, local path policy, page rendering, and the integration weave/generate path. I’m moving to the repo-level lint/check gate next.

# gpt-5.5_2026-05-21_0759_33

`deno task lint` and `deno task check` are green after removing the now-unused path-only helper. Since this touched logic and tests, I’m running the full test task as the final validation pass.

# gpt-5.5_2026-05-21_0800_36

Full `deno task test` is green: 471 tests passed. I’m marking the review checklist now and adding the validation commands so the note tells the next human exactly where this landed.

# gpt-5.5_2026-05-21_0801_24

I noticed the release note content landed before the YAML fields, which would break the Dendron frontmatter. I’m fixing that placement now; good catch from the diff.

# gpt-5.5_2026-05-21_0802_11

Implemented all non-canceled CodeRabbit items. The startup try/catch item remains `[c]` in the review note, as intended.

Changed in Weave:
- Fixed the `live-server` mount typo in [wd.general-guidance.md](../../../../../documentation/notes/wd.general-guidance.md).
- Filled [release-notes.v0.1.2.md](../../../../../documentation/notes/release-notes.v0.1.2.md).
- Threaded `workingAccessUrl` and floating repository locators into first-payload created page models.
- Made extracted-source weave validation compare the full floating repository locator, not just path.
- Preserved floating locator data through runtime extracted-source loading.
- Resolved canonical/extraction working source panels through floating repository checkout resolution.
- Tightened mesh-owned repository candidate roots to stay within `workspaceRoot`, while preserving host-local grants.
- Added/updated tests for these paths.

Updated in weave-dev-archive:
- Marked every item in [wa.review.2026-05-21_0020-v0-1-2-coderabbit.md](wa.review.2026-05-21_0020-v0-1-2-coderabbit.md) as `[x]` or `[c]`.
- Added validation commands to the review note.

Validation passed:
- `deno task fmt:check`
- `deno task lint`
- `deno task check`
- `deno task test` → 471 passed

Suggested Weave commit message:

```text
fix(weave): address v0.1.2 review findings

- preserve workingAccessUrl and floating repository locators on first payload ResourcePage models
- validate extracted-source floating repository locators by URL and repository-root path
- resolve canonical and extraction source panels through allowed floating repository checkouts
- keep mesh-owned floating source candidate roots inside the configured workspace boundary
- fill v0.1.2 release notes and fix the live-server branch fixture mount example
- add focused coverage for floating extracted sources and mesh-owned path-policy boundaries
```

Suggested weave-dev-archive commit message:

```text
docs(review): close v0.1.2 CodeRabbit checklist

- mark implemented review findings as complete
- keep the broad executeWeave startup try/catch suggestion canceled
- record formatting, lint, typecheck, and test validation
```