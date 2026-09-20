# Contract — shared CI baked Node 22 (.github#8)

status: FROZEN 2026-09-21 · issue: .github#8 · authority: Henry direct
ARC Node 22 dispatch · RULING 29 delivery route · contract-only bootstrap

## Frozen acceptance criteria

```governance-acs
1. The default Node path for a Node suite on runner `ci-aeros` with input `node-version: 22` selects the baked Node 22 toolchain and performs no `actions/setup-node` download.
2. Other runner labels and non-22 Node versions retain the existing `actions/setup-node` behaviour.
3. The baked path fails closed when the expected runtime is absent or the major version is not 22; it reports the selected executable and exact version.
4. Runtime acceptance records a successful ordinary workflow on a MacBook ARC pod and a separate full Node 22 archive download, with exact run, job, pod and node identity.
5. Rollback restores the unconditional Setup Node step and the prior ci-aeros image tag.
```

## Threat invariants

- A missing baked runtime cannot silently fall back to system Node 20.
- GitHub-hosted, macOS and explicitly selected alternate Node versions retain
  `actions/setup-node`.
- No credential or host mount is added.

## Self-application

This contract lands alone. The implementation pull request freezes against
this file after it reaches the protected default branch.
