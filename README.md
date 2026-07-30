# `.github` — org-shared workflows for `aerodeck-ai`

Central home for reusable GitHub Actions workflows. One definition, called by every repo, so a fix
lands everywhere at once.

## Why

Measured 2026-07-30 across all 66 repos in the org:

| | count |
|---|---|
| repos in `aerodeck-ai` | 66 |
| with any workflow | 22 |
| using a shared/reusable workflow | 0 |

`aerodeck-template` alone carried 29 workflow files; `hermes-agent` 22. Every repo hand-rolled its
own CI, so nothing was ever fixed once.

## `ci.yml` — the shared gate

Call it from any repo:

```yaml
name: CI
on: [push, pull_request]
jobs:
  ci:
    uses: aerodeck-ai/.github/.github/workflows/ci.yml@main
```

### It will not report a vacuous green

The rejected alternative was a `test-if-present` gate everywhere: it reports green when nothing
ran, so a repo with no tests looks exactly like a repo whose suite passed. That is the fail-open
pattern behind this estate's recurring "green check that can't see failure" defects.

Instead, when no suite is detected the job reports **UNTESTED** and fails — unless the caller sets
`declared-untested: true`, which records the repo as knowingly unproven and succeeds. Untested is a
visible state, never a silent pass.

```yaml
jobs:
  ci:
    uses: aerodeck-ai/.github/.github/workflows/ci.yml@main
    with:
      declared-untested: true   # no suite, knowingly unproven
```

### Inputs

| input | default | purpose |
|---|---|---|
| `runner` | `self-hosted` | Org runner pool. Pass `ubuntu-latest` to use GitHub-hosted. |
| `node-version` | `22` | Node version when a node suite is detected. |
| `test-command` | *(auto)* | Override detection. Set only when detection is wrong. |
| `declared-untested` | `false` | Record a suite-less repo as knowingly unproven. |

## Runners

Three runners are registered **org-level**, so any repo can use them without per-repo registration:

| runner | host |
|---|---|
| `aeros-runner-1` | aeros (systemd service, work dir on `/mnt/data`) |
| `mac-mini-runner-1` | mac mini |
| `personal-runner-1` | personal |

<!-- push-auth verified via gh credential helper 2026-07-30 -->
