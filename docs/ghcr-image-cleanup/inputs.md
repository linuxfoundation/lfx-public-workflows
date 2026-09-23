---
type: reference
title: Inputs, Permissions, and Outputs
description: The workflow_call interface for the GHCR image cleanup workflow.
tags:
  - github-actions
  - reference
status: stable
---

# Inputs, Permissions, and Outputs

## Inputs

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `image-name` | string | yes | — | Exactly one GHCR package name (e.g. `lfx-self-serve`, `lfx-v2-campaign-service/campaign-service`). Wildcards, lists, whitespace, and empty values are rejected. |
| `account` | string | no | `linuxfoundation` | GHCR account that holds the package. Organization name, or the literal value `user` for a personal account. Do not pass a GitHub username; any other value is treated as an organization. |
| `cut-off` | string | no | `30d` | Minimum age before a version is eligible for deletion (e.g. `30d`, `4w 2d`). |
| `image-tags` | string | no | `!latest !development !v* !*.*.*` | Space-separated protected-tag patterns (negative patterns protect). Override for repo-specific protection. |
| `dry-run` | boolean | no | `true` | Preview only. Scheduled runs always delete regardless; other triggers honor this. |
| `enable-preview-protection` | boolean | no | `false` | Protect deploy-preview images tied to open PRs. Requires `pull-requests: read`. |
| `preview-label` | string | no | `deploy-preview` | PR label identifying preview PRs. Used only when preview protection is on. |
| `preview-tag-prefix` | string | no | `ui-pr-` | Prefix for per-PR preview tags (`ui-pr-123`). Used only when preview protection is on. |

The default `image-tags` is a superset baseline. `lfx-self-serve` overrides it (it also
protects two-segment `!*.*` tags), so the default alone does not reproduce self-serve
protection. Set `image-tags` explicitly to match a specific repo's tag scheme.

`account` is an organization name, not a GitHub username. Personal packages must use
the literal value `user`; any other value is sent to the organization packages API.

## Caller permissions

Declare these on the calling job. The workflow itself declares no `secrets:` block.

```yaml
permissions:
  contents: read
  packages: write
  # pull-requests: read   # ONLY when enable-preview-protection: true
```

Do not use `secrets: inherit`. No secret is passed to this workflow.

### Package access prerequisite

`packages: write` is necessary but not sufficient. The caller repository's
`GITHUB_TOKEN` can delete versions only when the repository has admin access to
the target package: either the package is scoped to (linked to) the caller
repository, or the package's access settings grant that repository the `Admin`
role. Without this, cleanup fails with an authorization error despite
`packages: write`. Package access is managed on the package's settings page under
"Manage Actions access".

## Outputs

| Name | Description |
|------|-------------|
| `deleted-count` | Unique versions deleted (0 in dry-run). |
| `would-delete-count` | Unique versions that would be deleted (dry-run only). |
| `failed-count` | Failed deletions; the run exits non-zero when greater than 0. |

The human-readable summary is always written to the job's step summary.

## Behavioral guarantees

- **G1**: Deletes only versions older than `cut-off` that carry no protected tag.
- **G2**: Protects `latest`, `development`, `v*`, and `*.*.*` tags by default; overridable.
- **G3**: With `enable-preview-protection: true`, protects `ui-pr-<PR#>` versions for open
  PRs labeled `preview-label`; makes no PR API call when false.
- **G4**: Scoped to exactly one exact-match package; never wildcards.
- **G5**: Writes a run summary (trigger, dry-run, cut-off, selected, protected multi-arch
  children, would-delete/deleted, failed) to the step summary.
- **G6**: Exits non-zero when any deletion fails.
- **G7**: Manual runs default to a dry-run preview; scheduled runs delete.
