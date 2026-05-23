---
id: weave-task-audit-and-cle-tqcz54
title: 'weave task audit and cleanup'
desc: ''
created: 1779509457167
updated: 1779509457167
participants: [codex.gpt-5.5]
conversationEventKinds: [message.assistant, message.user, tool.call, tool.result]
---

# User_2026-05-21_0912_48

weave task audit and cleanup

Could you review all the wa.task.* notes created before  (5/20) and summarize for each: whether they can clearly be closed due to completeness or because they are no longer relevant/applicable or have been superseded by other tasks; for those that could be closed by deferring work, where we should capture that deferred work; and for any that need updating, make updates when obvious, and bring me back any questions. 

This is a big task, but it probably makes sense to have all the tasks in your context to do this reconcilliation.

# gpt-5.5_2026-05-21_0912_57

I’ll treat “before 5/20” as `wa.task.*` notes dated before 2026-05-20, and I’ll avoid renaming any task notes unless you explicitly ask. First I’m reading the repo guidance and product vision, then I’ll inventory the task notes and reconcile them against current docs/code.

I’ve got 29 pre-5/20 `wa.task` notes in the archive path. I’m going to map each one against nearby `wa.completed` notes, current checkboxes/status fields, and the current implementation so the close/keep/defer calls aren’t just filename archaeology.

The note set splits into a few eras: April page/source/bootstrap planning, early-May extraction/config cleanup, and mid-May branch-published/deploy work. I’m reading them in those clusters and checking whether later completed notes or current docs already absorbed the dangling work.

A lot of the “open” notes are actually old umbrellas with their useful pieces already moved into later, sharper tasks. I’m doing one implementation/doc check pass now so the dispositions can say “closed/superseded/defer here” with specific anchors rather than vibes.

I found a few straightforward stale spots worth fixing in-place: the extraction behavior spec checkbox is now done; the old Bob import-boundary note should hand its remaining CLI/runtime backfill to the new import task; append-onlyish inventory still named removed `prepare gh-pages`; and the roadmap still had the pre-append-only “graph-preserving” wording. I’m patching just those obvious documentation cleanups.

I audited the 29 `wa.task.*` notes dated before 2026-05-20, using 2026-05-20 as the cutoff. I did not rename any tasks to `completed`, per repo guidance.

I made a few obvious cleanup edits:
- Marked the extraction behavior spec update complete in [[wa.task.2026.2026-05-04-extraction-improvements]].
- Deferred Bob import/runtime items to `[[wa.task.2026.2026-05-21_0907-import]]` and remote access to `[[wa.task.2026.2026-05-20_2152-workingAccessUrl]]`.
- Added an audit note to [[wa.task.2026.2026-04-14_0018-configurable-test-tmp]].
- Updated stale roadmap wording in [roadmap.md](../../../../../documentation/notes/roadmap.md) and added the optional `WEAVE_TEST_TMP_ROOT` follow-up to [wd.todo.md](../../../../../documentation/notes/wd.todo.md).

**Audit Summary**

