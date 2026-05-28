---
id: review-and-implementhome-x4wied
title: '[$review-and-implement](/home/djradon/.codex/skills/review-…'
desc: ''
created: 1779947001912
updated: 1779947001912
participants: [codex.gpt-5.5]
conversationEventKinds: [message.assistant, message.user, tool.call, tool.result]
---

# User_2026-05-27_2225_20

[$review-and-implement](../../../../../../../../.codex/skills/review-and-implement/SKILL.md)

# gpt-5.5_2026-05-27_2225_41

Using the `review-and-implement` skill for this turn. I’ll first read the task note plus the repo’s required product vision and developer guidance, then I’ll come back with concrete suggestions before touching code.

I’ve got the repository guidance. The task note path in the IDE isn’t under the current repo root as written, so I’m locating the archive copy now before reviewing the actual task text.

The archive note is present under `dependencies/github.com/semantic-flow/weave-dev-archive/notes`. I’m reading that now, and I’ll also take a quick look at the ontology summary called out by the repo guidance so the review is anchored in the project’s model rather than just the implementation shape.

Small but important discovery: the shared artifact resolver already has exact `targetHistoricalState` support and tests for state/history mismatch. That means the task can likely shrink: page-definition should stop blocking exact state and delegate more cleanly, while fallback is the genuinely missing shared-resolver piece.

**Review**

This task is in good shape. I’d make two scope tweaks before implementation.

The main discovery: exact `targetHistoricalState` support already exists in the shared resolver. `page_definition.ts` still blocks it at [page_definition.ts](../../../../../src/runtime/weave/page_definition.ts:317), while the resolver already routes exact artifact state at [resolver.ts](../../../../../src/runtime/artifact_resolution/resolver.ts:262), implements it at [resolver.ts](../../../../../src/runtime/artifact_resolution/resolver.ts:526), and tests it at [resolver_test.ts](../../../../../src/runtime/artifact_resolution/resolver_test.ts:178). So the exact-state part should be mostly page-source adapter work, not new traversal logic.

The real missing piece is fallback. I’d avoid message-string matching here: the task wants fallback only for “unavailable/not found” style failures, but `ArtifactResolutionError` currently has no reason/code. Before enabling fallback, I’d add a small error classification, then allow fallback only for `unavailable` failures. No fallback for malformed specs, contradictory coordinates, policy-denied paths, unsupported locators, digest mismatches, ownership validation failures, or UTF-8 decode failures.

Suggested open-issue resolutions:

- Fallback: only after classified `unavailable` resolution failures.
- `targetHistoricalState` + `targetLocatedFile`: keep rejected for now.
- `expectsContentDigest`: yes, use the shared resolver’s existing digest behavior.
- Media/content-kind hints: no new negotiation; read text and fail closed on non-UTF-8.

One scope nit I’d push on: the task says “artifact-backed” broadly, but the shared exact/latest-state resolver is payload-focused right now. I’d implement this slice as payload-backed exact/latest/fallback unless we deliberately decide to broaden support-artifact historical-state resolution too.

I’m ready to implement with that conservative interpretation.

# User_2026-05-27_2241_01

Don't we have enums for how fallback should work? I couldn't find them in the ontology.

cases where two targetLocators is helpful:
- when you target an ArtifactHistory and either a state or located file, and the state or located file can't be found, you can fall 
- when you target a Historical state and a located file, but one located file isn't available, you could try another one. 

But I'm okay with a single locator restriction for now.

Regarding the nit, why would we ever target a a non-payload (i.e., I think only supporting artifacts) in this use case? So payload is fine. I think we can still target just a URL or a loose file though, right?

# gpt-5.5_2026-05-27_2241_13

I’ll check the live ontology terms before answering, because this is exactly the kind of vocabulary-memory trap that creates bad implementation assumptions.

The live core ontology has `ArtifactResolutionMode` individuals for working/latest-state, but I’m not seeing a fallback-policy class or enum in the current TTL. There are older notes that talk about “accept”/fallback policy, but the live model appears to have converged on explicit fallback specs instead.

Yep: you didn’t find fallback enums because the live ontology doesn’t appear to have them anymore. It has `ArtifactResolutionMode` with only `working` and `latestState`; fallback is currently modeled as an explicit linked `hasFallbackArtifactResolutionSpec`, not an enum/policy value.

That actually fits your cases pretty well without adding policy vocabulary yet:

- `targetArtifact + targetArtifactHistory`: means latest state within that history.
- `targetArtifact + targetArtifactHistory + targetHistoricalState`: exact state, bounded/validated by that history.
- If exact state fails and you want fallback to latest-in-history, the fallback spec can explicitly say `targetArtifact + targetArtifactHistory + latestState`.
- If one located file fails and you want another, the fallback spec can explicitly name the alternate `targetLocatedFile`.

