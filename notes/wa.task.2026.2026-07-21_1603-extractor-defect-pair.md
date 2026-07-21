---
id: watask20260721extractordefects
title: 'Extractor defect pair — all-terms file-URL crash + missing hasResourcePage claims'
desc: 'Fix the two Weave 0.3.0 extraction defects surfaced by the srd-rules big-mesh publication run (2026-07-21): (a) extract --all-terms enumerates checked-in file URLs as terms then dies (Not a directory); (b) extracted-term Knops get no sflo:hasResourcePage claims, forcing consumers to patch inventories. Boarded with a wake trigger that fired at the version-API lane; cut 2026-07-21 16:03 by the stagecraft flagship seat under the we-own-the-substrate law.'
---

## Goals

Make `weave extract` correct on big real meshes so downstream pipelines stop working around it: `--all-terms` must enumerate only genuine terms (never checked-in file URLs), and extracted-term Knops must carry their governed `sflo:hasResourcePage` claims without consumer-side patching.

## Summary

The srd-rules publication run (the first big-mesh consumer, ~6,900 files) surfaced both defects with exact evidence:

- **(a) all-terms filtering:** `weave extract --all-terms` includes checked-in file URLs such as `conditions.csvw.json` in the term set, then attempts `conditions.csvw.json/_knop/...` and refuses with `Not a directory`. The pipeline works around via supported single-term extraction over a governed non-file census.
- **(b) page-claim emission:** extracted term Knops receive no `sflo:hasResourcePage` claims, so the consumer's `prepare-srd-publication.mjs` deterministically adds the missing governed claims to Weave-created inventories before `weave generate` renders HTML.

Both workarounds live in the stagecraft repo's `tools/srd-extraction/` and are cited in the srd-rules cutover task's phase-A receipts (stagecraft-lab note `flag.task.2026-07-21_1116-srd-rules-package-repo`, "Named upstream candidate" entries).

## Decisions

- Fix BOTH defects in one lane off weave `main` (`b7cbf89`); they share the extraction surface and one consumer proof.
- The whole-mesh `weave validate` 4 GiB heap exhaustion and any parse-once restructuring are OUT (they stay on wd.todo — architecture work, not a defect lane).
- Consumer-side workaround removal (the stagecraft pipeline) is OUT — boarded stagecraft-side, fires after this lands.
- Branch `fix/extractor-defect-pair`; path-scoped commits; no push until the runner lands it per weave convention.

## Contract Changes

None to the public API/CLI surfaces beyond corrected behavior: `--all-terms` output narrows to genuine terms (the file-URL entries were never valid); extracted inventories gain claims they were always supposed to carry. If either correction requires a documented behavior-contract edit (wd notes), it rides the same lane.

## Testing

- Fail-on-old repro per defect: (a) a fixture mesh containing a checked-in file URL among term candidates — old code dies `Not a directory`, new code enumerates cleanly without the file URL; (b) an extraction fixture asserting the extracted Knop inventory carries the governed `sflo:hasResourcePage` claims — red on old.
- Focused suites for the touched extraction families, then one full `deno task ci` green at lane end.

## Non-Goals

Heap/scale work, parse-once, whole-mesh validation green, npm/release work, consumer pipeline edits.

## Implementation Plan

1. Locate the term-enumeration path for `--all-terms`; add the genuine-term filter at the enumeration source (not a post-filter on the crash site); fail-on-old.
2. Locate extracted-Knop inventory emission; emit the governed `sflo:hasResourcePage` claims where the mesh's page-generation contract expects them (mirror what `weave generate`/validate consider governed); fail-on-old.
3. Full ci; receipts to this note.

## Spec review

*(appended by a fresh review seat with PROPOSED verdicts/severities; execution blocked until the stagecraft PM's sign-off)*
