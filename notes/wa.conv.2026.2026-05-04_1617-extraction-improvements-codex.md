---
id: extraction-improvements-ki0klk
title: 'extraction improvements'
desc: ''
created: 1777936644771
updated: 1777936644771
participants: [djradon, codex.gpt-5.5]
conversationEventKinds: [message.assistant, message.user, tool.call, tool.result]
---

# djradon_2026-05-04_1544_17

extraction improvements

Please review [wd.task.2026.2026-05-04-extraction-improvements.md](../../../../../documentation/notes/wd.task.2026.2026-05-04-extraction-improvements.md) 

Regarding Open Issues:
- I think it can update everything. We have history, so people can revert if necessary.
- no, treat it as an error if the state doesn't mention the term
- remove it immediately
- remove it immediately
- 16/17 should migrate

# gpt-5.5_2026-05-04_1544_23

I’ll read the repo guidance and the task note first, then I’ll update the open issues in that Dendron note using the project’s documentation conventions.

The open issues map cleanly onto the existing Decisions section. I’m going to remove the resolved Open Issues block and add explicit decisions for batch scope, pinned-state mention validation, alias removal, and the 16/17 migration path.

The settled decisions exposed two stale follow-through lines: the test expectation for `--yes` should now be rejection, and the implementation-plan decision item can be marked complete. I’m tightening those so the note does not preserve ambiguity.

Updated [wd.task.2026.2026-05-04-extraction-improvements.md](../../../../../documentation/notes/wd.task.2026.2026-05-04-extraction-improvements.md) to resolve the Open Issues into Decisions.

Changes made:
- Removed the `Open Issues` section.
- Added decisions for broad `--all-terms` updates, pinned-state term validation errors, immediate removal of both aliases, and `16/17` migration.
- Updated the `--yes` test expectation to rejection.
- Marked the Fantasy Rules `08/09` vs `16/17` decision item complete.

No linter run; this was a documentation-only edit. The file is currently untracked in git.

Suggested commit message:

```text
docs: settle extraction-source open decisions

- resolve extraction improvements open issues into explicit decisions
- require pinned source states to mention the target term
- remove deprecated source and preview aliases immediately
- keep Fantasy Rules 08/09 pinned and migrate extracted terms in 16/17
```

# djradon_2026-05-04_1545_58

ok, ready to proceed?

# gpt-5.5_2026-05-04_1546_05

Yes. I’ll start by locating the existing extraction CLI/runtime paths and the relevant tests, then I’ll make the contract changes from the task note in the smallest coherent slice.

I found the current implementation still uses `--source-designator-path` and pins every extraction source. The main change points are `src/cli/run.ts`, `src/runtime/extract/extract.ts`, `src/core/extract/extract.ts`, and the extraction-source inventory resolution used by weave/page generation.

This is larger than a flag rename because `--source-state` needs to read and validate historical bytes, while current mode should stop pinning generated inventories. I’m going to implement the extraction-source selection as an explicit runtime object first, then reuse it for single-target and all-terms extraction.

The runtime now has a first pass at resolving current sources from working bytes and pinned sources from historical snapshots, including mention validation on the pinned bytes. Next I’m updating inventory resolution so generated pages/weave can treat missing mode as `Current` and only require requested target state for pinned sources.

The current/pinned extraction path now type-checks. I’m adding the dedicated `set extraction-source` command next; for batch mode I’m going to update only discovered existing Knops that already carry an `sfc:hasExtractionSource`, so the source artifact’s own Knop does not get treated as an extracted term by accident.

The dedicated command is wired in and the source-selection code still type-checks. I’m going to run the focused extraction tests now; I expect snapshot-style assertions to catch the remaining places that still assume pinned-by-default output.

The focused extraction tests pass after updating them for current-by-default extraction and the renamed flags. I’m adding a narrow runtime test for `set extraction-source` now because the new command needs coverage that it replaces the binding instead of appending one.

The new runtime test confirms `set extraction-source` replaces the existing inventory binding, and the focused integration/e2e tests pass. I’m updating the CLI docs and task checklist next, then I’ll run formatting, lint, type-check, and the focused tests again.

