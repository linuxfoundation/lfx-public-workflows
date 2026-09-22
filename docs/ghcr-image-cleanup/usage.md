---
type: how-to
title: Usage and Caller Examples
description: Copy-ready, SHA-pinned caller workflows for the GHCR image cleanup workflow.
tags:
  - github-actions
  - how-to
status: stable
---

# Usage

Add a small caller workflow to your repository. Pin the `uses:` reference to a full
commit SHA of `lfx-public-workflows` (annotate it with the release version in a comment). Do not
use `@main` or a moving tag.

Find a SHA to pin from the
[lfx-public-workflows releases](https://github.com/linuxfoundation/lfx-public-workflows/releases).

## Example: simple package (no deploy previews)

Mirrors the `lfx-v2-campaign-service` case.

```yaml
# Copyright The Linux Foundation and each contributor to LFX.
# SPDX-License-Identifier: MIT
---
name: "Cleanup Stale Container Images"
"on":
  schedule:
    - cron: "0 0 * * 0"   # Sundays 00:00 UTC
  workflow_dispatch:
    inputs:
      dry-run:
        description: "Preview candidate deletions without deleting anything."
        type: boolean
        required: false
        default: true
      cut-off:
        description: "Minimum age before a version is eligible for deletion."
        type: string
        required: false
        default: "30d"
permissions:
  contents: read
jobs:
  cleanup:
    permissions:
      contents: read
      packages: write
    uses: linuxfoundation/lfx-public-workflows/.github/workflows/ghcr-image-cleanup.yaml@<full-commit-sha> # v1.0.0
    with:
      image-name: lfx-v2-campaign-service/campaign-service
      image-tags: "!v* !latest !development"
      cut-off: ${{ github.event.inputs.cut-off || '30d' }}
      dry-run: ${{ fromJSON(github.event.inputs['dry-run'] || 'true') }}
```

## Example: with deploy-preview protection

Mirrors the `lfx-self-serve` case. Note the extra `pull-requests: read` permission and the
explicit `image-tags` override (self-serve protects two-segment `!*.*` tags).

```yaml
# Copyright The Linux Foundation and each contributor to LFX.
# SPDX-License-Identifier: MIT
---
name: "Cleanup Stale Container Images"
"on":
  schedule:
    - cron: "0 0 * * 0"   # Sundays 00:00 UTC
  workflow_dispatch:
    inputs:
      dry-run:
        description: "Preview candidate deletions without deleting anything."
        type: boolean
        required: false
        default: true
permissions:
  contents: read
jobs:
  cleanup:
    permissions:
      contents: read
      packages: write
      pull-requests: read
    uses: linuxfoundation/lfx-public-workflows/.github/workflows/ghcr-image-cleanup.yaml@<full-commit-sha> # v1.0.0
    with:
      image-name: lfx-self-serve
      image-tags: "!development !latest !*.*.* !*.*"
      enable-preview-protection: true
      dry-run: ${{ fromJSON(github.event.inputs['dry-run'] || 'true') }}
```

## Notes

- Scheduled runs delete; manual runs default to a dry-run preview. Review the job summary
  from a dry-run before relying on a scheduled delete.
- `image-name` must be one exact package. The workflow fails fast on wildcards or lists.
- See [inputs.md](./inputs.md) for the full input, permission, and output contract.
