---
id: 5ix5ch76ifm3tkjqo10njpb
title: 2026 05 26_2030 User Settings
desc: ''
updated: 1779852668402
created: 1779852668402
---

## Goals

- Define Weave's host-local, per-user settings model.
- Replace the global `~/.sf-local-access.ttl` access-grant file with access rights grouped by mesh inside the user's Weave settings store.
- Give logs/audit storage, host-local access grants, mesh profiles, caches, and future preferences one predictable per-user home.
- Keep portable mesh config clean: user settings state should not become part of the mesh-owned config graph.
- Choose names and terms that make the model clear before wiring the runtime.

## Summary

**User settings** is the right parent concept.

A user has a host-local Weave settings store. That store can include preferences, profiles, access rights, audit/log storage, and caches. Mesh-specific settings are grouped under deterministic mesh identifiers:

```text
$WEAVE_SETTINGS/
  settings.ttl
  global-profile.ttl
  global-access.ttl
  global-preferences.ttl
  meshes/
    <mesh-identifier>/
      profile.ttl
      access.ttl
      preferences.ttl

$XDG_STATE_HOME/weave/meshes/<mesh-identifier>/logs/
  operational.jsonl
  security-audit.jsonl

$XDG_CACHE_HOME/weave/meshes/<mesh-identifier>/cache/
```

`WEAVE_SETTINGS` overrides the settings root. When it is unset, Weave should use the XDG config location, `${XDG_CONFIG_HOME:-~/.config}/weave`. The current global access-grant idea (`~/.sf-local-access.ttl`) should become mesh-scoped `$WEAVE_SETTINGS/meshes/<mesh-identifier>/access.ttl` for mesh-specific grants. `$WEAVE_SETTINGS/global-access.ttl` is reserved for explicitly trans-mesh grants, but should not accidentally become a compatibility replacement for `~/.sf-local-access.ttl`.

Logs and caches should still follow XDG state/cache locations by default rather than living under the settings root. That keeps `WEAVE_SETTINGS` from becoming a pseudo-home directory while preserving the same deterministic mesh grouping in each XDG root. `WEAVE_LOG_DIR` should continue to override the default log directory.

This should also introduce Weave's first ontology artifact: a small Weave-owned vocabulary for local runtime concepts that do not belong in portable SFLO config.

Current implementation status: the first slice now has `ontology/weave-ontology.ttl`, Weave namespace constants, a user-settings resolver, mesh identifier derivation, mesh-scoped host-local access profiles under `access.ttl`, default XDG state log directories, and tests/docs for the cutover. `~/.sf-local-access.ttl` is no longer a runtime source. `global-access.ttl` is reserved and intentionally not applied by default. A follow-up cleanup keeps path-policy parsing scoped to `sfcfg:MeshConfig` for mesh-carried rules and names the host-local settings access file as an access profile path rather than a local config path.

## Discussion

The settings store is per OS user because it lives under that user's XDG config location or configured `WEAVE_SETTINGS` root. Inside that user-owned store, mesh-scoped settings need to be keyed by mesh identity, not by checkout path. A user can move a local copy, clone the same mesh elsewhere, or access the same mesh through different commands. If the mesh base stays the same, Weave should find that user's same local grants and audit history.

Using the raw `meshBase` directly as the directory name is tempting but brittle. It can leak too much information, be long, contain awkward path separators, vary by case/trailing slash/default port normalization, and collide in confusing ways. The better default is a readable slug plus a deterministic hash of the canonical `meshBase`.

Candidate identifier shape:

```text
<display-slug>-<mesh-base-hash>
```

Example:

```text
$WEAVE_SETTINGS/meshes/mesh-alice-bio-7d9a3f12c4e1/
```

The slug is deterministic and always part of the directory name. It is readable, but it is not the uniqueness anchor. Lookup should recompute the same full `<display-slug>-<mesh-base-hash>` identifier from the canonical `meshBase`; it should not depend on scanning candidate folders or treating the slug alone as identity. Slug generation should strip protocol and normalize host-root meshes carefully because for meshes based at a host root, the slug could otherwise just become the hostname. That is acceptable as a fallback label, but it must not be treated as authoritative identity.

Canonicalization should be explicit:

- Parse the `meshBase` as a URL.
- Lowercase scheme and host.
- Drop query and fragment.
- Normalize default ports.
- Normalize path/trailing slash according to mesh-base rules.
- Derive the hash from the canonical string.
- Derive the slug from the last meaningful path segment when present; otherwise use a sanitized host fallback.

