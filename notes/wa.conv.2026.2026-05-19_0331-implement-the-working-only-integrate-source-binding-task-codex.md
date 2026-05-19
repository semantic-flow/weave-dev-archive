---
id: implement-the-working-on-x3qne4
title: 'Implement the working-only integrate source-binding task.'
desc: ''
created: 1779186702567
updated: 1779186702567
participants: [djradon, codex.gpt-5.5]
conversationEventKinds: [message.assistant, message.user, tool.call, tool.result]
---

# djradon_2026-05-19_0320_08

Implement the working-only integrate source-binding task.

Context:
- Task note: /home/djradon/hub/semantic-flow/weave/dependencies/github.com/semantic-flow/weave-dev-archive/notes/wa.task.2026.2026-05-19_0249-source-binding-update.md
- We decided ordinary floating working-source publication should not need `weave set source-binding`.
- For external working sources, `weave integrate <source> <designatorPath> --mesh-root <publicationRoot> --grant-source-directory <sourceRoot>` should create a working-only source binding automatically.
- Users should not provide `--source-binding-id`; Weave should use a deterministic internal id for the single-source case.
- Floating working bindings should not include source repository ref/commit/path/digest or `expectsContentDigest`.
- Exact source-state evidence for pinned bindings is later work or maybe not necessary.
- Update branch fixture manifests/tests so initial branch integrations use working-only source bindings, and first-release replay weaves from existing working locators instead of `payload.update`, `prepare gh-pages`, or source-binding update.
- Do not regenerate full fixture ladders yet; run focused tests.

Please inspect current `integrate` CLI/runtime behavior, implement the change, update docs/manifests/tests as needed, run lint/check/focused fixture tests, and provide separate commit messages per repo.

# gpt-5.5_2026-05-19_0320_20

I’ll trace the existing `integrate` path and the task note first, then make the smallest set of changes that moves branch integrations onto working-only source bindings without touching full fixture regeneration.

The task note narrows this nicely: this is not a new setter, it’s an `integrate` inference change plus fixture replay cleanup. I’m going to find the CLI option resolver, the core planner shape, and the branch fixture manifests that still encode repository evidence.

The branch fixture is where the old shape is concentrated: manifest assertions still expect `--source-binding-id` plus repository ref/commit/path, and the publication-state test expects those facts in `sources.ttl`. I’ll update those expectations around the new internal `payload-source` binding and keep the old repository-backed path covered separately.

I’m going to change the runtime split now: the CLI will only accept complete repository locator options, and the runtime will synthesize a `payload-source` working binding whenever the approved working locator is extra-mesh. Repository-backed bindings stay supported, but custom source-binding ids leave the ordinary CLI surface.

The code path is patched, and I’ve added focused coverage for the new implicit working binding and for keeping repository metadata complete. I’m doing a quick targeted compile/test pass now before touching the manifest JSON, because type drift here would be noisy later.

I’m switching the branch conformance manifests to describe the new future replay: no user-supplied binding ids, no repository evidence on floating working bindings, no publication-root working source copies, and first release reduced to ordinary weave calls.

# gpt-5.5_2026-05-19_0332_25

The manifests now describe the working-only replay. I also changed the branch fixture test to validate the new manifest contract instead of pretending the already-generated old ladder has been regenerated. Next I’m formatting the TypeScript edits and then I’ll run the focused checks again.

# gpt-5.5_2026-05-19_0336_19

Focused integrate and fixture tests are green now. I’m running the repo-level lint/check gates next; after that I’ll inspect the cross-repo diffs and give you separate commit messages.

# gpt-5.5_2026-05-19_0337_35

Implemented the working-only integrate source-binding change.

In Weave, I updated `integrate` so external working sources granted with `--grant-source-directory` now get an automatic deterministic `#payload-source` binding. The CLI no longer exposes `--source-binding-id`, floating working bindings omit repository ref/commit/path/digest and `expectsContentDigest`, and repository-backed metadata still has to be provided as a complete set.

I also updated the branch fixture manifest expectations and docs so initial branch integrations use working-only source bindings, and first-release replay now weaves from existing working locators instead of `payload.update`, `prepare gh-pages`, or source-binding update. I did not regenerate the full fixture ladders.

Validation passed:
- `deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env src/core/integrate/integrate_test.ts tests/integration/integrate_test.ts tests/e2e/integrate_cli_test.ts`
- `deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env tests/scripts/fixture_ladder_test.ts tests/integration/branch_fantasy_rules_fixture_test.ts`
- `deno task lint`
- `deno task check`

Separate commit messages:

Weave repo:
```text
feat(integrate): infer working-only source bindings for external sources

- create deterministic payload-source bindings for extra-mesh working locators
- remove user-facing source binding id handling from integrate
- keep floating working bindings free of repository and digest evidence
- update integrate CLI/runtime docs and focused branch fixture tests
```

Semantic Flow Framework repo:
```text
test(branch-fixture): switch branch manifests to working source bindings

- remove source binding and repository args from initial branch integrations
- replay first release from existing working locators with weave commands only
- update Semantic Flow API/spec notes for floating working source bindings
```

I left unrelated pre-existing changes untouched, including the payload/version intent files and fixture ladder regeneration note in the Weave worktree.

# djradon_2026-05-19_1526_39

Are we done? Can you update [wa.task.2026.2026-05-19_0249-source-binding-update.md](wa.task.2026.2026-05-19_0249-source-binding-update.md)

# gpt-5.5_2026-05-19_1526_51

I’ll update the task note with the implementation outcome and validation status, then I’ll give you a crisp “done / remaining risk” read.

# gpt-5.5_2026-05-19_1527_03

The note still reads as a pre-implementation task. I’m going to mark the completed slice explicitly and leave the full ladder regeneration item open, since that was intentionally deferred.

# gpt-5.5_2026-05-19_1527_21

The task file is untracked in the archive repo, so `git diff` does not show its changes there. I’m doing a quick content check directly against the file.

# djradon_2026-05-19_1527_21

full ladder has been regenerated

# gpt-5.5_2026-05-19_1527_32

Got it. I’ll mark the regeneration item complete in the task note and adjust the result text so it reflects the current state, without renaming the task note.

# gpt-5.5_2026-05-19_1527_51

Updated the task note: full fixture ladder regeneration is now recorded as complete, and the implementation checklist is fully checked.

Yes, with that update, this task reads as done. I did not rename it to `completed`, per the repo instruction unless you explicitly ask.