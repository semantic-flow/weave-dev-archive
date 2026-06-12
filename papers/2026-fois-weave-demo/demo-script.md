# Demo Script

Primary demo: SFLO publication mesh.

Secondary topology comparison: Fantasy Rules sidecar and branch-published meshes.

Optional local vignette: Alice Bio for custom pages and imported Markdown.

## Reviewer-Clickable Path

Use these URLs in the paper if they remain stable:

- SFLO publication mesh: https://semantic-flow.github.io/sflo/
- SFLO ontology page: https://semantic-flow.github.io/sflo/ontology
- SFLO config ontology page: https://semantic-flow.github.io/sflo/config
- SFLO extracted term page example: https://semantic-flow.github.io/sflo/ontology/SemanticMesh/
- Fantasy Rules sidecar mesh: https://semantic-flow.github.io/mesh-sidecar-fantasy-rules/
- Fantasy Rules branch-published mesh: https://semantic-flow.github.io/mesh-branch-fantasy-rules/
- Weave repository: https://github.com/semantic-flow/weave
- SFLO repository: https://github.com/semantic-flow/sflo

## Live Walkthrough

1. Start on https://semantic-flow.github.io/sflo/.

Show that the root page is a generated ResourcePage with a human-readable title, summary, child identifiers, RDF properties, raw source, and Semantic Flow metadata.

2. Open https://semantic-flow.github.io/sflo/ontology.

Show the ontology artifact page. Point out that this is not merely a hand-written documentation page: it is generated from a governed payload artifact and linked to artifact history/source metadata.

3. Open an extracted ontology term page: https://semantic-flow.github.io/sflo/ontology/SemanticMesh/.

Show that an ontology term receives a dereferenceable generated page rather than being only a fragment inside the source ontology document.

4. Show local source repository layout.

Use the SFLO checkout:

```sh
tree -L 2 dependencies/github.com/semantic-flow/sflo
```

If `tree` is unavailable:

```sh
find dependencies/github.com/semantic-flow/sflo -maxdepth 2 -type f | sort | sed -n '1,80p'
```

5. Show Weave's publication command family, not as a full replay.

Use a few representative commands from `documentation/notes/wu.cli-reference.examples.sflo.md`:

```sh
weave mesh create --workspace "$SFLO_PUB" --mesh-base 'https://semantic-flow.github.io/sflo/' --publication-profile github-pages
weave integrate "$SFLO_SRC/semantic-flow-core-ontology.ttl" ontology --mesh-root "$SFLO_PUB" --grant-source-directory "$SFLO_SRC" --source-repository-current --source-repository-url 'https://github.com/semantic-flow/sflo.git'
weave set history ontology releases --mesh-root "$SFLO_PUB"
weave set next-state ontology "$SFLO_RELEASE_STATE" --mesh-root "$SFLO_PUB"
weave --mesh-root "$SFLO_PUB" --payload-manifestation-segment ttl --target 'designatorPath=ontology'
weave extract --all-terms --mesh-root "$SFLO_PUB" --source ontology --add-source-references --reference-role canonical --accept-preview
weave generate --mesh-root "$SFLO_PUB"
weave validate publication --mesh-root "$SFLO_PUB"
```

The exact command names should be checked before recording or submitting; the paper can use abbreviated pseudocode if the full local replay is too noisy.

6. Show source registry or metadata.

Use a source registry file such as:

```sh
sed -n '1,180p' /tmp/sflo/ontology/_knop/_sources/sources.ttl
```

If using the checked-in dependency or fixture instead of `/tmp/sflo`, adjust the path.

Point out repository/source facts and absence of host-local path leakage.

7. Show topology comparison.

Open:

- https://semantic-flow.github.io/mesh-sidecar-fantasy-rules/
- https://semantic-flow.github.io/mesh-branch-fantasy-rules/

The point is that sidecar and branch-published meshes use the same Semantic Flow artifact model even though their repository/publication topology differs.

8. Optional Alice Bio vignette.

Use local fixture pages under:

```sh
dependencies/github.com/semantic-flow/mesh-alice-bio/index.html
dependencies/github.com/semantic-flow/mesh-alice-bio/alice/index.html
dependencies/github.com/semantic-flow/mesh-alice-bio/carol/index.html
```

This is for showing custom page definitions, imported Markdown, authored regions, and generated panels in a more human-readable setting.

## Figure Ideas

- Figure 1: source repository to SemanticMesh diagram.
- Figure 2: ResourcePage screenshot with callouts for RDF facts, history, source, and metadata panels.
- Figure 3: small table comparing sidecar, branch-published, and whole-repository topology.
- Figure 4: minimal command flow: create, integrate, weave, extract, generate, validate.

## Claims To Keep Honest

- Say "pre-1.0 reference implementation", not mature platform.
- Say "CLI/runtime and generated static pages", not completed daemon/browser app.
- Say "governed local/repository source resolution", not arbitrary remote fetching.
- Say "supports reproducible publication workflows", not "fully solved release automation idempotence" until append-onlyish inventory lands.