| Task | Recommendation | Notes |
|---|---:|---|
| 2026-04-06 markdown payload publishing | Close / superseded | Broad early note; remaining publication work is now split across page renderer, import, working URL, and config tasks. |
| 2026-04-08 resource page definition and sources | Keep or split | Much is done, but exact/fallback page-source behavior still deserves a focused resolver task if this closes. |
| 2026-04-08 weave orchestration refactor | Keep | Runtime orchestration/target-preparation seam is still distinct from the 5/21 extraction refactor. |
| 2026-04-08 page definition ontology and config | Close | Superseded by page definition spec, ontology/config synthesis, and later implementation notes. |
| 2026-04-11 operational config runtime resolution | Close via deferral | Local resolution is largely implemented; remote concerns belong to `workingAccessUrl` / later target URL work. |
| 2026-04-13 shape generalization | Close via deferral | Replaced by concrete config, first-payload, append-onlyish, and extraction-refactor tasks. |
| 2026-04-13 Bob import boundary | Close via deferral | Remaining first-class import work is now captured in the 2026-05-21 import task. |
| 2026-04-13 page renderer refresh | Keep | Renderer/style/frontmatter/reference regeneration work still looks active. |
| 2026-04-14 configurable test tmp | Close via deferral | Current helper uses platform temp; optional override is now in `[[wd.todo]]`. |
| 2026-05-04 extraction improvements | Close | Checklist is now complete. |
| 2026-05-04 fingerprint verification | Close / superseded | Too skeletal now; future digest verification should be a new validation task. |
| 2026-05-04 refactor `planFirstPayloadWeave` | Question | Some pieces are done, but I’m not sure multi-pending first-payload behavior is actually settled. |
| 2026-05-04 split extraction from page selection | Keep / question | Still a real model change unless `_sources` is now considered the durable direction. |
| 2026-05-05 optional history / slim artifacts | Close via deferral | Superseded by grand config synthesis and append-onlyish work. |
| 2026-05-06 grand config synthesis | Keep, but narrow | Mostly done, but remaining unchecked items should probably be split into focused follow-ups. |
| 2026-05-13 refactor | Close via deferral | Narrow extraction work is done; future refactor lives in the 5/21 careful extraction task. |
| 2026-05-13 gh-pages branch deployments | Close | Core workflow is implemented and documented; follow-ups already have separate notes. |
| 2026-05-14 guarded branch rebuild | Keep | Still unchecked and useful for explicit rebuild-from-scratch behavior. |
| 2026-05-16 deploy profile | Keep | Still future-facing and called out as not part of current release. |
| 2026-05-16 manifest completeness check | Keep | Validation gap still appears real. |
| 2026-05-16 create SFLO branch mesh | Question | Docs/runbook exist, but I can’t tell whether the dogfood publication itself was completed. |
| 2026-05-17 append-onlyish inventory | Keep | Still active; roadmap now points to this wording. |
| 2026-05-17 performance optimization | Close via deferral | Timing/read-cache/progress work is done; parsed RDF reuse can be a future perf note. |
| 2026-05-17 prepare symmetry | Close | Resolved by remove-prepare task. |
| 2026-05-18 integrate manifest | Keep | Still a future feature. |
| 2026-05-19 lateststate improvement | Close via deferral | Main behavior is implemented; conformance/shared resolver work belongs elsewhere. |
| 2026-05-19 latest state conformance | Keep | Still valid as later fixture/conformance work. |
| 2026-05-19 fix misclassified children | Close | Fully checked off. |
| 2026-05-19 branch-based workingfile fix | Question | Core bug is fixed; remaining validation/reporting polish may deserve either this task or a smaller follow-up. |

**Questions**

1. For `split-extraction-from-page-selection`: do you still want to move away from `ExtractionSource` registries toward reference-link/page-selection semantics, or is `_sources` now the durable provenance model?
2. For `refactor-planFirstPayloadWeave`: is multi-pending first-payload weave still a blocker, or should I split remaining URPX/progress details into fresh tasks?
3. For `create-sflo-branch-mesh`: was the actual SFLO branch-mesh dogfood completed, or should that note remain active?
4. For `branch-based-workingfile-fix`: should we close the core bug and create a small validation/reporting follow-up, or keep this task open?

No tests/linter run; these were markdown/docs-only edits.

Suggested commit messages:

Weave repo:
`docs(tasks): update task audit fallout`

Archive repo:
`docs(tasks): reconcile stale Weave task notes`

# User_2026-05-21_1009_37

regarding "- for supporting DigitalArtifacts, allow "thin" mode: just working file, no histories" -- default is now thin mode for most (all?) supporting artifacts?

# gpt-5.5_2026-05-21_1009_45

I’ll check the current artifact/history defaults before answering, since this sounds like exactly the kind of TODO that may have gone stale after the config work.

The first pass shows history policy support is real, but I need to separate payload histories from support artifacts. I’m going into the config defaults and support-page planner now.

A useful distinction is emerging: “thin” is currently a policy value (`currentOnly`) applied by role, not a universal absence of historical vocabulary. I’m checking the built-in defaults to see which support roles actually get it.

