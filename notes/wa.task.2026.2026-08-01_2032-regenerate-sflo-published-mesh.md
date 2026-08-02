---
id: cplkv9b6zzh30mkatcehm69w
title: Regenerate SFLO Published Mesh From Scratch
desc: Rebuild the SFLO gh-pages mesh with current Weave and compatible tagged SFLO source, replacing the existing publication with one root commit.
updated: 1785641565000
created: 1785641565000
---

## Goals

- Regenerate the complete SFLO branch-published semantic mesh from scratch with a pinned current Weave revision and an immutable, vocabulary-compatible SFLO source release.
- Make the resulting `gh-pages` branch appear to contain the first published mesh version: one root publication commit with no old publication commits in its ancestry.
- Preserve the established public mesh base, payload topology, root welcome surface, and explicitly ruled host assets.
- Replace legacy generated `sflo:hasTargetArtifact` / `sflo:hasRequestedTargetState` bindings with current `sflo:targetArtifact` / `sflo:targetHistoricalState` bindings.
- Produce a deterministic, validated, reviewable publication with source, term-census, path, blob, and Git receipts.

## Summary

Dave ruled on 2026-08-01 that the SFLO `gh-pages` mesh will be regenerated from scratch with current code, making the result appear to be the first published version.

The existing `gh-pages` branch contains two commits: root `aed218c780969849206091bb960e7e706e8cf879` (`Publish initial SFLO semantic mesh`) and child `a7e7626f1534f9befef2c2a7b28f294ed84e18f4` (`lovely manual re-creation`). The second commit appears to have introduced the generated-vocabulary drift and expanded the tree to 7,234 files and 379 Knops.

The v0.2.0 source payloads on SFLO `main` use the legacy artifact-resolution vocabulary, while current Weave emits and resolves the renamed direct predicates `sflo:targetArtifact` and `sflo:targetHistoricalState`. Current Weave must therefore not regenerate a mesh whose payload still claims to define the old vocabulary. The present `next/v0.2.1` source has compatible vocabulary but still contains v0.2.0 release identities, so a properly prepared and tagged SFLO release is a hard prerequisite.

This task starts only after all Open Issues have rulings and the compatible SFLO source tag exists.

## Discussion

### Evidence

The original topology and operation order are documented in [[wa.completed.2026.2026-05-16_1707-create-sflo-branch-mesh]]: separate source and publication worktrees, `integrate` rather than `import`, mesh base `https://semantic-flow.github.io/sflo/`, root welcome payload, three ontology payloads, all-terms extraction, weave, generation, and validation.

Generic branch-publication boundaries and rebuild safety are documented in [[wa.completed.2026.2026-05-13_1655-support-gh-pages-branch-based-deployments]]. The current concrete replay is in [[wu.cli-reference.examples.sflo]], with the exact-state extraction improvement evidenced by [[wu.cli-reference.examples.urpx]] and [[wu.cli-reference.extract]]. Host controls are defined in [[wu.cli-reference.mesh.create]] and [[wu.cli-reference.validate]]. Runtime path and settings behavior are defined in [[wd.runtime]]. Intentional publication-history rewrites and `--force-with-lease` review are documented in [[wd.testing.fixture-ladder-regeneration]].

SFLO source release discipline is documented in [[ont.dev.release-runbook]] and [[ont.release-notes.v0.2.0]]. A publication release must be replayed from a detached worktree at an immutable tag commit. Core, config, and core SHACL are the `/sflo/` Pages payloads; job and provenance remain source-only until their non-`/sflo/` topology is settled. Weave manages `.nojekyll` for the GitHub Pages profile but does not manage `CNAME`.

Current Weave emits `sflo:targetArtifact` in `src/core/integrate/integrate.ts`, and emits `sflo:targetArtifact` plus `sflo:targetHistoricalState` in `src/core/extract/extract.ts`. The SFLO v0.2.0 payload instead defines the legacy predicates. The current `next/v0.2.1` payload defines the direct predicates but is not yet release-prepared.

### Existing publication baseline

- Old root commit: `aed218c780969849206091bb960e7e706e8cf879`.
- Old current commit: `a7e7626f1534f9befef2c2a7b28f294ed84e18f4`.
- Old tree: 7,234 files and 379 Knops.
- Old Knop distribution: root 1, `config` 225, `ontology` 153.
- Old payload mappings: core → `ontology`, config → `config`, core SHACL → `ontology/shacl`.
- Old release state: `v0.2.0`.
- Both `config/_knop` and `ontology/_knop` are intentional payload Knops.
- `.nojekyll` exists.
- `favicon.ico` exists and was introduced by the second commit.
- No `CNAME` exists.
- No `assets/` directory exists.
- The three published v0.2.0 payload blobs exactly match their source files at the v0.2.0 source commit.

Against SFLO `next/v0.2.1` commit `d765a52e2b2952f945fe1793976e5af0fcbf7c46`, the projected fresh mesh contains 362 Knops: root 1, three integrated payload Knops, and 358 extracted term Knops. Relative to the old mesh, 60 designators retire and 43 appear. These figures are planning evidence, not a substitute for freezing a new expected manifest after the final source tag exists.

### Exact ordered regeneration recipe

The following is the exact recipe under the recommended decisions: SFLO v0.2.1, the existing mesh base and payload mappings, all-governed-artifact versioned history, exact release-state extraction, one preserved favicon, one orphan publication commit, and an archived old tip. Replace the two explicitly marked ruled values only after the corresponding rulings and source release exist.

#### 1. Pin inputs and establish isolated worktrees

