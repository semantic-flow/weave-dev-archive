---
id: for-applicationttlhomedj-iufvvo
title: 'For [application.ttl](/home/djradon/hub/semantic-flow/weave…'
desc: ''
created: 1779851389302
updated: 1779851389302
participants: [codex.gpt-5.5]
conversationEventKinds: [message.assistant, message.user, tool.call, tool.result]
---

# User_2026-05-26_2009_33

For [application.ttl](../../../../../defaults/application.ttl) and [config-resolution.ttl](../../../../../defaults/config-resolution.ttl), shouldn't we set a base for those? I'd like to use semantic-flow.github.io/weave/defaults/

If we set a base, what needs to be modified in weave to consume it correctly?

# gpt-5.5_2026-05-26_2009_46

I’ll trace how those default TTL files are parsed and referenced before guessing. First I’m reading the repo guidance you pointed me at, then I’ll follow the config-loading path from the defaults into runtime behavior.

# gpt-5.5_2026-05-26_2009_57

Small surprise already: both TTL files in the worktree currently have `@base <https://semantic-flow.github.io/weave/defaults/> .` at the top. I’m checking whether that’s enough semantically, because Turtle parsing may already resolve relative IRIs while downstream code may still compare against old relative strings.

# gpt-5.5_2026-05-26_2010_32

The important bit I’m seeing: runtime parsing is already N3-based and the parser honors Turtle `@base`. The risky spots are the hand-authored identity maps in `effective_config.ts` and tests/guardrails, because those compare resolved absolute IRIs rather than Turtle lexemes.

# gpt-5.5_2026-05-26_2010_50

I’ve got the shape now. I’m going to run the focused config/defaults tests, because they’re exactly where this base-IRI contract would break if consumption were still file-relative.

# gpt-5.5_2026-05-26_2011_00

The first test run hit Deno’s permission gate before executing tests: `n3`/`readable-stream` wants `READABLE_STREAM` env access. I’ll rerun with the narrow env/read permissions those tests need so the result is about the code instead of the harness.

# gpt-5.5_2026-05-26_2011_31

Yes. The correct base is:

```ttl
@base <https://semantic-flow.github.io/weave/defaults/> .
```

And in the current worktree it’s already present in both [application.ttl](../../../../../defaults/application.ttl:1) and [config-resolution.ttl](../../../../../defaults/config-resolution.ttl:1).

What needs to consume it correctly is mostly already in place:

- [effective_config.ts](../../../../../src/runtime/config/effective_config.ts:119) already defines `WEAVE_DEFAULTS_NAMESPACE` as `https://semantic-flow.github.io/weave/defaults/`.
- The runtime identity maps compare against absolute resolved IRIs like `.../resource-page-presentation/semantic-site-default`, so `<...>` relative terms in the TTL resolve correctly after `@base`.
- [parseTurtle](../../../../../src/runtime/config/effective_config.ts:1750) uses the N3 parser, which honors Turtle `@base`.
- [loadWeaveEffectiveConfig](../../../../../src/runtime/config/effective_config.ts:882) still reads the local packaged files; the `@base` affects RDF identity, not filesystem loading.

One caveat: Weave does not currently use `sfcfg:hasConfigResolutionConfig <config-resolution>` to discover/load `config-resolution.ttl`; it explicitly loads both default files. That’s fine for now. If you want RDF-driven discovery later, that would need a resolver that maps `https://semantic-flow.github.io/weave/defaults/config-resolution` back to the embedded/local default artifact.

I verified the current behavior with:

```bash
deno test --allow-env=READABLE_STREAM --allow-read tests/integration/defaults_guardrails_test.ts src/runtime/config/effective_config_test.ts
```

Result: `22 passed`.

# User_2026-05-26_2015_24

I feel like we need DigitalArtifacts for the stylesheet. A stylesheet-metadata.ttl and then just regular digitalartifact, at /defaults/stylesheet-metadata and /defaults/stylesheet