I’m going to update that TODO rather than leave it as a stale open question. The accurate version is “mostly yes, but not quite all”: support artifacts inherit `currentOnly` by default, while config is explicitly versioned and ResourcePageDefinition still has a versioned legacy path to audit.

Mostly yes, but I wouldn’t say “all.”

Current defaults in [defaults/application.ttl](../../../../../defaults/application.ttl:7) make `currentOnly` the default history policy. That means support roles inherit thin/current-only behavior unless overridden. The explicit exceptions are:

- `payload`: `versioned`, but payloads are not supporting artifacts.
- `config`: `versioned`, so `_mesh/_config` is still not thin by default.
- `meshInventory`, `knopInventory`, and `runtimeMeta`: explicitly `currentOnly`.
- `meshMetadata`, `knopMetadata`, `referenceCatalog`, `resourcePageDefinition`, etc. inherit `currentOnly`.

One caveat: the ResourcePageDefinition path still has some legacy versioned progression behavior in the planner, so I would not declare the whole support-artifact story fully settled yet.

I updated [wd.todo.md](../../../../../documentation/notes/wd.todo.md:21) to replace the stale “allow thin mode” note with a narrower audit/documentation follow-up for the remaining exceptions.

No tests run; docs-only change. Suggested commit message: `docs(todo): clarify support artifact thin defaults`

# User_2026-05-21_1311_32

we're in the midst of a major refactor. Is "the ResourcePageDefinition path still has some legacy versioned progression behavior in the planner, " reflected in any task?

# gpt-5.5_2026-05-21_1311_41

I’ll verify against the task notes rather than trusting my memory here. I’m going to search the archive for ResourcePageDefinition/history-policy language and check the new refactor task you have open.

It is partially captured, but a bit awkwardly: the page-definition source task still describes `_knop/_page` as a normal support artifact with histories, while the newer thin-default TODO names the mismatch directly. I’m checking whether the ResourcePage generation decomposition task already owns the cleanup, because that may be the better home than the runtime module split.

I’m going to make one small note update so the active refactor work doesn’t accidentally absorb this semantic cleanup. The behavior belongs in the support-history/page-definition backlog, not in the move-only runtime decomposition.

Yes, but it was a little too implicit. It is now explicit.

It’s reflected in three places:

- [wd.todo.md](../../../../../documentation/notes/wd.todo.md:21) has the direct follow-up: audit/document the support-artifact history exceptions, including `config` and the legacy versioned `ResourcePageDefinition` progression.
- [wa.task.2026.2026-05-05-optional-history-and-slim-support-artifacts-by-default.md](wa.task.2026.2026-05-05-optional-history-and-slim-support-artifacts-by-default.md:159) already captures the policy question: `ResourcePageDefinition` and `ReferenceCatalog` may need history unless page-output durability/render manifests replace it.
- [wa.task.2026.2026-04-08_1545-resource-page-definition-and-sources.md](wa.task.2026.2026-04-08_1545-resource-page-definition-and-sources.md:556) reflects the current implemented behavior: `_knop/_page` versions like a normal support artifact.

I also added a guardrail to the active core extraction task: [wa.task.2026.2026-05-21_1037-core-weave-first-extraction-slice.md](wa.task.2026.2026-05-21_1037-core-weave-first-extraction-slice.md:72). It says not to change this behavior during the type/model extraction and points back to `[[wd.todo]]`.

My read: don’t let this sneak into the current major refactor. It’s semantic policy cleanup, not decomposition.

# User_2026-05-21_1359_31

Can you confirm that _page and _ref can be allowed to be current-only? If they are, ResourcePage generation should just make a best effort. So as long as weaving doesn't depend on them having history, that should be fine.

# gpt-5.5_2026-05-21_1445_39

I’ll check this at the code level, because the important line is “as long as weaving doesn’t depend on them having history.” I already suspect `_references` and `_page` are in different states.

