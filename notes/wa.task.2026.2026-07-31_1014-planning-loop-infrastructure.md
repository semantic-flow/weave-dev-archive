---
id: wataskplanningloop20260731
title: 'Planning Loop Infrastructure — wd.read-in.jimbo, wd.queues, queue gate, /loop prompt'
desc: 'Cut 2026-07-31 10:14 by Jimbo from Dave''s /loop request. Modeled on stagecraft-lab sd.shop-protocol, sc.queues, sc.read-in(.jimbo), sc.disc.2026-07-30_1732-compaction-discipline, and tools/queue-gate.mjs. Claude adversarial r0 (3 lenses: source-fidelity ACCEPT, weave-fit CHANGES-REQUIRED, design CHANGES-REQUIRED) folded 2026-07-31.'
updated: 1785518700000
created: 1785518048182
---

## Goals

- Let Dave run the weave/sflo planning session (Jimbo) on a recurring `/loop` prompt: each wake timestamps from the shell, detects compaction mechanically, reads only what changed since a mechanically recorded last wake, fires ready work, grooms the standing surfaces, and stops the loop only when nothing is do-able AND the day's grooming floors are satisfied.
- Mint the two surfaces the loop needs: [[wd.read-in.jimbo]] (what a fresh or compaction-recovering planning session loads) and [[wd.queues]] (the ordered READY slice, distinct from the [[wd.todo]] backlog).
- Enforce the queue contract mechanically with a Deno gate (`scripts/queue-gate.ts`) so adds, pops, and checks refuse instead of drifting. The Stagecraft lesson is verbatim in their gate's header: the pointers-only rule "was written in prose and violated twelve times by its own author within the hour, so it is a tool now."

## Summary

Stagecraft's loop machinery has four load-bearing parts: a governing process note (`sd.shop-protocol`), a per-seat read-in tree (`sc.read-in` + `sc.read-in.jimbo`), a gated ready-slice queue (`sc.queues` + `tools/queue-gate.mjs`), and a paste-source loop prompt that deliberately duplicates the session mechanics. Weave adopts the same shape at two-seat scale (Jimbo plans, Kim implements, Dave rules and lands): one read-in note instead of a tree, one queue note with two sections plus a separate ungated court note ([[wa.dave-court]]), one Deno-native gate with hermetic tests, and the loop prompt kept as a paste-source section inside the read-in. [[wd.todo]] remains the backlog and delivery record; [[wd.queues]] is only the ready slice, in order.

## Discussion

### What maps to what

| Stagecraft | Weave | Adaptation |
|---|---|---|
| `sc.queues` | [[wd.queues]] (new) | two sections: Kim, Jimbo — D1 below; Dave's court is separate, per D2 as ruled |
| task notes as backlog + `sc.todo` | [[wd.todo]] + `wa.task.*` (existing, unchanged) | already split correctly: wd.todo is the backlog, task notes own the detail |
| `sc.read-in` + `sc.read-in.jimbo` | [[wd.read-in.jimbo]] (new, single note) | two seats don't need a common/per-seat split: Kim gets self-contained briefs/prompts, never a read-in — "a bite that needs a read-in is not a bite yet; it is under-briefed" |
| `sc.disc.2026-07-30_1732-compaction-discipline` | § Compaction discipline inside [[wd.read-in.jimbo]] | small shop: fold, don't mint a separate note |
| `tools/queue-gate.mjs` (Node) | `scripts/queue-gate.ts` (Deno) + `tests/scripts/queue_gate_test.ts` | 17 top-level `scripts/*.ts` precedents (20 with `scripts/release/`), zero Node tooling, and [[wd.general-guidance]] mandates Deno-native |
| `sc.changelog.<date>` owed on pop | flip the [[wd.todo]] checkbox; [[wd.decision-log]] entry if a ruling happened | weave has no changelog notes; wd.todo is the delivery record |
| `sc.daves-court` | [[wa.dave-court]] (new, archive vault) | separate ungated note, matching Stagecraft's shape — D2, ruled by Dave 2026-07-31 |
| `sc.desk.prompt-templates.jimbo` (paste source) | `## Loop prompt — paste source` section in [[wd.read-in.jimbo]] | the duplication of session mechanics into both the read-in and the prompt is deliberate; Stagecraft records both rules "were lost once each," which is why they live in two places |
| *(no equivalent — the gap this task closes twice)* | `.jimbo-state.json` wake/groom stamps | Stagecraft's own prompt says "since your last wake" with no mechanism for knowing it; weave makes the last wake and the groom floors a file read, not a memory — D8 |

