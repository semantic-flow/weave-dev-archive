---
id: i4iu1pnwa3xu05kezam3pu1
title: 2026-05-20_2152-workingAccessUrl
desc: ''
updated: 1779339180859
created: 1779339180859
---

## Goals

- Add Weave runtime support for `sflo:workingAccessUrl` as the active current-byte locator for a `sflo:DigitalArtifact` whose working bytes intentionally remain remote.
- Preserve the no-copy distinction: ordinary current-byte resolution fetches the remote URL under active policy, while versioning captures the exact bytes observed in the normal historical manifestation files.
- Keep remote access explicit and fail closed. Mesh RDF may name a `workingAccessUrl`, but it cannot grant the host permission to contact that URL.
- Support public HTTP(S) sources such as raw repository files without making mutable branch URLs appear immutable or authoritative.
- Reuse the bounded acquisition and host-local origin policy established by [[wa.task.2026.2026-08-21_1810-import-refresh]] rather than growing a second network stack.
- Keep local working files, repository-floating locators, imported local copies, and remote working URLs as distinct source modes with clear operator-facing diagnostics.

## Summary

`sflo:workingAccessUrl` remains valid core vocabulary for the case where remote bytes themselves are the active working surface. The feature is still technically feasible, but it is no longer the default answer to “this ontology lives somewhere else.”

Weave now has explicit HTTP(S) `import`, governed local working copies, `sflo:ImportSource` provenance, canonical expected/observed digest behavior, floating repository locators, and a shared artifact-resolution service. When a local copy is acceptable, the preferred workflow is initial import followed by explicit `weave import --refresh`; see [[wa.task.2026.2026-08-21_1810-import-refresh]]. That workflow keeps ordinary weave/version/generate offline and makes each network acquisition an explicit operator action.

This task is the distinct no-copy mode. It should be activated when a real consumer requires one or more of the following:

- no durable local working copy
- versioning whatever bytes a remote current URL returns at operation time
- a remote working source that cannot or should not be materialized into the local workspace
- current-byte semantics where explicit import refresh would be the wrong lifecycle boundary

The task should not be implemented merely to record an upstream or primary URL. `ImportSource.targetAccessUrl`, repository source metadata, ordinary provenance, and manifestation/file locations can describe where bytes came from without making the remote URL the active runtime locator.

## Current State

- The SFLO core ontology defines `sflo:workingAccessUrl` as a remote/external current-byte hook whose use remains an operational-policy question.
- Weave inventory/page-model code can parse and display `workingAccessUrl` metadata, but current payload loading still requires a local path or floating repository locator and does not fetch the URL.
- The shared artifact resolver parses direct `sflo:targetAccessUrl` but deliberately rejects ambient URL fetching.
- `weave import` already has bounded HTTP(S) acquisition, timeout/size limits, digest calculation, and injected fetch tests, but the helper is import-private and follows redirects automatically.
- `sflo:expectsContentDigest` on `sflo:ArtifactResolutionSpec` and `sflo:observedContentDigest` on `sflo:ArtifactResolutionObservation` settle the expected/observed digest model. No new digest property is needed for remote resolution.
- Portable `sfcfg:RemoteAccessRule` vocabulary was removed from the live config ontology. Host-local path and remote trust belong to implementation- or service-specific runtime configuration. Do not restore machine trust to portable mesh config as part of this task.
- Weave's packaged in-process API is intentionally subprocess/network-clean. Remote resolution needs an injected capability so CLI/daemon runtime paths can opt in without making `versionPayloads` ambiently network-active.
- Compiled binary permissions currently need auditing because explicit HTTP acquisition requires network capability while application policy supplies the finer-grained authorization.

## Discussion

### Import Refresh Is The Default Remote-Origin Workflow

When duplication is acceptable, a governed local copy has substantial operational advantages:

- ordinary weave/version/generate can run offline
- the source bytes under review are stable until an explicit acquisition changes them
- network trust is exercised only by `weave import` or `weave import --refresh`
- the source registry retains origin and observation evidence
- a remote change cannot race two phases of one ordinary weave operation

This task therefore should not absorb import refresh or serve as an optimization for avoiding one working copy. Historical versioning already creates settled local manifestation files, so the no-copy benefit applies only to the mutable working surface.

### Current Payload Resolution Needs A Real Locator Model

Weave currently threads `workingLocalRelativePath` through inventory state, candidate models, manifestation naming, source rendering, page links, and payload loading. Some code carries `workingAccessUrl` alongside that required path only as display metadata. A remote-only payload cannot honestly supply the path.

Do not synthesize a fake local path from the URL merely to satisfy these interfaces. Refactor payload current-source state into a discriminated locator model, for example local `LocatedFile`, allowed local relative path, floating repository locator, or remote working URL, and separate locator identity from the filename/content-kind hint used for historical manifestation layout.

The ontology already has `sflo:preferredPayloadFileSlug`, but the exact filename/media hint contract should be chosen deliberately. A URL pathname can provide a default only when it ends in an unambiguous file name; redirects and content negotiation must not silently change artifact naming.