The search confirms the asymmetry. `_references` has a current-only planner path and a test for it; `_page` still goes through a history progression planner. I’ll pull the exact snippets before I give you the final answer, because this is a “yes, but not yet everywhere” situation.

I’m going to update the task note to capture the decision more cleanly: current-only is allowed for both, with ResourcePage generation best-effort, and the actual remaining code work is specifically removing `_page`’s planner/generator dependency on history.

Yes as a model/contract direction: `_knop/_page` and `_knop/_references` can be current-only. ResourcePage generation can be best-effort when their histories are absent. Historical regeneration should not force these support artifacts to be versioned.

But I’d push back on “confirmed” for implementation today:

- `_knop/_references`: already supported. `planFirstReferenceCatalogWeave` has a current-only path, and there’s a test proving it creates no `_knop/_references/_history001`.
- `_knop/_page`: not fully supported yet. `planPageDefinitionWeave` still always builds a versioned page-definition progression, and `loadActiveCustomIdentifierPage` currently refuses to use a ResourcePageDefinition unless `currentArtifactHistoryExists` is true.

I updated the notes to reflect that split:
- [wa.task.2026.2026-05-05-optional-history-and-slim-support-artifacts-by-default.md](wa.task.2026.2026-05-05-optional-history-and-slim-support-artifacts-by-default.md:159) now says both may be current-only, with best-effort page generation.
- [wd.todo.md](../../../../../documentation/notes/wd.todo.md:21) now narrows the remaining work to removing the `_page` history dependency.

No tests run; docs-only change. Suggested archive commit message: `docs(tasks): allow current-only page and reference support artifacts`

# User_2026-05-21_1510_31

Is "_knop/_page: not fully supported yet. planPageDefinitionWeave still always builds a versioned page-definition progression, and loadActiveCustomIdentifierPage currently refuses to use a ResourcePageDefinition unless currentArtifactHistoryExists is true." covered in [wd.todo.md](../../../../../documentation/notes/wd.todo.md)  or anywhere else? Should we complete the refactoring before addressing?

# gpt-5.5_2026-05-21_1510_45

I’ll do a quick local cross-check so I don’t overstate it. The key is whether it’s merely in a generic TODO or pinned to the current refactor notes strongly enough to keep scope clean.

Yes. It is covered in two useful places:

- [wd.todo.md](../../../../../documentation/notes/wd.todo.md:21) has the concise backlog item.
- [wa.task.2026.2026-05-05-optional-history-and-slim-support-artifacts-by-default.md](wa.task.2026.2026-05-05-optional-history-and-slim-support-artifacts-by-default.md:266) has the concrete implementation checkboxes:
  - refactor `_knop/_page` weave planning to allow current-only
  - refactor custom identifier generation to render best-effort without `_knop/_page/_history001`

It’s also guarded in [wa.task.2026.2026-05-21_1037-core-weave-first-extraction-slice.md](wa.task.2026.2026-05-21_1037-core-weave-first-extraction-slice.md:72) as something not to change during the type/model extraction.

My recommendation: finish the current behavior-preserving refactor slice before addressing it, but don’t wait for every refactor in the universe. This is a semantic behavior change crossing core planning and runtime page generation, so it should be its own small cleanup slice after the relevant files/modules are stable. Mixing it into a move-only refactor would make review and regression hunting worse.

# User_2026-05-22_1416_58

in [[wa.task.2026.2026-05-16_1707-create-sflo-branch-mesh]] mark everything a x

Do either of @wa.task.2026.2026-05-20_2152-workingAccessUrl or wa.task.2026.2026-04-08_1545-resource-page-definition-and-sources inform each other?

# gpt-5.5_2026-05-22_1444_55

I’ll compare the two task notes directly and anchor the answer in their actual wording, since this is exactly where old roadmap threads can overlap in sneaky ways.

The overlap is real, but it is not a merge. `workingAccessUrl` is the remote current-byte resolver for artifacts; the page-definition task is the consumer/model that deliberately keeps direct page-source remote access out of the first pass.

Yes. They inform each other, but they should stay separate.