The user settings store and its mesh settings groups should point to the mesh. The portable mesh config should not point back to the user settings store. Putting a host-local path in mesh-owned config would make the config less portable and would leak machine/user assumptions into the graph.

Candidate first Weave ontology namespace:

```text
https://semantic-flow.github.io/weave/ontology/
```

Candidate prefix:

```text
weave:
```

Candidate user settings metadata:

```turtle
@prefix weave: <https://semantic-flow.github.io/weave/ontology/> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .

<> a weave:UserSettings ;
  weave:hasProfile <global-profile.ttl> ;
  weave:hasAccessProfile <global-access.ttl> ;
  weave:hasPreferencesProfile <global-preferences.ttl> ;
  weave:hasMeshSettings <meshes/mesh-alice-bio-7d9a3f12c4e1/> .

<meshes/mesh-alice-bio-7d9a3f12c4e1/> a weave:MeshSettingsGroup ;
  rdfs:label "mesh-alice-bio" ;
  weave:forMeshBase "https://semantic-flow.github.io/mesh-alice-bio/"^^xsd:anyURI ;
  weave:meshIdentifier "mesh-alice-bio-7d9a3f12c4e1" ;
  weave:hasProfile <meshes/mesh-alice-bio-7d9a3f12c4e1/profile.ttl> ;
  weave:hasAccessProfile <meshes/mesh-alice-bio-7d9a3f12c4e1/access.ttl> ;
  weave:hasPreferencesProfile <meshes/mesh-alice-bio-7d9a3f12c4e1/preferences.ttl> .
```

The first slice should keep mesh settings metadata deliberately small: canonical `meshBase`, deterministic `meshIdentifier`, and a readable display name using `rdfs:label`. Workspace-root observations, last-used paths, command history, and other host-local diagnostics can wait until there is a concrete feature that needs them.

Profile/access/preferences vocabulary should be reusable across global and mesh scopes. The same `weave:hasProfile`, `weave:hasAccessProfile`, and `weave:hasPreferencesProfile` shape can hang off `weave:UserSettings` for trans-mesh user state and off `weave:MeshSettingsGroup` for mesh-scoped state. Scope comes from the subject, not from inventing separate global-only and mesh-only vocabularies.

Candidate host-local access metadata should move away from portable `sfcfg:` operational config terms:

```turtle
@prefix weave: <https://semantic-flow.github.io/weave/ontology/> .

<> a weave:HostLocalAccessProfile ;
  weave:forMeshSettings <./> ;
  weave:hasLocalPathGrant [
    a weave:LocalPathGrant ;
    weave:allowsLocalPathBase "/home/alice/notes"
  ] .
```

This does not have to become a large ontology pass immediately. The first implementation can introduce a small Weave-local vocabulary and parser while keeping the public contract intentionally small.

Important split after implementation review: **host-local user settings grants use `weave:`; portable mesh-carried path grants stay in `sfcfg:` for now.** The runtime still needs `sfcfg:LocalPathAccessRule`-style grants for mesh config because those grants describe mesh-relative operational policy carried with a mesh. That is a different scope than a user's host-local access approval in `$WEAVE_SETTINGS/meshes/<mesh-identifier>/access.ttl`. The old `~/.sf-local-access.ttl` global home file is the legacy thing to remove, not every `sfcfg:` path grant.

## Open Issues

- Exact first-term names beyond the first slice. `weave:UserSettings`, `weave:MeshSettingsGroup`, `weave:HostLocalAccessProfile`, and `weave:LocalPathGrant` are the current first-slice choices.
- How to handle a changed `meshBase`. The mesh settings group key should probably change with it, with any migration being explicit.
- Whether `global-access.ttl` should ever be loaded by default for all meshes. It is reserved, but first-slice access should stay mesh-scoped unless a concrete trans-mesh grant use case justifies broader behavior.
- Whether later preferences need one RDF shape across global and mesh-scoped files or separate shapes. No preference terms are needed in the first slice.

## Decisions