### Runtime Policy

Remote trust must be host-local and deny-by-default. The policy should match at least:

- operation/locator kind: `workingAccessUrl`, distinct from replaying `ImportSource.targetAccessUrl`
- scheme
- exact origin, including normalized default ports

The reusable Weave-local policy/fetch layer should come from [[wa.task.2026.2026-08-21_1810-import-refresh]]. Every redirect hop must be authorized before the next request. Private/authenticated access, cookies, arbitrary headers, and credential-bearing URLs stay out of the first slice.

### Operation-Scoped Byte Snapshot

A mutable URL can change between requests. A composed weave must fetch a selected remote working source once per operation and reuse the resulting bytes across validation, planning, versioning, and any explicit current-source consumers in that operation. It must not validate one response and version a later response.

Across separate operations, a mutable URL is intentionally allowed to yield new bytes. A caller-supplied digest expectation fails closed if those new bytes do not match. A commit/tag URL may be stable by convention, but the runtime should rely on URL/digest policy rather than infer immutability from GitHub path syntax.

### Multiple Current Locators

SFLO says coexisting current locators should identify the same bytes. The first remote slice should avoid implicit precedence. If `workingAccessUrl` coexists with a local or repository working locator, reject the source as ambiguous unless an explicit profile selects an agreement-check mode. A later agreement mode may resolve both and compare digests, but ordinary resolution should not silently choose one.

### Historical Evidence

Successful versioning must copy the exact operation-scoped remote bytes into the normal historical manifestation file. A stored `sflo:expectsContentDigest`, when present on the applicable resolution/source binding, is checked before any state is minted.

Persisting `sflo:hasContentDigest` on the new manifestation or LocatedFile would also advance the separate fingerprint/integrity line. If that standing digest is required for acceptance, make the overlap with [[wa.task.2026.2026-05-04-fingerprint-verification]] explicit rather than implying that current version rendering already writes it. Failed fetch or mismatch evidence must not be persisted as a successful observation.

### Authoring Surface

Runtime support alone is not friendly enough for the product vision, but an authoring command should follow rather than distort the resolver design. Do not make a plain HTTP positional argument silently switch `integrate` into remote-current semantics.

The likely surface is an explicit remote working-source option on `integrate` or a narrow source-binding command. It must:

- fetch once under policy when bytes are needed to validate content/type
- record `sflo:workingAccessUrl` on the payload artifact
- record an `sflo:IntegrationSource` with `sflo:targetAccessUrl`, working resolution mode, optional expected digest, and acquisition observation when an intentional fetch occurred
- record an honest filename/content-kind hint without inventing `workingLocalRelativePath`
- avoid writing a governed local working copy

The exact CLI spelling remains open until the locator-model refactor exposes the clean request type.

## Open Issues

- What real consumer requires no-copy remote working resolution after `weave import --refresh` exists? Capture that workflow before admitting implementation to the active queue.
- What is the smallest honest filename/media hint required for remote-only payload version layout: URL basename, `preferredPayloadFileSlug` plus media type/extension, or an explicit authoring option?
- Should the first runtime slice support only payload versioning, or also extraction from a remote-current source within the same operation-scoped byte snapshot? Prefer payload versioning first unless the motivating consumer requires extraction.
- What explicit authoring syntax should record `workingAccessUrl` without conflating it with ordinary import or local integrate?
- Should validation report policy denial as an operational resolvability finding only under an explicit profile, or whenever a selected operation needs current bytes?
- Is standing historical manifestation digest output part of this task or a coordinated fingerprint slice?

## Decisions

- Keep this task open as the live no-copy remote-current mode; do not treat it as the ordinary upstream-location/import workflow.
- Prefer `weave import --refresh` when a governed local copy is acceptable.
- First implementation supports HTTP(S) only and remains deny-by-default.
- Host-local Weave policy, not portable `sfcfg` mesh config, grants remote access.
- Reuse one bounded remote acquisition implementation and one policy matcher across import refresh and `workingAccessUrl`, while retaining distinct locator-kind grants.
- Fetch remote working bytes once per operation and reuse that byte snapshot throughout the operation.
- Do not invent a local path for remote-only sources.
- Reject ambiguous multiple current locators in the first slice rather than defining silent precedence.
- A mutable branch URL is “latest-ish” working input, not immutable evidence. Exactness comes from pinned coordinates and/or a caller-supplied expected digest.
- Versioning captures the observed bytes in normal historical manifestation files. Standing digest serialization, if required, must be scoped explicitly.
- Resource page generation may display `workingAccessUrl` metadata but must not fetch merely to render a page unless an explicit working-source consumer requires the bytes.
- Private/authenticated remote sources remain out of scope.
- Generic `targetAccessUrl` fetching remains out of scope.

## Contract Changes

