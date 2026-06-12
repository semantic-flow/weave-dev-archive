---
id: jhdv2d05ksgk8udsnm1qmq8
title: 2026 05 29 0954 Fois/Semantics Demonstration Papers
desc: ''
updated: 1781217241815
created: 1780073658016
---

## Goals

- Produce two distinct near-term submissions under the FOIS/JOWO umbrella, while keeping SEMANTiCS and FOIS Ontology Showcase explicitly secondary.
- If submitting to SEMANTiCS Posters and Demos, produce a 5-page CEUR-ART paper plus demo video link by 2026-06-13, 23:59 AoE.
- If submitting to FOIS Demonstrations, produce a non-anonymous CEUR-ART short paper, 5-9 pages including bibliography, by the extended 2026-06-17 deadline.
- If submitting to FOMI, target the formal-ontology-in-practice angle by 2026-06-17, but only if the paper can say something stronger than a general tool demo.
- Treat a separate FOIS Ontology Showcase submission as optional and secondary. Only do it if SFLO has a crisp independent ontology story by 2026-06-14; otherwise avoid splitting the sprint.

## Summary

As of 2026-06-11, this is a paper production sprint with a hard human-availability stop before vacation on 2026-06-13. SEMANTiCS is the fastest deadline but should be dropped unless nearly free. FOIS Demonstrations is the clean topical fit for a Weave implementation demo, FOMI is the clean topical fit for the DigitalArtifact model, and FOIS Ontology Showcase is mostly a separate SFLO ontology paper rather than a Weave tool paper.

Recommended path after the vacation constraint: produce two coordinated but non-duplicative submission packages by 2026-06-13. The FOMI paper should be a short model/practice paper on the DigitalArtifact model and the practical ontology-publication problems it addresses. The FOIS Demonstrations paper should be a short tool/method paper on Weave's implementation workflow, generated ResourcePages, and repeatable static publication. The same SFLO public site, figures, and screenshots can be reused as shared evidence, but the thesis and contribution claims must remain distinct.

This split is stronger than forcing everything into one paper. The DigitalArtifact material has enough conceptual substance for FOMI: byte-grounded artifact identity, optional artifact layers, historical states, manifestations, located files, coordinates, provenance, major ontology revisions, and the limits of Git as an ontology-publication record. The Weave material has enough implementation substance for FOIS Demonstrations: create/integrate/weave/extract/generate/validate, source registries, path policy, static ResourcePages, and sidecar/branch publication topologies.

Provisional FOMI title: "Semantic Flow: Byte-Grounded Digital Artifact Histories for Static RDF Publication".

Provisional Weave demo title: "Weave: Filesystem-Native Semantic Mesh Publication for Dereferenceable Ontology Resources".

The combined paper pair should make a practical applied-ontology claim: Semantic Flow plus Weave lets ontology maintainers publish stable, dereferenceable identifiers and human-inspectable pages from ordinary repository files while preserving artifact history, source binding, provenance, and static-host compatibility. The FOMI paper should explain the model and lessons; the FOIS demo should show a complete workflow, not just screenshots of generated pages.

## Discussion

### Current Reorientation

Weave is on `next/v0.3.0`, clean and aligned with `origin/next/v0.3.0`. The package version in `deno.json` is `0.2.2`. The weave-dev archive repo has unrelated dirty files plus this new untracked task note; do not treat archive status as a clean slate.

The product vision is still compact: Weave provides a filesystem-oriented reference implementation of the Semantic Flow API, plus command-line and web tooling for minting dereferenceable identifiers and managing DigitalArtifacts and SemanticMeshes. The implemented surface is mostly CLI/runtime/static-page generation; daemon and web remain future-facing and should not be oversold in the paper.

Recent work since the last sustained push produced a credible demo substrate:

- ResourcePages are now a real user-facing surface, including generated panels, custom page definitions, authored Markdown regions, history/source panels, and Semantic Flow metadata.
- The Alice Bio ladder now reaches Carol, exercising integration, extraction, import, nested payloads, custom pages, and generated ResourcePages.
- The Fantasy Rules fixtures demonstrate ontology/SHACL publication, term extraction, sidecar publication, and branch-published topology.
- Artifact resolution, config-source resolution, Knop config inheritance, per-target effective config, user settings, and path-grant policy all landed enough to make the operational story more honest.
- Branch-published publication behavior is now described as ordinary mesh create, integrate, weave, extract, generate, validate, and git output handling rather than a special `prepare gh-pages` API.
- The main correctness backlog is append-onlyish inventory and publication/link cleanup, especially if we want to claim idempotent release automation in strong terms.