- Use **user settings** as the parent concept and **mesh settings group** for the per-mesh grouping under it.
- Use `MeshSettingsGroup`, not `MeshSettingsBundle`, because these directories are host-local user state rather than portable packages.
- Use one mesh settings group per canonical `meshBase` within the user's settings store.
- Use a deterministic readable slug plus a hash. Both are always included in the directory name; the hash is the authoritative uniqueness component.
- Strip and normalize host/protocol details during slug generation; host-root meshes can use a sanitized host fallback, but not as identity.
- Do not add a portable mesh config property that points to the user settings path.
- Add user settings metadata that points from a mesh settings group to the mesh base.
- Create Weave's first ontology for Weave-owned runtime/local terms instead of adding these terms to `sfcfg:`.
- Use `https://semantic-flow.github.io/weave/ontology/` as the candidate namespace and `weave:` as the candidate prefix.
- Put the first ontology artifact at `ontology/weave-ontology.ttl`.
- Use XDG locations by default. `WEAVE_SETTINGS` overrides the settings root; otherwise use `${XDG_CONFIG_HOME:-~/.config}/weave`.
- Store logs under `${XDG_STATE_HOME:-~/.local/state}/weave/meshes/<mesh-identifier>/logs/` by default, with `WEAVE_LOG_DIR` still taking precedence.
- Store caches under `${XDG_CACHE_HOME:-~/.cache}/weave/meshes/<mesh-identifier>/cache/` by default.
- Reserve `$WEAVE_SETTINGS/global-profile.ttl`, `$WEAVE_SETTINGS/global-access.ttl`, and `$WEAVE_SETTINGS/global-preferences.ttl` for trans-mesh user state.
- Reserve `$WEAVE_SETTINGS/meshes/<mesh-identifier>/profile.ttl`, `$WEAVE_SETTINGS/meshes/<mesh-identifier>/access.ttl`, and `$WEAVE_SETTINGS/meshes/<mesh-identifier>/preferences.ttl` for mesh-scoped user state.
- Reuse profile/access/preferences vocabulary across global and mesh-scoped files; do not fork the vocabulary just because the file scope differs.
- Do not define preference terms in the first slice.
- Do not automatically migrate from `~/.sf-local-access.ttl` and do not warn about it. There are no users to protect yet; remove legacy references and behavior instead.
- Use `weave:` terms for host-local user settings access grants.
- Keep portable mesh-carried path grants in the `sfcfg:` config vocabulary for now. They are mesh-relative operational policy, not user settings state.
- Treat the future durability of `sfcfg:LocalPathAccessRule` as config-policy ontology work, not as a compatibility shim for `~/.sf-local-access.ttl`.

## Contract Changes

- New user settings root convention:

```text
$WEAVE_SETTINGS/
```

When `WEAVE_SETTINGS` is unset:

```text
${XDG_CONFIG_HOME:-~/.config}/weave/
```

- New per-mesh settings group convention:

```text
$WEAVE_SETTINGS/meshes/<mesh-identifier>/
```

- New default host-local access file:

```text
$WEAVE_SETTINGS/meshes/<mesh-identifier>/access.ttl
```

- New default log directory:

```text
${XDG_STATE_HOME:-~/.local/state}/weave/meshes/<mesh-identifier>/logs/
```

- New default cache directory:

```text
${XDG_CACHE_HOME:-~/.cache}/weave/meshes/<mesh-identifier>/cache/
```

- New user settings metadata file:

```text
$WEAVE_SETTINGS/settings.ttl
```

- Reserved trans-mesh profile file:

```text
$WEAVE_SETTINGS/global-profile.ttl
```

- Reserved trans-mesh access file:

```text
$WEAVE_SETTINGS/global-access.ttl
```

- Reserved trans-mesh preferences file:

```text
$WEAVE_SETTINGS/global-preferences.ttl
```

- New mesh profile metadata file:

```text
$WEAVE_SETTINGS/meshes/<mesh-identifier>/profile.ttl
```

- Reserved mesh-scoped preferences file:

```text
$WEAVE_SETTINGS/meshes/<mesh-identifier>/preferences.ttl
```

- New Weave ontology artifact:

```text
ontology/weave-ontology.ttl
```

- New Weave ontology namespace:

```text
https://semantic-flow.github.io/weave/ontology/
```

