---
id: weave-user-documentation-bxe4iw
title: 'weave user documentation update'
desc: ''
created: 1779376722348
updated: 1779376722348
participants: [codex.gpt-5.5]
conversationEventKinds: [message.assistant, message.user, tool.call, tool.result]
---

# User_2026-05-21_0807_32

weave user documentation update

We have @wu.cli-reference.integrate -- I'd like to create dendron notes for all the major commands. Can you provide a list of notes for me to create? (

Each command note would have a succinct summary, some usage examples, and a list of impacting environment variable that are link to particular parts of [wu.environment-variables.md](../../../../../documentation/notes/wu.environment-variables.md) ? On that page I think each var gets its own heading and a succinct description with links to relative parts of the wu.cli-reference. 

Do you think that's a good idea)?

# gpt-5.5_2026-05-21_0807_41

I’ll ground this in the repo’s doc conventions first, then I’ll look at the existing integrate reference and env-var page so the note list matches what’s already there.

The env-var note is currently just a shell, so the structure can be shaped cleanly. I’m going to inventory the existing CLI reference notes and command surfaces now, so the proposed Dendron names aren’t just vibes with frontmatter.

Yes, good idea. I’d do it, with one guardrail: keep `[[wu.cli-reference]]` as the compact index/cheat sheet, and make the per-command notes task-oriented rather than a second exhaustive copy of `weave --help`.

Recommended notes:

- `[[wu.cli-reference.weave]]` - composed validate/version/generate flow. Env: `WEAVE_LOG_DIR`, `WEAVE_TIMING`, `WEAVE_GENERATED_AT`, indirect `HOME`.
- `[[wu.cli-reference.validate]]` - `mesh` vs `publication`, targeted validation. Env: `WEAVE_LOG_DIR`, `WEAVE_TIMING`, `HOME`.
- `[[wu.cli-reference.version]]` - create historical states without page generation. Env: `WEAVE_LOG_DIR`, `WEAVE_TIMING`, indirect `HOME`.
- `[[wu.cli-reference.generate]]` - render ResourcePages. Env: `WEAVE_LOG_DIR`, `WEAVE_TIMING`, `WEAVE_GENERATED_AT`.
- `[[wu.cli-reference.mesh.create]]` - initialize a mesh, sidecar mesh roots, publication profiles. Env: `WEAVE_LOG_DIR`.
- `[[wu.cli-reference.integrate]]` - already exists; expand/align it with the template. Env: `WEAVE_LOG_DIR`, `HOME`, `USERPROFILE`.
- `[[wu.cli-reference.extract]]` - single target and `--all-terms` extraction. Env: `WEAVE_LOG_DIR`, indirect `HOME`.
- `[[wu.cli-reference.set.extraction-source]]` - migrate extracted terms between working and exact source bindings. Env: `WEAVE_LOG_DIR`, indirect `HOME`.
- `[[wu.cli-reference.payload.update]]` - replace working payload bytes. Env: `WEAVE_LOG_DIR`.
- `[[wu.cli-reference.knop.create]]` - create non-payload Knop surfaces. Env: `WEAVE_LOG_DIR`.
- `[[wu.cli-reference.knop.add-reference]]` - create reference catalog/link surfaces. Env: `WEAVE_LOG_DIR`.
- `[[wu.cli-reference.set.history]]` - persist payload history intent. Env: `WEAVE_LOG_DIR`.
- `[[wu.cli-reference.set.next-state]]` - persist next state intent. Env: `WEAVE_LOG_DIR`.

I’d also create two shared reference notes:

- `[[wu.cli-reference.target-syntax]]` - `--target`, recursive targeting, payload naming fields.
- `[[wu.cli-reference.root-designator]]` - `/` as the root designator sentinel.

For `[[wu.environment-variables]]`, I’d use one heading per real Weave-relevant variable:

- `WEAVE_LOG_DIR`
- `WEAVE_TIMING`
- `WEAVE_GENERATED_AT`
- `HOME` / `USERPROFILE`

