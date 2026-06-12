# Weave FOIS 2026 Demonstration Paper

This folder holds sprint material for the FOIS 2026 Demonstrations submission.

Primary target: FOIS 2026 Demonstrations.

Paired target: a separate FOMI 2026 paper on the Semantic Flow DigitalArtifact model lives in `../2026-fomi-digitalartifact/`.

Hard internal deadline: 2026-06-13, before vacation.

Official FOIS deadline: 2026-06-17.

Working title: Weave: Filesystem-Native Semantic Mesh Publication for Dereferenceable Ontology Resources

Current stance:

- main planning/draft artifact: `paper-planning-codex.md`
- primary demo: ontology repository publication as a Semantic Flow mesh
- preferred concrete demo source: SFLO or Fantasy Rules
- secondary vignette: Alice Bio ResourcePages/custom page composition if space permits
- contribution boundary: this paper is about Weave as implementation and demonstration; DigitalArtifact theory belongs in the FOMI paper
- production format: draft in Markdown, assemble final PDF in CEUR-ART 1-column style


Do not submit substantially identical papers to multiple venues if venue policies prohibit simultaneous submissions. The intended split is genuinely distinct: FOMI gets the Semantic Flow DigitalArtifact model/practice argument; FOIS Demonstrations gets the Weave workflow and static publication demo.

## Format Decision

CEUR strongly recommends LaTeX CEURART, and FOMI specifically points authors at the CEUR-Art LaTeX template in one-column mode. CEUR also supplies a LibreOffice/ODT 1-column CEUR-ART template, but that should now be treated as the fallback path rather than the primary authoring route.

Working plan:

- keep content in Markdown while drafting quickly, or move directly into `.tex` once structure stabilizes
- assemble final submissions in CEUR-ART LaTeX 1-column style
- use Overleaf or install a local TeX toolchain for compilation
- use LibreOffice/ODT only if LaTeX becomes a blocker
- use the latest 1-column CEUR template
- include the mandatory declaration on generative AI use

Local status on 2026-06-11: LibreOffice is installed at `/usr/bin/libreoffice`; no local LaTeX compiler or `latexmk` is currently on PATH. Libertinus fonts are not currently installed, and `fc-match` falls back to Noto Sans.
