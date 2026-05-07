---
id: exterminate-sfc-5iefva
title: 'exterminate sfc'
desc: ''
created: 1778012457885
updated: 1778012457885
participants: [djradon, codex.gpt-5.5]
conversationEventKinds: [message.assistant, message.user, tool.call, tool.result]
---

# djradon_2026-05-04_2218_44

exterminate sfc

I just ran a weave on a different ontology, and one page had "sfc:ArtifactResolutionMode/Pinned" -- it should be "sflo:ArtifactResolutionMode/Pinned" - can you find all references to "sfc:" and if they obviously should be "sflo:" please fix them. Bonus points if you can figure out the source of this confusion.

# gpt-5.5_2026-05-04_2218_50

I’ll inspect the repo guidance first, then search for `sfc:` across code, ontology, docs, and any generated examples so we can distinguish real config-ontology terms from mistaken core-ontology terms.

The repo is currently using `sfc:` as an alias for the core ontology in several code paths and tests. That is likely the direct source: generated HTML abbreviates core IRIs with the configured prefix map, so `https://semantic-flow.github.io/ontology/core/ArtifactResolutionMode/Pinned` becomes `sfc:ArtifactResolutionMode/Pinned` wherever core was registered as `sfc`.

There are two related namespace issues in play. The modern core ontology advertises prefix `sflo` for `https://semantic-flow.github.io/ontology/core/`, while Weave still has many older `sflo` references pointed at `https://semantic-flow.github.io/semantic-flow-ontology/`; I’m keeping this pass scoped to the obvious `sfc:` alias misuse and the renderer that produced the bad page label.

I’m going to make the page renderer treat the core ontology namespace canonically as `sflo`, even when a source Turtle file declares it as `sfc`. That directly fixes the visible `sfc:ArtifactResolutionMode/...` leak without pretending the older `semantic-flow-ontology` namespace migration is finished.

The renderer and its assertions are updated. I’m running the focused page tests now, then I’ll run the linter as requested for code changes.

The direct `deno test` invocation needs repo permissions because a dependency checks `READABLE_STREAM`. I’m switching to the project’s test task so the same flags used by the repo are applied.

The full test task passed because the project task runs the whole suite when given an extra file argument. I’m running lint now and then I’ll give you the remaining `sfc:` inventory/source-code picture instead of burying the namespace debt.

Implemented the visible fix for generated pages.

Changed:
- [src/runtime/weave/pages.ts](../../../../../src/runtime/weave/pages.ts): core ontology IRIs now compact to `sflo:*`, even if source Turtle declared `@prefix sfc: <https://semantic-flow.github.io/ontology/core/>`.
- [src/runtime/weave/pages_test.ts](../../../../../src/runtime/weave/pages_test.ts): updated coverage so extraction source modes render as `sflo:ArtifactResolutionMode/...`.

Verification:
- `deno task test src/runtime/weave/pages_test.ts` passed, and because of the task wiring it ran the full suite: `278 passed`.
- `deno task lint` passed.

Source of the confusion: the modern core ontology says `https://semantic-flow.github.io/ontology/core/` has preferred prefix `sflo`, but Weave still has older code/data using `sflo` for `https://semantic-flow.github.io/semantic-flow-ontology/`, plus `sfc` for the newer core namespace. A blind `sfc:` to `sflo:` rewrite would create invalid/ambiguous Turtle in files that already use `sflo` for the legacy namespace, so I fixed the page renderer’s canonical display behavior rather than papering over the bigger namespace migration.

Suggested commit message:

```text
fix: render core ontology CURIEs with sflo prefix

- canonicalize page-rendered RDF prefix labels for the modern core ontology namespace
- preserve sfc-declared source Turtle compatibility while displaying core IRIs as sflo
- update extraction source page tests for ArtifactResolutionMode display
```

# djradon_2026-05-04_2222_52

what "source Turtle declared @prefix sfc:"?

# gpt-5.5_2026-05-04_2223_01

I meant the RDF source panel content that the page renderer parses, not your ontology file.

Concrete example from the repo: [pages_test.ts](../../../../../src/runtime/weave/pages_test.ts:336) has this fixture:

```turtle
@prefix sfc: <https://semantic-flow.github.io/ontology/core/> .

<bob/_knop/_inventory#extraction-source> a sfc:ExtractionSource ;
  sfc:hasTargetArtifact <alice/bio> ;
  sfc:hasRequestedTargetState <alice/bio/_history001/_s0002> ;
  sfc:hasArtifactResolutionMode <https://semantic-flow.github.io/ontology/core/ArtifactResolutionMode/Pinned> .
```

