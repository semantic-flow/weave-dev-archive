---
id: weave-drop-meshinheritab-zgsy6b
title: 'weave: drop MeshInheritableConfig'
desc: ''
created: 1779917056605
updated: 1779917056605
participants: [codex.gpt-5.5]
conversationEventKinds: [message.assistant, message.user, tool.call, tool.result]
---

# User_2026-05-27_1406_01

weave: drop MeshInheritableConfig

Please review [[wa.task.2026.2026-05-27_1347-drop-MeshInheritableConfig]] and let me know if you have any questions or suggestions for improvement.

# gpt-5.5_2026-05-27_1406_14

I’ll read the repo guidance and product vision first, then the task note itself so the feedback is grounded in the project’s own vocabulary. I’ll keep this as a document/task review unless the note reveals something that needs a quick code lookup.

The task note is coherent at a first pass: it has a clean concept split between mesh-owned config and Knop-authored subtree defaults. I’m going to check the live ontology and the follow-up discovery task now, because the useful review questions are probably hiding where the document crosses repo boundaries.

One useful thing popped out already: the dependent discovery task is still phrased in the old role-specific source vocabulary, exactly as this task predicts. I’m checking whether the Weave runtime actually references those terms today, and whether the ontology has SHACL shapes that should be named explicitly in the plan.

Reviewed it. The direction looks good to me: dropping `KnopConfig` and the mesh-inheritable authoring surface before config-source discovery hardens those terms is the right move.

My main pushback: don’t make `sfcfg:hasInheritableConfig` a subproperty of `sfcfg:hasConfig` if `hasConfig` now means local config. The task currently says to do that in [the ontology contract](<wa.task.2026.2026-05-27_1347-drop-MeshInheritableConfig.md:108>), but the summary defines `hasConfig` and `hasInheritableConfig` as different roles. With RDFS reasoning, inheritable config would also be inferred as ordinary `hasConfig`, which muddies the local/inheritable split. I’d either keep inheritable as a sibling property, or add a neutral superproperty like `hasConfigAttachment` if shared tooling needs one.

Other suggestions:

- Explicitly resolve the layer-role fallout. Current Weave defaults still include `configLayerRole_meshInheritable` in [defaults/config-resolution.ttl](<../../../../../defaults/config-resolution.ttl:31>), and runtime constants still parse it in [effective_config.ts](<../../../../../src/runtime/config/effective_config.ts:257>). I’d add a concrete checklist item to remove or rename that role, not leave it under “if any live code references them.”
- Clarify conventional file bootstrap semantics: does `_mesh/_config/config.ttl` become mesh-local by path, or must it assert `<meshIri> sfcfg:hasConfig <>`? Same question for `<knop>/_knop/_config/config.ttl`. The task says filesystem location is not the only authority, but conventional discovery still grants initial authority, so naming that as bootstrap behavior would prevent confusion.
- Enumerate “recognized config-bearing subjects.” The note says `SemanticMesh` and `Knop`, but source/default/config-resolution surfaces may also matter. SHACL/runtime validation needs a closed list.
- Add SHACL owner-shape expectations explicitly. OWL `rdfs:domain sflo:Knop` infers the subject is a Knop; it does not validate that only Knops can use the property. The fail-closed behavior needs SHACL/SPARQL or runtime checks.
- The follow-up config-source task is definitely stale: it still centers `hasMeshConfigSource`, `hasKnopLocalConfigSource`, and `hasKnopInheritableConfigSource`. Updating that task should probably happen in the same commit as the ontology wording change.

No blocking questions from me beyond the subproperty issue. That one is worth settling before implementation.

# User_2026-05-27_1409_48

Agreed on the subproperty issue. Proceed if you're ready.

# gpt-5.5_2026-05-27_1410_07

Ready. I’ll turn the review into the actual cleanup now: first I’ll check repo boundaries and references, then I’ll edit the task notes/ontology/runtime surfaces in the smallest coherent slice and run the relevant Deno checks.

The main repos are clean, and the archive task note already has local edits. I’ll read that note before touching it so I preserve your latest changes instead of trampling them.

The task note’s only existing diff is the timestamp plus the “it’ll probably need a bunch…” aside, so I can fold that into the cleanup safely. Next I’m mapping the validation scripts and the exact ontology/test surfaces.

