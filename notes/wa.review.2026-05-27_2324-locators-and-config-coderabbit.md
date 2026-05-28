---
id: 2py814zip3yte09k41htfff
title: 2026 05 27_2324 Locators and Config Coderabbit
desc: ''
updated: 1779951326173
created: 1779949459817
---

Inline comments:
In `@src/runtime/weave/page_generation.ts`:
- Around line 186-225: The function ownerDesignatorPathForPage is reconstructing
a designator from page.path (handling _knop and _history) which causes wrong
results for simple pages; instead, use the owning designator already threaded
into the page model: for non-mesh pages return page.ownerDesignatorPath (or
page.designatorPath for kinds that already use it like
identifier/customIdentifier/knop) and stop inferring from page.path in
ownerDesignatorPathForPage; leave the existing special-case branches only if
page.ownerDesignatorPath is undefined as a strict fallback, and ensure
resourcePagePresentationForTarget consumes the ownerDesignatorPath value rather
than any path-derived value.


In `@tests/support/fixture_snapshot.ts`:
- Around line 205-214: The snapshot reads use the mutable ref resolved.gitRef
causing potential cache/content mismatches; update the methods that call git
(notably `#listGitTree` and the git show usage in `#buildSnapshot`) to pass the
resolved commit SHA (e.g., resolved.commitSha) instead of resolved.gitRef so
ls-tree and git show read the exact commit used as the cache key; ensure any
other git invocations in the same file (the git show block around the other diff
region) are changed the same way.

---

Outside diff comments:
In `@src/runtime/artifact_resolution/resolver.ts`:
- Around line 910-925: The code returns stateIri from requiredNamedNodeObject
and immediately calls resolvePayloadStateSnapshot without verifying the node is
typed as sflo:HistoricalState, allowing malformed inventories to succeed in
latest-state mode but fail in exact-state mode; add the same type validation
used by resolvePayloadExactState before calling resolvePayloadStateSnapshot:
after obtaining stateIri, assert the quad/triple exists that types stateIri as
sflo:HistoricalState (use the existing helper that checks types - e.g.,
requiredTypeTriple/requiredTypeObject or similar validation used by
resolvePayloadExactState), and if missing throw the same "unavailable" error
message (or the same descriptive message pattern used for other failures) so
latest-state resolution rejects non-HistoricalState nodes consistently with
resolvePayloadExactState.

In `@src/runtime/weave/page_definition.ts`:
- Around line 239-290: The code bypasses the resolver for
targetLocalRelativePath and rejects targetLocatedFile; change the logic so both
single-entry targetLocalRelativePaths and any targetLocatedFile cases delegate
to resolveArtifactResolutionSpecQuads() instead of directly calling
normalizeTargetLocalRelativePath/readAllowedSourceText or throwing;
specifically, in the branch that currently handles targetLocalRelativePaths
(symbols: targetLocalRelativePaths, normalizeTargetLocalRelativePath,
readAllowedSourceText) call resolveArtifactResolutionSpecQuads(designatorPath,
regionSpec, artifactResolutionModes, artifactResolutionFallbackSpecs,
requestedTargetHistories, requestedTargetStates, ...) to obtain a resolved
artifact spec and then honor expectsContentDigest/fallback semantics when
loading content, and remove the hard throw for targetLocatedFile so that
resolveArtifactResolutionSpecQuads() is used for targetLocatedFile resolution as
well (symbols: targetLocatedFile, resolveArtifactResolutionSpecQuads,
expectsContentDigest, artifactResolutionFallbackSpecs).