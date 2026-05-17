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

# djradon_2026-05-16_1642_54

OK, smoketests pass. the linux binary runs. how do I inspect the draft github release body?

# gpt-5.5_2026-05-16_1643_06

Nice, that’s the good release-path kind of boring. I’ll check the workflow’s release-management step so I can give you the exact UI/CLI route for the draft body, not a generic GitHub shrug.

# gpt-5.5_2026-05-16_1643_22

The draft release tag should be `v0.1.0` from root `deno.json`.

Best CLI inspection:

```bash
gh api repos/semantic-flow/weave/releases --paginate \
  --jq '.[] | select(.tag_name == "v0.1.0") | {tag_name, draft, name, html_url, body}'
```

Just the body:

```bash
gh api repos/semantic-flow/weave/releases --paginate \
  --jq '.[] | select(.tag_name == "v0.1.0") | .body'
```

UI route: go to https://github.com/semantic-flow/weave/releases while logged in with repo write/admin access. Draft releases should be visible there; open the `v0.1.0` draft and inspect/edit the body.

To compare against what the workflow intended to upload, preview the local stripped release notes:

```bash
awk '
  BEGIN { in_frontmatter = 0; frontmatter_done = 0 }
  NR == 1 && $0 == "---" { in_frontmatter = 1; next }
  in_frontmatter && $0 == "---" { in_frontmatter = 0; frontmatter_done = 1; next }
  !in_frontmatter && frontmatter_done { print }
' documentation/notes/release-notes.v0.1.0.md
```

That `awk` is the same frontmatter-stripping logic the workflow uses.

# djradon_2026-05-16_1702_59

The publish to npm failed:

 npm error need auth This command requires you to be logged in to https://registry.npmjs.org/
npm error need auth You need to authorize this machine using `npm adduser`
npm error A complete log of this run can be found in: /home/runner/.npm/_logs/2026-05-16T23_54_20_676Z-debug-0.log
Command failed with exit code 1: npm publish --tag latest --provenance

I logged in locally, but that won't fix the github action. What do I do?

# djradon_2026-05-16_1706_40

The publish to npm failed:

 npm error need auth This command requires you to be logged in to https://registry.npmjs.org/
npm error need auth You need to authorize this machine using `npm adduser`
npm error A complete log of this run can be found in: /home/runner/.npm/_logs/2026-05-16T23_54_20_676Z-debug-0.log
Command failed with exit code 1: npm publish --tag latest --provenance

I logged in locally, but that won't fix the github action. What do I do?

# gpt-5.5_2026-05-16_1706_52

Local npm login only authenticates your machine; the GitHub runner is a fresh machine, so the workflow needs its own publish credential or trusted-publishing setup. I’m going to verify the workflow’s auth path and cross-check npm’s current official guidance before recommending the next move.

# gpt-5.5_2026-05-16_1707_43

The workflow is already wired to read `secrets.NPM_TOKEN` as `NODE_AUTH_TOKEN`; the failure means GitHub either did not have that secret, the secret was empty/unavailable to this run, or the token lacks publish permission for these packages. One wrinkle: trusted publishing is now the better long-term answer, but npm requires the package to already exist before you can attach trusted publishing, so first publish may still need a real token.

# gpt-5.5_2026-05-16_1708_05

Local `npm login` won’t help because GitHub Actions runs on a clean runner. The workflow expects a repo secret named `NPM_TOKEN`:

```yaml
NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

I checked `gh secret list --repo semantic-flow/weave` from here and it returned nothing, so it looks like the repo does not currently have that secret.

Do this for the immediate publish:

1. On npmjs.com, create a granular access token with publish rights.
   - Access Tokens → Generate New Token.
   - Give it read/write access to the `@semantic-flow` package scope/packages.
   - If 2FA is enabled for publishing, enable “Bypass 2FA” for the token.
   - Do not IP-restrict it unless you have GitHub-hosted runner IP ranges handled.

2. Add it to GitHub:

```bash
gh secret set NPM_TOKEN --repo semantic-flow/weave
```

Paste the npm token when prompted.

3. Before rerunning, check whether anything partially published:

```bash
npm view @semantic-flow/weave@0.1.0 version
npm view @semantic-flow/weave-linux-x64@0.1.0 version
npm view @semantic-flow/weave-windows-x64@0.1.0 version
npm view @semantic-flow/weave-macos-x64@0.1.0 version
npm view @semantic-flow/weave-macos-arm64@0.1.0 version
```

If they all 404, rerun the workflow publish on the same commit.

For future releases, I’d move to npm trusted publishing after the first publish. npm’s docs now recommend trusted publishing over long-lived tokens, but `npm trust` requires the package to already exist. Configure trusted publishing separately for all five packages with:

- GitHub org/user: `semantic-flow`
- Repository: `weave`
- Workflow filename: `release-manual.yml`
- Environment: blank, unless you later add a GitHub Actions environment

Sources: npm’s docs on [trusted publishing](https://docs.npmjs.com/trusted-publishers/), [access tokens](https://docs.npmjs.com/about-access-tokens/), and [provenance publishing](https://docs.npmjs.com/generating-provenance-statements/).

# djradon_2026-05-16_1730_18

OK, I set up trusted publishing. Do we need to change or clean anything up?

# gpt-5.5_2026-05-16_1730_36

Trusted publishing changes the release contract a bit: we should stop depending on `NPM_TOKEN`, and preferably let npm’s OIDC path handle provenance instead of forcing the old token-plus-provenance mode. I’ll check the publish script shape, then I’ll clean the workflow/runbook so the next run is unambiguous.

# gpt-5.5_2026-05-16_1730_53

I’m going to keep the script’s `--provenance` switch as a manual/token fallback, but remove it from the GitHub workflow path. With trusted publishing, npm should publish via OIDC and generate provenance automatically; an explicit token env var just keeps the old failure mode alive.

# gpt-5.5_2026-05-16_1730_59

All five `0.1.0` packages are visible on npm now, so the immediate publish succeeded at least once. I’m cleaning the workflow for future releases rather than touching package contents: no `NPM_TOKEN`, no explicit `--provenance` in the trusted-publishing path.

# gpt-5.5_2026-05-16_1731_10

I’m updating the runbook wording so it no longer tells future-you to chase `NPM_TOKEN` after the project has moved to trusted publishing. It will still mention token/provenance only as an emergency/manual fallback, not the normal path.

# gpt-5.5_2026-05-16_1731_27

The release logic change is just YAML plus docs; there isn’t a Deno code path to update. I’m still running the repo linter and whitespace check, then I’ll summarize exactly what to remove/verify outside the repo.

# gpt-5.5_2026-05-16_1731_45

Yes. I cleaned the normal release path so it matches trusted publishing:

- Removed `NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}` from `.github/workflows/release-manual.yml`.
- Stopped passing `--provenance` in the workflow publish path; npm trusted publishing generates provenance automatically.
- Updated `documentation/notes/dev.release-runbook.md` so it describes trusted publishing as normal, with token publishing only as a manual fallback.

I also checked npm: all five `0.1.0` packages are visible now:
`@semantic-flow/weave`, `weave-linux-x64`, `weave-windows-x64`, `weave-macos-x64`, `weave-macos-arm64`.

Still worth doing outside the repo:
- Confirm trusted publishing is configured for all five packages, not just the wrapper.
- If you added an `NPM_TOKEN` repo secret during debugging, delete it after the next trusted-publishing release succeeds:
  `gh secret delete NPM_TOKEN --repo semantic-flow/weave`

Verified locally:
- `deno task lint`
- `git diff --check`

Commit message:

```text
release: use npm trusted publishing

- remove token-based npm auth from the manual release workflow
- let npm trusted publishing generate provenance automatically
- update the release runbook for the OIDC publishing path
```

npm docs: [trusted publishing](https://docs.npmjs.com/trusted-publishers/), [provenance](https://docs.npmjs.com/generating-provenance-statements/).