`workingAccessUrl` informs the page-definition task because page sources can target governed `DigitalArtifact`s, and those artifacts may eventually have remote current-byte locators. The relevant page-definition TODO is the Phase 5 item about adding `workingAccessUrl` handling to governed-artifact current resolution.

The page-definition task informs `workingAccessUrl` by setting the boundary: page generation should not casually fetch live remote page content. It should prefer imported/governed in-tree artifacts, or only follow remote current bytes behind explicit operational policy. That matches the `workingAccessUrl` task’s own constraint that ResourcePage generation should not fetch the network merely to render pages.

Important distinction:

- `sflo:workingAccessUrl`: remote current-byte locator for a `DigitalArtifact`.
- `sflo:targetAccessUrl`: direct remote target on an `ArtifactResolutionTarget` / page source.
- Import: copy/acquire remote bytes into a governed in-tree artifact.

So: `workingAccessUrl` can eventually support page-definition artifact-backed sources, but it should not collapse the page task’s import boundary or become direct `targetAccessUrl` fetching by accident. I’d treat `workingAccessUrl` as upstream resolver infrastructure; page-definition should consume it deliberately later.

# User_2026-05-22_2032_48

let's working on refining [[wa.task.2026.2026-04-08_1545-resource-page-definition-and-sources]] . Keep in mind, a major refactor is partially completed and continuing, see [[wa.task.2026.2026-05-21_0849_careful-extraction-refactor]] .

One thing I want to do is be able to keep/re-use the existing Semantic Site's look and feel, even if that's just the default. Is the current default using the page template-based approach?

# gpt-5.5_2026-05-22_2033_16

I’ll trace the current ResourcePage rendering path rather than infer from the older task prose. The distinction I’m looking for is “template-like renderer functions” versus an actual page-template artifact/config mechanism.

The code has split further since the old note: runtime page generation now lives in focused modules. I’m reading `pages.ts` and the page-definition loader next, since that’s where the actual default HTML and optional custom page path live now.

One important finding: the current default does have a `ResourcePageTheme` interface internally, but there is only a hardwired `defaultResourcePageTheme` backed by TypeScript string-rendering and inline CSS. That is not yet a mesh/config/template artifact system.

I’m going to patch the task note with that distinction, because it affects the design direction: preserving the Semantic Site look should start as default theme/presentation reuse, not as proof that the template vocabulary already exists.

No. The current default is not using a page-template-artifact approach.

What exists now is an internal renderer seam: `src/runtime/weave/pages.ts` has a `ResourcePageTheme` interface and a hardwired `defaultResourcePageTheme`, but the default theme is TypeScript rendering functions plus inline CSS. Custom identifier pages can use `_knop/_page/page.ttl` for page content composition and `_knop/_assets` stylesheets, but that is not a general template/chrome selection system.

I updated the task note to say that explicitly and to capture the design direction: preserving the existing Semantic Site look and feel should be treated first as the built-in default ResourcePage presentation baseline, not as something requiring `_knop/_page` or user-selectable template artifacts.

Docs-only change. Suggested archive commit message: `docs(tasks): clarify ResourcePage default theme direction`

# User_2026-05-22_2043_14

If we move to the same model for the default and customized pages (i.e., a ResourcePagePresentationConfig with Outer/InnerResourcePageTemplate, ResourcePageStylesheet; using ResourcePageDefinition / hasResourcePageSource, and making the current panels like "Children" re-usable and the default logic over-ridable) we'll be in a much better position to make consistent sites.

# gpt-5.5_2026-05-22_2043_23

That direction sounds right to me, with one caveat: I want to check the current ontology shape before I bless the exact names, because the “same model for default and custom” idea is strong but only if the template/panel pieces don’t smuggle runtime logic into authored RDF.

I like this direction. I’m going to capture it in the task note, but with the guardrail that “same model” should mean a shared render/presentation model, not necessarily that every default page must persist its own `_knop/_page/page.ttl`.

Yes, I agree. That’s the right target shape.