```sh
export WEAVE_ROOT='/home/djradon/hub/semantic-flow/weave'
export WEAVE_CLI="$WEAVE_ROOT/src/main.ts"
export WEAVE_COMMIT='ff2b0f6507e60b93664675d5893669192ee7c0ff'

export SFLO_REPO="$WEAVE_ROOT/dependencies/github.com/semantic-flow/sflo"
export SFLO_SOURCE_REF='v0.2.1'
export SFLO_SOURCE_COMMIT='<RULED FULL V0.2.1 TAG COMMIT SHA>'
export SFLO_RELEASE_VERSION='0.2.1'
export SFLO_RELEASE_STATE="v$SFLO_RELEASE_VERSION"

export SFLO_OLD_ROOT='aed218c780969849206091bb960e7e706e8cf879'
export SFLO_OLD_GH_PAGES='a7e7626f1534f9befef2c2a7b28f294ed84e18f4'
export SFLO_REGEN_BRANCH='regen/gh-pages-first-publication-2026-08-01'
export SFLO_ARCHIVE_TAG='archive/gh-pages-before-regeneration-2026-08-01'
export SFLO_GENERATED_AT='<RULED ISO-8601 INSTANT WITH EXPLICIT OFFSET>'
export SFLO_EXPECTED_KNOP_COUNT='362'

export SFLO_RUN_ROOT
SFLO_RUN_ROOT="$(mktemp -d /tmp/sflo-regeneration.XXXXXX)"
export SFLO_SRC="$SFLO_RUN_ROOT/source"
export SFLO_PUB="$SFLO_RUN_ROOT/publication"
export WEAVE_LOG_DIR="$SFLO_RUN_ROOT/logs"
export WEAVE_SETTINGS="$SFLO_RUN_ROOT/settings"

mkdir -p "$WEAVE_LOG_DIR" "$WEAVE_SETTINGS"
test "$SFLO_SOURCE_COMMIT" != '<RULED FULL V0.2.1 TAG COMMIT SHA>'
test "$SFLO_GENERATED_AT" != '<RULED ISO-8601 INSTANT WITH EXPLICIT OFFSET>'
```

```sh
test "$(git -C "$WEAVE_ROOT" rev-parse HEAD)" = "$WEAVE_COMMIT"
test -z "$(git -C "$WEAVE_ROOT" status --porcelain)"
test -z "$(git -C "$SFLO_REPO" status --porcelain)"

git -C "$SFLO_REPO" fetch --prune origin
git -C "$SFLO_REPO" fetch origin "refs/tags/$SFLO_SOURCE_REF:refs/tags/$SFLO_SOURCE_REF"

test "$(git -C "$SFLO_REPO" rev-parse "$SFLO_SOURCE_REF^{commit}")" = "$SFLO_SOURCE_COMMIT"
git -C "$SFLO_REPO" merge-base --is-ancestor "$SFLO_SOURCE_COMMIT" refs/remotes/origin/main

test "$(git -C "$SFLO_REPO" rev-parse refs/remotes/origin/gh-pages)" = "$SFLO_OLD_GH_PAGES"
test "$(git -C "$SFLO_REPO" rev-parse "$SFLO_OLD_GH_PAGES^")" = "$SFLO_OLD_ROOT"
test "$(git -C "$SFLO_REPO" rev-list --count "$SFLO_OLD_GH_PAGES")" = '2'
test "$(git -C "$SFLO_REPO" ls-remote --heads origin gh-pages | cut -f1)" = "$SFLO_OLD_GH_PAGES"

git -C "$SFLO_REPO" worktree list --porcelain
git -C "$SFLO_REPO" worktree add --detach "$SFLO_SRC" "$SFLO_SOURCE_COMMIT"
git -C "$SFLO_REPO" worktree add --orphan -b "$SFLO_REGEN_BRANCH" "$SFLO_PUB"

test "$(git -C "$SFLO_SRC" rev-parse HEAD)" = "$SFLO_SOURCE_COMMIT"
test -z "$(git -C "$SFLO_SRC" status --porcelain)"
test -z "$(git -C "$SFLO_PUB" status --porcelain)"
```

#### 2. Capture old-tree and source receipts

```sh
git -C "$SFLO_REPO" ls-tree -r --name-only "$SFLO_OLD_GH_PAGES" > "$SFLO_RUN_ROOT/old-paths.txt"
git -C "$SFLO_REPO" ls-tree -r "$SFLO_OLD_GH_PAGES" > "$SFLO_RUN_ROOT/old-blobs.txt"

git -C "$SFLO_REPO" ls-tree -r --name-only "$SFLO_OLD_GH_PAGES" \
  | awk '/(^|\\/)\\_knop\\/\\_meta\\/meta\\.ttl$/ { sub(/\\/_knop\\/_meta\\/meta\\.ttl$/, ""); if ($0 == "_knop/_meta/meta.ttl") $0 = "/"; print }' \
  | sort -u > "$SFLO_RUN_ROOT/old-knops.txt"

git -C "$SFLO_REPO" show "$SFLO_OLD_GH_PAGES:favicon.ico" > "$SFLO_RUN_ROOT/old-favicon.ico"

if git -C "$SFLO_REPO" cat-file -e "$SFLO_OLD_GH_PAGES:CNAME" 2>/dev/null; then
  echo 'Unexpected CNAME in the ruled old publication commit.' >&2
  exit 1
fi

if git -C "$SFLO_REPO" cat-file -e "$SFLO_OLD_GH_PAGES:assets" 2>/dev/null; then
  echo 'Unexpected assets directory in the ruled old publication commit.' >&2
  exit 1
fi
```

