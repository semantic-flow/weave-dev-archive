---
id: 6dc2f094-f1b7-458b-84a1-e64d8ccf96b0
title: Named Repository Floating Locators
desc: 'Replace generated floating-locator blank nodes with the ruled source-registry fragment and unblock append planning'
created: 1788219564775
---

## Parent Plan

[[wa.plan.2026.2026-05-17-append-onlyish-inventory]]

## Goals

- Make persisted `RepositorySourceFloatingLocator` objects deterministic named resources at `<D/_knop/_sources#payload-source-repository-locator>`.
- Update the SFLO ontology/SHACL contract, Weave producers/readers, user/developer documentation, and affected fixtures together.
- Route current integrate MeshInventory growth through the shared append planner now that its requested graph is named-node-only.

## Summary

Dave ruled deterministic named floating locators on 2026-08-31 and accepted the existing source-registry fragment style. Today `integrate` and shared payload renderers emit `sflo:hasRepositorySourceFloatingLocator [ ... ]`, SFLO SHACL admits `sh:BlankNodeOrIRI`, and readers accept either representation. That prevents the locator from participating in append/no-op/conflict planning with stable RDF identity.

The canonical locator is the current payload's repository-working locator slot, not an arbitrary source-binding child. A custom `IntegrateSourceBinding.bindingId` therefore does not change the locator IRI: every payload has at most one `<D/_knop/_sources#payload-source-repository-locator>`. Different coordinates for that identity are a conflict/repair case, not a second current locator.

## Discussion

Add a shared designator-path helper for the locator IRI. The payload subject and its `IntegrationSource` may both point to the same named locator, and each document that carries the relationship must include the locator's type, URL, and repository-root path facts so it is self-contained.

SFLO SHACL should require `sh:IRI` for `hasRepositorySourceFloatingLocator`; the ontology comment should explain stable persisted identity without hard-coding a particular implementation's path layout. Weave owns the exact `_sources#payload-source-repository-locator` convention.

Current Weave-produced fixtures should be regenerated or deliberately updated to named locators. Reader/shape validation should fail a wrong or anonymous current locator rather than silently canonicalize it. Generic RDF parsing and the append planner may still preserve unrelated carried blank nodes; this task removes only generated floating-locator blank nodes.

## Open Issues

- Exact/ref-backed `RepositorySourceLocator` and observation-spec blank nodes are separate existing identity questions. Report them; do not broaden this child beyond `RepositorySourceFloatingLocator`.
- Published release meshes are release artifacts. Update checked-in/source fixture ladders here; do not republish SFLO/URPX or cut releases without a separate release task.

## Decisions

- Canonical locator: `<D/_knop/_sources#payload-source-repository-locator>`.
- The locator is one current-payload slot independent of custom source-binding IDs.
- Persisted blank or differently named floating locators are stale pre-v1 shapes and fail closed; fixture regeneration is preferred over a compatibility shim.
- Exact repository locators, observations, progression migration, and general blank-node elimination remain separate.

## Contract Changes

- SFLO validation changes `hasRepositorySourceFloatingLocator` object kind from blank-node-or-IRI to IRI.
- Weave output changes from inline `[...]` syntax to a named relationship plus locator subject block.
- Current integrate MeshInventory writes become exact-prefix append/no-op/conflict operations.
- Stale anonymous or noncanonical locator shapes are rejected with migration-oriented diagnostics.

## Testing

- SFLO fail-on-old named-valid and blank-invalid SHACL cases plus RDF/release guardrails.
- Weave fail-on-old exact named output in MeshInventory, KnopInventory, and source registry.
- Integrate MeshInventory exact-prefix append, semantic-union, locator-coordinate conflict, and runtime zero-write coverage.
- Reader/shape tests for canonical named acceptance and anonymous/wrong-name refusal.
- Existing floating resolution, extract, weave, ResourcePage, CLI, branch-published, and fixture-ladder coverage.
- Run full SFLO `deno task ci` and full Weave `deno task ci` before landing.

## Non-Goals

- No exact/ref-backed repository-locator identity migration, observation blank-node migration, progression producer correction, first-Knop WIP, remote fetch, operational trust, CLI flag, or public API change.
- No release publication, OS-level append, repair surface, or general RDF canonicalization.

## Implementation Plan

- [ ] Record SFLO and Weave fail-on-old tests before production edits.
- [ ] Land the named-locator ontology/SHACL contract and shared Weave path/render/read helpers.
- [ ] Migrate integrate MeshInventory through append/no-op/conflict planning and retain zero-write refusal.
- [ ] Update affected fixtures and docs without touching published release branches.
- [ ] Run cross-repo focused/full gates, land in dependency order, and record receipts.
