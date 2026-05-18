---
id: sflo-release-runbook-and-uzdfgy
title: 'sflo "release runbook" and (late) release notes for v0.1.0'
desc: ''
created: 1779078942377
updated: 1779078942377
participants: [djradon, codex.gpt-5.5]
conversationEventKinds: [message.assistant, message.user, tool.call, tool.result]
---

# djradon_2026-05-17_2005_03

sflo "release runbook" and (late) release notes for v0.1.0

# gpt-5.5_2026-05-17_2005_09

I’ll treat this as a documentation task for the `sflo` area: first I’m going to read the repo guidance and existing ontology/release docs so the new notes land in the right style and place.

The worktree is currently clean. I found the top-level guidance; now I’m checking the `sflo` tree layout and any local instructions before editing.

There isn’t an `sflo` directory under this checkout root, even though your IDE tab paths are showing one. I’m going to locate the sibling/dependency copy so I don’t create docs in the wrong place.

The `sflo` content is a nested repo at `dependencies/github.com/semantic-flow/sflo`, and it already has your in-progress changes: modified `README.md`, new `LICENSE`/`NOTICE`, new release runbook and v0.1.0 notes, plus several deletions. I’ll work with those edits as the current intent and only fill/refine the release docs.

The two target notes exist but are just frontmatter shells, so this is a fill-in job rather than a new-file placement decision. I’m going to inspect the ontology files and recent history enough to make the v0.1.0 notes concrete instead of hand-wavy.

The tag `v0.1.0` is already on `main`, which matters: the runbook should include both normal future-release steps and the “late notes after tag” path. I found an older release-runbook task note too, so I’m reading that before writing the durable developer note.

There’s a local `gh-pages` worktree alongside `main`, so the release runbook can be more useful than “push tag and hope”: it should spell out the source-tag verification and the Pages publication check. I’m reading that worktree shape now.

One useful wrinkle: the tagged source contains job/prov ontology files, but the local `gh-pages` publication currently shows core/config/core-SHACL release payloads under `/sflo/`, not job/prov under their non-`/sflo` namespaces. I’m going to document that as a known publication boundary instead of pretending the public surface is cleaner than it is.

I have enough context to write the durable docs now. I’m going to replace the two empty note bodies with a release runbook and a late v0.1.0 release note, keeping internal note links in Dendron wikilink form and leaving your other in-progress sflo changes alone.

The docs are now filled in. I’m validating the Turtle files with `riot` and checking the resulting diff for whitespace issues; if the validator uncovers a real release-doc mismatch, I’ll adjust the text rather than sweeping it under the rug.

The Turtle validation passed cleanly. One more pass now: I’m checking the working-tree status and trailing whitespace on the two new notes, since untracked files do not show up in ordinary `git diff --check`.

# djradon_2026-05-17_2014_06

Don't worry about "late notes after tag", we'll just do it correctly moving forward.

# gpt-5.5_2026-05-17_2014_18

Good call. I’ll make the release docs future-oriented and treat v0.1.0 notes as normal release notes, without the “late after tag” ceremony.

I’ve stripped out the late-tag language. I’m doing a quick consistency check now so there aren’t any stale “late” references left hiding in the two docs.

The stale “late” language is gone and `git diff --check` is clean. The remaining v0.1.0 note still names the current Pages boundary plainly, which I think is useful release honesty without turning it into tag-process archaeology.

Updated the sflo docs:

- Added the durable release runbook: [ont.dev.release-runbook.md](../../sflo/notes/ont.dev.release-runbook.md:8)
- Filled v0.1.0 release notes without the “late after tag” framing: [ont.release-notes.v0.1.0.md](../../sflo/notes/ont.release-notes.v0.1.0.md:8)

Validation:
- `riot --validate semantic-flow-core-ontology.ttl semantic-flow-core-shacl.ttl semantic-flow-config-ontology.ttl semantic-flow-job-ontology.ttl semantic-flow-prov-ontology.ttl` passed
- `git diff --check` passed

Suggested commit message:

```text
docs(ontology): document v0.1.0 release process

- add v0.1.0 ontology release notes
- add source-tag and Pages publication runbook
- document current Pages publication boundary for job/prov vocabulary
```

# djradon_2026-05-17_2133_59

It makes sense to do a simple weave as part of the pages publication, at least in some cases, but "weave prepare" may have been run already. We need to verify that it is idempotent if we include it in the release action. I think ideally we do checks about whether a weave needs to run, and I think weave --dry-run could do that... might need to tighten up the contract for what happens when no files would be changed. 