```sh
(cd "$SFLO_SRC" && deno task release:validate --version "$SFLO_RELEASE_VERSION" --require-tag)
(cd "$SFLO_SRC" && deno task ci)

riot --validate \
  "$SFLO_SRC/semantic-flow-core-ontology.ttl" \
  "$SFLO_SRC/semantic-flow-core-shacl.ttl" \
  "$SFLO_SRC/semantic-flow-config-ontology.ttl" \
  "$SFLO_SRC/semantic-flow-job-ontology.ttl" \
  "$SFLO_SRC/semantic-flow-prov-ontology.ttl"

sha256sum \
  "$SFLO_SRC/semantic-flow-core-ontology.ttl" \
  "$SFLO_SRC/semantic-flow-config-ontology.ttl" \
  "$SFLO_SRC/semantic-flow-core-shacl.ttl" \
  > "$SFLO_RUN_ROOT/source-sha256.txt"
```

#### 3. Create the mesh at the established Pages base

```sh
deno run -A "$WEAVE_CLI" mesh create \
  --workspace "$SFLO_PUB" \
  --mesh-base 'https://semantic-flow.github.io/sflo/' \
  --publication-profile github-pages
```

Replace the generated config with the ruled durable SFLO policy:

```sh
cat > "$SFLO_PUB/_mesh/_config/config.ttl" <<'TTL'
@prefix sfcfg: <https://semantic-flow.github.io/sflo/config/> .

<> a sfcfg:MeshConfig ;
  sfcfg:hasPublicationProfile sfcfg:publicationProfile_githubPages ;
  sfcfg:hasPolicyBinding
    <#history-versioned>,
    <#resource-page-presentation-all-panels> .

<#history-versioned> a sfcfg:PolicyBinding ;
  sfcfg:bindsPolicy <#versioned-history> ;
  sfcfg:appliesToPolicyTarget <#any-governed-artifact> .

<#versioned-history> a sfcfg:PolicyDefinition ;
  sfcfg:hasHistoryTrackingPolicy sfcfg:historyTrackingPolicy_versioned .

<#resource-page-presentation-all-panels> a sfcfg:PolicyBinding ;
  sfcfg:bindsPolicy <#semantic-site-all-panels> ;
  sfcfg:appliesToPolicyTarget <#any-governed-artifact> .

<#semantic-site-all-panels> a sfcfg:PolicyDefinition ;
  sfcfg:hasResourcePagePresentationPolicy <https://semantic-flow.github.io/weave/defaults/resource-page-presentation/semantic-site-all-panels> .

<#any-governed-artifact> a sfcfg:AnyGovernedArtifactPolicyTarget .
TTL
```

Initialize mesh support history before any payload candidate exists:

```sh
deno run -A "$WEAVE_CLI" \
  --mesh-root "$SFLO_PUB" \
  --generated-at "$SFLO_GENERATED_AT"
```

#### 4. Integrate and weave the root welcome payload

```sh
cat > "$SFLO_PUB/welcome.ttl" <<'TTL'
@base <https://semantic-flow.github.io/sflo/> .
@prefix dcterms: <http://purl.org/dc/terms/> .

<https://semantic-flow.github.io/sflo> dcterms:title "Semantic Flow Ontology and Related Resources" ;
  dcterms:description "The Semantic Flow core ontology and other related resources provide a way to create identifiers that are dereferenceable, resilient, and explorable. It formalizes how Semantic Flow designators, supporting artifacts, and optional payload resources can be combined to make these identifiers useful." .
TTL
```

```sh
if rg '\\>' "$SFLO_PUB/welcome.ttl"; then
  echo 'Turtle IRI delimiters in welcome.ttl must use plain >.' >&2
  exit 1
fi

deno run -A "$WEAVE_CLI" integrate "$SFLO_PUB/welcome.ttl" / \
  --mesh-root "$SFLO_PUB"

deno run -A "$WEAVE_CLI" \
  --mesh-root "$SFLO_PUB" \
  --target 'designatorPath=/' \
  --generated-at "$SFLO_GENERATED_AT"
```

Preserve the ruled manual asset:

```sh
cp "$SFLO_RUN_ROOT/old-favicon.ico" "$SFLO_PUB/favicon.ico"
test "$(git hash-object "$SFLO_PUB/favicon.ico")" = "$(git -C "$SFLO_REPO" rev-parse "$SFLO_OLD_GH_PAGES:favicon.ico")"
```

#### 5. Integrate the three tagged source payloads

```sh
deno run -A "$WEAVE_CLI" integrate "$SFLO_SRC/semantic-flow-core-ontology.ttl" ontology \
  --mesh-root "$SFLO_PUB" \
  --grant-source-directory "$SFLO_SRC" \
  --source-repository-current \
  --source-repository-url 'https://github.com/semantic-flow/sflo.git'

deno run -A "$WEAVE_CLI" integrate "$SFLO_SRC/semantic-flow-config-ontology.ttl" config \
  --mesh-root "$SFLO_PUB" \
  --grant-source-directory "$SFLO_SRC" \
  --source-repository-current \
  --source-repository-url 'https://github.com/semantic-flow/sflo.git'

deno run -A "$WEAVE_CLI" integrate "$SFLO_SRC/semantic-flow-core-shacl.ttl" ontology/shacl \
  --mesh-root "$SFLO_PUB" \
  --grant-source-directory "$SFLO_SRC" \
  --source-repository-current \
  --source-repository-url 'https://github.com/semantic-flow/sflo.git'
```

Verify portable floating bindings and reject local-path or accidental pinning evidence:

```sh
rg 'hasRepositorySourceFloatingLocator|sourceRepositoryPathFromRoot' \
  "$SFLO_PUB/ontology/_knop/_sources/sources.ttl" \
  "$SFLO_PUB/config/_knop/_sources/sources.ttl" \
  "$SFLO_PUB/ontology/shacl/_knop/_sources/sources.ttl" >/dev/null

if rg 'targetLocalRelativePath|workingLocalRelativePath|sourceRepositoryRef|sourceRepositoryCommit|expectsContentDigest|hasContentDigest|/home/|/tmp/' \
  "$SFLO_PUB/ontology/_knop/_sources/sources.ttl" \
  "$SFLO_PUB/config/_knop/_sources/sources.ttl" \
  "$SFLO_PUB/ontology/shacl/_knop/_sources/sources.ttl" \
  "$SFLO_PUB/ontology/_knop/_inventory/inventory.ttl" \
  "$SFLO_PUB/config/_knop/_inventory/inventory.ttl" \
  "$SFLO_PUB/ontology/shacl/_knop/_inventory/inventory.ttl"
then
  echo 'Expected portable floating repository bindings without host-local paths or accidental pinning.' >&2
  exit 1
fi
```

#### 6. Create the three release states in one deterministic payload batch

```sh
deno run -A "$WEAVE_CLI" set history ontology releases \
  --mesh-root "$SFLO_PUB"

deno run -A "$WEAVE_CLI" set next-state ontology "$SFLO_RELEASE_STATE" \
  --mesh-root "$SFLO_PUB"

deno run -A "$WEAVE_CLI" set history config releases \
  --mesh-root "$SFLO_PUB"

deno run -A "$WEAVE_CLI" set next-state config "$SFLO_RELEASE_STATE" \
  --mesh-root "$SFLO_PUB"

deno run -A "$WEAVE_CLI" set history ontology/shacl releases \
  --mesh-root "$SFLO_PUB"

deno run -A "$WEAVE_CLI" set next-state ontology/shacl "$SFLO_RELEASE_STATE" \
  --mesh-root "$SFLO_PUB"
```

```sh
deno run -A "$WEAVE_CLI" \
  --mesh-root "$SFLO_PUB" \
  --payload-manifestation-segment ttl \
  --target 'designatorPath=ontology' \
  --target 'designatorPath=config' \
  --target 'designatorPath=ontology/shacl' \
  --generated-at "$SFLO_GENERATED_AT"
```

Verify byte identity between tagged source and locally woven release payloads:

```sh
cmp \
  "$SFLO_SRC/semantic-flow-core-ontology.ttl" \
  "$SFLO_PUB/ontology/releases/$SFLO_RELEASE_STATE/ttl/semantic-flow-core-ontology.ttl"

cmp \
  "$SFLO_SRC/semantic-flow-config-ontology.ttl" \
  "$SFLO_PUB/config/releases/$SFLO_RELEASE_STATE/ttl/semantic-flow-config-ontology.ttl"

cmp \
  "$SFLO_SRC/semantic-flow-core-shacl.ttl" \
  "$SFLO_PUB/ontology/shacl/releases/$SFLO_RELEASE_STATE/ttl/semantic-flow-core-shacl.ttl"
```

#### 7. Extract all mesh-scoped terms from exact release snapshots

The order is contractual because existing Knops are skipped and source references are only created for terms newly extracted by that invocation. Core wins first-source attribution over config; config wins over SHACL.

```sh
deno run -A "$WEAVE_CLI" extract --all-terms \
  --mesh-root "$SFLO_PUB" \
  --source-state "ontology/releases/$SFLO_RELEASE_STATE" \
  --add-source-references \
  --reference-role canonical \
  --accept-preview

deno run -A "$WEAVE_CLI" extract --all-terms \
  --mesh-root "$SFLO_PUB" \
  --source-state "config/releases/$SFLO_RELEASE_STATE" \
  --add-source-references \
  --reference-role canonical \
  --accept-preview

deno run -A "$WEAVE_CLI" extract --all-terms \
  --mesh-root "$SFLO_PUB" \
  --source-state "ontology/shacl/releases/$SFLO_RELEASE_STATE" \
  --add-source-references \
  --reference-role canonical \
  --accept-preview
```

#### 8. Weave all extracted terms, regenerate, and validate

```sh
deno run -A "$WEAVE_CLI" \
  --mesh-root "$SFLO_PUB" \
  --generated-at "$SFLO_GENERATED_AT"

deno run -A "$WEAVE_CLI" generate \
  --mesh-root "$SFLO_PUB" \
  --generated-at "$SFLO_GENERATED_AT"

deno run -A "$WEAVE_CLI" validate mesh \
  --mesh-root "$SFLO_PUB"

deno run -A "$WEAVE_CLI" validate publication \
  --mesh-root "$SFLO_PUB"
```

Parse every generated Turtle file:

```sh
find "$SFLO_PUB" -type f -name '*.ttl' -print0 \
  | xargs -0 -n 100 riot --validate
```

#### 9. Freeze and compare the new census

```sh
find "$SFLO_PUB" -type f -path '*/_knop/_meta/meta.ttl' -printf '%P\n' \
  | sed -e 's#/_knop/_meta/meta\\.ttl$##' -e 's#^_knop/_meta/meta\\.ttl$#/#' \
  | sort -u > "$SFLO_RUN_ROOT/new-knops.txt"

test "$(wc -l < "$SFLO_RUN_ROOT/new-knops.txt" | tr -d ' ')" = "$SFLO_EXPECTED_KNOP_COUNT"

comm -23 "$SFLO_RUN_ROOT/old-knops.txt" "$SFLO_RUN_ROOT/new-knops.txt" \
  > "$SFLO_RUN_ROOT/retired-knops.txt"

comm -13 "$SFLO_RUN_ROOT/old-knops.txt" "$SFLO_RUN_ROOT/new-knops.txt" \
  > "$SFLO_RUN_ROOT/added-knops.txt"

wc -l \
  "$SFLO_RUN_ROOT/old-knops.txt" \
  "$SFLO_RUN_ROOT/new-knops.txt" \
  "$SFLO_RUN_ROOT/retired-knops.txt" \
  "$SFLO_RUN_ROOT/added-knops.txt"
```