The renderer’s `collectPrefixMap()` was reading that `@prefix sfc:` from raw Turtle panels and then using it to compact full core IRIs. So even when the object was written as a full IRI:

```turtle
<https://semantic-flow.github.io/ontology/core/ArtifactResolutionMode/Pinned>
```

the generated HTML displayed it as:

```text
sfc:ArtifactResolutionMode/Pinned
```

That was the leak. The fix makes the renderer ignore `sfc` as the display prefix for the modern core namespace and display that namespace as `sflo` instead.

# djradon_2026-05-05_0848_22

We need to finalize the sflo ontology and SHACL URLs, which means deciding whether to do a full-repo mesh or a sidecar mesh, and finalizing the repo name (currently "ontology"). Also considering removing "core" from the filename.

# gpt-5.5_2026-05-05_0848_29

I’ll ground this in the current repo layout and ontology metadata before giving a recommendation, because the URL decision is mostly about what future publishing and mesh boundaries should make boring.

I don’t see a live sibling `/home/djradon/hub/semantic-flow/ontology`; this Weave checkout has the ontology under `dependencies/github.com/semantic-flow/ontology`, and there’s an `ontology_DONT-USE` sibling. I’m checking the dependency repo metadata and current ontology filenames/URLs so the recommendation doesn’t assume the wrong working copy.

My recommendation:

Use a **full-repo mesh** for `semantic-flow/ontology`, keep the repo name **`ontology`**, and make the core ontology URL **not include `core`**.

Concretely:

```text
Core ontology namespace / ontology IRI:
https://semantic-flow.github.io/ontology/

Core ontology document:
https://semantic-flow.github.io/ontology/ontology.ttl
or
https://semantic-flow.github.io/ontology/sflo.ttl

SHACL shapes graph IRI:
https://semantic-flow.github.io/ontology/shacl/

SHACL document:
https://semantic-flow.github.io/ontology/shacl/shapes.ttl
```

I would avoid:

```text
https://semantic-flow.github.io/ontology/core/
```

unless you genuinely expect multiple peer ontologies where “core” is one module among several. Right now `sflo` is the main vocabulary. Making users write or see `sflo = .../ontology/core/` is already causing confusion.

Full-repo mesh is the better fit because the ontology repo’s whole purpose is publication. A sidecar mesh is useful when RDF publishing is attached to some other primary project, like docs for an app or a domain repo. Here the mesh should own the repo: ontology docs, SHACL, releases, generated pages, and term pages are the product.

Keep the repo name `ontology`. It gives you the clean GitHub Pages project path:

```text
https://semantic-flow.github.io/ontology/
```

Renaming the repo to `sflo` would make the canonical URL less descriptive and would couple the repo path to one prefix. `ontology` also leaves room for `config`, `prov`, `job`, and SHACL without making `sflo` sound like the container for everything.

On filenames: yes, remove `core` from the filename if we also remove it from the URL. I’d use `ontology.ttl` if this repo is the Semantic Flow ontology publication, or `sflo.ttl` if you want the prefix to be visually obvious. I slightly prefer `ontology.ttl`; it reads better in a repo already named `ontology`.

One pushback: don’t finalize URLs while preserving the old split where `sflo` sometimes means `https://semantic-flow.github.io/semantic-flow-ontology/` and sometimes means `https://semantic-flow.github.io/ontology/core/`. That will keep leaking into generated pages, SHACL, examples, and tests. The real cleanup is to pick one canonical `sflo` namespace and migrate hard before v1.

# djradon_2026-05-05_0856_07

Right now we do have peer ontologies: config, job, and prov, so if https://semantic-flow.github.io/ontology/ represents the ontology, the other ontology IRIs become inconsistent, https://semantic-flow.github.io/ontology/config etc. 

What about https://semantic-flow.github.io/ontologies/sflo, https://semantic-flow.github.io/ontologies/config, https://semantic-flow.github.io/ontologies/job, https://semantic-flow.github.io/ontologies/prov.

# gpt-5.5_2026-05-05_0856_24

Yes, `https://semantic-flow.github.io/ontologies/...` is cleaner if you are willing to make the repo/page path match it.

I’d revise my recommendation:

```text
https://semantic-flow.github.io/ontologies/sflo/
https://semantic-flow.github.io/ontologies/config/
https://semantic-flow.github.io/ontologies/job/
https://semantic-flow.github.io/ontologies/prov/
```

That treats `sflo`, `config`, `job`, and `prov` as peers. It also avoids the awkward implication that `/ontology/` is both the repo/site root and one ontology’s namespace.

The practical catch: GitHub Pages project URLs follow the repo name. A repo named `ontology` naturally publishes at:

```text
https://semantic-flow.github.io/ontology/
```

To get this exact clean URL without routing tricks, I’d rename the repo to:

```text
ontologies
```

Then use a full-repo mesh rooted at:

```text
https://semantic-flow.github.io/ontologies/
```

For files, I’d align them with ontology names, not “semantic-flow-core”:

```text
sflo.ttl
config.ttl
job.ttl
prov.ttl
sflo-shacl.ttl
```

For SHACL, I would not put it at `/ontologies/sflo/` as if it were the ontology namespace itself. I’d use either:

```text
https://semantic-flow.github.io/ontologies/sflo-shacl/
```

or, if you want it clearly subordinate to the ontology it validates:

```text
https://semantic-flow.github.io/ontologies/sflo/shacl/
```

I slightly prefer `https://semantic-flow.github.io/ontologies/sflo-shacl/` for the shapes graph IRI, because it is a distinct RDF graph with its own identity, not a term inside the `sflo` vocabulary namespace.

So my current preferred set is:

```text
sflo ontology:   https://semantic-flow.github.io/ontologies/sflo/
config ontology: https://semantic-flow.github.io/ontologies/config/
job ontology:    https://semantic-flow.github.io/ontologies/job/
prov ontology:   https://semantic-flow.github.io/ontologies/prov/
sflo SHACL:      https://semantic-flow.github.io/ontologies/sflo-shacl/
```

This does mean `core` should disappear from the public name unless you actually want users to say “the core ontology” forever. Internally, “core” was useful while sorting modules; externally, `sflo` is the thing.

# djradon_2026-05-05_0859_01

I also like https://semantic-flow.github.io/sflo/ as just a "SemanticSite (still to be defined), and then https://semantic-flow.github.io/sflo/ontology, https://semantic-flow.github.io/sflo/ontology/shacl, https://semantic-flow.github.io/sflo/config, https://semantic-flow.github.io/sflo/config/shacl, etc.

# gpt-5.5_2026-05-05_0859_16

I like that better than `/ontologies/...` if the goal is a coherent Semantic Flow publication site, not just an ontology registry.

Preferred shape:

```text
SemanticSite root:
https://semantic-flow.github.io/sflo/

Core ontology:
https://semantic-flow.github.io/sflo/ontology/

Core SHACL:
https://semantic-flow.github.io/sflo/ontology/shacl/

Config ontology:
https://semantic-flow.github.io/sflo/config/

Config SHACL:
https://semantic-flow.github.io/sflo/config/shacl/

Job ontology:
https://semantic-flow.github.io/sflo/job/

Prov ontology:
https://semantic-flow.github.io/sflo/prov/
```

That gives you a stable product/site root, and lets each vocabulary module live as a first-class section. It also solves the inconsistency problem: `ontology`, `config`, `job`, and `prov` are not all pretending to be children of an `ontology` namespace.

The main pushback: if you do this, the repo probably should not remain named `ontology`. For GitHub Pages, `https://semantic-flow.github.io/sflo/` naturally wants a repo named `sflo` unless you publish through the org site repo or use redirects. A repo named `ontology` publishing a site whose canonical root is `/sflo/` will keep creating mental and operational friction.

I’d make `https://semantic-flow.github.io/sflo/ontology/` the canonical `sflo:` namespace. It is slightly verbose, but good verbose: `sflo` is the project/site, `ontology` is the core vocabulary module. Then config can import:

```turtle
@prefix sflo: <https://semantic-flow.github.io/sflo/ontology/> .
@base <https://semantic-flow.github.io/sflo/config/> .
```

For files, I’d still drop `core`:

```text
ontology.ttl
ontology-shacl.jsonld or shacl.jsonld
config.ttl
config-shacl.jsonld or config/shacl.jsonld
job.ttl
prov.ttl
```

Net: yes, this is probably the cleanest direction if `SemanticSite` is real. Full-repo mesh rooted at `/sflo/`, repo renamed to `sflo`, with ontology/config/job/prov as site sections.