- Introduce a locator-discriminated payload working-source model instead of requiring `workingLocalRelativePath` for every payload.
- Add an optional injected remote-byte capability to the runtime resolution context. Absence of that capability or applicable policy fails closed without touching the network.
- Resolve `sflo:workingAccessUrl` for selected payload current-byte operations under exact origin/scheme policy and operation-scoped byte reuse.
- Keep the packaged in-process API network-clean unless a future public API deliberately accepts an injected acquisition capability.
- Extend version/weave execution so remote bytes flow through the same historical manifestation planning as local payload bytes without fabricating local locator RDF.
- Add clear diagnostics for malformed URL, missing policy, denied origin/scheme, redirect denial, timeout, size limit, HTTP failure, ambiguous locators, content/type failure, and digest mismatch.
- Add an explicit authoring surface only after the locator model and runtime resolver support it honestly.
- Update [[sf.spec.2026-05-18-publication-source-binding]], [[sf.spec.2026-04-03-weave-behavior]], [[wu.cli-reference.integrate]], [[wu.cli-reference.weave]], [[wu.cli-reference.version]], [[wd.runtime]], and [[wd.codebase-overview]] when the corresponding behavior ships.

## Testing

- Unit-test remote policy selection for the `workingAccessUrl` locator kind, including absence of capability/policy, malformed URL, scheme/origin normalization, and denied redirects before the next request.
- Unit-test payload locator parsing for remote-only, each existing local/repository mode, and ambiguous combinations.
- Unit-test filename/content-kind hint validation without fake local paths.
- Integration-test remote-only payload versioning through a local HTTP server and prove the historical manifestation bytes equal the fetched bytes.
- Integration-test operation-scoped byte reuse by changing the server response between possible phases and proving one operation captures one response.
- Integration-test separate operations against a mutable URL: unchanged/no-op behavior when bytes are unchanged and a later state when bytes change.
- Integration-test expected-digest success and mismatch with no historical writes on mismatch.
- Regression-test local mesh, sidecar local, floating repository, import-refresh, page generation, and packaged API behavior.
- Test ambiguous multiple locators, redirect loops, timeout, maximum size, HTTP errors, malformed content, and no local path leakage.
- Add authoring CLI/e2e coverage only when that slice is admitted.
- Use injected/local HTTP servers in CI rather than public network dependencies.
- Run focused artifact-resolution and weave/version tests, followed by `deno task fmt`, `deno task lint`, `deno task check`, and `deno task test`.

## Non-Goals

- Importing or refreshing a governed local copy; see [[wa.task.2026.2026-08-21_1810-import-refresh]].
- Avoiding historical manifestation copies. Versioning still materializes settled bytes inside the mesh.
- Generic remote `sflo:targetAccessUrl` resolution for config, pages, references, or extraction.
- Authenticated/private URLs, secrets, cookies, credential storage, or arbitrary request shaping.
- Persistent HTTP caching or offline success for a remote-only working source.
- Inferring that a URL is canonical/authoritative merely because it is the active working locator.
- Automatically remapping an upstream ontology namespace to a different mesh base; alternate namespace/base support is a separate ontology and extraction concern.
- General URL-root-relative filesystem abstractions.

## Implementation Plan

### Phase 0: Confirm The No-Copy Consumer

- [ ] Record the concrete consumer workflow that cannot use a governed local import plus `weave import --refresh`.
- [ ] Decide whether the first slice needs payload versioning only or also extraction/current-source reading.
- [ ] Decide the remote-only filename/content-kind hint contract.
- [ ] Refine the portable behavior specs before implementation.

### Phase 1: Reuse Remote Acquisition Policy

- [ ] Land or reuse the host-local remote-origin policy and bounded per-hop-authorized fetch layer from [[wa.task.2026.2026-08-21_1810-import-refresh]].
- [ ] Add the distinct `workingAccessUrl` locator-kind policy match and injected runtime capability.
- [ ] Audit compiled CLI permissions and preserve the network-clean packaged API boundary.

### Phase 2: Generalize Payload Current Sources

- [ ] Replace required-local-path payload source state with a discriminated locator model and separate filename/content-kind hint.
- [ ] Preserve all existing local, sidecar, repository-floating, binary, RDF, and page behavior through focused regression tests.
- [ ] Reject ambiguous multiple locators in the first slice.

### Phase 3: Resolve And Version Remote Bytes

- [ ] Add operation-scoped fetch/snapshot reuse for selected remote payloads.
- [ ] Verify stored/caller digest expectations before successful use or historical writes.
- [ ] Thread the acquired bytes through weave/version planning and historical manifestation writes.
- [ ] Add policy, network, redirect, mutable-source, digest, and no-path-leakage tests.

### Phase 4: Add Authoring UX And Documentation

- [ ] Choose and implement an explicit remote-current source-binding surface without overloading ordinary import or local integrate syntax.
- [ ] Record `workingAccessUrl`, `IntegrationSource` coordinates, and honest observation evidence without a local working path.
- [ ] Update user/developer documentation, behavior specs, and relevant release notes.
- [ ] Run `deno task fmt`, `deno task lint`, `deno task check`, and `deno task test`.
- [ ] Provide a semantic commit message with a summary line and detailed developer-facing bullets.
