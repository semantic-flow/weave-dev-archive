---
id: kgswc-2026-paper-target-20260630
title: 2026 06 30 1159 KGSWC 2026 Paper Target
desc: ''
created: 1782845940000
---

## Goals

- Treat KGSWC 2026 as the next plausible paper-submission target for Semantic Flow / Weave.
- Use the missed FOIS/FOMI sprint drafts as concept scratch, not as a frozen submission.
- Let ongoing Semantic Flow and Stagecraft engineering clarify the paper thesis before drafting hardens.

## Summary

KGSWC 2026 is now the next target venue to track. The official important dates list paper submission on 2026-07-10, notification on 2026-08-10, camera-ready on 2026-08-20, and conference dates on 2026-11-16 through 2026-11-18. The call asks for unpublished work, double-blind review, Springer LNCS formatting, full papers of 12-15 pages, and short papers of 6-8 pages.

This is a better future target than trying to resurrect the missed FOIS/FOMI deadline immediately. Semantic Flow is still in flux, and Stagecraft may give the paper a stronger applied story: Semantic Flow as a persisted roleplaying-data substrate, with Weave as the filesystem-native implementation and static/inspectable publication path.

## Discussion

KGSWC's topic list includes several good matches:

- Linked Data
- Vocabularies, Schemas, Ontologies
- Services, APIs, Processes and Cloud Computing
- Scientific Semantic Web
- Knowledge Graph into LLMs

The strongest paper shape is probably a single integrated paper rather than the earlier two-paper FOIS/FOMI split:

- Semantic Flow model: byte-grounded `DigitalArtifact`, histories, states, manifestations, located files, resolution coordinates, and ResourcePages
- Weave implementation: filesystem-native CLI/runtime, static pages, source provenance, and repository/application mesh topologies
- Stagecraft pressure: persisted roleplaying data as an application-driven use case, if enough concrete work lands before drafting

Do not force a paper around unstable vocabulary. The right stance is to let append-onlyish inventory, history/current policy, ResourcePage inspection, and Stagecraft persistence requirements sharpen the claims.

## Open Issues

- Should the submission be a full paper or short paper?
- What is the strongest running example by early July: SFLO publication, Fantasy Rules, Stagecraft persisted data, or a combined example?
- How much Stagecraft can be shown without making the paper about a private application rather than the reusable Semantic Flow framework?
- Does the KGSWC submission system require any additional abstract-registration date not visible on the important-dates page?
- What exact LNCS template/setup should be used locally or in Overleaf?

## Decisions

- KGSWC 2026 is the next paper target to track.
- The paper should not drive near-term engineering unless a paper claim exposes a real product/documentation gap.
- Existing FOIS/FOMI drafts remain reusable scratch, especially the DigitalArtifact and Semantic Mesh material.

## Contract Changes

- None.

## Testing

- No code tests required for this planning task.
- Before submission, verify public examples and any demo commands against the current Weave version and generated pages.

## Non-Goals

- Do not resume paper drafting at the expense of append-onlyish inventory or Stagecraft-driven persistence work.
- Do not freeze Semantic Flow vocabulary before the ontology/framework stabilizes.
- Do not submit substantially recycled FOIS/FOMI draft text without reworking the venue fit and current implementation claims.

## Implementation Plan

- [ ] Confirm whether KGSWC has a separate abstract-registration deadline or only the 2026-07-10 paper deadline.
- [ ] Decide full paper versus short paper.
- [ ] Reassess the running example after the next engineering slice.
- [ ] Harvest stable material from the FOIS/FOMI drafts into a new KGSWC outline.
- [ ] Verify LNCS formatting and submission requirements.
