---
id: watask202609081846repositorysourceurlpresentation
title: Render repository-backed working sources as one browse URL
desc: Keep repository-backed payload pages free of a duplicate Working File row and render a GitHub repository locator as one file URL instead of a repository link plus unlinked path fragments.
created: 1788918360000
---

## Parent Plan

None. Small ResourcePage presentation correction found while reviewing the published URPX Semantic Flow staging site.

## Goals

- Confirm repository-backed identifier pages do not display a separate Working File row.
- Render a safe GitHub repository URL and repository-relative path as one browseable `blob/HEAD/<path>` URL.
- Keep unsafe repository URLs unlinked and escaped.

## Summary

The current ResourcePage renderer already suppresses Working File when a floating repository locator is present. Repository Source is rendered as a link to the repository, a visual slash, and a separate unlinked path. For GitHub sources this should be one file URL.

## Discussion

The locator remains structured RDF. This task changes HTML presentation only. A floating locator has no ref, so `HEAD` is the honest GitHub browse coordinate. Exact release identity remains on the linked tag-named HistoricalState, not on the floating working-source row.

## Open Issues

None.

## Decisions

- Recognize canonical HTTP(S) GitHub repository URLs with or without `.git`.
- Strip `.git`, preserve and encode repository path segments, and insert `/blob/HEAD/`.
- Keep the current safe fallback for non-GitHub or unsafe URLs.

## Contract Changes

None to RDF or the Semantic Flow API. ResourcePage HTML presentation changes for GitHub floating repository sources.

## Testing

- GitHub source renders as one anchor whose label and target are the complete browse URL.
- Repository-backed pages omit Working File.
- Unsafe repository URLs remain unlinked.
- Run `deno task test`, `deno task check`, and `deno task lint`.

Receipts, 2026-09-08:

- focused GitHub floating-source renderer test: pass
- focused unsafe repository URL test: pass
- integration page-generation test: pass
- `deno task test`: 942 passed, 0 failed
- `deno task check`: pass
- `deno task lint`: pass

## Non-Goals

- Adding a ref to `RepositorySourceFloatingLocator`.
- Changing source registry serialization.
- Provider-specific browse URLs for non-GitHub hosts.

## Implementation Plan

- [x] Add safe GitHub browse-URL derivation.
- [x] Update repository-source rendering and tests.
- [x] Document working-source presentation.
- [x] Run repository validation.