The only guardrail I’d add is: same model should not require every default page to persist a `_knop/_page/page.ttl`. The runtime can synthesize the default `ResourcePageDefinition` / panel set from the resource context and effective config. But both default and customized pages should flow through the same conceptual pipeline:

`ResourcePageDefinition` / `hasResourcePageSource` for content inputs, `ResourcePagePresentationConfig` for chrome, `OuterResourcePageTemplate` / `InnerResourcePageTemplate` for shell/body layout, and `ResourcePageStylesheet` for shared styling.

I updated the task note to capture that direction, including reusable panel models for things like children, properties, references, histories, raw source, provenance, and Semantic Flow metadata. The important engineering point is that runtime code should still compute those panel models; templates/config should include, order, suppress, restyle, or replace rendering without becoming responsible for RDF discovery or graph navigation.

Docs-only change. Suggested archive commit message: `docs(tasks): align default and custom ResourcePage presentation model`

# User_2026-05-22_2051_37

Maybe we should consider extracting (some of) the remaining items into a new task note that captures this "Resource page config and templating for default pages" direction.. that old note is a little crufty.

# gpt-5.5_2026-05-22_2051_51

I agree; the old note is carrying both implemented history and future design, and the new default/custom presentation direction deserves its own cleaner task. I’ll add a focused task note and leave the old one pointing to it instead of trying to make the old note do every job forever.

I’m going to make the new note own the default/custom presentation unification, then mark the old note’s overlapping Phase 4 bullets as deferred into it. That keeps the old task as historical/page-source work instead of another forever-bucket.

Yes. I split that direction into a new focused task note:

[[wa.task.2026.2026-05-22_2253-resourcepage-config-and-templating]]

It now owns the “ResourcePage config and templating for default pages” thread: shared default/custom presentation pipeline, synthesized default page definitions, reusable panel models, `ResourcePagePresentationConfig`, outer/inner templates, stylesheets, and preserving the Semantic Site look as the default baseline.

I also updated the old page-definition/source task to point to the new task and marked the overlapping Phase 4 presentation/template items as `[d]` deferred to it:

[[wa.task.2026.2026-04-08_1545-resource-page-definition-and-sources]]

No tests run; docs-only. Suggested archive commit message: `docs(tasks): split ResourcePage presentation config work`

# User_2026-05-22_2056_16

which should we do first, 1545 or 2253? And wait until the refactor is complete, right?

# gpt-5.5_2026-05-22_2056_30

I’ll answer from the current task split rather than reopen the notes: this is sequencing and dependency management more than a code question.

Do **1545 first**, but only the remaining source-semantics parts. Then do **2253**.

My read:

1. Finish the current behavior-preserving refactor enough that the relevant seams are stable.
   Especially the ResourcePage model-builder / render-helper extraction work. No semantic changes while files are still moving.

2. Do the narrowed `1545` work next.
   It owns page-definition source behavior: `ResourcePageSource`, artifact-backed sources, exact/current/latest modes, fallback, import boundary, `targetAccessUrl` / `workingAccessUrl` decisions. `2253` will be much cleaner if the inputs and source-resolution semantics are settled first.

3. Do `2253` after that.
   It owns presentation: default/custom unification, panels, templates, stylesheets, config. It should consume the source model, not define it.

I would not wait for every refactor task in the universe to finish. I would wait until the specific files/modules you’d touch for `1545` have stopped moving. Then land `1545` in small semantic slices. After that, `2253` can be a more pleasant design/implementation path instead of trying to build presentation abstractions over unstable source behavior.

# User_2026-05-22_2101_05

Regarding "Decide whether first-pass page generation should ever follow `workingAccessUrl` directly, or continue requiring imported in-tree artifacts for remote-origin content even if the broader artifact model permits external current-byte locators." -- I think only in cases where the fingerprint is present and checks out? Or if we want to be more liberal, disallow script in the source target. What do you think?

# gpt-5.5_2026-05-22_2101_20

I think your instinct is good, but I’d make it stricter than “fingerprint or no script.” Fingerprint solves reproducibility/integrity; script filtering solves rendering safety. They cover different failure modes, so the first-pass rule should probably require both for direct remote page content.

