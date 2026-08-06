---
id: ojh3juejurrcr8hx4130d3j
title: 2026 05 04 Fingerprint Verification
desc: ''
updated: 1777933342068
created: 1777933304728
---

## Goals

historical states should never change. Record a fingerprint for each file, and invent a new API/CLI surface that verifies files have not changed.

## Summary

## Discussion

## Open Issues

## Decisions

## Contract Changes

## Testing

## Non-Goals

## Implementation Plan

- [ ]

## RULED 2026-08-06 (Dave)

**DEFERRED.** Not cancelled — the gap is real (payload LocatedFiles carry no digest even though `sflo:hasContentDigest` exists and the resolver already hashes every read), but no consumer has asked and both real consumers hold stronger anchors than a mesh-local digest: Git commits and tags, tagged source files, Accord fixtures.

Revive on either trigger: the SFLO runbook wanting a recurring integrity gate instead of manual byte comparison, or Stagecraft committing to historical-payload integrity in its CI.

**Overwrite ruling: `--overwrite-existing-state` updates the digest.** Fingerprinted states are therefore NOT immutable, and the feature must be documented as **current byte-consistency**, never as proof that a state never changed. Any wording implying historical immutability is wrong and must not ship.

Dave's reason is the decisive one, and it is stronger than the packaging argument: **states must be retractable.** PII exposure is the motivating case — a state containing personal data has to be correctable or removable, and a guarantee of historical immutability would make that impossible by design. Immutability was the wrong goal, not merely the harder option.

This lands where [[wa.task.2026.2026-05-17-append-onlyish-inventory]] already pointed: that note lists "privacy or security retraction" among legitimate non-append modes and reserves explicit repair/retraction modes. Retraction is therefore an existing thread, not a new one, and whichever task implements it owns the digest behavior too.

Full proposed surface (`weave validate mesh --integrity payload-history`), findings taxonomy, and ten acceptance tests are in the 2026-08-02 codex analysis, retained for revival.