Why is "weave prepare" only used in branch-based meshes? It seems like basically the same process as sidecar-based meshes.

# gpt-5.5_2026-05-17_2139_05

I’ll trace the `prepare gh-pages` implementation and the existing `weave --dry-run` behavior before answering, because this is exactly the kind of place where the intuitive product shape can be right but the current command contract is narrower than we remember.

# gpt-5.5_2026-05-17_2139_10

The CLI reference currently frames `prepare gh-pages` as branch-publication only, and that’s partly a naming artifact: the implementation has publication-root/source-root separation baked in, not necessarily “branch” as the essential concept. I’m reading the runtime planner now to see whether idempotence is already real or just implied.

# gpt-5.5_2026-05-17_2139_56

There are already tests that show `prepare gh-pages` is idempotent for bootstrap and for a materialized source. What is missing is a named contract for the dry-run no-op case, and top-level `weave` itself currently has no `--dry-run` flag, so using “weave dry run” as a release-action gate would require a new planner surface rather than just docs.

# gpt-5.5_2026-05-17_2140_18

The targeted tests pass. One subtle but important bit: `prepare gh-pages` itself already calls the weave operation when it has something to bootstrap or materialize; what we still need to know is whether an additional unconditional `weave` after a prepared publication root is a no-op or whether it rewrites generated pages/history. I’m going to verify that before I suggest a release action shape.

# gpt-5.5_2026-05-17_2140_46

I found a real contract issue: `prepare gh-pages` is idempotent, but an extra unconditional top-level `weave` after it is not currently a no-op. In a quick temp run, the extra `weave` created no new histories but rewrote the generated HTML pages, which means a release action should not blindly add `weave` until we tighten page-generation idempotence or add a dry-run/planner gate.

# gpt-5.5_2026-05-17_2141_09

I’m going to make this a little more concrete in the repo: add a no-op dry-run assertion for `prepare gh-pages`, make the dry-run output say explicitly when no publication files would change, and soften the CLI docs so “branch-published” reads as the current concrete use case rather than the only conceptual shape.

# gpt-5.5_2026-05-17_2141_20

I’ve codified the idempotence check for `prepare gh-pages --dry-run` and clarified the docs. Now I’m running the focused deploy tests plus lint/check, because this touches CLI-facing behavior and a runtime description string.

# gpt-5.5_2026-05-17_2141_43

I’m also going to reflect the release-runbook implication in the sflo runbook: `prepare gh-pages` may already weave the needed pages, while a separate unconditional `weave` is not yet a clean no-op because generated HTML can be rewritten. That keeps the release instruction from accidentally creating churn.

# gpt-5.5_2026-05-17_2142_34

You’re right to be suspicious of adding a plain `weave` to the release action. I verified the current behavior:

`prepare gh-pages` is idempotent for an already-prepared publication root with the same source bytes and source metadata. I added a test that locks that down and tightened dry-run output so it explicitly says: “No publication file changes would be made.” See [gh_pages.ts](../../../../../src/runtime/deploy/gh_pages.ts:396) and [deploy_gh_pages_test.ts](../../../../../tests/integration/deploy_gh_pages_test.ts:386).

But an extra unconditional top-level `weave` is not currently idempotent: in a temp run after `prepare`, it created no new histories but rewrote generated HTML pages. Likely culprit is generated page render timestamps. So I added a warning to the sflo runbook not to add that separate `weave` step until we have a real top-level `weave --dry-run`/planner contract or stable no-op ResourcePage generation: [ont.dev.release-runbook.md](../../sflo/notes/ont.dev.release-runbook.md:116).

On “why branch-based only”: it isn’t conceptually only branch-based. The real distinction is a detached publication root. I updated the CLI docs to say that `gh-pages` is the currently supported concrete target, while same-branch docs-rooted sidecar meshes should usually use normal `weave mesh create`, `weave integrate`, and `weave --mesh-root docs`: [wu.cli-reference.md](../../../../../documentation/notes/wu.cli-reference.md:185).

Validation passed:
- `deno test --allow-read --allow-write --allow-run --allow-env tests/integration/deploy_gh_pages_test.ts tests/e2e/deploy_gh_pages_cli_test.ts`
- `deno task lint`
- `deno task check`
- `deno task fmt:check`
- `git diff --check`

Suggested commit message:

```text
fix(deploy): make prepare dry-run no-op contract explicit

- report when prepare gh-pages would make no publication file changes
- verify settled prepare dry-runs stay empty for unchanged source bindings
- clarify detached publication root semantics in CLI docs
```