If the final v0.2.1 tag changes only release metadata from the probed `d765a52` vocabulary, expect 379 old, 362 new, 60 retired, and 43 added. Any other count requires a reviewed source-vocabulary explanation and an updated frozen manifest before publication.

Verify vocabulary migration and absence of host-local paths:

```sh
rg 'sflo:targetArtifact' "$SFLO_PUB" --glob '*.ttl' >/dev/null
rg 'sflo:targetHistoricalState' "$SFLO_PUB" --glob '*.ttl' >/dev/null

if rg 'sflo:(hasTargetArtifact|hasRequestedTargetState)' "$SFLO_PUB" --glob '*.ttl'; then
  echo 'Legacy generated artifact-resolution vocabulary remains.' >&2
  exit 1
fi

if rg -F "$SFLO_RUN_ROOT" "$SFLO_PUB" || rg -F "$SFLO_REPO" "$SFLO_PUB"; then
  echo 'Host-local regeneration path leaked into publication output.' >&2
  exit 1
fi
```

Verify the ruled publication boundary and controls:

```sh
test -f "$SFLO_PUB/.nojekyll"
test -f "$SFLO_PUB/favicon.ico"
test ! -e "$SFLO_PUB/CNAME"
test ! -e "$SFLO_PUB/assets"

test -d "$SFLO_PUB/config/_knop"
test -d "$SFLO_PUB/ontology/_knop"
test -d "$SFLO_PUB/ontology/shacl/_knop"

test ! -d "$SFLO_PUB/job"
test ! -d "$SFLO_PUB/prov"

test -f "$SFLO_PUB/ontology/releases/$SFLO_RELEASE_STATE/ttl/semantic-flow-core-ontology.ttl"
test -f "$SFLO_PUB/config/releases/$SFLO_RELEASE_STATE/ttl/semantic-flow-config-ontology.ttl"
test -f "$SFLO_PUB/ontology/shacl/releases/$SFLO_RELEASE_STATE/ttl/semantic-flow-core-shacl.ttl"

rg '<summary>Semantic Flow metadata</summary>' "$SFLO_PUB" --glob '*.html' >/dev/null
test -z "$(git -C "$SFLO_SRC" status --porcelain)"
```

#### 10. Create and review the one-root publication commit

```sh
git -C "$SFLO_PUB" add --all
git -C "$SFLO_PUB" diff --cached --check
git -C "$SFLO_PUB" status --short

git -C "$SFLO_PUB" commit \
  -m 'Publish SFLO semantic mesh' \
  -m "Build from SFLO $SFLO_SOURCE_REF at $SFLO_SOURCE_COMMIT with Weave $WEAVE_COMMIT.

Regenerate the GitHub Pages mesh from scratch using the current artifact-resolution vocabulary, exact release-state term extraction, and the ruled SFLO publication topology."

test "$(git -C "$SFLO_PUB" rev-list --count HEAD)" = '1'
test "$(git -C "$SFLO_PUB" rev-list --parents -n 1 HEAD | wc -w | tr -d ' ')" = '1'

git -C "$SFLO_PUB" diff --stat "$SFLO_OLD_GH_PAGES" HEAD
git -C "$SFLO_PUB" diff --name-status "$SFLO_OLD_GH_PAGES" HEAD \
  > "$SFLO_RUN_ROOT/old-to-new-name-status.txt"
```

#### 11. Preserve or retire the old commits according to the ruling

If the archive-tag recommendation is accepted, preserve both old commits through the old tip before replacing `gh-pages`:

```sh
git -C "$SFLO_REPO" tag -a "$SFLO_ARCHIVE_TAG" "$SFLO_OLD_GH_PAGES" \
  -m 'Archive SFLO gh-pages before scratch regeneration'

git -C "$SFLO_REPO" push origin "refs/tags/$SFLO_ARCHIVE_TAG"
```

If the ruling forbids a public archive ref, skip those commands and record both immutable commit IDs in this task and the publication receipt. Do not imply that force-updating the branch immediately erases the underlying Git objects.

#### 12. Publish with an exact lease

```sh
test "$(git -C "$SFLO_REPO" ls-remote --heads origin gh-pages | cut -f1)" = "$SFLO_OLD_GH_PAGES"

git -C "$SFLO_PUB" push \
  --force-with-lease=refs/heads/gh-pages:"$SFLO_OLD_GH_PAGES" \
  origin \
  HEAD:refs/heads/gh-pages

export SFLO_NEW_GH_PAGES
SFLO_NEW_GH_PAGES="$(git -C "$SFLO_PUB" rev-parse HEAD)"

test "$(git -C "$SFLO_REPO" ls-remote --heads origin gh-pages | cut -f1)" = "$SFLO_NEW_GH_PAGES"
```

#### 13. Verify the deployed Pages surface

Confirm in repository settings that GitHub Pages still serves `gh-pages` from `/` and that no custom domain is configured.