# gpt-5.5_2026-05-26_2015_47

I’ll model this through the existing ontology instead of just renaming strings. I’m going to inspect the stylesheet/defaults path, the artifact metadata patterns, and where the stylesheet IRI is interpreted at runtime, then I’ll make the smallest coherent change.

# gpt-5.5_2026-05-26_2015_58

There isn’t an existing diff in the files I’ll touch, which makes this clean to scope. The current `default-stylesheet` node is both config concept and artifact; I agree that’s a little muddy if we want the stylesheet bytes to be a normal `DigitalArtifact` with separate metadata.

# gpt-5.5_2026-05-26_2016_48

I’m going to rename the metadata file rather than keep a compatibility alias: pre-1.0 guidance says avoid shims, and `default-stylesheet` would keep the muddled identity around. The runtime will still use the same built-in presentation identity, but its stylesheet artifact IRI will become `https://semantic-flow.github.io/weave/defaults/stylesheet`.

# gpt-5.5_2026-05-26_2017_09

The edit is in: `application.ttl` now points to `<stylesheet>`, and the old `default-stylesheet.ttl` has become a self-describing `stylesheet-metadata.ttl` that models both the metadata RDF document and the CSS artifact. I’m checking for stale references and then I’ll run the focused tests plus lint.

# gpt-5.5_2026-05-26_2017_21

One more guardrail is worth adding here: otherwise a future edit could quietly drift back to `default-stylesheet` or collapse metadata and CSS identity again while the parser tests still pass. I’m adding a small RDF assertion around the two canonical stylesheet artifact IRIs.

# gpt-5.5_2026-05-26_2017_47

`fmt --check` caught a small wrapping preference in the guardrail test. I’m letting Deno format the touched TypeScript files, then I’ll run the focused tests and the repo lint as requested by the project guidance.

# gpt-5.5_2026-05-26_2018_04

Focused tests are green, and `deno task lint` passed across the repo. I’m doing a final diff/status pass now so I can separate my changes from the pre-existing dirty worktree in the closeout.

# gpt-5.5_2026-05-26_2018_36

Done. I made the stylesheet artifact live at `/defaults/stylesheet` and moved the RDF description into [stylesheet-metadata.ttl](../../../../../defaults/stylesheet-metadata.ttl:5).

Changes:
- [application.ttl](../../../../../defaults/application.ttl:92) now references `<stylesheet>` instead of `<default-stylesheet>`.
- Removed `defaults/default-stylesheet.ttl`.
- Added `defaults/stylesheet-metadata.ttl` with:
  - `<stylesheet-metadata>` as `sflo:DigitalArtifact, sflo:RdfDocument`
  - `<stylesheet>` as `sfcfg:ResourcePageStylesheet, sflo:DigitalArtifact`
  - working located files for both `stylesheet-metadata.ttl` and `stylesheet.css`
- Updated [effective_config.ts](../../../../../src/runtime/config/effective_config.ts:132) so runtime recognizes the canonical stylesheet IRI as `https://semantic-flow.github.io/weave/defaults/stylesheet`.
- Added a guardrail test in [defaults_guardrails_test.ts](../../../../../tests/integration/defaults_guardrails_test.ts:83).

I deliberately did not keep a `default-stylesheet` alias; that would preserve the confusing identity split we’re cleaning up.

Verification:
- `deno test --allow-env=READABLE_STREAM --allow-read tests/integration/defaults_guardrails_test.ts src/runtime/config/effective_config_test.ts` passed: `23 passed`
- `deno task lint` passed

Suggested commit message:

```text
refactor(defaults): model built-in stylesheet as canonical DigitalArtifact

- replace the default ResourcePage stylesheet IRI with /defaults/stylesheet
- move stylesheet RDF into stylesheet-metadata.ttl as a separate metadata artifact
- model both stylesheet metadata and CSS bytes with working LocatedFiles
- update effective config stylesheet identity mapping and tests
- add guardrail coverage for the stylesheet metadata/artifact split
```