I found one real runtime seam: Weave has an inheritance helper that still models mesh-inheritable sources separately from Knop offers. I’ll read that before editing, because this is where the ontology decision needs to become actual behavior instead of just renamed terms.

The implementation shape is clear now. I’m going to remove the mesh-inheritable role from ontology/defaults/runtime, add the two generic inheritable attachment terms as siblings rather than subproperties, and tighten tests/docs around that.

I’m starting with the ontology because it’s the source of truth. The key edit is deliberately small: no `subPropertyOf` from inheritable to local config/source, and no live mesh-inheritable layer role.

The SFLO side now has the new vocabulary plus SHACL guardrails, and I added a retired-term check so those old names don’t sneak back into active RDF. I’m moving to Weave’s small runtime/default cleanup next.

Weave’s code cleanup is small and contained: the separate mesh-inheritable source path is gone, and the defaults no longer list that layer. Now I’m updating the framework/task notes so the next runtime task doesn’t reintroduce the old vocabulary.

The edits are in place. I’m going to let Deno format the TypeScript test/helper changes, then I’ll run the focused SFLO and Weave validation paths plus lint.

SFLO checks are green, including release validation. I’m running the Weave checks now, with the config tests targeted and the repo linter/checker because the runtime types changed.

Weave lint/check and focused config tests are green. I’m doing a final diff sanity pass now, including whitespace checks and the task checklist updates.

One more ontology inconsistency surfaced in the final read: `hasEffectiveConfig` was still a subproperty of `hasConfig`. Since `hasConfig` now means authored local config, that would create the same reasoning leak we just avoided for inheritable config. I’m going to sever that too.

That extra effective-config subproperty fix is patched and recorded in the ontology notes/task. I’m rerunning the SFLO checks touched by that follow-up.

Implemented the cleanup across the four touched repos. The big bits:

- SFLO ontology now uses `hasConfig` / `hasConfigSource` for local config and `hasInheritableConfig` / `hasInheritableConfigSource` for Knop descendant offers.
- Removed `KnopConfig`, role-specific config attachment/source terms, and `configLayerRole_meshInheritable`.
- Also removed `hasEffectiveConfig` as a subproperty of authored local `hasConfig`, for the same reasoning-leak reason.
- Weave no longer has a mesh-inheritable runtime/default layer or inheritance projection path.
- Updated framework docs and both task notes, including the config-source discovery follow-up.

Validation passed:

- SFLO: `deno task fmt`, `lint`, `check`, `test`, `release:validate`, `git diff --check`
- Weave: `deno task fmt`, `lint`, `check`, targeted config tests, `git diff --check`
- Framework/archive docs: `git diff --check`

Suggested commit messages:

```text
ontology: simplify config attachment roles

- Replace role-specific config attachment vocabulary with generic local and Knop-inheritable properties.
- Remove KnopConfig, mesh-inheritable layer role, and the hasEffectiveConfig -> hasConfig subproperty leak.
- Add SHACL subject constraints and guardrails for retired config terms.
```

```text
config: remove mesh-inheritable runtime layer

- Drop configLayerRole_meshInheritable from Weave defaults and parser constants.
- Simplify Knop inheritance projection to ancestor Knop offers only.
- Update config runtime tests for the surviving layer order.
```

```text
docs: document generic config attachment roles

- Update config docs/specs for local versus Knop-inheritable attachment semantics.
- Clarify conventional mesh and Knop config file locations.
- Align config-source authority wording with subject-plus-property attachment rules.
```

```text
task: align config role cleanup planning

- Resolve the inheritable subproperty question in the drop-MeshInheritableConfig task.
- Update config-source discovery planning to use hasConfigSource and hasInheritableConfigSource.
- Mark the cleanup task checklist complete after validation.
```

# User_2026-05-27_1424_04

OK, what's next?

# gpt-5.5_2026-05-27_1424_24

Next, I’d freeze this cleanup as its own commit stack before starting config-source discovery. It touched ontology, framework docs, Weave runtime, and task notes, so mixing the next runtime feature into the same diff would make review harder.

Recommended order:

1. Review/commit per repo:
   - `sflo`: ontology, SHACL, guardrails, release notes, decision log
   - `semantic-flow-framework`: `sf.config` and config behavior spec
   - `weave`: remove mesh-inheritable runtime/default layer
   - `weave-dev-archive`: task note updates