I’m going to update the selected checklist item to make the decision explicit. The note should say that fingerprinted direct fetch is a narrow exception, not the general rule.

I agree with your instinct, but I’d make it **both**, not either/or.

A fingerprint/digest solves integrity and reproducibility: “these are the exact bytes I expected.” But it does not make those bytes safe to render. A perfectly fingerprinted HTML blob can still contain script.

Script filtering/sanitization solves active-content safety. But it does not solve drift: an unfingerprinted `workingAccessUrl` can silently change between runs.

So I updated the task note with this decision: direct `workingAccessUrl` page rendering is allowed only as a narrow, explicitly policy-enabled, digest-checked path, and the rendered content is still treated as untrusted. No raw script execution, event-handler attributes, `javascript:`/active URL surfaces, or unsafe embedded HTML. Otherwise, use import/governed in-tree artifacts or settled historical state rendering.

Docs-only change. Suggested archive commit message: `docs(tasks): constrain remote page-source rendering`

# gpt-5.5_2026-05-22_2101_42

I agree with your instinct, but I’d make it **both**, not either/or.

A fingerprint/digest solves integrity and reproducibility: “these are the exact bytes I expected.” But it does not make those bytes safe to render. A perfectly fingerprinted HTML blob can still contain script.

Script filtering/sanitization solves active-content safety. But it does not solve drift: an unfingerprinted `workingAccessUrl` can silently change between runs.

So I updated the task note with this decision: direct `workingAccessUrl` page rendering is allowed only as a narrow, explicitly policy-enabled, digest-checked path, and the rendered content is still treated as untrusted. No raw script execution, event-handler attributes, `javascript:`/active URL surfaces, or unsafe embedded HTML. Otherwise, use import/governed in-tree artifacts or settled historical state rendering.

Docs-only change. Suggested archive commit message: `docs(tasks): constrain remote page-source rendering`

# User_2026-05-22_2134_52

OK, I think you can update [wa.task.2026.2026-04-11_1723-operational-config-for-runtime-resolution.md](wa.task.2026.2026-04-11_1723-operational-config-for-runtime-resolution.md) . Integrate has broadened, and I think we will allow it to broaden further, carefully. I think we can resolve those phase 3 items and create a new task for remote integrate. Or maybe we just defer remote integrate, perhaps import is adequate for now. 

One new API surface we should have is, after something has been integrated, being able to convert it to a regular in-mesh (or sidecar or other-local-branch) workinig file. I think [wa.task.2026.2026-05-21_0907-import.md](wa.task.2026.2026-05-21_0907-import.md)  needs refinement: importing is going to be very different for whole-mesh-repos, sidecar repos, and branch-based meshes, right?. 

Also, instead of using the bob page content, which we ended up using already (how did we do that?) we should probably find new content that can legitimately extend the 3 fixture ladders.

# gpt-5.5_2026-05-22_2135_07

I’ll treat this as a docs/task-note cleanup pass: first I’ll read the two task notes and the nearby fixture/roadmap context, then I’ll patch the notes so the remaining work is clearer instead of leaving remote integrate and import boundaries tangled together.

# gpt-5.5_2026-05-22_2135_20

I’ve got the current notes. The import note is still Bob-centered and too “one mesh shape fits all”; the operational-config note also still carries remote-integrate questions as if they belong there. I’m reading the remaining phase/checklist sections now so I can patch the actual stale items instead of adding more prose on top.

# gpt-5.5_2026-05-22_2136_21

I’ll fold this into the task notes rather than spinning up a remote-integrate note right away. The shape I’m seeing is: keep remote integrate deferred, make import/localize the explicit acquisition surface, and make the import task topology-aware for whole-mesh, sidecar, and branch-based meshes.

# gpt-5.5_2026-05-22_2136_26

I refreshed the repo guidance and the two task notes. The operational-config note already cleanly distinguishes modeled remote policy from active dereferencing, so I’m going to make the remaining remote items explicit deferrals and move the concrete “make it local” surface into the import note.

