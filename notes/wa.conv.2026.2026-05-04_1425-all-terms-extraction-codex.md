---
id: all-terms-extraction-u3z8t5
title: 'all-terms extraction'
desc: ''
created: 1777929919622
updated: 1777929919622
participants: [djradon, codex.gpt-5.5]
conversationEventKinds: [message.assistant, message.user, tool.call, tool.result]
---

# djradon_2026-05-04_1412_42

all-terms extraction

Let's pick up [wd.task.2026.2026-05-04-all-terms-extraction.md](../../../../../documentation/notes/wd.task.2026.2026-05-04-all-terms-extraction.md) -- 

Regarding open issues, 
- how would source documents opt out? But maybe on the CLI, you get shown a list of identifiers that would be created, and have to confirm that you want them all?
- it's be nice to be able to narrow (by type/Class, regex, identifier hierarchy etc), but we can address later.
- blank nodes can be ifnored
- only extract new terms, ignore existing terms. 
- regarding "Should term discovery ignore source-document support artifacts and generated page RDF if the source payload graph references them, or is the mesh-base IRI filter alone sufficient?"... I think this is mostly covered in "only extract new terms"... seems unlikely SF support artifact references would be present if they didn't already exist. But to make it clear, no SF support artifacts should ever get created because the are reference in an extraction source.

# gpt-5.5_2026-05-04_1412_49

I’ll read the repo guidance and the task note first, then inspect the relevant extraction code before making changes. After that I’ll update the task/doc and implementation together so the behavior matches the decisions you just made.

The task note is currently still written as “future behavior” and leaves the exact decisions open. Your latest answers settle the main policy points: batch extraction should be preview/confirm oriented, blank nodes are out, existing extracted terms are skipped, and support artifacts are never created from source references.

I’m going to implement this as a runtime batch operation plus CLI support. The CLI needs a noninteractive path for automation, so I’ll add confirmation by default and a `--yes` flag for tests/scripts; the runtime itself will expose the discovered/created/skipped lists so callers can show a preview.

Before editing, I’m tightening one important interpretation: “only extract new terms” means terms whose Knop is not already registered in the mesh inventory. I’m also treating `_mesh`, any `_knop` path, other underscore-prefixed support paths, generated `index.html` pages, and RDF file paths as non-extractable support/document artifacts for `--all-terms`.

The implementation edits are in place now. I’m updating the task note and decision log to match the behavior that landed, then I’ll format/check/lint and run the focused extract tests.

One behavior-spec wikilink referenced in existing docs does not resolve to a local file in this checkout, so I’m keeping the behavior details in the Weave task note and decision log for this pass. Next I’m running formatting and type checks; that will catch the likely rough edges from adding the batch result type.

Type checking is clean after one small logger typing fix. I’m running the focused integration and CLI extract tests now; if those pass I’ll run the repo linter as requested for significant code changes.

The first test run exposed a real ordering bug: support-artifact IRIs were being normalized before the support-artifact exclusion check, so `bob/_knop` failed validation instead of being skipped. I’m moving the support-path filter ahead of safe designator normalization.

Focused integration and e2e extract tests pass after that fix. I’m running the full linter now, then I’ll inspect the diff for any accidental churn and prepare the commit message.

Lint and targeted type checks are clean. I’m doing a final status/diff pass now, mainly to identify the touched files and make sure there are no unrelated workspace changes mixed into the summary.

The status shows unrelated pre-existing edits in the weave page/task files that I did not touch; I’m leaving them alone. I noticed one planned CLI validation case was documented but not covered yet, so I’m adding the positional-plus-`--all-terms` rejection test before the final summary.

Implemented `all-terms` extraction.

Key changes:
- Added runtime batch extraction with preview support in [extract.ts](../../../../../src/runtime/extract/extract.ts:212).
- Added CLI support for `weave extract --all-terms --source-designator-path ... [--yes]` in [run.ts](../../../../../src/cli/run.ts:288).
- Discovery now:
  - expands RDF named nodes via parsing
  - ignores blank nodes
  - skips existing Knops
  - skips support/generated artifact paths like `_mesh`, `_knop`, `index.html`, and `.ttl`
  - fails closed for unsafe mesh-scoped IRIs
- Updated task note, decision log, and [CLI reference](../../../../../documentation/notes/wu.cli-reference.md:199).
- Added integration and e2e coverage in [extract_test.ts](../../../../../tests/integration/extract_test.ts:130) and [extract_cli_test.ts](../../../../../tests/e2e/extract_cli_test.ts:269).

Verification run:
- `deno check src/runtime/extract/extract.ts src/cli/run.ts tests/integration/extract_test.ts tests/e2e/extract_cli_test.ts`
- `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --allow-read --allow-write --allow-run=git,deno --allow-env tests/integration/extract_test.ts`
- `WEAVE_GENERATED_AT=2026-05-03T00:00:00.000Z deno test --allow-read --allow-write --allow-run=git,deno --allow-env tests/e2e/extract_cli_test.ts`
- `deno task lint`

Note: the worktree also has pre-existing unrelated edits in `wd.task.2026.2026-05-02-fantasy-rules-sidecar.md`, `src/runtime/weave/pages.ts`, `src/runtime/weave/pages_test.ts`, and `src/runtime/weave/weave.ts`; I left those untouched.

Commit message:

```text
feat(extract): add source-scoped all-terms extraction

- add runtime preview and execution paths for all-terms extraction
- discover named mesh-scoped RDF terms from a selected woven source artifact
- skip blank nodes, existing Knops, support artifacts, and generated page/file paths
- fail closed for unsafe discovered mesh-scoped IRIs
- add CLI preview/confirmation with --yes for noninteractive runs
- cover runtime extraction behavior and CLI validation
- update all-terms task note, decision log, and CLI reference
```
