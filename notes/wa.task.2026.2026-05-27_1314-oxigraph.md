---
id: vkr445weibz6r5k3swcds0l
title: 2026 05 27_1314 Oxigraph
desc: ''
updated: 1779912960580
created: 1779912857238
---

## Goals

Use Oxigraph for:

holding parsed config/source/metadata/inventory graphs in memory
named graphs per source/layer/scope, e.g. app defaults, mesh config, command overrides, Knop source registries
querying ancestry, attachments, policy bindings, exact targets, source provenance
debugging/explanation queries
dependency discovery for cache invalidation

## Summary

## Discussion

Codex suggests this model:

Build an EffectiveConfigGraphCache keyed by mesh root plus a fingerprint of config-bearing files/inventories/source registries.
Load relevant RDF into named graphs.
Use SPARQL/query helpers to gather candidate config layers and policy bindings for a Knop.
Compile the final ResolvedConfigForScope into plain runtime objects.
Cache that object by { scopeIri, commandOverrideFingerprint, graphFingerprint }.
Invalidate when staged writes touch _mesh/_config, _mesh/_inventory, _knop/_sources, _knop/_inventory, page definitions, or command overrides.

## Open Issues

## Decisions

## Contract Changes

## Testing

## Non-Goals

- Do not put all resolver semantics in SPARQL; Layer ordering, selector specificity, conflict resolution, fail-closed diagnostics, host trust, and command overrides should stay in TypeScript domain code. SPARQL can collect candidates; the resolver should still decide.



## Implementation Plan

- [ ]