---
id: weave-release-manual-tro-s1v9dq
title: 'weave release manual troubleshooting'
desc: ''
created: 1778973297687
updated: 1778973297687
participants: [djradon, codex.gpt-5.5]
conversationEventKinds: [message.assistant, message.user, tool.call, tool.result]
---

# djradon_2026-05-16_1558_19

weave release manual troubleshooting

The binaries built, but the smoke tests are failing:

No such file or directory (os error 2): readfile '/home/runner/work/weave/weave/dist/npm/npm-packages-metadata.json'
Error: Process completed with exit code 1.

The system cannot find the path specified. (os error 3): readfile 'D:\a\weave\weave\dist\npm\npm-packages-metadata.json'

No such file or directory (os error 2): readfile '/Users/runner/work/weave/weave/dist/npm/npm-packages-metadata.json'
Error: Process completed with exit code 1.

etc

# gpt-5.5_2026-05-16_1558_30

I’ll trace where `npm-packages-metadata.json` is supposed to be produced and how the release smoke tests invoke the built binaries. I’m also going to read the repo guidance first so the fix lands in the local style.

I see the worktree already has changes in `deno.json` and a new v0.1.1 release note, so I’ll treat those as yours unless I can clearly attribute a change I make. Now I’m following the missing metadata file through the release scripts.

The failure shape is already pretty suggestive: the smoke script is falling back to its default `dist/npm`, which means the `--input-dir` from the workflow is not being consumed. I’m checking whether this is an argument parser bug in the smoke script rather than an artifact upload problem.

I found the specific bug: `parseSmokeNpmInstallArgs()` treats a literal `--` as “stop parsing and return defaults”. GitHub’s `deno task smoke:npm-install -- --input-dir ...` passes that separator through to the script, so the script ignores the real `--input-dir` and looks in `dist/npm`. I’m going to make the smoke parser handle `--` the same way the assemble and publish parsers already do, then pin it with a test.

I’m also going to add a small runbook troubleshooting note for this exact symptom. It is the sort of failure future-us will otherwise have to rediscover from a CI log breadcrumb.

The code change is tiny but it touches release tooling, so I’m running the focused smoke parser tests first and then the linter. If either gives us a second clue, I’ll chase it.

Focused smoke tests and lint are clean. I’m doing one more quick repo sanity pass now: formatting check plus the diff, so I can tell you exactly what changed and whether anything else in the dirty worktree was left alone.

The targeted fix is behaving; I’m expanding slightly to the adjacent npm release-script tests because they share the same command-line conventions.

Fixed the release smoke failure. Root cause was the smoke script treating the leading `--` from `deno task smoke:npm-install -- ...` as “stop parsing”, so it ignored `--input-dir` and fell back to `dist/npm`.

Changed:
- [scripts/smoke-npm-install.ts](../../../../../scripts/smoke-npm-install.ts:52) now tolerates the task separator.
- [tests/scripts/smoke_npm_install_test.ts](../../../../../tests/scripts/smoke_npm_install_test.ts:11) covers the workflow-style invocation.
- [documentation/notes/dev.release-runbook.md](../../../../../documentation/notes/dev.release-runbook.md:125) has a troubleshooting note for this exact symptom.

Verified:
- `deno test --allow-read --allow-write tests/scripts/smoke_npm_install_test.ts`
- `deno test --allow-read --allow-write tests/scripts/assemble_npm_packages_test.ts tests/scripts/publish_npm_packages_test.ts tests/scripts/smoke_npm_install_test.ts`
- `deno task lint`
- `deno task fmt:check`
- `git diff --check`

Suggested commit message:

```text
release: honor npm smoke task input directory

- tolerate the deno task argument separator in npm install smoke parsing
- add coverage for workflow-style smoke task invocation
- document the dist/npm metadata failure mode in the release runbook
```

I left the existing `deno.json` change and untracked `release-notes.v0.1.1.md` alone.

# djradon_2026-05-16_1609_34

This is weird: " added 2 packages in 3s
Expected npm-installed weave --version to print weave 0.1.0, got weave 0.1.0
Error: Process completed with exit code 1."

# gpt-5.5_2026-05-16_1609_45

That message is a classic “the strings render the same, but one contains an invisible byte” situation. I’m going to make the smoke test show escaped output on mismatch and normalize the version line before comparing, so CI tells us the truth instead of smiling politely while failing.

The current comparison only uses `trim()`, which hides too much in the failure message and strips too little for ANSI/control-code output. I’m checking the CLI version path now so the fix stays local to the right layer.

# djradon_2026-05-16_1609_57

and the 2nd 0.1.0 is in blue

# gpt-5.5_2026-05-16_1610_29

Yep, that pins it: the installed CLI is emitting ANSI color around the version, so the log renderer paints the second `0.1.0` blue while the plain text looks identical. I’m going to compare a decolorized one-line version of stdout, and make mismatch errors use `JSON.stringify(...)` so invisible control codes become visible next time.

I’m not going to paper over this by pretending the raw strings match. The pragmatic smoke-test fix is to strip ANSI for the comparison while preserving the raw captured output for diagnostics; I’ll add a unit-level test for exactly the blue-version case.

The new comparison is in place. I’m running the focused smoke tests and lint now; if Deno dislikes the control-code regex, I’ll adjust it without getting clever.

The focused test and lint pass. One judgment call: stripping ANSI in the smoke script makes the release gate robust, but the purist fix would be making `weave --version` plain in the CLI itself. I’m checking Cliffy’s knobs before I decide whether that belongs in this patch or a follow-up.

Local `deno run` needs `--allow-env` before it reaches version output in this repo, so that quick probe hit permissions rather than the color question. I’m rerunning it with the same permissions the CLI tests use.

The source-run CLI is plain locally, so this seems specific to the packaged/native path or the CI color environment. I’m going to document the blue-version clue in the runbook too, since that failure mode is surprisingly non-obvious from the text alone.

The runbook now calls out the colored-output tell. I’m doing the focused release-script test set again, plus formatting/lint/diff checks, and then I’ll give you the concise diagnosis.

Yep, the blue `0.1.0` was the invisible difference: ANSI color codes in the captured installed-command output.

I updated [scripts/smoke-npm-install.ts](../../../../../scripts/smoke-npm-install.ts:160) so the smoke comparison strips ANSI before comparing, and the failure message now uses `JSON.stringify(...)` so future invisible characters show up as escaped bytes instead of gaslighting us. Added coverage in [tests/scripts/smoke_npm_install_test.ts](../../../../../tests/scripts/smoke_npm_install_test.ts:123), and noted the blue-version clue in [dev.release-runbook.md](../../../../../documentation/notes/dev.release-runbook.md:125).

Verified:
- `deno test --allow-read --allow-write tests/scripts/assemble_npm_packages_test.ts tests/scripts/publish_npm_packages_test.ts tests/scripts/smoke_npm_install_test.ts`
- `deno task lint`
- `deno task fmt:check`
- `git diff --check`

Commit message:

```text
release: decolorize npm smoke version output

- compare npm smoke --version output after stripping ANSI escapes
- print escaped raw version output on mismatch
- document the colored-version release smoke failure mode
```