So I’d say: no fallback enum for this slice. Use explicit fallback specs. Add fallback policy enums later only if we want algorithmic fallback like “try earlier same history” or “try any located file for this state.”

On “loose file / URL”: yes in the ontology. `targetLocalRelativePath`, `targetAccessUrl`, and direct `targetLocatedFile` are valid `ArtifactResolutionSpec` target forms, and `targetArtifact` is optional when those are enough. But for Weave page generation right now:

- loose local file: yes, already supported via `targetLocalRelativePath`, but exact only, no mode/fallback broadening yet.
- direct URL: modeled, but should remain unsupported in page generation until explicit network policy exists.
- payload artifact: yes, that’s the right artifact-backed scope for this task. I agree there’s no obvious page-content use case for targeting support artifacts here.

# User_2026-05-27_2242_53

ok, agreed. ready to implement?

# gpt-5.5_2026-05-27_2243_04

Yes. I’m going to keep it tight: payload-backed exact state through the existing resolver, one explicit fallback spec via the shared resolver, and no broadening for local helper files or URLs.

# gpt-5.5_2026-05-27_2243_37

The implementation seam is clear now. I’m going to add a graph-aware resolver entry point rather than smuggling fallback parsing into page-definition code; that keeps fallback behavior available to future `ArtifactResolutionSpec` consumers without turning the page layer into a mini resolver.

# gpt-5.5_2026-05-27_2248_57

I’ve made the first code pass. Now I’m adding the tests that lock the boundary: exact state renders, fallback only follows unavailable exact/latest artifact resolution, local helper sources stay exact, and URL sources stay out of page generation.

# gpt-5.5_2026-05-27_2250_21

Focused tests are green. I’m doing a quick diff pass now to catch accidental broadening or awkward API shape before running the heavier check/lint pass.

# gpt-5.5_2026-05-27_2250_31

One correctness wrinkle surfaced during the diff pass: the existing exact-state resolver already checked artifact/history ownership, but it did not actually require the requested state to be declared as a `HistoricalState`. I’m tightening that while we’re here because the task explicitly asks for exact state verification.

# gpt-5.5_2026-05-27_2251_27

The state verification tighten-up is in and the focused tests still pass. I’m moving to the repo-level validation now: type check first, then lint.

# gpt-5.5_2026-05-27_2251_49

Validation passed: focused tests, `deno task check`, and `deno task lint` are all green. I’m updating the task/backlog notes now, without renaming the task note.

# gpt-5.5_2026-05-27_2253_43

The code validation is already clean. I’m doing one last formatting check over the Deno-managed files, then I’ll wrap with the exact test commands and a commit-message draft.

# gpt-5.5_2026-05-27_2254_13

Implemented. The main change is in [resolver.ts](../../../../../src/runtime/artifact_resolution/resolver.ts): it now has a graph-aware `resolveArtifactResolutionSpecQuads` entry point, classifies resolution failures, and allows one explicit fallback spec only after unavailable exact/latest payload-artifact resolution. [page_definition.ts](../../../../../src/runtime/weave/page_definition.ts) now delegates artifact-backed `ResourcePageSource` resolution to that shared path, so exact `targetHistoricalState` works while local helper files and `targetAccessUrl` stay fail-closed.

I added focused coverage in [resolver_test.ts](../../../../../src/runtime/artifact_resolution/resolver_test.ts) and [page_definition_test.ts](../../../../../src/runtime/weave/page_definition_test.ts), and updated [wd.todo.md](../../../../../documentation/notes/wd.todo.md) plus the task note checklist.

Validation run:

```bash
deno test --preload=tests/support/test_tmp_harness.ts --allow-read --allow-write --allow-run=git,deno --allow-env src/runtime/artifact_resolution/resolver_test.ts src/runtime/weave/page_definition_test.ts
deno task check
deno task lint
deno task fmt:check
```

Commit message for Weave:

```text
feat(weave): support exact and fallback ResourcePageSource resolution

- add graph-aware ArtifactResolutionSpec resolution with one explicit fallback spec
- classify resolver failures so fallback only runs for unavailable exact/latest payload sources
- route artifact-backed ResourcePageSource loading through the shared resolver
- preserve fail-closed handling for local helper broadening, direct URLs, unsupported locators, validation failures, and digest mismatches
- add focused resolver and page-definition tests for exact state, fallback, diagnostics, and rejection paths
- update the Weave backlog note for the completed ResourcePageSource slice
```

Commit message for the archive note if you want it separate:

```text
docs(archive): record ResourcePageSource resolution implementation

- capture explicit fallback-spec decisions and no fallback enum assumption
- mark the ResourcePageSource resolution task checklist complete
```