```sh
mkdir -p "$SFLO_RUN_ROOT/fetched"

curl -fsSL --retry 12 --retry-delay 10 \
  'https://semantic-flow.github.io/sflo/' \
  -o "$SFLO_RUN_ROOT/fetched/index.html"

curl -fsSL --retry 12 --retry-delay 10 \
  "https://semantic-flow.github.io/sflo/ontology/releases/$SFLO_RELEASE_STATE/ttl/semantic-flow-core-ontology.ttl" \
  -o "$SFLO_RUN_ROOT/fetched/semantic-flow-core-ontology.ttl"

curl -fsSL --retry 12 --retry-delay 10 \
  "https://semantic-flow.github.io/sflo/config/releases/$SFLO_RELEASE_STATE/ttl/semantic-flow-config-ontology.ttl" \
  -o "$SFLO_RUN_ROOT/fetched/semantic-flow-config-ontology.ttl"

curl -fsSL --retry 12 --retry-delay 10 \
  "https://semantic-flow.github.io/sflo/ontology/shacl/releases/$SFLO_RELEASE_STATE/ttl/semantic-flow-core-shacl.ttl" \
  -o "$SFLO_RUN_ROOT/fetched/semantic-flow-core-shacl.ttl"

cmp "$SFLO_SRC/semantic-flow-core-ontology.ttl" "$SFLO_RUN_ROOT/fetched/semantic-flow-core-ontology.ttl"
cmp "$SFLO_SRC/semantic-flow-config-ontology.ttl" "$SFLO_RUN_ROOT/fetched/semantic-flow-config-ontology.ttl"
cmp "$SFLO_SRC/semantic-flow-core-shacl.ttl" "$SFLO_RUN_ROOT/fetched/semantic-flow-core-shacl.ttl"

riot --validate \
  "$SFLO_RUN_ROOT/fetched/semantic-flow-core-ontology.ttl" \
  "$SFLO_RUN_ROOT/fetched/semantic-flow-config-ontology.ttl" \
  "$SFLO_RUN_ROOT/fetched/semantic-flow-core-shacl.ttl"

curl -fsSL --retry 12 --retry-delay 10 \
  'https://semantic-flow.github.io/sflo/ontology/ArtifactHistory/' \
  -o "$SFLO_RUN_ROOT/fetched/artifact-history.html"

curl -fsSL --retry 12 --retry-delay 10 \
  'https://semantic-flow.github.io/sflo/ontology/ArtifactResolutionSpec/' \
  -o "$SFLO_RUN_ROOT/fetched/artifact-resolution-spec.html"

curl -fsSL --retry 12 --retry-delay 10 \
  'https://semantic-flow.github.io/sflo/config/PolicyBinding/' \
  -o "$SFLO_RUN_ROOT/fetched/policy-binding.html"

curl -fsSL --retry 12 --retry-delay 10 \
  'https://semantic-flow.github.io/sflo/favicon.ico' \
  -o "$SFLO_RUN_ROOT/fetched/favicon.ico"

cmp "$SFLO_RUN_ROOT/old-favicon.ico" "$SFLO_RUN_ROOT/fetched/favicon.ico"
```

### Legitimate differences from the old mesh

- Generated source, extraction, and reference bindings use `sflo:targetArtifact` and `sflo:targetHistoricalState`.
- Removed pre-1.0 vocabulary designators disappear and newly introduced direct-resolution, policy-binding, and related designators appear.
- The term census changes; against the currently probed source this is 379 → 362, with 60 retired and 43 added.
- The published ontology release state changes from v0.2.0 to the newly ruled compatible release.
- Generated inventories, histories, state segments, HTML, metadata panels, and timestamps change because the mesh is rebuilt with current Weave from empty state.
- The Git branch changes from two commits to one unrelated root commit.
- File and blob churn across generated support artifacts is expected and may be broad, as described for deterministic regeneration in [[wd.testing.fixture-ladder-regeneration]].

### Differences that must not occur

- The mesh base must not change from `https://semantic-flow.github.io/sflo/`.
- The root canonical IRI must not gain a trailing slash.
- Core, config, and core SHACL must not move from `ontology`, `config`, and `ontology/shacl`.
- Published release payload bytes must not differ from the three files at the ruled source tag.
- Job or provenance payloads must not enter the `/sflo/` Pages mesh.
- Common surviving term IRIs must not change designator paths or stop resolving.
- Generated RDF must not contain legacy artifact-resolution predicates, host-local checkout paths, traversal outside the mesh, or accidental branch/commit coordinates under the floating-source ruling.
- `.nojekyll` must not disappear.
- `favicon.ico` must not change under the preservation ruling.
- `CNAME` or an `assets/` directory must not appear without a new ruling.
- The source worktree must not be modified.
- The regenerated `gh-pages` commit must not have a parent.
- Publication must not be pushed if the remote `gh-pages` tip has moved from the recorded lease SHA.
- Generated RDF or HTML must not be manually repaired after generation; failures return to source, configuration, or Weave.

## Acceptance Criteria