- Existing `WEAVE_LOG_DIR` behavior remains an explicit override.
- Existing docs that mention `~/.sf-local-access.ttl` need to move to the user settings path model.
- Existing runtime/tests/docs that describe the old global host-local access file should move to Weave ontology terms or be deleted.
- Existing runtime/tests that use `sfcfg:LocalPathAccessRule`, `sfcfg:hasLocalPathAccessRule`, `sfcfg:hasLocalPathBase`, or `sfcfg:hasLocalPathLocatorKind` inside mesh config should stay in place unless the config-policy ontology task replaces that portable vocabulary explicitly.
- The config-policy work in [[wa.task.2026.2026-05-25_1609-config-policy-ontology-and-runtime]] should treat user settings and mesh settings group resolution as host-local runtime state, not portable config resolution.

## Testing

- Unit-test mesh identifier derivation for host-root mesh bases, path-root mesh bases, uppercase hosts, default ports, trailing slashes, query/fragment rejection or normalization, and unusual but valid path segments.
- Unit-test user settings root and mesh settings group resolution using fake `WEAVE_SETTINGS`, `XDG_CONFIG_HOME`, and home-directory fallback environments.
- Unit-test default log/cache root resolution using fake `XDG_STATE_HOME`, `XDG_CACHE_HOME`, and fallback environments.
- Runtime-test that local access grants are read from `access.ttl` under the selected user's mesh settings group.
- Runtime-test that `global-access.ttl` is not accidentally treated as a legacy all-mesh grant file in the first slice.
- CLI-test that granting source-directory access writes the per-mesh `access.ttl` and does not write `~/.sf-local-access.ttl`.
- Logging-test that default operational and security audit logs land under the selected XDG state per-mesh `logs/` directory.
- Logging-test that `WEAVE_LOG_DIR` still wins when set.
- Documentation guardrail: user-facing docs should no longer recommend `~/.sf-local-access.ttl`.
- Legacy guardrail: Weave runtime/tests/docs should no longer use `~/.sf-local-access.ttl` or `sfcfg:HostLocalOperationalConfig` for host-local user access grants after the cutover.
- Scope guardrail: mesh config tests may still use `sfcfg:` path grants for portable mesh-relative policy; do not confuse those with user settings access files.
- Ontology smoke-test or parser test should confirm the first Weave ontology artifact can be parsed as Turtle.

## Non-Goals

- Do not make user settings or mesh settings groups portable mesh config.
- Do not design a cloud account or multi-user identity system.
- Do not solve remote authorization policy here.
- Do not redesign the full SFLO config ontology in this task.
- Do not build a preferences UI in the first pass.

## Implementation Plan

- [x] Add Weave's first ontology artifact for local runtime/user-settings terms.
- [x] Add a small user-settings resolver that accepts canonical `meshBase` and returns the settings root, global profile/access/preferences paths, mesh settings group directory, mesh profile metadata path, access path, preferences path, log directory, and cache directory.
- [x] Add mesh identifier derivation: canonical mesh base string, deterministic display slug, truncated hash, filesystem-safe final directory name.
- [x] Add runtime namespace constants for the Weave ontology.
- [x] Introduce a minimal Weave user-settings renderer/parser path using `weave:UserSettings`, `weave:MeshSettingsGroup`, `weave:HostLocalAccessProfile`, and local access grant terms.
- [x] Change host-local access grant reads/writes from `~/.sf-local-access.ttl` to `$WEAVE_SETTINGS/meshes/<mesh-identifier>/access.ttl`.
- [x] Reserve but do not implicitly apply `$WEAVE_SETTINGS/global-access.ttl`.
- [x] Remove legacy `~/.sf-local-access.ttl` reads/writes.
- [x] Keep mesh-carried `sfcfg:` path grant handling in place for portable mesh config policy.
- [x] Stop treating generic `sfcfg:OperationalConfig` subjects as mesh path-policy carriers.
- [x] Rename runtime surfaces from local-config wording to host-local access profile wording where they point at `$WEAVE_SETTINGS/meshes/<mesh-identifier>/access.ttl`.
- [x] Change default runtime/CLI logging from workspace `.weave/logs` to the selected XDG state per-mesh `logs/`, preserving `WEAVE_LOG_DIR`.
- [x] Update CLI/user docs and examples that mention `~/.sf-local-access.ttl`.
- [x] Add tests for identifier derivation, user settings path resolution, mesh settings group path resolution, access grant storage, and log directory defaults.
- [ ] Decide whether `sfcfg:LocalPathAccessRule` should be formalized in the config ontology during [[wa.task.2026.2026-05-25_1609-config-policy-ontology-and-runtime]] or replaced by a better portable policy term.