# gpt-5.5_2026-05-22_2136_31

There are unrelated archive changes already in that repo, so I’ll leave those alone. I’m editing only the two task notes you called out.

# gpt-5.5_2026-05-22_2137_30

The import note needs a slightly bigger reshaping: I’m going to preserve the current command idea, but stop treating Bob as the next fixture, add topology-specific behavior, and capture the “localize an integrated artifact” API as part of the import family.

# gpt-5.5_2026-05-22_2139_30

I also fixed one stale wikilink prefix in the operational-config note while I was there. Since this is documentation-only, I’m going to sanity-check the diff rather than run the Deno test stack.

# gpt-5.5_2026-05-22_2139_51

Updated both notes.

In [wa.task.2026.2026-04-11_1723-operational-config-for-runtime-resolution.md](wa.task.2026.2026-04-11_1723-operational-config-for-runtime-resolution.md:40), I resolved Phase 3 toward: no remote integrate task yet, keep active remote fetching deferred, and center remote-origin acquisition on explicit import/localize workflows. I also marked the remote locator items deferred and the integrate/import split decided at [line 366](wa.task.2026.2026-04-11_1723-operational-config-for-runtime-resolution.md:366).

In [wa.task.2026.2026-05-21_0907-import.md](wa.task.2026.2026-05-21_0907-import.md:11), I refined import as a topology-aware acquisition/localization surface:
- whole-mesh, sidecar, and branch-based meshes are now explicitly covered
- added the localize-from-integrated API shape at [line 57](wa.task.2026.2026-05-21_0907-import.md:57)
- deferred remote-fetching `integrate`
- kept Bob as historical regression/context only, and added the need for new non-Bob content across the three fixture ladders

On “how did we already use Bob?”: the notes describe it as a carried import/page-source fixture where `workingAccessUrl` was recorded as origin metadata on `bob/page-main`; generation still follows the local governed working file, not a live remote fetch.

Validation: `git diff --check` passed. I didn’t run Deno tests because these were docs/task-note-only edits.

# User_2026-05-22_2141_58

I'm confused, didn't we create a task for "if possible, legacy Alice Bio-specific (and other fixture ladder) render helpers should be replaced by generalized renderers after the move-only phase?"

# gpt-5.5_2026-05-22_2142_08

I’ll trace the exact wording across the task notes and the nearby renderer-extraction tasks. This sounds like it may have been split across a general refactor note plus one or more focused renderer tasks, so I’ll check before answering from memory.

# gpt-5.5_2026-05-22_2142_19

I found the breadcrumb: the broad refactor note points to mesh-inventory renderer extraction as the next likely slice, and there is a note with that exact title, but it looks like it was only stubbed out. I’ll check whether any completed slice absorbed that work before I give you the clean answer.

# gpt-5.5_2026-05-22_2142_31

Yes, sort of. You’re not misremembering, but the task seems to have been created only as a stub.

The likely intended owner is [wa.task.2026.2026-05-22_2139-core-weave-mesh-inventory-renderer-extraction.md](wa.task.2026.2026-05-22_2139-core-weave-mesh-inventory-renderer-extraction.md:1), titled “Core Weave Mesh Inventory Renderer Extraction”, but it currently has empty sections and just a blank checklist item.

The broader refactor note still carries the real dangling work at [wa.task.2026.2026-05-21_0849_careful-extraction-refactor.md:46](wa.task.2026.2026-05-21_0849_careful-extraction-refactor.md:46) and [line 123](wa.task.2026.2026-05-21_0849_careful-extraction-refactor.md:123): mesh-inventory renderer extraction, then remaining rendered-history/render helpers and legacy fixture-sensitive renderers.

Also, [wa.completed.2026.2026-05-22_2252-payload-render-helpers.md](wa.completed.2026.2026-05-22_2252-payload-render-helpers.md:50) explicitly deferred the mesh-inventory renderers because the generic boundary wasn’t clear yet.

So: yes, 2139 is probably the task, but it needs to be filled in. The clean next move is to flesh out 2139 and change the 0849 checklist item to point at it.