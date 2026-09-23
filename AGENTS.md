# AGENTS.md

Guidance for AI coding agents (Claude Code, and any agent that reads
`CLAUDE.md`, which is a symlink to this file) working in this repository.

## Overview

**LFX Shared Public Workflows** (`linuxfoundation/lfx-public-workflows`) is the
public home for reusable GitHub Actions workflows shared across projects in the
`linuxfoundation/` GitHub org. Because it is public, its workflows can be
referenced by both public and private consumer repositories.

Everything here is a reusable workflow (`on: workflow_call`) or a composite
action, consumed by other repositories, not an application. There is no build
or runtime; changes are validated by static checks and by exercising the
workflow from a consumer repository.

## Repository layout

Reusable workflows live in `.github/workflows/` (each triggered by
`on: workflow_call`), and composite actions in `.github/actions/`. Repository
owners are listed in `OWNERS.md`, with `.github/CODEOWNERS` driving automatic
review requests. The security policy is in `SECURITY.md`, and licenses in
`LICENSE` (MIT, source) and `LICENSE-docs` (CC-BY-4.0, documentation). The
pinning policy is documented in the Conventions section below.

## Conventions

### Pinning policy (required)

Third-party GitHub Actions and container images MUST be pinned to an immutable
reference with a trailing version comment. See
[`CONTRIBUTING.md`](CONTRIBUTING.md) for the full policy, introduced in
[PR #18](https://github.com/linuxfoundation/lfx-public-workflows/pull/18).

- Actions: full 40-character commit SHA, e.g.
  `uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1`
- Container images: `@sha256:` digest, e.g.
  `ghcr.io/snok/container-retention-policy@sha256:884037f...4e9b4112 # 3.1.0`
- Consumers pin these reusable workflows by full commit SHA.
- No moving refs (`@v4`, `@main`, `@master`, branch or tag names).

### License headers

Every workflow and script file starts with:

```yaml
# Copyright The Linux Foundation and each contributor to LFX.
# SPDX-License-Identifier: MIT
```

Markdown files in this repo do not carry a license header (match the existing
`README.md`, `SECURITY.md`, `OWNERS.md`).

### Reusable workflow design

- Prefer `on: workflow_call` with typed `inputs` and `outputs`.
- Request least-privilege `permissions`; a scope requested by a called
  workflow cannot exceed the caller's grant, so gate optional scopes behind a
  conditional job rather than requesting them unconditionally.
- Bind inputs and context values through step-level `env:`; do not interpolate
  `${{ ... }}` inside `run:` scripts (script-injection prevention).

## Git workflow

### DCO signoff and signed commits are required

All commits to this repository MUST be both DCO signed-off and cryptographically
signed. Always commit with:

```bash
git commit -S --signoff
```

- `--signoff` adds the `Signed-off-by:` trailer required by the DCO check.
- `-S` produces a GPG/SSH-signed commit so it shows as **Verified** on GitHub.

The `DCO` status check will fail without the `Signed-off-by:` trailer. Do not
add the trailer by hand; let `--signoff` generate it from your Git identity.

### Commit messages

Use Conventional Commits:

```text
<type>(<scope>): <subject>

<body>
```

Types: `feat` · `fix` · `docs` · `chore` · `refactor` · `test` · `style`.
Reference the PR or issue in the body when applicable.

### Pull requests

- Reviewers are requested automatically via [`.github/CODEOWNERS`](.github/CODEOWNERS);
  `OWNERS.md` remains the human-readable list of repository owners.
- Add the `ai-assisted` label when an AI agent helped author or review the
  change.

## Validation

There is no build. Validate workflow changes with:

```bash
actionlint .github/workflows/<file>.yml   # workflow + embedded shell (shellcheck)
yamllint -d relaxed .github/workflows/<file>.yml
zizmor .github/workflows/<file>.yml       # GitHub Actions static/security analysis
```

For a full check, exercise the workflow from a consumer repository (for example
a manual `workflow_dispatch` in dry-run mode) before relying on a scheduled run.

## Security

Report vulnerabilities per [`SECURITY.md`](SECURITY.md). Do not open a public
issue for a security report.
