---
id: i4iu1pnwa3xu05kezam3pu1
title: 2026-05-20_2152-workingAccessUrl
desc: ''
updated: 1779339180859
created: 1779339180859
---

## Goals

- Add Weave runtime support for `sflo:workingAccessUrl` as a current-byte locator for `sflo:DigitalArtifact` payloads.
- Support the common public-repository workflow where a source artifact is available through a raw HTTP(S) URL, especially mutable branch URLs such as GitHub `raw.githubusercontent.com/.../main/...`.
- Keep remote access explicit and fail-closed: mesh RDF may name a `workingAccessUrl`, but commands should only fetch it when the active operational policy permits that locator kind, scheme, and origin.
- Preserve the distinction between mutable working-source locators and immutable historical evidence. A branch raw URL can mean "usually latest"; a historical state should still capture the bytes and digest observed at weave/version time.
- Document how `workingAccessUrl` relates to local working files and the repo-floating source locator covered in [[wa.task.2026.2026-05-19_2349-branch-based-workingfile-fix]].

## Summary

`sflo:workingAccessUrl` is already present in the SFLO core ontology as the remote/external current-byte hook for a `DigitalArtifact`. Weave currently treats it as model vocabulary only: local commands read source bytes from filesystem paths, `file:` URLs, mesh-local `hasWorkingLocatedFile`, `workingLocalRelativePath`, or the newer repository-floating locator, but they do not fetch HTTP(S) working bytes.

The repository-floating locator is portable in the mesh, but operationally it still depends on a local checkout and on the user or CI process keeping the intended branch checked out. A raw URL is simpler for public sources: it can identify "the current source bytes over there" without a local checkout. For a URL pinned to a commit or tag, it is strong provenance. For a URL pinned to a branch such as `main`, it is intentionally mutable: caveat ingestor, but also a useful representation of "usually the latest source artifact."

This task should implement the first explicit remote-current-source slice in Weave, centered on HTTP(S) `workingAccessUrl`, with remote-access policy in operational config and digest-based fail-closed behavior when exactness is requested.

## Discussion

We currently have three useful current-source models:

- Local mesh/source files: `sflo:hasWorkingLocatedFile` and `sflo:workingLocalRelativePath` work well when the current bytes live inside the mesh root or in an explicitly allowed local path.
- Repository-floating files: `sflo:hasRepositorySourceFloatingLocator` plus `sflo:sourceRepositoryUrl` and `sflo:sourceRepositoryPathFromRoot` work well when the mesh should record repository identity and repo-root path without persisting the local checkout path, branch, ref, commit, or digest.
- Remote working files: `sflo:workingAccessUrl` should work well when the current bytes are directly fetchable by URL and the runtime is allowed to use the network.

The raw GitHub URL case is likely to be attractive because it is a one-value locator that works from any machine. For example, a mesh can point at a raw `main` URL for the source ontology and then a release command can fetch whatever bytes are currently published there. That does introduce a real race: the URL can change between validation and versioning, or between two release attempts. That should be treated as a property of mutable working locators, not as a reason to reject the workflow. Users who need immutable evidence can use commit/tag URLs or a digest expectation.

`workingAccessUrl` should not behave like import. Following it should not create or require a mesh-local working copy. The URL remains the current working locator; the historical state captures the bytes observed at the moment Weave versions or releases the artifact.

The config ontology already contains the right policy direction: `sfcfg:RemoteAccessRule`, `sfcfg:hasRemoteLocatorKind`, `sfcfg:remoteLocatorKind_workingAccessUrl`, allowed schemes, and allowed origins. Weave needs the runtime side of that model.

## Open Issues

- What exact operational-config surface should Weave accept first: a mesh-carried `sfcfg:RemoteAccessRule`, a host-local config file, CLI flags, or a narrow built-in option such as `--allow-working-access-url-origin`?
- Should `weave integrate https://... artifact/path` record the URL directly as `sflo:workingAccessUrl`, or should the first implementation require an explicit flag such as `--working-access-url` to avoid accidental remote-source semantics?
- Should integrate fetch the URL immediately to parse/validate the source and compute an observed digest, or should it only record the locator and leave fetching to `weave` / `version`? Recommendation: fetch during integrate when the command needs source semantics, but avoid writing a historical-state digest until a state is actually minted.
- Which digest property should carry an expected remote-byte digest for fail-closed exactness? If no existing SFLO term fits cleanly, add one deliberately rather than overloading a historical-state digest.
- Redirect policy needs a decision. GitHub raw URLs may redirect; the first implementation should either allow safe HTTPS redirects and record the final URL in diagnostics, or fail unless the final origin is also permitted.
- Size, timeout, and content-type limits need conservative defaults. Turtle parsing should still be based on the bytes and extension/media hints Weave already uses, but remote fetches should not be unbounded.
- If multiple current locators are present for the same artifact (`workingLocalRelativePath`, `hasWorkingLocatedFile`, `hasRepositorySourceFloatingLocator`, `workingAccessUrl`), should Weave choose by explicit precedence or require agreement? The SFLO decision log says mismatch should fail closed in operational profiles that rely on multiple locators; Weave should make that concrete.
- Private/authenticated URLs are valuable later, but they raise credential-storage and logging questions. They should probably be deferred from the first slice.

