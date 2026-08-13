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

## `control-plane-integrity.yml` — protects the gates themselves

Call it alongside `ci.yml`:

```yaml
jobs:
  ci:
    uses: aerodeck-ai/.github/.github/workflows/ci.yml@main
  control-plane-integrity:
    uses: aerodeck-ai/.github/.github/workflows/control-plane-integrity.yml@main
```

### Why

`aerodeck-template` PR #1936 merged via `berlai-services[bot]` with **zero human reviews**
(`gh api pulls/1936/reviews` -> length 0, confirmed live 2026-08-13) while touching a gate checker
script. It happened to be caught by a different lane (the file was independently design-locked) —
nothing generic protects `.github/workflows/**`, gate manifests, CODEOWNERS, or gate checker
scripts across the other 65 repos. This gate closes that class:

- **NEGATIVE-WORKFLOW** — a PR adds a brand-new file inside a protected path (e.g. a new workflow
  that grants itself extra permissions). Checked regardless of git status (added/modified/removed),
  not just already-known files.
- **NEGATIVE-RENAME** — a PR renames a protected file out from under its own name to dodge a
  path-string match, or renames a file INTO a protected path. Both `filename` and
  `previous_filename` are checked on a GitHub-reported rename.

A PR touching a protected path is blocked until a control-plane owner
(`henryberliand-design` / `stillmusic-tech`) approves it, or the `founder-session-approval` label +
structured session comment lane is used (same mechanism as `aerodeck-template`'s
`design-lock-gate.yml`, Henry ruling 2026-08-06).

### Extending coverage

Protected by default: `.github/workflows/**`, `CODEOWNERS`, `design-locks.json`,
`canonical-routes.json`, `scripts/check-*.mjs`, `control-plane-owners.json` itself. A repo with its
own runner/deploy/gate config to protect drops a `control-plane-owners.json` at its root:

```json
{
  "owners": ["henryberliand-design", "stillmusic-tech"],
  "protectedGlobs": ["deploy/**", ".github/runner-config.yml"]
}
```

Read from the PR's **base branch**, never the PR's own copy — a PR cannot widen its own escape
hatch in the same change it's making.

## Runners

Three runners are registered **org-level**, so any repo can use them without per-repo registration:

| runner | host |
|---|---|
| `aeros-runner-1` | aeros (systemd service, work dir on `/mnt/data`) |
| `mac-mini-runner-1` | mac mini |
| `personal-runner-1` | personal |

<!-- push-auth verified via gh credential helper 2026-07-30 -->
