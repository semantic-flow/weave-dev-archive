---
id: e7068d4722fa0f2c19584c1f
title: 2026-08-21_1810 import refresh
desc: ''
created: 1787361063000
---

## Goals

- Add `weave import --refresh <designatorPath>` as the explicit replay surface for a previously recorded `sflo:ImportSource`.
- Reacquire bytes from the stored import source and replace the artifact's existing governed local working copy without requiring the operator to restate the source URL or destination path.
- Keep refresh inside the import boundary: acquisition and provenance change, while versioning, extraction, page generation, and publication remain separate operations.
- Make persisted-URL replay fail closed. A URL recorded in mesh RDF is provenance and replay intent, not authority to make a network request; refresh must require applicable host-local remote-origin policy before fetching it.
- Preserve the distinction between a remote upstream/acquisition location, a governed local working copy, and the mesh identifier that manages or describes that copy.
- Append honest acquisition evidence for every successful refresh instead of overwriting the previous observation.
- Leave room for a later import destination outside the mesh but inside an allowed workspace without coupling that destination expansion to refresh itself.

## Summary

The first `weave import` implementation can fetch an HTTP(S) URL, copy the bytes into a mesh-root-relative governed working file, create an `sflo:ImportSource` in the target Knop source registry, compute an observed digest, and later replace the same working file when the caller repeats the source, designator, destination, and `--replace-working` arguments.

That is sufficient for explicit reacquisition, but it makes a routine refresh restate information Weave has already recorded. A successful URL import already leaves the durable pieces needed to replay the acquisition:

- the target payload artifact and designator path
- the active governed local working file
- an `sflo:ImportSource` source binding with `sflo:targetAccessUrl`
- optional `sflo:expectsContentDigest`
- an `sflo:ArtifactResolutionObservation` with the digest and time observed during the previous acquisition

`weave import --refresh <designatorPath>` should resolve that existing binding, reacquire the source under explicit remote-origin policy, verify any stored expectation, atomically replace the existing local working bytes, and append a new observation. It should not mint a HistoricalState. A later `weave version` or composed `weave` records the refreshed bytes in artifact history.

This is the preferred ordinary workflow when an ontology or other artifact has an upstream byte location that differs from its mesh identifier but a governed local copy is acceptable. The distinct no-copy mode in which the remote URL remains the active working surface belongs to [[wa.task.2026.2026-05-20_2152-workingAccessUrl]].

## Discussion

### Import, Replacement, And Refresh

The three related command shapes have different intent:

- Initial import establishes the source, destination, artifact, and first observation: `weave import <source> <designatorPath> --working-file <meshRelativePath>`.
- Explicit replacement restates the source and destination and is appropriate when either may be changing: repeat the ordinary import command with `--replace-working`.
- Refresh replays the existing source binding and active destination: `weave import --refresh <designatorPath>`.

Refresh should not accept a new source URL or working-file destination in its first slice. Changing those coordinates is a source-binding mutation, not a replay. Use an explicit replacement command today and design a narrower source-binding maintenance surface later if changing recorded coordinates becomes common.

### Destination Is Orthogonal

Import means that outside bytes are copied into a governed local working surface. It is still import whether that local destination is:

- inside the mesh root, where it can be represented as a mesh-addressable `sflo:LocatedFile` through `sflo:hasWorkingLocatedFile`; or
- outside the mesh root but inside an explicitly allowed workspace/source boundary, where it would use `sflo:workingLocalRelativePath` and would not automatically become a published mesh file.

The current CLI implements only mesh-root-relative `--working-file` destinations. The first refresh slice should replay that supported shape rather than simultaneously widening import writes beyond the mesh. A later workspace-destination task can extend initial import and refresh together under existing local-path policy.

A transient temp file is not a valid governed destination. An implementation may use temp files internally for atomic writes, but the persisted active locator must identify the durable local working copy.

### Stored URLs Are Not Network Authority

Initial URL import is explicit because the operator supplies the URL in the command being executed. Refresh is different: the URL comes from repository-controlled RDF. A cloned or modified mesh could otherwise cause `weave import --refresh` to contact an unexpected internal or external service.

The refresh runtime must therefore require an applicable positive host-local grant for the stored URL's scheme and exact origin. Portable mesh config must not grant broader host network trust. The grant model should distinguish refresh of `ImportSource.targetAccessUrl` from later use of `sflo:workingAccessUrl`, even if both consume a shared Weave-local remote policy matcher.