### What deliberately does not port

The four-seat topology, the claim ledger, detached codex seats and watchers (`seat.sh`/`seat-watch.sh`), the `lab-edit.mjs` patch pipeline, every mail/inbox mechanism, and the `note-invariants.json` manifest beyond the queue's own headings. Weave's existing flow — `lane/*` branch off main → push → PR → CodeRabbit + CI → merge on green, with pushes, releases, PM GO, and consumer replies staying with Dave — is unchanged by this task. One Stagecraft mechanism initially dropped was put back by review r0: the READ-IN OWED return line, restored as the bite-return delta line in D3.

### Queue contract (ported from sc.queues, amended r0)

- [[wd.queues]] is the READY slice, in order. A task enters when it is fireable and leaves at delivery. The backlog stays in [[wd.todo]].
- The sections are SURFACES, not standing seats (Stagecraft ruled this of their own queue on 2026-07-31: "the sections below are SURFACES, not seats"): every item is the planning session's to fire or route; the heading names who disposes of the item, not who is sitting somewhere waiting for it.
- One line per item: `N. [[wa.task…]] — <at most one clause of why-next or blocked-on>`, comment ≤ 140 chars. If it needs a second line, it belongs in the task note.
- The admission test, mechanised: a line may appear only if its truth can change only by editing this file. The gate refuses SHAs, status words (landed/closed/complete/completed/delivered/shipped/merged), and percentages.
- An entry may appear once per section, not once per file: one task legitimately holds a fireable Kim slice and a Jimbo grooming slice at the same time. (A task waiting on Dave holds its queue entry AND a [[wa.dave-court]] card — the two surfaces are allowed to name the same task.)
- Only items whose owning task note exists enter the queue; a backlog item without a task note stays in [[wd.todo]] until one is cut. The refusal is law, not a bug.
- Pop the slice, not the task: a partially-delivered task keeps its entry until every slice closes.
- Blocker lines (`waits on`, `blocked on`, `pending`, `awaiting`, `until`) are reported on every check, not refused — the ported lesson: a blocker that quietly cleared looks exactly like a blocked lane; re-verify "is it still true, and is it still that section's?"
- Reordering is a hand edit: the gate is the only ADMITTING writer; re-prioritizing is renumber-by-hand, then `check` (which validates contiguous numbering). Stated honestly rather than pretending the gate owns all writes.

### Gate design (`scripts/queue-gate.ts`)

- Library-first: exported functions take a `rootDir` parameter; a thin CLI under `import.meta.main` resolves the repo root from `import.meta.url`. Tests import the functions against temp-dir fixture trees and never touch the real repo state (see Testing — this is what keeps them green in CI, where three of the six vaults do not exist).
- Subcommands `init` / `add <task-note> <section> "<comment>"` / `pop <task-note>` / `check` / `wake` / `groomed <duty>`; exit 0 ok · 1 violation · 2 usage; invoked as `deno task queue <subcommand> …`.
  - `init` writes the wd.queues skeleton (frontmatter, charter paragraph, both headings); refuses if the file exists. A missing queue file on any other subcommand is an explicit refusal naming `init`, never an ENOENT crash.
  - `add` refuses on the admission test, comment length, unknown section, per-section duplicates, and unresolvable notes.
  - `pop` prints what is owed: flip the [[wd.todo]] checkbox; add a [[wd.decision-log]] entry if the delivery ratified a decision; "if this closed the task, renames are now link-safe"; and the ported reminder that popping a task with only one slice landed is the wrong command.
  - `check` validates headings by exact identity, every line's contract, contiguous numbering per section, and reports (not refuses) blocker lines and renamed targets.
  - `wake` prints the previous wake stamp and any groom duty whose once-per-day floor is unmet today, then rotates the wake stamp in `.jimbo-state.json` (git-ignored). `groomed <duty>` stamps a duty. This is D8 — it makes "since the last wake" and "has the floor fired today" file reads.
