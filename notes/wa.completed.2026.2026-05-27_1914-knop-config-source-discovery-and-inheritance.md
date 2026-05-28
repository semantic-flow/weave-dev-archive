---
id: ale6owbna2va4zqwrrir9z9
title: 2026 05 27_1914 Knop Config Source Discovery and Inheritance
desc: >-
  Teach Weave to discover Knop-local and Knop-inheritable config sources and
  project inherited config at runtime.
updated: 1779934461288
created: 1779934461288
---

## Goals

- Implement the next config runtime slice after [[wa.task.2026.2026-05-27_1246-config-source-discovery-and-resolution]]: Knop-local and Knop-inheritable config discovery.
- Treat `sflo:Knop sfcfg:hasConfigSource ?source` as a Knop-local config source attachment whose authority comes from the Knop attachment subject.
- Treat `sflo:Knop sfcfg:hasInheritableConfigSource ?source` as a descendant config offer, projected at runtime to matching descendant Knops.
- Support inline `sfcfg:hasConfig` and `sfcfg:hasInheritableConfig` on Knops if the existing effective-config compiler can consume the same config graph shape without extra ceremony.
- Resolve discovered `sfcfg:ConfigSource` values through the shared artifact resolver and the same fail-closed policy as mesh-local config sources.
- Use the existing pure inheritance helper in `src/runtime/config/inheritance.ts` for projection semantics instead of inventing a second inheritance model.
- Keep inherited projection runtime-derived. Do not materialize an explicit config mapping for every Knop unless a later performance or observability task proves that it is necessary.
- Leave room for an Oxigraph-backed graph query/index later, but do not require Oxigraph for the first implementation.

## Summary

The mesh-local prerequisite is effectively complete: Weave can now discover and resolve config sources attached to the mesh, and unsupported Knop attachment subjects are intentionally outside that first slice. This task adds the Knop half of the contract.

The first mover for Knop config source discovery should be authored in the current Knop metadata graph, not in inventory, source provenance, or an implicit filesystem scan. For example, the current metadata for a Knop can say that the Knop has a local config source whose target is `D/_knop/_config/config.ttl`; that target file then contains the config payload and any recursive config-source declarations it needs. The attachment statement is the bootstrap authority, while `_knop/_config/config.ttl` is just the resolved artifact content.

Inherited config should be projected by the runtime while resolving effective config for a target scope. We should not create one explicit RDF mapping per descendant Knop as part of this task. A small in-memory index of ancestor Knop attachments is enough for the first implementation; Oxigraph becomes interesting if the ancestor/source/policy query surface stops being straightforward.

## Discussion

### Prerequisite Status

[[wa.task.2026.2026-05-27_1246-config-source-discovery-and-resolution]] should now be treated as the completed prerequisite for this task, with one important caveat: it deliberately stopped before Knop attachment subjects. [[wa.task.2026.2026-05-24_1748-shared-artifact-resolution-runtime-service]] also provides the shared resolver cleanup this task should reuse.

### Authoring Location

The first-mover `sflo:Knop sfcfg:hasConfigSource ?source` statement belongs in current Knop metadata. The metadata graph is where Weave records current support facts about the Knop itself. Inventory answers what resources belong to the mesh, source provenance answers where observed resource statements came from, and `_knop/_config/config.ttl` is config content once a source attachment has authorized loading it.

A representative pattern:

```turtle
@prefix sflo: <https://semantic-flow.github.io/sflo/ontology/> .
@prefix sfcfg: <https://semantic-flow.github.io/sflo/config/> .

<.> a sflo:Knop ;
  sfcfg:hasConfigSource [
    a sfcfg:ConfigSource ;
    sflo:targetLocalRelativePath "_knop/_config/config.ttl"
  ] .
```

The exact subject IRI/base handling should match the existing current-metadata conventions for a Knop. The important part is that the attachment subject is the Knop and the resolved target path stays within the same local artifact authority rules used by mesh-local config sources.

### Discovery Model

For a target Knop, the runtime needs the ancestor Knop path from the mesh/root scope through the target scope. It should inspect current metadata for those scopes and collect four attachment families:

- `sfcfg:hasConfig` on `sflo:Knop`: local inline config for that Knop.
- `sfcfg:hasConfigSource` on `sflo:Knop`: local config source for that Knop.
- `sfcfg:hasInheritableConfig` on `sflo:Knop`: inline config offered to descendants.
- `sfcfg:hasInheritableConfigSource` on `sflo:Knop`: config source offered to descendants.

The existing config-source discovery behavior for `sflo:SemanticMesh` remains mesh-local. Mesh config may provide defaults or top-level runtime policy, but it is not the same thing as Knop ancestry.

### Projection Model

Projection should be computed during effective-config resolution. The first implementation should build an in-memory list of Knop scopes and their authored config attachments, call `resolveKnopInheritedConfigSources`, then map the resulting projected sources into the effective-config compiler.

The effective precedence should be explicit, not an accident of helper return order:

- Command-line/runtime override config wins over discovered config.
- Target Knop-local config wins over inherited Knop config.
- Nearer ancestor inherited config wins over farther ancestor inherited config when all other config-layer comparison fields are equal.
- Mesh-local config remains a broader default layer below Knop-specific config unless an existing config priority field intentionally says otherwise.

If the effective-config compiler currently cannot express "nearer inherited source wins" cleanly, this task should add that ordering in the smallest local way possible rather than changing the config model broadly.

### Oxigraph

Oxigraph is a good candidate once we need durable graph queries over many current metadata graphs, historical metadata, cross-mesh inheritance, or richer policy joins. It should not be a prerequisite for this task. A simple runtime index is easier to test and keeps the first Knop inheritance implementation close to the existing Deno/RDF helpers.