- [ ] Every Open Issue below has an explicit recorded ruling.
- [ ] A compatible SFLO source release is prepared, merged to `main`, tagged, pushed, and validated with `--require-tag`.
- [ ] The source worktree is detached at the exact ruled tag commit and remains clean throughout regeneration.
- [ ] Weave runs at the exact ruled commit.
- [ ] The mesh is created from an empty orphan publication worktree with base `https://semantic-flow.github.io/sflo/` and profile `github-pages`.
- [ ] The root welcome payload is integrated and generated at `/` with the slashless canonical root subject.
- [ ] Exactly three source payloads are integrated at `ontology`, `config`, and `ontology/shacl`.
- [ ] The three locally woven and remotely fetched release payloads are byte-identical to the tagged source files.
- [ ] All-term extraction uses exact `--source-state` coordinates in core, config, SHACL order with canonical source references.
- [ ] The final Knop, added-designator, and retired-designator manifests match the frozen post-tag expectations or have an explicitly reviewed source-vocabulary explanation.
- [ ] Generated support RDF consistently uses `sflo:targetArtifact` and `sflo:targetHistoricalState`; no legacy generated target predicates remain.
- [ ] `weave validate mesh`, `weave validate publication`, full Turtle syntax validation, source CI, and source release validation pass.
- [ ] No regeneration path, source checkout path, or parent traversal leaks into published RDF.
- [ ] `.nojekyll` exists, `favicon.ico` matches the ruled old blob, and `CNAME` plus `assets/` remain absent.
- [ ] The new publication commit is a root commit and the entire `gh-pages` branch has exactly one commit.
- [ ] Any old-commit archive ref is handled exactly as ruled.
- [ ] The non-fast-forward publication uses an explicit lease against `a7e7626f1534f9befef2c2a7b28f294ed84e18f4`.
- [ ] GitHub Pages still serves `gh-pages` from `/`, has the ruled custom-domain setting, and serves the root, three release payloads, representative surviving terms, and representative new terms.
- [ ] The durable SFLO CLI example and release runbook reflect the final exact-state regeneration recipe.
- [ ] No source or generated publication file is hand-edited to conceal a failed generation or validation step.

## Open Issues

- DECISION: Confirm that the compatible source release is `v0.2.1`, prepared from the current `next/v0.2.1` line, merged to `main`, and built from a detached tag commit.
- DECISION: Record the exact SFLO source tag commit SHA after release preparation.
- DECISION: Confirm audited Weave commit `ff2b0f6507e60b93664675d5893669192ee7c0ff` as the meaning of “current code,” or record a replacement commit and repeat the compatibility probe.
- DECISION: Confirm mesh base `https://semantic-flow.github.io/sflo/` and Pages publication at the branch root.
- DECISION: Confirm the payload set and mappings: core → `ontology`, config → `config`, core SHACL → `ontology/shacl`; exclude job and provenance.
- DECISION: Confirm history `releases`, state `v0.2.1`, and manifestation segment `ttl`.
- DECISION: Confirm all-terms extraction from exact release states in core, config, SHACL order with canonical references and first-source-wins attribution.
- DECISION: Confirm portable floating repository integration bindings plus external tag/commit/digest receipts. If public RDF must contain exact repository commit/digest provenance, create and complete a Weave source-binding task before this regeneration.
- DECISION: Confirm the all-governed-artifact `versioned` history policy. If “first published version” instead requires current-only generated support histories, design and validate that alternate policy before changing the recipe.
- DECISION: Freeze one `--generated-at` instant and record why that instant represents the publication event.
- DECISION: Confirm one orphan root commit plus an explicit force-with-lease update of `gh-pages`.
- DECISION: Confirm whether the old two-commit chain is preserved under the proposed annotated archive tag, preserved only in an external bundle/receipt, or intentionally left without a durable ref.
- DECISION: Confirm preservation of the existing welcome content and slashless canonical root subject.
- DECISION: Confirm preservation of `favicon.ico`, recreation of `.nojekyll`, and continued absence of `CNAME` and `assets/`.
- DECISION: Verify and confirm the GitHub Pages repository setting: source branch `gh-pages`, directory `/`, site URL `https://semantic-flow.github.io/sflo/`, no custom domain.
- DECISION: Restore a Java JDK for `riot`, or explicitly approve an equivalent independent Turtle parser gate.
- DECISION: Freeze the expected Knop and added/retired designator manifests after the source tag exists; retain 362/60/43 only if the tagged vocabulary matches the probed `d765a52` term graph.

## Decisions

### Rulings — Dave, 2026-08-01 (evening)

- **Source release: cut as `v0.3.0`, DONE.** Dave chose a minor bump over the branch's original `0.2.1` after review showed the content is breaking (dcterms→dcat predicate migration, artifact-resolution targets renamed to specs, `owl:versionInfo` off version-independent IRIs, narrowed config policy). Tagged `v0.3.0` at commit `ee3a21dfeab1c96a6442ed0679c903896535b69d`, pushed to `origin/main` with the annotated tag; `release:validate --require-tag`, `deno task ci` (27/27), and `riot --validate` over all five files all pass. **Every `$SFLO_RELEASE_VERSION='0.2.1'` / `releases/v0.2.1` in the recipe below becomes `0.3.0` / `releases/v0.3.0`.**
- **A validator fix rode along**, recorded here because it changes a release gate: `scripts/release_validate.ts` required `owl:versionInfo` on the version-independent ontology IRI, which this line deliberately removed (a version-independent resource must not claim a single version). It now derives each file's declared version from `dcat:hasVersion` and still requires `owl:versionInfo` on the release resource. Two tests that asserted the retired message were updated.
- **`favicon.ico`: preserve byte-for-byte** (RULED). It is the only at-risk host asset; `.nojekyll` stays Weave-managed, `CNAME` and `assets/` stay absent.
- **Old commits: keep the archive tag** (RULED) — annotated `archive/gh-pages-before-regeneration-2026-08-01` at `a7e7626`, preserving parent `aed218c`, before `gh-pages` is replaced by the single root commit.
- **Turtle validation: `riot` is usable** (RESOLVED, not a ruling). Java is present via sdkman; it is simply absent from non-interactive `PATH`. Every recipe step invoking `riot` must first `source "$HOME/.sdkman/bin/sdkman-init.sh"`. Verified against the core ontology.

### Prior