### Venue Fit

SEMANTiCS Posters and Demos is viable only as a compact prototype/demo submission: 5 pages including references, single-anonymous, in-person presentation, demo video expected for demos, deadline 2026-06-13. This is the highest hustle/highest logistics-pressure path.

FOIS Demonstrations is the strongest topical fit for Weave itself. The call asks for tools and methods for creating, maintaining, integrating, publishing, evaluating, and implementing ontologies. Weave can answer directly: a filesystem-native tool/method for publishing ontology resources as inspectable Semantic Flow meshes.

FOIS Ontology Showcase is a better fit for SFLO as an ontology, not for Weave as a tool. If submitted, it should describe the Semantic Flow ontology's domain, reuse, requirements, FAIR behavior, evaluation, and sustainability. That is a different paper.

FOMI is better for the DigitalArtifact model than for a pure Weave demo. Its 2026 call explicitly includes ontology governance, maintenance/update, ontology reuse/evolution/versioning/change management, ontologies with LLMs, tool support for collaborative ontology development, semantic APIs, and practical experiences with ontology-based software. That lines up with a paper about modeling digital ontology artifacts, their histories, manifestations, byte locations, and revision boundaries. A Weave demo can be included as the running case, but the paper should foreground the model and the practical lessons rather than present Weave as the main object.

### Vacation-Hard-Stop Strategy

Because the author will be unavailable after 2026-06-13, do not plan on using the official 2026-06-17 FOIS/FOMI slack. Everything intended for submission needs to be ready, uploaded, and sanity-checked before vacation.

The practical strategy is:

- draft the FOMI DigitalArtifact paper first because it carries the conceptual boundary work
- make the FOIS Demonstrations paper a parallel but narrower implementation artifact
- keep both papers short and concrete; neither should become a sprawling architecture paper
- reuse shared screenshots/figures only where they support different claims
- avoid new implementation unless a blocker appears in the demo script
- prefer slightly rough but honest submissions over scope expansion
- do not submit substantively identical papers to multiple venues if their policies prohibit work submitted elsewhere

Affiliation for EasyChair/paper metadata should use the professional/legal entity intended for publication, currently presumed to be "Spectacular Voyage LLC" unless the author decides otherwise.

### Demonstration Fit

The FOIS Demonstrations call asks for demonstration subjects, actions/processes, focused characteristics/results, and significance for applied ontology. Weave can answer those directly:

- Subject: a filesystem-native Semantic Flow mesh managed by Weave.
- Process: create/integrate/import/extract/weave/generate/validate a mesh from ontology/source files.
- Focus: dereferenceable identifiers, ResourcePages, source/provenance records, historical states, static-host publication, and fail-closed local path policy.
- Significance: ontology publication becomes reproducible, inspectable, and repository-friendly without forcing authors into a database-backed or server-side publishing stack.

### Candidate Demo Stories

Primary recommendation: SFLO or Fantasy Rules branch/sidecar publication as the main FOIS demo, with Alice Bio as a brief human-readable vignette. FOIS reviewers will probably care more about ontology publication than about biographical content, even though Alice Bio is the richer UI tour.

Candidate A: "Publish an ontology repository as a Semantic Flow mesh." Use SFLO or Fantasy Rules. Show source ontology and SHACL files, integrate them as governed payloads, weave release states, extract ontology terms into dereferenceable identifiers, generate ResourcePages, and validate no local path leakage. This best matches FOIS.

Candidate B: "Semantic mesh as an inspectable artifact ledger." Use Alice Bio. Show a small mesh evolving from data integration to extracted identifiers, imported Markdown, references, custom pages, and Carol. This is easier to demo live and makes the product legible, but it is less ontology-conference-shaped.

Candidate C: "Two publication topologies, one semantic model." Compare sidecar and branch-published Fantasy Rules. This is compelling but risks becoming too much infrastructure explanation for a short paper.

Recommended Weave-demo paper shape: lead with Candidate A, use one figure/table from Candidate B if needed to show custom page composition, and mention Candidate C as evidence that topology is operational rather than semantic.