## Decisions

- `workingAccessUrl` is a mutable current-source locator unless paired with commit/tag URL structure or an expected digest. Branch raw URLs are allowed but should be described as "latest-ish" rather than stable evidence.
- First implementation should support HTTP(S) only. Reject other schemes until a concrete use case and policy shape exist.
- Remote access remains deny-by-default. A command must have an applicable operational allowance before fetching `sflo:workingAccessUrl`.
- Versioning/release commands should fetch remote working bytes at the moment they mint the historical state; the historical state should then capture the observed bytes in the normal manifestation files and digest/provenance trail.
- A digest expectation, when supplied, must fail closed if the fetched bytes do not match.
- `workingAccessUrl` should be documented next to repo-floating source locators in [[wu.cli-reference.integrate]] because users will choose between those two models for public repositories.

## Contract Changes

- `weave integrate` may accept an HTTP(S) source URL once remote access has been explicitly allowed. It should record the URL as `sflo:workingAccessUrl` on the payload artifact rather than converting it into a local path.
- Current-byte resolution for weave/version/release flows should include a remote resolver path: read `sflo:workingAccessUrl`, check the active remote policy, fetch bytes, validate/parse as needed, and surface clear diagnostics on policy denial, network failure, parse failure, or digest mismatch.
- `weave validate` should report unsupported or disallowed `workingAccessUrl` locators when the active validation profile asks it to validate operational resolvability.
- Existing local-path behavior should not change. Mesh-local files, sidecar local files, and repository-floating locators remain valid and should keep using their existing predicates.
- Resource page generation should continue preferring settled historical-state content when available. `workingAccessUrl` may be displayed as current locator metadata, but generation should not fetch the network merely to render current resource pages unless an explicit working-resolution mode requires it.

## Testing

- Add unit tests for remote-access policy matching: locator kind, scheme, origin, denied-by-default behavior, and malformed URL failures.
- Add an integration test with a local HTTP server standing in for a raw public URL. Verify `integrate` records `sflo:workingAccessUrl` and does not persist a host-local filesystem path.
- Add weave/version tests proving that the fetched remote bytes become the historical-state manifestation bytes.
- Add a mutable-URL test where the server changes bytes between two runs. Without an expected digest, the later run should capture the later bytes; with an expected digest, it should fail closed.
- Add denial tests proving that a disallowed URL fails before or at fetch resolution with a clear diagnostic.
- Add redirect tests once redirect behavior is chosen.
- Add documentation examples for raw GitHub `main` URLs and pinned commit URLs.

## Non-Goals

- Do not implement authenticated/private URL fetching in the first slice.
- Do not make `workingAccessUrl` an import/copy mechanism. It resolves current bytes; it does not create a mesh-local working source.
- Do not implement `sflo:targetAccessUrl` page-source fetching as part of this task unless the remote resolver abstraction makes it trivial and separately tested.
- Do not make mutable branch URLs appear immutable. Users who need immutability should use commit/tag URLs or digest expectations.
- Do not add general URL-root-relative filesystem abstractions. The useful concrete cases are repository-floating locators and direct remote access URLs.

## Implementation Plan

- [ ] Inspect the current `sfcfg:RemoteAccessRule` ontology shape and choose the smallest runtime config surface Weave can support without painting itself into a corner.
- [ ] Add a remote-access policy parser/matcher for `remoteLocatorKind_workingAccessUrl`, allowed scheme, and allowed origin.
- [ ] Add a bounded HTTP(S) fetch helper with timeout, maximum byte size, redirect policy, final-URL diagnostics, and digest calculation.
- [ ] Extend integrate source resolution so an explicitly allowed HTTP(S) source can be recorded as `sflo:workingAccessUrl`.
- [ ] Extend payload current-byte resolution to fetch `sflo:workingAccessUrl` when no local/repository current locator is selected, or when policy/precedence explicitly selects the URL locator.
- [ ] Decide and implement multiple-locator precedence/agreement checks.
- [ ] Add tests for policy denial, successful fetch, mutable URL behavior, digest mismatch, and no local path leakage.
- [ ] Update [[wu.cli-reference.integrate]] and any relevant release/runbook notes with examples for raw `main` URLs and pinned commit URLs.
