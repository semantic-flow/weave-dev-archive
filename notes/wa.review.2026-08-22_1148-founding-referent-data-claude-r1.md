---
id: 6b7f0d3b-78f0-4beb-8b93-66d7fde6a82f
title: Founding Referent Data Claude Review R1
desc: 'Read-only Claude Opus max-effort verification of the post-r0 FoundingReferentData task draft'
created: 1787424504000
---

## Scope

Second read-only Claude Opus max-effort review of [[wa.task.2026.2026-08-22_1112-founding-referent-data]] after the disposition of [[wa.review.2026-08-22_1129-founding-referent-data-claude]]. The review verified closure of the four r0 blockers and searched for contradictions introduced by the dispositions. No files were edited by the reviewer.

## Prior Blockers

- **B1 base/absolute-IRI semantics — closed.** The example uses absolute `D`; the task forbids `@base`, injects no parser base, and requires absolute named nodes.
- **B2 ordinary-weave preservation — closed as a requirement.** Preservation is explicit and has a create → weave round trip. One shared preservation-helper change covers `weave`, `version`, and validation planning.
- **B3 exact bytes/digest — closed.** The task names `Uint8Array`, `PlannedBinaryFile`, binary writes, and BOM/CRLF receipts; the binary model/writer already exists.
- **B4 active `ReferentMetadata` decision — closed at requirement level.** The task names the active decision and all three live guidance amendments.

## BLOCKING

None.

## MAJOR

### M1. The document profile still admits forbidden SFLO entailments

The draft allowed unrestricted `rdf:type` objects and rejected only a domain-selected subset of SFLO operational predicates. That permits assertions such as `<D> a sflo:PayloadArtifact` or `<D> sflo:hasWorkingLocatedFile <...>`, contradicting the non-byte-bearing boundary and causing union-graph SHACL targets to treat `D` as an SF support type.

**Fix:** reject every predicate in the SFLO or SFCFG namespace and reject every `rdf:type` object in those namespaces. Add negative tests for `sflo:PayloadArtifact` typing and `sflo:hasWorkingLocatedFile`.

### M2. `SemanticFlowResource` typing conflicts with deferred pages

The task typed `FoundingReferentData` as `SemanticFlowResource` while forbidding a founding page. Live guidance says a ResourcePage should accompany every `SemanticFlowResource`; the task amended the payload/dataset guidance but not this page expectation.

**Fix:** amend that guidance explicitly or drop `SemanticFlowResource` from the first-slice class hierarchy.

### M3. Create-only recovery and digest verification are jointly unrecoverable unless stated explicitly

Editing founding bytes after creation produces a permanent validation failure because there is no operation to update the digest, and validated weave paths reject findings.

**Fix:** select Warning severity, add a digest-refresh escape hatch, or state explicitly that hand editing permanently fails validation and reset-and-replay is the only first-slice repair.

### M4. Accord coverage needs a woven sibling

The r0 disposition added create-time Accord coverage but not the create → weave preservation transition that closes former blocker B2.

**Fix:** require a carried pair: founding-created and founding-created-woven. The woven manifest asserts the slot, artifact/file typing, working relation, digest, byte-identical `data.ttl`, and absence of founding page/history.

### M5. Scale prediction understates CPU growth and the CI smoke lacks a budget

The aggregate byte work is quadratic, but a nested existing-Knop/quad scan makes aggregate comparisons cubic. The 552 smoke was assigned to ordinary CI without a budget.

**Fix:** state cubic comparisons and quadratic bytes, name the hot loop, and either budget the CI smoke or make it an explicit opt-in scale task/receipt.

## ADVISORY

- Name the new public `MeshValidationFindingCode`, for example `content-digest-mismatch`.
- Prove no-network behavior by stubbing `globalThis.fetch` and running without network permission rather than adding a resolver dependency to `ExecuteKnopCreateOptions`.
- Define failure-atomic rollback precisely: remove created files and new empty directories, restore prior MeshInventory bytes, and report rollback failure distinctly. State whether the improvement applies to founding and no-founding create paths.
- Add a `knop add-reference` preservation regression because it shares the Knop-inventory preservation helper.
- Make the reset-and-replay warning explicit required content in user documentation, not merely a general documentation update.

## NIT

- Detect forbidden `@base` through lexer tokens rather than a text scan.
- Compare the parsed subject lexically with the normalized planner `D`; deliberately reject equivalent differently spelled IRIs.
- Check source/target path identity before generic existence checks so the diagnostic is deterministic.
- Name the local-path policy locator kind. Reusing the `workingLocalRelativePath` kind from integrate is available; adding a new kind would expand policy vocabulary.
- State whether digest validation follows whole-mesh or target-scoped validation selection.

## Checked Clean

- No-adoption and explicit source input are consistent.
- Root refusal is coherent with existing root create.
- Page generation is not accidentally triggered by the new path.
- A digest on the founding `LocatedFile` conforms to the live bearer contract.
- Optional-slot Warning severity and reuse of the global working-file cardinality shape match current SHACL structure.
- The requested digest-helper extraction and founding-path helper match live code opportunities.

## Verdict

**GO WITH CHANGES.**

All four r0 blockers are genuinely closed. Nothing found requires redesign. Resolve the five major contract/coverage points and the named advisories before implementation.