Recommended FOMI paper shape: lead with the DigitalArtifact problem. Explain why RDF/ontology publication needs a byte-grounded layer that is neither WEMI nor "just Git." Use FRIR/WEMI as touchpoints and contrast; show the SFLO ontology metadata stanza as a compact worked example; then discuss `DigitalArtifact`, `ArtifactHistory`, `HistoricalState`, `ArtifactManifestation`, `LocatedFile`, coordinates, skip-level properties, and major ontology revisions. Weave should appear as the reference implementation that made the model testable in practice.

### Two-Paper Boundary

FOMI paper:

- Title: "Semantic Flow: Byte-Grounded Digital Artifact Histories for Static RDF Publication"
- Primary claim: ontology/RDF publication needs a byte-grounded DigitalArtifact model to make artifact state, manifestation, file location, source coordinates, and revision boundaries reviewable.
- Contribution: the Semantic Flow DigitalArtifact model and the practice lessons it encodes.
- Evidence: FRIR/WEMI contrast, SFLO ontology metadata stanza, artifact chain/skip-level figure, coordinate table, major ontology revision scenario, and Weave as reference implementation.
- Avoid: detailed command walkthrough, screenshots-as-main-evidence, or broad claims about Weave maturity.

FOIS Demonstrations paper:

- Title: "Weave: Filesystem-Native Semantic Mesh Publication for Dereferenceable Ontology Resources"
- Primary claim: Weave demonstrates a filesystem-native workflow for publishing ontology resources as static, inspectable SemanticMeshes.
- Contribution: the tool/method demonstration and repeatable publication workflow.
- Evidence: demo script, generated ResourcePages, source registry, validated public URLs, command flow, sidecar/branch topology comparison, and implementation limitations.
- Avoid: re-arguing the full DigitalArtifact ontology theory; keep only enough model vocabulary to make the demo intelligible.

Shared assets are allowed, but shared text should be minimal. Both papers can use SFLO as the running example; they should ask different questions of it.

### Proposed FOMI Argument

Problem: ontology projects need stable, dereferenceable identifiers and human-readable documentation, but publication workflows often blur source files, generated pages, histories, provenance, host-specific deployment details, and major revision boundaries.

Approach: Semantic Flow models governed digital artifacts, artifact histories, historical states, manifestations, located files, artifact-resolution coordinates, skip-level shortcuts, and generated pages as explicit RDF-visible publication structure.

Contribution: a byte-grounded DigitalArtifact model for repository-native ontology publication, plus practice lessons about what the model intentionally leaves to domain ontologies.

Limitations: Semantic Flow is not WEMI, does not solve semantic ontology diffing or migration, and does not replace richer domain-level modeling.

### Proposed FOIS Demonstrations Argument

Problem: ontology projects need stable, dereferenceable identifiers and human-readable documentation, but publication workflows often blur source files, generated pages, histories, provenance, and host-specific deployment details.

Approach: Semantic Flow models identifiers, Knops, DigitalArtifacts, ArtifactHistories, source bindings, and ResourcePages. Weave implements those models against ordinary files and static hosting.

Demo: starting from ontology/source files, Weave registers sources, versions payloads, extracts terms, records provenance, and generates ResourcePages that expose RDF facts, histories, source links, and generated metadata.

Contribution: a reproducible tool workflow and reference implementation for publishing ontology resources as inspectable SemanticMeshes.

Limitations: v0.x implementation; daemon/web surfaces are not the current demo; append-onlyish inventory and remote current-byte resolution remain active work; no claim of complete ontology evaluation environment.

## Open Issues

- Which main demo mesh should carry the FOIS demo paper: SFLO, Fantasy Rules, or Alice Bio?
- Which worked example should carry the FOMI model paper: SFLO ontology metadata stanza, a major version revision scenario, or both?
- Does EasyChair permit both a FOIS Demonstrations submission and a FOMI/JOWO workshop submission by the same author on distinct contributions? The working assumption is yes, but the submissions must not be duplicate papers.
- Are we willing and able to support in-person presentation for SEMANTiCS in Ghent, FOIS/FOMI in Vitória, or both?
- Who are the authors, and who can register/present if accepted?
- What exact author affiliation should appear in EasyChair and the paper metadata? Current guess: Spectacular Voyage LLC.
- What presentation preference should be declared for FOIS: virtual, onsite, or flexible?
- For SEMANTiCS, can we produce a demo video link by 2026-06-13?
- How public/stable are the exact demo repositories, generated pages, and command sequences we want reviewers to inspect?
- How much GenAI disclosure is required under CEUR/SEMANTiCS policy for LLM-assisted drafting or code/documentation support?
- Can we get one clean repeatable demo script without racing unfinished append-only inventory/link-policy work?

