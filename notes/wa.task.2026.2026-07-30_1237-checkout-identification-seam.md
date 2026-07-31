---
id: watask20260730checkoutseam
title: 'Capability-injected checkout-identification seam (CLI reroute + lib parity for floating sources)'
desc: 'Task sketch (parked, future): make repository-source checkout identification a single injected capability seam — the CLI plugs in the git-backed implementation, the lib accepts a caller-supplied one — so validate/version share one code path and floating-locator meshes become lib-viable. Captured 2026-07-30 at PM direction during the validateMesh F1 ruling (typed refusal chosen for v1).'
---

## Status

**Sketch only — parked.** Cut 2026-07-30 by Jimbo at PM direction: the v1 `validateMesh` F1 ruling chose typed refusal for floating-repository sources ([[wa.task.2026.2026-07-29_1219-programmatic-validate-mesh-api]] ruling record), and the PM required the fullest version of the necessary/convenience split be captured as its own task. Activate when a Node/lib consumer actually needs floating-source validation, or when CLI/lib code-path unification pays for itself.

## Goals

- One code path for repository-source floating-locator resolution across CLI and lib, with the capability injected rather than environment-detected: the CLI supplies a git-backed checkout identifier; lib consumers may supply their own; absence yields the same typed refusal the v1 API ships.
- Eliminate the environment-dependent degradation that motivated F1: today the Node build feature-detects `Deno.Command` and silently degrades to "no checkout", surfacing as a misleading missing-working-payload error.

## Design sketch

The entire git surface on the validate/version read path is checkout *identification*, not content access (`src/runtime/operational/repository_source.ts` → lazy `repository_source_git.ts`): (1) resolve the repository root enclosing a path; (2) enumerate remotes/URLs to match the declared repository identity; then ordinary file reads from that checkout. So the seam is small:

```ts
export interface CheckoutIdentifier {
  identifyCheckout(path: string): Promise<
    { repositoryRoot: string; remoteUrls: readonly string[] } | undefined
  >;
}
```

- `repository_source.ts` consumes an injected `CheckoutIdentifier` instead of dynamically importing the git implementation itself.
- CLI wiring constructs the git-backed implementation (current `tryRunGit` logic, unchanged behavior) and passes it down; compiled binaries keep `--allow-run=git`.
- API requests (`validateMesh`, potentially `versionPayloads` later) accept an optional `checkoutIdentifier`; when absent, floating-locator candidates refuse with the same stable code as v1 — behavior is capability-determined, never environment-guessed.
- The fs-purity guard keeps holding: the git implementation lives behind CLI wiring, off the `src/api` graph; the seam interface itself is subprocess-free.

## Contract Changes (when activated)

- Additive optional request field on the API surface(s); the v1 typed refusal remains the absent-capability behavior, so this is strictly additive.
- CLI behavior unchanged by contract; internally rerouted through the seam (regression-pinned).

## Testing (when activated)

- Same-findings parity: a floating-source mesh validated via CLI (git-backed) vs lib with a caller-supplied identifier returning the same answers.
- Absent-capability refusal parity with v1 behavior; fs-purity guard; off-tree Node smoke with a stub identifier.

## Non-Goals

- Content fetch from refs (remote or local) — the seam identifies checkouts only; `workingAccessUrl`-style remote resolution stays with its own parked task.
- Changing v1 `validateMesh` (ships typed refusal first, per the F1 ruling).

## Open Issues

- Does `versionPayloads` adopt the seam in the same slice or keep its up-front refusal?
- Exact refusal code sharing between API surfaces (one `unsupported-source`-family meaning).
- Whether the seam interface belongs in `src/api` types (public) or stays a runtime-internal injection until a consumer asks.
