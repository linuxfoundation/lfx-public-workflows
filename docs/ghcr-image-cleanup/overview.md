---
type: concept
title: Overview and Security Model
description: What the GHCR image cleanup workflow does, when it runs, and how it is kept safe as a public shared workflow.
tags:
  - github-actions
  - security
status: stable
---

# Overview

The GHCR image cleanup workflow deletes stale container-image versions from a single
GHCR package so registries do not accumulate unbounded old builds. It wraps the
digest-pinned `snok/container-retention-policy` container and adds LFX defaults, a run
summary, a dry-run policy, and optional deploy-preview protection.

It is a reusable workflow (`on: workflow_call`). It has no schedule of its own; each
consumer repository owns its `schedule` and `workflow_dispatch` triggers in a small
caller workflow and delegates the work here.

## What it does

- Selects tagged and untagged versions of one exact-match package older than a cut-off.
- Protects release tags, `latest`, and the default-branch build by default.
- Optionally protects deploy-preview images tied to still-open pull requests.
- Writes a run summary (selected, protected, would-delete/deleted, failed) to the job
  summary, and exits non-zero if any deletion fails.

## When it runs

- **Scheduled** runs delete (dry-run is forced off).
- **Manual** (`workflow_dispatch`) and other triggers default to a dry-run preview so a
  maintainer can inspect candidates before anything is deleted.

## Security model

This workflow is public and callable by any repository. It runs in the caller's context
with the caller's token, so calling it never exposes LFX secrets. The following controls
keep it safe:

- **No `secrets:` block.** It uses the automatically provisioned `GITHUB_TOKEN` via
  `secrets.GITHUB_TOKEN`, scoped by the permissions the caller declares. Callers must not
  use `secrets: inherit`.
- **Least privilege.** The caller grants `packages: write` (to delete versions) and adds
  `pull-requests: read` only when preview protection is enabled.
- **No script injection.** Every input and context value is bound through step-level
  `env:` and referenced as a quoted shell variable; no `${{ }}` interpolation appears
  inside any `run:` block.
- **Single exact-match scope.** The package name must be one exact value. Wildcards,
  lists, whitespace, and empty values are rejected before any deletion runs, so a
  misconfigured value cannot delete unrelated packages.
- **Pinned supply chain.** The cleanup container is pinned by digest, and consumers pin
  this workflow by full commit SHA (see [usage.md](./usage.md)).
