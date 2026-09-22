---
okf_version: "0.2"
type: knowledge-bundle
title: GHCR Image Cleanup Reusable Workflow
description: Shared GitHub Actions workflow that prunes stale GHCR container-image versions for LFX repos.
tags:
  - github-actions
  - ghcr
  - ci
  - cleanup
status: stable
---

# GHCR Image Cleanup Reusable Workflow

A shared, reusable GitHub Actions workflow in `linuxfoundation/lfx-public-workflows` that
prunes stale GitHub Container Registry (GHCR) image versions for a single package.
Consumer repositories adopt it with a thin caller workflow instead of duplicating the
cleanup logic.

- Workflow: [`.github/workflows/ghcr-image-cleanup.yaml`](../../.github/workflows/ghcr-image-cleanup.yaml)

## Contents

- [overview.md](./overview.md) — what it does, when it runs, and the security model.
- [inputs.md](./inputs.md) — inputs, caller permissions, outputs, and behavioral guarantees.
- [usage.md](./usage.md) — copy-ready, SHA-pinned caller examples.