I would not treat example shell variables like `WEAVE_CLI`, `WEAVE_ROOT`, `SFLO_SRC`, or `SFLO_PUB` as Weave environment variables. They’re useful in example runbooks, but Weave itself does not read them. I’d also keep `WEAVE_KEEP_TEST_TMP` in developer docs like `[[wd.testing]]`, not user-facing CLI docs.

# User_2026-05-21_0813_29

agreed. files are created. Please change each title to look like the command (lower-casing) as you go, e.g. "weave set next-state"

# gpt-5.5_2026-05-21_0813_34

Got it. I’ll scan the new `wu.cli-reference.*` notes, then I’ll update only their frontmatter `title` values to the literal lowercase command/topic labels.

I spotted one odd filename: `documentation/notes/[[wu.cli-reference.validate.md`. I’ll include it for the title cleanup if it has normal Dendron frontmatter, but I’ll leave the filename alone since you only asked for titles.

I’m going to update the frontmatter now. I’ll cover the command-note files I found, including the three `u.cli-reference.*` files and the bracketed validate file, but I won’t rename anything without an explicit go-ahead.

Done. I updated the frontmatter `title` values for the command notes to lowercase command-style titles, including `weave set next-state`, `weave knop add-reference`, `weave mesh create`, etc. I also set the helper notes to `target syntax` and `root designator`.

Heads up: I found `documentation/notes/[[wu.cli-reference.validate.md` and three `documentation/notes/u.cli-reference.*.md` files. I updated their titles too, but left filenames unchanged. No linter run; this was frontmatter-only doc cleanup.

# User_2026-05-21_0818_25

ok, fixed the sloppy files. proceed with filling out the note shells.

# gpt-5.5_2026-05-21_0818_37

Nice, I’ll fill the shells in the Dendron style and keep the main reference as the compact hub. I’ll first rescan the note set and env-var page so I can wire the links cleanly.

# gpt-5.5_2026-05-21_0818_45

The cleaned filenames look sane now, and most command notes are empty shells apart from frontmatter. I’m going to build them from the existing compact CLI reference, then add reciprocal environment-variable anchors.

# gpt-5.5_2026-05-21_0819_57

I found that the top-level `weave` command did not have its own note yet. Since it is one of the major command surfaces, I’m going to add that note too while filling the shells.

# gpt-5.5_2026-05-21_0821_13

The first pass is in place. I’m doing a quick cross-link pass now so the compact CLI reference points to the new detailed notes instead of becoming an orphaned parallel structure.

# gpt-5.5_2026-05-21_0821_35

The cross-link scan caught my own bad `rg` regex, not a docs problem. I’ll rerun the sanity checks cleanly and then do a Markdown format check on the touched notes.

# gpt-5.5_2026-05-21_0821_46

`deno fmt --check` in this workspace is ignoring Markdown unless I force the extension, so I’m rerunning it with `--ext md`. Small tool wrinkle, not a content issue.

# gpt-5.5_2026-05-21_0822_08

The Markdown formatter is trying to rewrite Dendron frontmatter quote style when run outside the repo config, while the repo config deliberately excludes Markdown. I’ll use the project linter for the codebase sanity check and leave the Dendron frontmatter style alone.

# gpt-5.5_2026-05-21_0822_31

Filled out the command-note shells and added a missing top-level [wu.cli-reference.weave.md](../../../../../documentation/notes/wu.cli-reference.weave.md). I also expanded [wu.environment-variables.md](../../../../../documentation/notes/wu.environment-variables.md) with per-variable headings and linked the command notes to the relevant env sections.

I added a compact detailed-notes index to [wu.cli-reference.md](../../../../../documentation/notes/wu.cli-reference.md), and appended env/related sections to the existing [wu.cli-reference.integrate.md](../../../../../documentation/notes/wu.cli-reference.integrate.md).

Validation: `deno task lint` passed. I also tried a Markdown fmt check, but the repo config intentionally excludes Markdown, and forcing Deno fmt would only rewrite Dendron frontmatter quote style, so I left that alone.

Suggested commit message:

```text
docs: add detailed CLI command reference notes

- fill user-facing Dendron notes for major weave CLI commands
- add top-level weave command note and shared target/root syntax notes
- document Weave environment variables with links back to relevant command notes
- link the compact CLI reference to the expanded command notes
```