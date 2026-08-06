---
id: lw7a7nzv37gd7o405loyfa6
title: Dave Court
desc: 'Open decision cards for Dave — ungated; position is the status, swept at ruling time'
updated: 1785606064695
created: 1785606064695
---

This note is Dave's court: open decision cards only, one card per decision, each with a stated lean. Position is the status — a card's presence means the decision is open, and it is swept at ruling time, with the ruling recorded where it belongs (the owning task note and/or [[wd.decision-log]]). A ruling that spawns work becomes a task note and enters [[wd.queues]]. This note is deliberately not gate-enforced: its writers are Jimbo (cards) and Dave (rulings), and drift here is visible to its only reader. A task waiting on Dave may hold a [[wd.queues]] entry AND a card here — the two surfaces are allowed to name the same task.

## Fingerprint verification — build, defer, or cancel?

**Lean: BUILD-LATER**, per the 2026-08-02 codex analysis of [[wa.task.2026.2026-05-04-fingerprint-verification]].

The gap is real: Weave cannot detect a historical payload edited after versioning, and cannot let a consumer prove fetched bytes match what was versioned. `renderFirstPayloadWovenKnopInventoryTurtle` writes a LocatedFile with type declarations and **no digest**, even though `sflo:hasContentDigest` exists in the ontology and the resolver already computes SHA-256 on every read. `validate` is explicitly planner/preflight coverage, not integrity coverage.

But no consumer has asked. Stagecraft's "releases are not verifiable" complaint was about identifying the *Weave executable's* build commit, not mesh payloads. And both real consumers currently hold **stronger** anchors than a self-recorded digest would be — Git commits and tags, tagged source files, Accord fixtures. The SFLO publication did exactly this check by hand on 2026-08-01 with `cmp` against tagged source, which is more trustworthy than a mesh-local digest, since an attacker who can edit a payload can edit its digest too.

**Revival triggers to rule on:** (a) the SFLO runbook wants a recurring integrity gate instead of manual byte comparison, or (b) Stagecraft commits to running historical-payload integrity in its CI.

**A second ruling is required before any implementation** — this one is a genuine contract conflict, not a preference. `--overwrite-existing-state` deliberately rewrites a current state in place. So either:

- fingerprinted states become **non-overwritable** (corrections mint a new state), and the feature can honestly claim historical immutability; or
- overwrite updates the digest too, and the docs must describe the guarantee as **current byte-consistency**, not proof a state never changed.

Shipping without ruling this would make the feature mean something we never decided.

If BUILD-LATER is accepted, the analysis proposes `weave validate mesh --integrity payload-history` rather than a new top-level verb — integrity belongs in the surface the real consumer already runs. Full proposed scope, findings taxonomy, and ten acceptance tests are in the analysis; they go into the task note when this revives.

## Embedded RDFa / JSON-LD — park, or rule the identity graph normative?

**Lean: PARK**, per the 2026-08-02 codex design analysis of [[wa.task.2026.2026-06-12-rdfa-and-jsonld-support]]. But there is a cheap first slice if you want it, and one of the two revival triggers is *your* ruling, not a consumer's.

**Serialization, if it ever ships: JSON-LD** in one `<script type="application/ld+json">` block. RDFa loses because it couples graph correctness to presentation markup — restyling or hiding a panel would silently remove triples, and the custom-page renderer bypasses the shared document model entirely. Microdata is dismissed: no credible path for RDF datatypes, language tags, or shared blank nodes.

**Why park.** No consumer has asked. Stagecraft's exercised use is validation and packaging, not page consumption, and its own requirements note explicitly says not to prioritize embedding absent direct need. Search engines are a weak justification — Google parses all three formats, but rich results are limited to supported feature types, and custom SFLO/RDFS assertions are not one. The strongest hypothetical consumer is a generic RDF-aware crawler; none is named or tested.

**The deeper reason: the page model is lossy.** Before rendering, Weave already drops literal datatypes and language tags, incoming triples, blank-node identity, inventory predicates and ordinals, and digest/manifestation evidence. A faithful graph cannot be reconstructed from the rendered panels — it would need a new parsed-dataset seam at `sourcePanelsForFacts`. Embedding a lossy projection and calling it the graph would be worse than embedding nothing.

**The cheap slice, if you want it now:** the three-triple page-identity graph only — `<resource> sflo:hasResourcePage <page/index.html>`, and the page typed as `sflo:ResourcePage, sflo:LocatedFile`. That has a clean seam (canonical, pagePath, and meshBase are all known before rendering), needs no RDF traversal, no new dependency, and costs ~547 bytes/page — about 0.77 MiB across SFLO's 1,467 pages. Compare the rejected option: embedding full carried payloads would add **29.41 MiB** to a 24.61 MiB publication, more than doubling it.

**Seven questions the analysis says you'd need to rule** before any slice — most consequential: is the identity graph normative for *every* generated page (universal beats "optional identity semantics", which would be unreliable); does the custom-page fallback carry it too; and must `weave validate` reject HTML/Turtle disagreement in the same slice. The proposed round-trip contract is that Turtle stays authoritative, embedded JSON-LD is a generated projection and never an import source, and a mismatch means the HTML is stale.

Note this cuts against the June intent recorded in the note — a page-identity graph distinguishing the presented resource from the page file was *your* idea, and the analysis calls that design rationale rather than consumer evidence. Ruling it normative is a legitimate way to revive this; it just needs to be an explicit product decision rather than drift.

## Markdown site pipeline — four contract rulings

**Direction RULED 2026-08-06.** Dave accepted the unified/remark/rehype pipeline (by asking for the ESM change that unblocks it), named the endpoint as **building sites via the public API**, and ranked the work **below the existing three queue items**. Cut as [[wa.task.2026.2026-08-06_0854-markdown-site-pipeline]], queue item 4. The prior `markdown-it` lean is superseded — see [[wa.discussion.2026-08-08-markdown-renderer]] for the blind evaluation that reversed it, and note the two evaluations answered different questions, so neither was wrong.

What remains open are four rulings that gate slice 1 (contract and fixtures). Nothing downstream can be specified without them.

1. **Dendron identity.** Is a note's canonical identity its dot-hierarchy filename, its frontmatter `id`, an RDF identifier, or a combination? This decides what happens on collision across vaults, and it is the one question everything else inherits from.
2. **Which wikilink forms are contractual in v1?** Recommendation: naked target, alias, and heading anchor only — not note references, block references, or tags. Narrow now is easy to widen later; the reverse is not.
3. **Missing or unpublished target behavior.** Build failure, warning plus plain text, disabled link, stub page, or link to another canonical location? Failing the build is the safest default for a publication pipeline but the harshest for authoring.
4. **Default publish state for conversation transcripts.** This is a privacy decision, not a rendering one — transcripts carry model output and whatever was pasted into them. Recommendation: **unpublished by default, opt-in required.**

A fifth is sequenced later but worth knowing now: **page generation is not currently exported from `src/api/mod.ts`.** "Sites via the public API" requires that export, and both existing programmatic surfaces ([[wd.programmatic-validate-api]], [[wd.programmatic-version-api]]) were ratified as contract notes before implementation. This one should be too, rather than arriving as an incidental export.

Also folded into that task rather than cut separately: `resolveMarkdownHref` accepts any URI scheme, so authored `[x](javascript:...)` reaches `href` verbatim. Not exploitable today (self-authored content, no authored regions published); `rehype-sanitize` fixes it as a side effect. **Take the three-line allowlist immediately if authored content starts publishing before the pipeline lands.**