2. Before merge, run fuller CI if you want extra confidence:
   - `deno task ci` in `sflo`
   - `deno task ci` in `weave` if you’re ready for the longer test run

3. Then start [[wa.task.2026.2026-05-27_1246-config-source-discovery-and-resolution]] with a narrow first slice:
   - discover `sfcfg:hasConfigSource` on the mesh subject
   - resolve only safe local/workspace-bounded config sources
   - compile resolved config into the mesh-local layer
   - reject `hasConfigSource` on unrecognized subjects

Tiny pushback: I would not start Knop config discovery yet. Mesh-local config-source discovery is now the best first executable slice, and it will prove the generic attachment model without pulling inheritance traversal into the same change.

# User_2026-05-27_1430_48

Regarding open questions, my instincts are:

- I think we allow attachment via config, otherwise we can't chain.
- hasConfigSource on unrecognized subject -- I'm okay with that being a hard error.
- don't we already have an ordering/priority semantics?
- this is your pushback, and I don't really understand it. mesh-local config-source discovery would include knop-based config, no?

# gpt-5.5_2026-05-27_1431_06

That all sounds sane. I’m going to update the config-source task note to close the open questions in that direction, and I’ll also spell out the “Knop-based config” distinction because that’s the bit where terminology can trick us.

# gpt-5.5_2026-05-27_1431_19

I’m going to patch this as decisions rather than leave them as open issues. The important wording is: “mesh-local first slice can resolve a Knop-owned artifact if the mesh attaches it; it doesn’t yet scan Knop subjects as independent attachment points.”

# gpt-5.5_2026-05-27_1432_11

One last small tightening: given the project’s fail-closed posture, I don’t love “Knop attachment statement may be reported” for the first slice. I’m changing that to “reject as unsupported until Knop source discovery exists,” while still allowing a mesh attachment to resolve a Knop-owned artifact.

# gpt-5.5_2026-05-27_1432_35

Agreed, and I updated [the task note](<wa.task.2026.2026-05-27_1246-config-source-discovery-and-resolution.md>) accordingly.

My read:

- Yes, allow config-source attachment from config itself. Otherwise chaining reusable config sources is awkward or impossible.
- Yes, `hasConfigSource` / `hasInheritableConfigSource` on unrecognized subjects should be a hard error.
- Yes, we already have policy ordering semantics: layer, selector specificity, and `sfcfg:policyPriority`. What we should not add is source-list ordering as another override mechanism. Sources can be processed deterministically, but conflicts should resolve through existing config semantics or fail closed.
- On “Knop-based config”: yes, mesh-local discovery can resolve a Knop-owned `ConfigArtifact` if the mesh subject attaches it. My pushback was narrower: don’t scan Knop subjects as attachment points yet. So `<mesh> hasConfigSource <source-that-targets-knop-owned-config>` is in; `<knop> hasConfigSource <source>` as an independently discovered Knop-local layer is later.

I also tightened the first slice to reject Knop attachment-subject statements as unsupported until Knop source discovery exists. `git diff --check` passes for the archive note.

# User_2026-05-27_1434_33

So we're only doing mesh-sourced config in the first slice, but knop-sourced is in this task, just second-slice?

# gpt-5.5_2026-05-27_1434_43

Yes, exactly.

For this task:

- **First slice:** mesh attachment discovery only.
  - Discover `<meshIri> sfcfg:hasConfigSource ?source`.
  - Allow that source to target config bytes or a `ConfigArtifact` wherever it lives, including under a Knop.
  - Compile resolved content as **mesh-local** config because the mesh subject attached it.
  - Allow chaining from accepted mesh-local resolved config graphs.

- **Second slice in the same task:** Knop attachment discovery.
  - Discover `sflo:Knop sfcfg:hasConfigSource ?source` as **Knop-local** config.
  - Discover `sflo:Knop sfcfg:hasInheritableConfigSource ?source` as **Knop-authored descendant defaults**.
  - Add ancestor traversal, inheritance projection, and cycle handling across inherited source graphs.

So “Knop-sourced bytes/artifacts” can appear in slice one if the **mesh** points at them. “Knop subject as authority/attachment point” waits for slice two.