Redirects must be checked before each hop. The existing import helper uses automatic redirect following, which is not sufficient because a request may reach a disallowed origin before the final response can be inspected. The reusable fetch layer should use bounded manual redirect handling, authorize every hop, retain final-URL diagnostics, and keep authentication/private URL support out of the first slice.

### Refresh Evidence

Every successful refresh is an intentional acquisition event and should append a new `sflo:ArtifactResolutionObservation` linked from the existing `sflo:ImportSource`. The observation should carry:

- the exact digest computed over the acquired bytes
- the observation time
- an `sflo:observedArtifactResolutionSpec` that identifies the governed local destination actually written
- the final URL when the shared observation model gains an honest coordinate for it; do not invent or overload a local-path field merely to retain redirect diagnostics

The existing import renderer uses a single deterministic `-observation-001` fragment and replacement rewrites the source registry. Refresh needs append-aware observation allocation and source-registry parsing that can retain multiple observations. Existing observation `001` remains valid; the first refresh should allocate the next unused ordinal.

If the acquired bytes are unchanged, refresh should still append an observation because a real acquisition and verification occurred. The working file can be reported as unchanged/skipped, while the source registry is updated with the new evidence.

### Expected Digests

`sflo:expectsContentDigest` on the existing `sflo:ImportSource` remains a pre-acquisition requirement. Refresh must verify it before writing either the working file or successful observation evidence. A mismatch fails closed and must not persist a successful observation.

The first refresh syntax should not silently change the stored expectation. A mutable upstream that is intentionally followed should normally have no fixed expectation and will accumulate observed digests. A pinned acquisition can retain its expected digest. Updating an expectation belongs to an explicit source-binding change rather than ordinary replay.

### Relationship To Extraction

Refresh only updates the governed source payload. A typical ontology flow remains explicit:

1. initial `weave import`
2. `weave` or `weave version` to settle the imported payload
3. `weave extract --all-terms --source <designatorPath>` when the source contains mesh-scoped terms
4. a later `weave` to version and generate the extracted term surfaces

After a refresh, the operator chooses whether to version the new payload and whether extraction/source-reference maintenance is required. Refresh must not automatically re-extract terms or rewrite derived Knops.

## Open Issues

- What exact Weave-local host settings vocabulary and CLI helper should create a refresh origin grant? The semantic requirement is an exact scheme/origin positive grant scoped to import replay; the spelling should align with the existing host-local access profile rather than reintroducing removed portable `sfcfg:RemoteAccessRule` terms.
- Should a later slice replay portable local `ImportSource.targetLocalRelativePath` bindings as well as HTTP(S) bindings? The first consumer is remote ontology refresh, so URL replay is sufficient for the initial task.
- Should observation identifiers use the next numeric suffix, a timestamp-derived suffix, or another append-safe identifier? Prefer the next unused numeric suffix if concurrent writer behavior remains single-process and fail-closed.
- Should an unchanged refresh rewrite the working file atomically or skip that write after digest comparison? Prefer skipping the working-file replacement while still appending observation evidence.

## Decisions

- Refresh is a mode of import, exposed as `weave import --refresh <designatorPath>`, not a new top-level `weave refresh` operation.
- First-slice refresh replays exactly one existing `sflo:ImportSource` for the target payload artifact. Missing, multiple, malformed, unsupported, or source/destination-incoherent bindings fail closed.
- First-slice refresh supports stored HTTP(S) `sflo:targetAccessUrl` acquisition. It does not infer live remote access from `sflo:workingAccessUrl`.
- The existing governed local working locator remains active before and after refresh.
- The first slice supports the current mesh-root-relative imported working-file shape. Workspace-external destinations remain a separate import-destination extension.
- Persisted URLs require host-local positive origin policy before network access. Mesh-carried RDF cannot grant that trust.
- Redirect authorization is per hop, with bounded redirects, timeout, and maximum response bytes.
- A successful refresh appends acquisition evidence to the existing `ImportSource`; it does not replace prior observations.
- Stored digest expectations are verified but not changed by refresh.
- Refresh does not version, extract, generate, commit, or publish.
- Ordinary `weave`, `version`, `generate`, and artifact resolution continue to ignore `ImportSource.targetAccessUrl`; only the explicit refresh acquisition path follows it.

## Contract Changes