The CLI docs, sidecar follow-up note, decision log, and codebase overview now reflect the new source-resolution contract. The behavior spec referenced by the task note does not appear in this repo checkout, so I’m leaving that checklist item open rather than pretending it was updated.

Formatting and lint are clean now. I’m running type-check and the focused test set once more after formatting, then I’ll inspect the diff for any accidental churn.

Focused extraction coverage is green. Since this touches shared weave source loading, I’m running the full `deno task test` once to catch regressions outside the extraction test files.

Full tests found two broader regressions. One is an expected error-type mismatch in a staged batch test; the other shows current-mode page generation reading working ontology bytes where that fixture expects the pinned release snapshot, so I’m tightening the affected behavior rather than weakening the assertions.

The full suite exposed two older assumptions that conflicted with the new pinned/current semantics and the current fixture shape. I adjusted those tests to match the new contract, and format/lint/type-check are clean again. Running the full test suite one final time now.

While reviewing the diff I found one real issue: current-mode extracted pages were adding raw-source panels under the source artifact page path instead of the extracted term page path. I’m correcting that before finalizing.

Implemented the extraction improvements slice.

Changed:
- `weave extract` now uses `--source`, `--source-state`, and `--accept-preview`.
- `--source-designator-path` and `--yes` are removed.
- New extractions default to current-tracking `sfc:ExtractionSource`.
- `--source-state` creates pinned extraction sources and validates against historical bytes.
- Added `weave set extraction-source`, including `--all-terms` migration/update support.
- Generated/weaved extracted pages now resolve current vs pinned extraction sources correctly.
- Updated CLI docs, decision log, sidecar follow-up task notes, codebase overview, and tests.

Validation:
- `deno task fmt`
- `deno task lint`
- `deno task check`
- `deno task test` passed: `277 passed | 0 failed`

Remaining open from the task note:
- Behavior spec update is still unchecked because the referenced `sf.spec.2026-04-05-extract-behavior` note is not in this checkout.
- SHACL max-count constraint is still unchecked.

Suggested commit message:

```text
feat: support current and pinned extraction sources

- make newly extracted sources current-tracking by default
- replace extract source flags with --source and --source-state
- replace --yes with --accept-preview for all-terms confirmation
- add set extraction-source for single-target and all-terms migrations
- resolve generated extracted pages through current or pinned source contracts
- update extraction docs, task notes, decision log, and regression coverage
```

# djradon_2026-05-04_1608_35

Current-mode extracted pages must update on later weave from the current source artifact, without requiring re-extraction. Is that in place?

# gpt-5.5_2026-05-04_1608_48

Yes, for **current-mode** extracted inventories.

The implemented path is:

- A current extraction writes `sfc:hasArtifactResolutionMode .../Current` and does **not** write `sfc:hasRequestedTargetState`.
- During later `weave` / `generate`, the extracted page source loader resolves the source artifact’s **current latest historical state at generation time**.
- So after the source artifact advances, re-weaving/regenerating the extracted page should read the new source state without running `weave extract` again.

Important caveat: old `08/09` fixture inventories are still pinned. Those need `weave set extraction-source ... --source ...` once to migrate them to current mode. After that, they should track future source advances without re-extraction.

I did not add the explicit “source advances, current extracted page refreshes” regression test yet. The code path is in place, but that test item is still represented in the task note’s testing section rather than checked off.

# djradon_2026-05-04_2214_38

Can't remember if the unstaged changes are yours, but regardless, please provide a commit mesg

# gpt-5.5_2026-05-04_2214_45

I’ll quickly inspect the current worktree summary so the commit message matches what is actually unstaged now.

# gpt-5.5_2026-05-04_2214_53

The current unstaged set is different from the extraction-source work I just finished. I’ll read the diff before naming it, because this looks like page/weave behavior plus two new task notes.

# djradon_2026-05-04_2214_53

(weave repo)

# gpt-5.5_2026-05-04_2214_57

For the current unstaged changes in the Weave repo, I’d use:

```text
feat: surface extraction source metadata on resource pages

- add extraction-source metadata to identifier page models and rendering
- show source artifact, pinned source state when present, and resolution mode
- thread extraction-source state through generated page contexts
- add page rendering coverage for current extraction-source metadata
- add follow-up task notes for first-payload weave refactor and page-selection split
```