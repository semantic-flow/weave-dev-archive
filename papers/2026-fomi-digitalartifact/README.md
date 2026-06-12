# FOMI 2026 DigitalArtifact Paper

This folder holds sprint material for a FOMI 2026 short paper on the Semantic Flow DigitalArtifact model.

Primary target: FOMI 2026, as a JOWO workshop co-located with FOIS 2026.

Paired target: the separate Weave implementation/demo paper lives in `../2026-fois-weave-demo/`.

Hard internal deadline: 2026-06-13, before vacation.

Official JOWO/FOMI deadline: 2026-06-17.

Working title: Semantic Flow: Byte-Grounded Digital Artifact Histories for Static RDF Publication

Current stance:

- main planning/draft artifact: `outline.md`
- primary contribution: the Semantic Flow DigitalArtifact model as a practical ontology-publication layer
- running example: the opening metadata stanza of `semantic-flow-core-ontology.ttl` and the generated SFLO publication surface
- reference implementation: Weave, used as evidence that the model is operational
- shared reference pool: `../2026-fois-weave-demo/potential-references.md`
- contribution boundary: this paper is about the model and practice lessons; the FOIS Demonstrations paper is about the Weave workflow and demo

Do not submit substantially identical papers to multiple venues if venue policies prohibit simultaneous submissions. The intended split is genuinely distinct: FOMI gets the Semantic Flow DigitalArtifact model/practice argument; FOIS Demonstrations gets the Weave workflow and static publication demo.

## Production Notes

- Draft in Markdown or directly in LaTeX, depending on speed.
- Assemble final PDF in CEUR-ART LaTeX 1-column style.
- Use Overleaf or install a local TeX toolchain; current local status on 2026-06-11 is that LibreOffice is installed, but no LaTeX compiler or `latexmk` is on PATH.
- Treat LibreOffice/ODT as fallback only, not the primary FOMI path.
- Include the mandatory declaration on generative AI use.
- Reuse figures/screenshots from the Weave demo only when they support the DigitalArtifact model argument rather than the tool walkthrough.