- Add CLI admission for `weave import --refresh <designatorPath>`, mutually exclusive with the ordinary import source positional, `--working-file`, `--replace-working`, and source-changing options.
- Add a refresh request/result shape that names the target designator and reports a sanitized source origin, destination, observed digest, appended observation IRI, and created/updated/unchanged paths without echoing userinfo, sensitive query parameters, or other secret-bearing request data.
- Load the target Knop source registry, require exactly one applicable `sflo:ImportSource`, and resolve its `targetAccessUrl`, expected digest, target artifact, resolution mode, and prior observations.
- Resolve the current governed local destination from payload inventory and require it to agree with the import observation/binding evidence that the implementation relies on.
- Extract the current bounded HTTP import logic into a shared runtime acquisition helper, add manual redirect policy, and preserve dependency injection for deterministic tests.
- Extend the Weave host-local access profile with remote origin grants appropriate for stored-source replay; do not add host trust rules to portable SFLO config.
- Preserve the current no-ambient-fetch boundary in the shared artifact resolver and packaged in-process API.
- Update source-registry rendering/parsing to preserve and return multiple import observations.
- Ensure compiled CLI binaries carry the network capability required by explicitly authorized import/refresh operations while application policy remains the finer-grained gate.
- Update [[wu.cli-reference.import]], [[wd.runtime]], and [[wd.codebase-overview]]. Update [[sf.spec.2026-05-18-publication-source-binding]] or add a narrower behavior spec if the refresh contract would otherwise be buried in Weave-only documentation.

## Testing

- Unit-test refresh CLI normalization and rejection of mixed ordinary-import/refresh arguments.
- Unit-test remote-origin policy matching by operation/locator kind, scheme, exact origin, default ports, malformed URLs, absence of grants, and malformed host-local settings.
- Unit-test manual redirect handling: allowed same-origin redirect, allowed explicitly granted cross-origin redirect, denied cross-origin redirect before the next request, redirect loop, and redirect-count bound.
- Unit-test source-registry parsing with zero, one, and multiple `ImportSource` bindings and multiple observations.
- Core-plan test append-aware observation rendering while preserving all prior source-registry facts.
- Integration-test successful URL refresh, changed bytes, unchanged bytes, stored expected-digest success, digest mismatch with no writes, timeout, size rejection, HTTP failure, malformed source registry, missing working file, and atomic rollback.
- Integration-test that refresh never mints a HistoricalState and that a later explicit version operation captures the refreshed bytes.
- Regression-test that ordinary weave/generate/resolution still performs no fetch from `ImportSource.targetAccessUrl`.
- E2E-test the packaged CLI permission path so HTTP(S) import and refresh work in a compiled binary under application-level origin policy.
- Use an injected/local HTTP server for normal CI. Do not make test success depend on GitHub or another public service.
- Run focused import, artifact-resolution, source-registry, CLI, and packaging tests, followed by `deno task fmt`, `deno task lint`, `deno task check`, and `deno task test`.

## Non-Goals

- Live no-copy resolution through `sflo:workingAccessUrl`; see [[wa.task.2026.2026-05-20_2152-workingAccessUrl]].
- Generic `sflo:targetAccessUrl` fetching by ordinary artifact resolution, config discovery, page generation, or extraction.
- A new top-level refresh operation.
- Changing the recorded source URL, working destination, or digest expectation during refresh.
- Import destinations outside the mesh root in the first slice.
- Automatic versioning, extraction, generation, publication, git operations, or deployment.
- Authenticated/private URLs, credential storage, cookies, arbitrary request headers, or recursive linked-asset import.
- Persistent HTTP response caching or offline refresh success.
- Treating `ImportSource.targetAccessUrl` as a claim that the URL is uniquely canonical or authoritative; it records acquisition/replay coordinates. A stronger source-role claim requires separate vocabulary and evidence.
- Alternate namespace/base mapping for external ontology term IRIs.

## Implementation Plan

- [ ] Refine the portable behavior in [[sf.spec.2026-05-18-publication-source-binding]] and derive the first failing integration tests.
- [ ] Add refresh CLI request normalization and fail-closed argument combinations.
- [ ] Add import-source replay loading with exactly-one binding selection, destination agreement checks, and multi-observation parsing.
- [ ] Define and implement the Weave-local remote-origin grant shape for stored import sources.
- [ ] Extract and harden the bounded HTTP acquisition helper with per-hop redirect authorization and injected fetch support.
- [ ] Add append-aware import observation planning/rendering without discarding prior source-registry facts.
- [ ] Implement atomic refresh execution, unchanged-byte handling, digest verification, and result/log summaries.
- [ ] Add focused unit, integration, e2e, and packaged-binary tests.
- [ ] Update [[wu.cli-reference.import]], [[wd.runtime]], [[wd.codebase-overview]], and relevant release notes when the feature ships.
- [ ] Run `deno task fmt`, `deno task lint`, `deno task check`, and `deno task test`.
- [ ] Provide a semantic commit message with a summary line and detailed developer-facing bullets.