- Note-existence resolution across all six vaults registered in the repo-root `dendron.yml`: parse `workspace.vaults`; for each vault the notes directory is `<fsPath>/notes` when `selfContained: true`, else `<fsPath>` itself (sflo-dendron-notes keeps notes at the vault root). A `wa.task.*` target that fails to resolve is retried as its `wa.completed.*`/`wa.cancelled.*` sibling and reported as "renamed — update or pop the entry" rather than refused, because renames are Dave's asynchronous act and a valid-when-admitted entry must not flip the gate red. This GENERALIZES Stagecraft's `vaultNoteDirs` — theirs backs only their wikilink scan (their `taskExists` is deliberately single-vault); the multi-vault rationale is their wikilink scanner's comment: checking one vault "made every legitimate cross-vault link a refusal, which is how a guard teaches people to bypass it." Weave's queue must point across vaults, so the generalization is load-bearing here.
- Exports `SECTION_HEADINGS` and `sectionsPresent`; the anti-vacuous-pass check is the heart of the port: `check` scans only lines inside a recognised section, so with no recognised heading it would validate nothing and print clean — Stagecraft audited this as "rank 9 of thirteen checks that pass on an empty population." A missing or re-punctuated heading is therefore a refusal, not a smaller scan.
- Invocation contract: `deno task queue` is the supported entry point — `deno task` pins cwd to the repo root, which keeps the task's relative `--allow-*` paths valid. `import.meta.url` self-location fixes path RESOLUTION only, not permission scoping; a direct `deno run` from elsewhere fails on permissions by design.

### Read-in design ([[wd.read-in.jimbo]])

Sections, adapted from `sc.read-in.jimbo` + the parts of `sc.read-in` that survive at two-seat scale:

1. **Session mechanics** — timestamp-first every turn (run `date` from the shell, command-driven, never typed from memory); no compaction quiz, with the five recovery rules inline: detect mechanically (a compaction summary above the turn is the trigger; the transcript's `compact_boundary` entries with `trigger=auto` are the record); re-derive before you publish (no figure that arrived through a summary is republished without re-reading its file); re-read whole the surfaces you write ([[wd.queues]], [[wa.dave-court]], [[wd.todo]], [[wd.read-in.jimbo]]); re-check lanes, branches, and running work from disk, never memory; treat every recited constant as recall, not truth.
2. **Governs** — [[wd.general-guidance]], `AGENTS.md`, and `wa.jimbo-guidance` (archive vault), whole. The read-in points at law; it never restates it (the Stagecraft anti-accretion rule).
3. **Live state** — [[wd.queues]] whole; [[wa.dave-court]] whole (open decision cards only — position is the status, swept at ruling time); [[wd.todo]] "Current Work And Next Pick" plus section headings (the audit index on demand, not by default); release state (latest `release-notes.*` plus any draft).
4. **Conventions** — task-note template and lifecycle (the `wa.task.*` → `wa.completed.*` rename is Dave's act, per `AGENTS.md`); the landing pattern; the two Codex seats (adversarial spec reviewer of task notes; `codex exec` implementation on `lane/*` branches); PM-owned outward acts; the Deno-native rule; the bite-return delta line (D3): every fired bite's return ends with `READ-IN/QUEUE DELTA: none | <what belongs where>`, and Jimbo applies deltas at harvest — this is the only mechanism by which under-briefing self-corrects.
5. **The active arc** — the tier that rots; the only tier loaded in full text; carries an explicit staleness warning discipline. Seeded with the current arc (0.6.0 draft: validateMesh API + bounded-memory validate, Stagecraft reply pending).
6. **Loop prompt — paste source** — the canonical `/loop` text (draft below), so Dave always has a current copy and edits to the mechanics propagate to both places in one edit session. Also states the seating order explicitly: a fresh session reads this read-in whole, runs `deno task queue wake` once to seed the stamp, and only then arms the loop — the prompt's "you are already seated" is an assumption the seating act must have made true.
7. **Not read-in** — `wa.conv.*`, `wa.completed.*`, `wu.*`, historical release notes, `sflo-dendron-notes` (historical-only per [[wd.general-guidance]]). They are the record, not the context.

Load cadence (ported): load at seating; reload after a compaction — that is the real trigger; never on an ordinary wake, because re-reading per wake fills the window, forces compaction, and compaction is precisely what destroys the read-in.

### Loop prompt (draft — finalized in slice 3)

```
/loop 10m — one planning wake (Jimbo). You are already seated; do NOT re-read the read-in.

TIMESTAMP FIRST: run `date '+%Y-%m-%d %H:%M %Z'` from the shell as this turn's first command and lead your reply with it. Command-driven, never typed from memory.

COMPACTION: do not quiz yourself. Detection is mechanical — if a compaction summary appears above this turn, apply wd.read-in.jimbo § Session mechanics: re-derive before you publish (no figure that reached you through a summary is republished without re-reading its file), re-read whole the surfaces you write (wd.queues, wa.dave-court, wd.todo, wd.read-in.jimbo), re-check lanes and running work from disk (git branch/status across the repos, the harness task list), and treat every recited constant as recall, not truth.

READ ONLY WHAT CHANGED: run `deno task queue wake` — it prints the last wake stamp and any groom floor unmet today, then rotates the stamp. git log --oneline --since='<the printed stamp>' across weave, weave-dev-archive, semantic-flow-framework, sflo, and accord, plus any lane/* branch with work in flight. Dave and Kim also commit here — read the diffs of anything you did not expect.

FIRE BEFORE YOU REPORT. If wd.queues holds a do-able item, ensure it is a farmable bite and fire it THIS wake — analysis and review bites as subagents; implementation per the standing grant recorded in wd.read-in.jimbo § Conventions. Running work is a slot busy, not a reason to wait. Items in the Jimbo section you do yourself or delegate. Only defer if genuinely overwhelmed — returns arriving faster than you can dispose of them — never "I'd rather finish writing this first".

GROOM per wd.read-in.jimbo — read-in currency first, then wd.queues (run `deno task queue check`), wa.dave-court (open cards only; sweep ruled ones), wd.todo sync, task-note closure flags, wd.decision-log. Each fires on its trigger AND has a once-per-day floor; `wake` reports unmet floors and `deno task queue groomed <duty>` stamps them, so the floors are a file read, not a memory.

If nothing is do-able AND `wake` reported no unmet floors, say so in one line and stop the loop rather than idling. If floors are unmet, satisfy them before stopping. Same prompt continues.
```

## Open Issues

- **Closure flag noise.** The two landed 07-29 `wa.task.*` notes are still un-renamed. With r0's amendment the gate only REPORTS renamed/landed targets rather than refusing, so the cost is a repeated report line per wake until Dave renames — judged acceptable; revisit if the report list grows past a handful.
- **Groom-duty list location.** `wake` needs the duty list to report unmet floors. Slice 1 hardcodes it in the gate (pinning it mechanically, like the headings); if the duties churn, move the list to the state file or the read-in and have the gate read it.
- **Loop-session identity.** The prompt assumes the /loop fires into the seated Jimbo conversation (Stagecraft's model). The read-in's paste-source section states the seating order (read-in whole → seed stamp → arm loop); if Dave instead arms the loop in a fresh session, wake 1 will have a stamp seeded by seating, so the mechanism still binds.

## Decisions

*(D2, D3, D7 RULED by Dave 2026-07-31 11:45; the rest proposed — ratify at spec review r1)*

- **D1 — Queue name and sections.** `wd.queues` (plural, matching the Stagecraft surface it ports) with exactly two sections: `## Kim — implementation`, `## Jimbo — planning`. The heading strings are exact identities the gate pins; renaming one is a refusal. Duplicate check is per-section, not per-file. (The Dave section moved out per D2 as ruled.)
- **D2 — Dave's court is a separate ungated note: [[wa.dave-court]] (archive vault). RULED by Dave 2026-07-31**, replacing the proposed fold-into-wd.queues. This matches Stagecraft's shape (their court was never gated) and dissolves r0 blocker F1 outright: no `Q:`-line exemption, no status-word scoping — the gate governs wd.queues alone and shrinks to two sections. The court note carries its own charter in prose: open cards only, one card per decision with a stated lean, position is the status, swept at ruling time; a ruling that spawns work becomes a task note and enters the queue. Not gate-enforced — accepted, since the court's writer set is Jimbo (writing cards) and Dave (ruling), and drift there is visible to its only reader. Trade-off accepted with the vault choice: loop-wake writes now span two repos (weave for wd.queues/wd.todo, weave-dev-archive for the court), which fits kato/task traffic already living in the archive vault.
- **D3 — What "fire" means at weave scale**, in order of preference: (a) analysis, review, and survey bites → Claude subagents in-session; (b) implementation bites → `codex exec` on a `lane/*` branch — **RULED by Dave 2026-07-31: standing grant for loop wakes**; (c) when a bite needs a ruling first, write the Kim prompt into the task note and add a [[wa.dave-court]] card. Every fired bite's brief mandates the return delta line (`READ-IN/QUEUE DELTA: none | <what belongs where>`). Pushes, releases, and consumer replies remain Dave's regardless of the grant.
- **D4 — Cadence.** Start at `/loop 10m` (weave moves slower than the Stagecraft lab's 5m); Dave adjusts by editing one character in the prompt.
- **D5 — CI posture.** The gate itself is not wired into `ci.yml` or the aggregate `ci` task: CI materializes only accord, semantic-flow-framework, and the three mesh fixture repos, so real queue entries pointing at `wa.task.*` cannot resolve there. But the gate's TESTS run in CI automatically (the `check`/`test` tasks glob all of `scripts/` and `tests/`), which is why Testing mandates hermetic fixtures. The "run `check` before any commit touching wd.queues" rule is prose, not a hook — the repo has no hook infrastructure, and adding some is out of scope; accepted risk, revisit if drift is observed (the wake-time `check` bounds the damage to one cycle).
- **D6 — The SURVEY repo set is named, never implied.** Stagecraft's own protocol carries an unresolved "⚠️ UNVERIFIED REPOSITORY SET: 'the four repos' does not name them" defect; weave does not import it. The set: weave, weave-dev-archive, semantic-flow-framework, sflo, accord. `sflo-dendron-notes` is historical-only and excluded; mesh fixture repos only when a task touches them.
- **D7 — Kim is the persona name for the implementation seat regardless of vendor. RULED by Dave 2026-07-31.** Recent weave build receipts credit "Codex (codex exec)" while prompts address Kim; the read-in states the unification and the `## Kim — implementation` heading is now safe for slice 1 to pin.
- **D8 — Wake and groom stamps are mechanical.** `.jimbo-state.json` at the repo root (git-ignored) holds the last-wake stamp and per-duty groom dates; `deno task queue wake` prints-then-rotates, `groomed <duty>` stamps. This closes two gaps r0 found: "since the last wake" previously had no mechanism (wake 1 had no bound at all, and post-compaction the bound was recall — which the same prompt forbids trusting), and the once-per-day floors previously ran on memory. Seating seeds the stamp so wake 1 is bounded.

## Contract Changes

- New `deno.json` task: `"queue": "deno run --allow-read=documentation/notes,dependencies,dendron.yml,.jimbo-state.json --allow-write=documentation/notes/wd.queues.md,.jimbo-state.json scripts/queue-gate.ts"` (subcommand and args passed through by `deno task`; cwd pinned to repo root by `deno task`, keeping the relative allow-paths valid).
- New `.gitignore` line: `.jimbo-state.json`.
- New notes: [[wd.queues]] (documentation vault, via `queue init`) and [[wd.read-in.jimbo]] (documentation vault); [[wa.dave-court]] (archive vault, hand-minted with its charter — the gate does not govern it).
- One pointer paragraph added to [[wd.general-guidance]] (the loop process exists; the queue is gated; the read-in is the seating source). Pointer only — the law lives in the new notes.
- One-line touch to [[wd.codebase-overview]]: extend the `scripts/` genre description ("release, packaging, fixture, and maintenance scripts") to include dev-process governance tooling, so the overview stays accurate (its before-merge update rule is unconditional).
- No changes to `src/`, the CLI surface, the published library, or CI workflows.

## Testing

- `tests/scripts/queue_gate_test.ts` (Deno.test, matching the `tests/scripts/*_test.ts` convention). **Hermetic by constraint, not by habit:** every test builds a temp-dir fixture tree (its own `dendron.yml`, fixture vaults in both layouts — `selfContained: true` with a `notes/` subdir and a root-notes vault mirroring sflo-dendron-notes — plus a fixture queue file) and calls the exported library functions with that root. No test reads the repo's real `dendron.yml`, `dependencies/`, or `wd.queues.md` — because CI runs these tests in a checkout where three of the six vaults do not exist.
- Ported Stagecraft behaviors: green with both headings present; red when one heading is re-punctuated (reported as a missing identity, never a silently smaller scan); red when both sections are renamed; an explicit proof the vacuous pass is real (an entry under an unrecognised heading is not scanned); import side-effect safety (`import.meta.main` guard).
- Weave-specific: `add` refuses a SHA, a status word (exercising "completed" specifically, so the regex cannot silently narrow), a percentage, and a >140-char comment, each with the reason in the refusal; `add` refuses a pointer to a nonexistent note; `add` resolves notes in both vault layouts; cross-section duplicates allowed, same-section duplicates refused; missing queue file refuses naming `init`; `init` refuses when the file exists; `check` reports (not refuses) a target renamed to `wa.completed.*` and validates contiguous numbering; `pop` refuses an item not present; `pop` prints the wd.todo/decision-log obligations; `wake` prints the prior stamp and unmet floors, and rotates; `groomed` stamps.
- Gates before merge, per the [[wd.general-guidance]] house rule verbatim: update [[wd.codebase-overview]], then `deno task fmt` and `deno task ci` green.

## Non-Goals

- No automation of `wa.task.*` → `wa.completed.*` renames (stays Dave's act, per `AGENTS.md`); the gate reports renamed targets, it never renames.
- No port of the claim ledger, detached seats/watchers, `lab-edit.mjs` pipeline, inbox/mail mechanisms, or a general `note-invariants.json` manifest (revisit only if the queue/read-in contracts demonstrably drift).
- No `closure` subcommand in slice 1 (Stagecraft's audits rename-to-changelog agreement; weave's rename cadence is Dave's and undated — defer until a real drift is observed).
- No `move`/reorder subcommand in slice 1: reordering is a hand edit plus `check`'s numbering validation (see Queue contract); add one only if hand-reordering demonstrably drifts.
- No git hooks or other before-commit enforcement infrastructure (D5's accepted risk).
- No new Dendron schema entries; `wd.queues` and `wd.read-in.jimbo` are plain `wd.*` notes.
- No changes to wd.todo's structure, the kato recording setup, the `wa.conv.*` genre, or the prompts-for-Kim conventions.

## Implementation Plan

- [x] Spec review r1: performed 2026-08-01 (Claude in-session verification against the live repo + Codex read-only refutation via `codex exec`); corrections folded — see the r1 disposition section; PM GO given by Dave's direct "review and then execute" instruction 2026-08-01 (D2/D3/D7 already ruled by Dave 2026-07-31)
- [x] Slice 1 — the gate: `scripts/queue-gate.ts` (library-first + thin CLI: init/add/pop/check/wake/groomed) + hermetic `tests/scripts/queue_gate_test.ts` (28 tests) + the `queue` task in `deno.json` + the `.gitignore` line + the one-line [[wd.codebase-overview]] touch; `deno task fmt` + `deno task ci` green
- [x] Slice 2 — the surfaces: `deno task queue init` minted [[wd.queues]]; seeded from wd.todo "Current Work And Next Pick" via `deno task queue add`, FILTERED (r1 F7 records what stayed out). Minted [[wd.read-in.jimbo]] (sections per Discussion, active arc seeded with the post-0.6.0 state) and [[wa.dave-court]] (charter + the open Dave decisions; the weave-lib@0.5.0 deprecation was already DONE 2026-07-30, so no card — r1 F2)
- [x] Slice 3 — the prompt: loop-prompt paste-source section finalized against D3/D4/D7 as ruled; pointer paragraph in [[wd.general-guidance]]; [[wd.decision-log]] entry recording D1–D8
- [ ] Dry run: one supervised wake (`/loop 10m`, one cycle) with Dave watching; adjust prompt/read-in from what the wake actually needed rather than what was predicted
- [x] Board in [[wd.todo]] Current Work with a wikilink here; report follow-ups discovered in the dry run rather than implementing them

## Adversarial review r0 — disposition (Claude, 3 lenses, 2026-07-31)

Pre-Codex internal pass; Codex spec review r1 remains owed. Verdicts: source-fidelity ACCEPT (4 findings), weave-fit CHANGES-REQUIRED (6), design CHANGES-REQUIRED (10). All findings folded above; the load-bearing ones:

- **Blocker, design:** the ported gate would refuse legitimate Dave-section decision cards (questions have no task note; "renames owed" is a status word). → first fixed by amending D2 (`Q:` lines, status-word scan scoped to Kim/Jimbo); **superseded same day by Dave's ruling** — the court is a separate ungated [[wa.dave-court]], so the exemptions were removed and the gate shrank to two sections.
- **Blocker, design:** `--since='<last wake>'` had no mechanism — wake 1 unbounded, post-compaction bound was recall. Stagecraft's prompt has the same hole; weave does not import it. → D8: `.jimbo-state.json` + `wake`/`groomed` subcommands; stop-loop rule now conditioned on floors satisfied (a stopped loop must not silently kill the daily grooms).
- **Major, weave-fit:** vault resolution is not "fsPath entries" — selfContained vaults keep notes in `<fsPath>/notes`, sflo-dendron-notes at the root; following the draft literally, every `add` of a `wa.task.*` entry would have refused. → exact rule in Gate design + both layouts in tests.
- **Major, weave-fit:** the gate's tests are CI-wired automatically (test/check globs) in a checkout missing three vaults. → hermetic-fixture constraint in Testing; D5 rewritten.
- **Major, design:** bootstrap (`add` before the queue exists), seeding note-less wd.todo items, per-file duplicate check vs the validateMesh arc's real shape, and heading identity pinned before the Kim/Codex name is ratified. → `init` subcommand + refusal-not-crash; filtered seeding; per-section duplicates; D7 cut and sequenced before slice 1.
- **Minor, design (restored port):** the READ-IN OWED return line was dropped without acknowledgment. → bite-return delta line in D3 and read-in Conventions.

## Spec review r1 — disposition (2026-08-01)

Executor note: the note assigned r1 to Codex alone; Dave's direct "review and then execute" instruction (2026-08-01) seated Claude as reviewer-then-executor, with a Codex read-only refutation (`codex exec --sandbox read-only`) run alongside as the independent pass. Findings, all folded:

- **F1, minor, fixed:** the wikilink spelled `wd.consumer-feedback-0.5.1` (dashed) resolves nowhere — the note is `wd.consumer-feedback.0.5.1`. Corrected at both wd.todo occurrences; the minted notes use the dotted name.
- **F2, minor, folded:** slice 2's example court card "the weave-lib@0.5.0 deprecation run" was already DONE 2026-07-30 (verified in [[wd.consumer-feedback.0.5.1]] Open Issues), so the court seeds without it. Open cards: send the Stagecraft reply; rename the two 07-29 notes; arm the supervised dry run; ratify D1/D4/D5/D6/D8.
- **F3, design tightening:** `check` refuses ANY unrecognised `## ` heading inside wd.queues rather than passively not scanning it — an entry under a novel heading would otherwise be invisible to every future check, which is the vacuous pass wearing a new hat. The heading refusal is the tested proof.
- **F4, design gap:** `pop <task-note>` is ambiguous when a task legitimately holds entries in both sections (which the contract allows). The gate takes `pop <note> [kim|jimbo]` and refuses the ambiguous bare form with "pop the slice, not the task."
- **F5, design detail:** the SHA scan requires a digit in the hex token — 7-char English words drawn from the hex alphabet ("defaced") are not SHAs — while 40-char all-letter hex is still refused.
- **F6, contract addition:** `@std/yaml` (jsr) joined `deno.json` imports for dendron.yml parsing; Deno-native and consistent with the existing `@std/*` usage, but it was absent from Contract Changes as drafted.
- **F7, seeding record:** entered the queue — extractor-defect-pair, planner-generalization residual, append-onlyish inventory (Kim); Stagecraft requirements collection and this task's dry-run slice (Jimbo). Stayed out per the admission law: binary payload advancement (no owning task note yet) and every checked/parked item.
- **F8, Codex refutation** (`codex exec --sandbox read-only`, second r1 pass over spec + built implementation): CHANGES-REQUIRED, 8 findings — dispositions:
  - **C1 blocker** (wake rotation can orphan the printed interval on interruption or re-seating): accepted as a property of D8's print-then-rotate design; mitigation folded into the read-in seating order — the seating session surveys since the stamp its wake printed, owning that interval. The suggested two-phase acknowledge cursor is deferred until a real loss is observed.
  - **C2 blocker** (archive half uncommitted while committed weave notes reference it): a sequencing artifact of the review running mid-flight; resolved by the archive commit that carries this disposition.
  - **C3 major** (any resolvable note was admissible, and a path-shaped name could resolve outside the vault roots): FIXED — entries must be plain `<vault>.task.…` note names; path-shaped and non-task names refuse, in `add` and `check` both; tests added.
  - **C4 major** (empty or multiline comment admitted, leaving malformed queue text the next parse refuses): FIXED — refused before writing; tests added, including proof the queue stays green after the refusal.
  - **C5 major** (admission-scan bypasses): mechanical holes FIXED — all-letter hex ≥8 chars ("deadbeef") and hex runs past 40 chars now refuse; "N percent" refuses alongside "N%"; 7-char hex-alphabet English words ("defaced") stay admissible by design. The residual point — a comment can still phrase external truth in words no lexical scan catches — is ACCEPTED: the mechanised test is the named markers, the rest stays writer judgment (the live queue's extractor line was reworded under this standard anyway).
  - **C6 minor** (`wake`/`groomed` succeeded without the queue, against the "any other subcommand" refusal sentence): FIXED — every subcommand except `init` refuses a missing queue; existence-only for `wake`/`groomed`, so a drifted-but-present queue cannot block the wake that would surface the drift.
  - **C7 major** (decision-log recorded D1–D8 with no ratification status while the court keeps five open): FIXED — the entry now carries the ruled/proposed split and points at the court card.
  - **C8 minor** (general-guidance misattributed the queue's writers): FIXED — init/add/pop write the queue, check validates, reordering is a hand edit, wake/groomed stamp only `.jimbo-state.json`.

## Rulings — Dave, 2026-07-31 11:45

- **D3(b) is a STANDING GRANT:** `codex exec` implementation bites on `lane/*` branches may fire from loop wakes without a per-session ask. Pushes, releases, and consumer replies remain Dave's.
- **D7 CONFIRMED:** Kim is the implementation-seat persona regardless of vendor; the `## Kim — implementation` heading is pinned.
- **D2 REPLACED:** Dave's court is a separate note, [[wa.dave-court]] in the archive vault, not a section of wd.queues — restoring Stagecraft's ungated-court shape and dissolving r0 blocker F1's workaround. D1 shrinks the gated queue to two sections; propagated through the queue contract, gate design, read-in, loop prompt, Testing, Contract Changes, and slices 2–3 in this note.

## Ruling — Dave, 2026-08-01

- **Pushes move to Jimbo:** branch pushes and PR opening for weave and weave-dev-archive are the planning seat's responsibility from now on ("You should take responsibility for pushes"). Releases, merge/landing GO, and consumer replies remain Dave's. Supersedes the push clauses in D3(b) and "What deliberately does not port"; propagated to [[wd.read-in.jimbo]] § Conventions and [[wd.decision-log]].
- **Seating prompt minted:** the opening prompt joins the loop prompt as a second paste-source section in [[wd.read-in.jimbo]], closing the gap where the seating act existed only as prose.
- **Renames move to Jimbo (second ruling, same day):** the `wa.task.*` → `wa.completed.*` rename is the planning seat's closure duty, done (with wikilink updates) before a task is considered finished. Supersedes the "stays Dave's act" clauses in D3, the Non-Goals, the gate's pop text, and `AGENTS.md`; the gate itself still never renames — it only reports. First exercised on the two 07-29 notes the same day.
- **D8 RULED AS AMENDED:** wake/groom stamps stay mechanical in git-ignored `.jimbo-state.json`, and the human-auditable maintenance log lives in a monthly note (`wd.maintenance.2026-08` style): `deno task queue groomed` appends its line there mechanically; hand maintenance (renames, queue hand-edits, closure sweeps) is logged there by hand.
- **D3 (a)/(b) clarified** (raised as "regarding D4 — don't we need a) AND b)?"): the fire rule is a dispatch by bite type, not a preference order — a wake fires as many do-able bites as it holds, analysis (a) and implementation (b) together when both are ready.