- Dave ruled on 2026-08-01 that the SFLO published mesh will be regenerated from scratch with current code and made to appear as the first published version.
- Regeneration is a clean replay, not an incremental mutation or another manual recreation of the existing `gh-pages` tree.
- The v0.2.0 source payload is not a valid input to current Weave because its published vocabulary is incompatible with the artifact-resolution vocabulary current Weave generates.
- The untagged `next/v0.2.1` checkout is not directly publishable because it still claims v0.2.0 release identities.
- No publication push occurs until the remaining Open Issues are moved into this section as explicit rulings.

## Contract Changes

- The public generated-source contract migrates from legacy `sflo:hasTargetArtifact` / `sflo:hasRequestedTargetState` predicates to `sflo:targetArtifact` / `sflo:targetHistoricalState`.
- The published source release advances from v0.2.0 to the ruled compatible release and must not reuse v0.2.0 release URLs for different bytes.
- Exact-state extraction makes generated extraction and canonical-reference bindings point at the selected historical source state rather than re-resolving a mutable working source.
- The `gh-pages` Git contract changes from a two-commit history to a one-root-commit publication branch.
- The mesh base, root IRI, payload designators, GitHub Pages root, and `/sflo/` payload boundary do not change.
- No Weave CLI or portable Semantic Flow code contract is expected to change unless exact repository provenance in public RDF is required.
- [[wu.cli-reference.examples.sflo]] should adopt the newer exact-state extraction pattern already demonstrated by [[wu.cli-reference.examples.urpx]].
- [[ont.dev.release-runbook]] should record the final scratch-regeneration and guarded publication sequence.

## Testing

- Run source `deno task release:validate --version 0.2.1 --require-tag`, source `deno task ci`, and independent Turtle parsing on all five tagged ontology files.
- Verify the source tag commit is on `main`, matches the recorded full SHA, and remains clean in a detached worktree.
- Run Weave mesh and publication validation after final generation.
- Parse every generated Turtle file independently.
- Compare the three tagged source payloads byte-for-byte with both local release-state outputs and the deployed Pages responses.
- Record old and new path manifests, blob manifests, Knop manifests, added designators, retired designators, counts, and a reviewed old-to-new name-status diff.
- Review broad generated changes semantically rather than expecting a small line diff.
- Confirm expected new vocabulary pages resolve and unchanged vocabulary pages continue to resolve.
- Confirm retired legacy term surfaces are absent where their removal is intentional.
- Scan source registries, inventories, and the whole output for host-local regeneration paths and legacy target predicates.
- Verify the root page, metadata panels, three release payloads, `.nojekyll`, favicon, and representative ontology/config/SHACL terms after Pages deployment.
- Verify the new branch contains exactly one root commit and that remote publication used the recorded lease.
- Run the final command sequence a second time in a second disposable environment with the same source commit and `--generated-at` value if byte-reproducibility is required; compare complete tree IDs.

## Non-Goals

- Preparing or modeling the SFLO v0.2.1 source release inside this publication task; that is a prerequisite source-release task.
- Editing the SFLO source checkout during publication regeneration.
- Publishing the job or provenance ontologies under `/sflo/`.
- Preserving the old generated mesh’s semantic drift, term census, support-history segment numbers, or HTML.
- Adding compatibility shims for legacy pre-1.0 vocabulary.
- Retrofitting v0.2.0 with changed bytes or changed vocabulary.
- Adding Weave-managed `CNAME` behavior.
- Changing the GitHub Pages URL or adding a custom domain.
- Creating publication automation or a separate branch-publish command family.
- Hand-editing generated RDF or HTML.
- Regenerating unrelated Weave fixture ladders.

## Implementation Plan

- [ ] Slice 1 — Resolve every Open Issue and move the accepted rulings into Decisions.
- [ ] Slice 2 — Complete the separate SFLO source-release prerequisite, push the source tag, and record its exact commit and SHA-256 payload receipts.
- [ ] Slice 3 — Pin the audited Weave commit, create job-local settings/log directories, create a detached source worktree, and create an empty orphan publication worktree.
- [ ] Slice 4 — Capture old path/blob/Knop/asset receipts and verify the expected old branch tip before generation.
- [ ] Slice 5 — Create the Pages mesh, install the ruled mesh-local policy, and initialize support history before adding payload candidates.
- [ ] Slice 6 — Recreate, integrate, and weave the root welcome payload; preserve only the explicitly ruled host asset.
- [ ] Slice 7 — Integrate core, config, and core SHACL using portable repository bindings; verify no host-local path or accidental pinning leakage.
- [ ] Slice 8 — Set `releases/v0.2.1`, weave the three payloads in one batch with `ttl` manifestations, and prove source/output byte identity.
- [ ] Slice 9 — Extract all terms from exact release snapshots in core, config, SHACL order with canonical references.
- [ ] Slice 10 — Weave all pending terms, explicitly regenerate pages, validate the mesh/publication, and parse all generated Turtle.
- [ ] Slice 11 — Freeze and review the Knop census, added/retired designators, vocabulary migration, generated path changes, payload topology, controls, and no-leak checks.
- [ ] Slice 12 — Create and inspect the one-root publication commit and confirm the source worktree remains unchanged.
- [ ] Slice 13 — Preserve or retire the old two-commit chain exactly as ruled.
- [ ] Slice 14 — Recheck the remote lease and replace `gh-pages` using the exact `--force-with-lease` form.
- [ ] Slice 15 — Verify GitHub Pages settings, deployed payload byte identity, representative old/new pages, root page, favicon, and absence of unruled host controls.
- [ ] Slice 16 — Update [[wu.cli-reference.examples.sflo]] and [[ont.dev.release-runbook]] with the final proven recipe and receipts.
- [ ] Slice 17 — Jimbo closes the task by renaming the task note to its completed form, updating affected wikilinks, and logging the rename in the applicable monthly `wd.maintenance.*` note.