Revisit Oxigraph if any of these become true:

- Ancestor discovery needs query planning across many metadata graphs rather than a known Knop path.
- We need to answer "which Knops are affected by this inherited config source?" repeatedly and cheaply.
- Policy joins become graph-shaped enough that hand indexing starts duplicating SPARQL.
- Validation needs the same inferred projection graph that runtime uses.

## Open Issues

- Document the current-metadata subject IRI pattern outside this task note: current Knop metadata uses `@base <meshBase>` with `<${toKnopPath(scopeKey)}>` as the active `sflo:Knop` subject.
- Decide whether explicit RDF terms already exist for inheritance offer/acceptance policies, or whether the first runtime slice should use only the default behavior represented by `resolveKnopInheritedConfigSources`.
- Inline `sfcfg:hasConfig` / `sfcfg:hasInheritableConfig` did not ship in the first implementation. Keep it as a tiny follow-up if the inline graph shape is still useful after source-based config settles.
- Decide how much trace output should be user-visible versus debug-only: inherited config is hard to reason about without a source/ancestor/projection trace, but normal CLI output should stay quiet.
- Root Knop metadata is part of the ancestor path when `_knop/_meta/meta.ttl` exists. Missing ancestor metadata files are skipped rather than treated as errors.

## Decisions

- The prerequisite mesh-local config-source work is complete enough for this task to proceed.
- First-mover Knop config-source attachment statements are authored in current Knop metadata.
- `_knop/_config/config.ttl` is a likely conventional target for Knop config content, but it should not be auto-loaded by convention without an authorizing metadata attachment in the first implementation.
- Inherited config projection is runtime-derived and should not be persisted as one explicit mapping per Knop.
- Do not make Oxigraph a dependency for the first Knop config inheritance slice.
- Reuse the shared artifact resolver and preserve the same local-only, digest-aware, fail-closed posture established by the mesh-local config-source task.
- Initial runtime wiring applies Knop config only when the operation resolves to exactly one selected/weaveable target. Multi-target operations need per-target effective config before Knop-local config can be broadened safely.

## Contract Changes

- Effective config resolution for Knop-scoped operations can include Knop-local and inherited Knop config sources, not only mesh-local config and command-provided config.
- Current Knop metadata may attach `sfcfg:ConfigSource` values using `sfcfg:hasConfigSource` and `sfcfg:hasInheritableConfigSource`.
- The runtime should expose enough trace data to explain which Knop authored a source, whether the source was local or inherited, and how it was projected to the target Knop.
- Invalid, unsupported, ambiguous, remote, or unsafe config source attachments should fail closed with diagnostics consistent with mesh-local config-source discovery.
- The runtime should not imply that every Knop has an authored config attachment merely because it receives projected inherited config.

## Testing

- Unit test Knop attachment parsing from current metadata for `hasConfigSource` and `hasInheritableConfigSource`.
- Unit test target Knop-local config source application.
- Unit test ancestor inheritable config source projection to descendants.
- Unit test that descendant-only inherited config does not apply to the authoring Knop unless the source is also attached locally or an explicit self-including policy is supported.
- Unit test that nearer ancestor inherited config beats farther ancestor inherited config when both set the same effective value at the same priority.
- Unit test that target Knop-local config beats inherited config.
- Unit test that command/runtime override config beats Knop-local and inherited config.
- Unit test unsafe paths, unsupported remote sources, digest mismatch, invalid attachment subjects, and missing/incorrect `sfcfg:ConfigSource` typing.
- Integration test one user-visible flow, preferably `weave version` or page generation, where an ancestor Knop inherited config source changes descendant behavior without changing unrelated Knops.
- Keep the existing pure `resolveKnopInheritedConfigSources` tests as the semantic baseline, and add integration coverage only where RDF discovery and effective-config compilation meet.

## Non-Goals

- No persistent per-Knop projection graph.
- No Oxigraph dependency in the first implementation.
- No broad historical "effective config at time T" resolver.
- No remote network fetching or host grants from portable config.
- No CLI/editor workflow for authoring config files.
- No automatic scan of every `_knop/_config/config.ttl` file without a current metadata attachment.
- No broad config ontology redesign unless a specific missing term blocks the runtime contract.

## Implementation Plan

- [x] Confirm the mesh-local prerequisite and shared resolver cleanup are merged or present in the working tree.
- [x] Audit current Knop metadata loading paths and identify the smallest place to collect ancestor Knop metadata for a target operation.
- [x] Confirm current Knop subject/base IRI conventions and add local examples to tests.
- [x] Add a Knop config attachment reader that recognizes `hasConfigSource` and `hasInheritableConfigSource` on `sflo:Knop` subjects in current metadata.
- [ ] Add inline `hasConfig` / `hasInheritableConfig` support if the effective-config compiler can accept it without widening the task.
- [x] Build a target-scope ancestor attachment index in memory.
- [x] Project inherited sources through `resolveKnopInheritedConfigSources`.
- [x] Map projected sources into deterministic effective-config ordering with nearer ancestors beating farther ancestors and local target config beating inherited config.
- [x] Resolve local and projected `ConfigSource` values through the shared artifact resolver.
- [x] Add trace/debug records for attachment subject, attachment predicate, authored source, offered-by scope, projection, resolved artifact, and effective ordering.
- [x] Fail closed for unsupported attachment subjects, unsafe local paths, unsupported remotes, digest mismatches, and ambiguous inheritance policy.
- [x] Add focused unit tests for parsing, projection integration, ordering, and failure cases.
- [x] Add one integration test through a real runtime command or generation path.
- [x] Update [[wd.todo]] after the task becomes actionable or complete.
- [x] Run focused tests and the normal Deno check/lint workflow for touched runtime modules.