## Decisions

- Target both near-term papers: FOMI 2026 short paper on the DigitalArtifact model, and FOIS 2026 Demonstrations short paper on Weave as the implementation/demo.
- Recommended length for each: 5-9 pages including bibliography.
- Submit both FOIS/FOMI targets before 2026-06-13 rather than relying on the official 2026-06-17 deadline.
- Recommended SEMANTiCS posture: submit only if a 5-page draft plus video is realistic by 2026-06-13; otherwise let it go.
- Recommended FOMI posture: model/practice paper foregrounding digital artifacts, history/state, publication coordinates, and ontology revision boundaries.
- Recommended FOIS Demonstrations posture: implementation/demo paper foregrounding the Weave command workflow, generated ResourcePages, static hosting, source registries, and validation.
- Recommended scope: no SEMANTiCS and no Ontology Showcase unless both target papers are already submit-ready.
- Draft in Markdown or directly in LaTeX, then assemble the final PDFs in CEUR-ART LaTeX first. Use Overleaf or install a local TeX toolchain if needed. LibreOffice/ODT is now only the fallback path.
- Do not claim daemon or browser app maturity. The current credible artifact is CLI/runtime plus generated static ResourcePages.
- Do not promise network fetching or broad remote current-byte resolution. The current story is governed local/repository source resolution and explicit import.
- Do not present append-onlyish inventory idempotence as completed until [[wa.task.2026.2026-05-17-append-onlyish-inventory]] lands.

## Contract Changes

- No code or ontology contract changes are required just to submit the paper.
- If the paper claims a durable behavior not currently covered by docs/tests, either narrow the claim or create a follow-up task rather than forcing new implementation into the paper sprint.

## Testing

- Smoke-test the chosen demo script from a clean checkout or clean publication worktree.
- Run `deno task lint` and at least the focused tests that cover the demo path if any last-minute code/doc examples change.
- Validate generated demo pages locally with `live-server` or static file browsing.
- Check generated public files for host-local path leakage before using screenshots or URLs.
- Verify all public URLs and repository links in the paper resolve from outside the local machine.
- Confirm CEUR-ART PDF compiles and stays within page limits before submission day.
- Current local authoring-tool status on 2026-06-11: LibreOffice is installed, but no local LaTeX toolchain is on PATH (`latexmk`, `pdflatex`, `lualatex`, `xelatex`, `tectonic`, and `pandoc` were not found).

## Non-Goals

- Do not build major new Weave features for the paper unless they are already in the current backlog and small enough to land safely.
- Do not submit a generic Semantic Flow theory paper to a demonstrations track.
- Do not use the paper to re-open daemon/web scope.
- Do not split attention beyond the two target papers unless both are already submit-ready.

## Implementation Plan

- [ ] 2026-06-11: make go/no-go call for SEMANTiCS Posters and Demos.
- [ ] 2026-06-11: make a hard target decision; revised recommendation is both FOMI DigitalArtifact and FOIS Demonstrations Weave.
- [ ] 2026-06-11: create separate CEUR-ART paper workspaces and draft skeletons.
- [ ] 2026-06-11: confirm EasyChair author metadata, including Spectacular Voyage LLC affiliation if appropriate.
- [ ] 2026-06-12: if SEMANTiCS is go, draft a 5-page version and record or assemble a short demo video.
- [ ] 2026-06-12: make go/no-go call for two-paper submission.
- [ ] 2026-06-12: freeze the FOMI model story and FOIS demo story.
- [ ] 2026-06-12: confirm authors, presentation mode, and registration feasibility.
- [ ] 2026-06-13: draft one-page outline for each paper with title, abstract, figures, and limitations.
- [ ] 2026-06-13: choose public artifact URLs and verify they resolve.
- [ ] 2026-06-14: produce first complete 5-9 page FOMI and FOIS CEUR-ART drafts.
- [ ] 2026-06-14: decide whether any additional companion submission is realistic; default no.
- [ ] 2026-06-15: run the demo script and capture shared screenshots/figures.
- [ ] 2026-06-15: tighten claims against actual v0.2.2/`next/v0.3.0` behavior.
- [ ] 2026-06-16: final edit, references, FAIR/public artifact check, CEUR/GenAI compliance check for both papers.
- [ ] 2026-06-17: submit PDFs to EasyChair for the chosen FOIS Demonstrations and FOMI/JOWO tracks.
