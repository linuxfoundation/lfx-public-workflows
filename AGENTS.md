# AGENTS.md

Guidance for AI coding agents (Claude Code, and any agent that reads
`CLAUDE.md`, which is a symlink to this file) working in this repository.

## Overview

**LFX Shared Public Workflows** (`linuxfoundation/lfx-public-workflows`) is the
public home for reusable GitHub Actions workflows shared across projects in the
`linuxfoundation/` GitHub org. Because it is public, its workflows can be
referenced by both public and private consumer repositories.

This repo publishes reusable workflows (`on: workflow_call`) and composite
actions for other repositories to consume. It is not an application. There is
no build or runtime. The same tree also holds this repository's own CI:
`.github/workflows/mega-linter.yml` runs on `pull_request` and is not a
reusable workflow. Changes are validated by those static checks and by
exercising a reusable workflow from a consumer repository.

## Repository layout

Reusable workflows live in `.github/workflows/` and are triggered by
`on: workflow_call`. The exception is `.github/workflows/mega-linter.yml`,
which is this repository's CI (`on: pull_request`). Composite actions live in
`.github/actions/`. Repository owners are listed in `OWNERS.md`, with
`.github/CODEOWNERS` driving automatic review requests. The security policy is
in `SECURITY.md`, and licenses in `LICENSE` (MIT, source) and `LICENSE-docs`
(CC-BY-4.0, documentation). The pinning policy is documented in the Conventions
section below.

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

New LFX-owned workflow, composite action, and script files start with:

```yaml
# Copyright The Linux Foundation and each contributor to LFX.
# SPDX-License-Identifier: MIT
```

Do not apply that MIT header to vendor-derived files that already carry a
different license. `.github/actions/helm-chart-oci-publisher/action.yml` keeps
the upstream Heimdall copyright, the LFX copyright, and
`SPDX-License-Identifier: Apache-2.0`.

Markdown files in this repo do not carry a license header (match the existing
`README.md`, `SECURITY.md`, `OWNERS.md`).

### Reusable workflow design

- Prefer `on: workflow_call` with typed `inputs` and `outputs`.
- Request least-privilege `permissions`; a scope requested by a called
  workflow cannot exceed the caller's grant, so gate optional scopes behind a
  conditional job rather than requesting them unconditionally.
- Bind inputs and context values through step-level `env:`; do not interpolate
  `${{ ... }}` inside `run:` scripts (script-injection prevention).
- `actions/checkout` sets `persist-credentials: false` unless the job
  genuinely needs the persisted token.

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

There is no build. Pull requests run MegaLinter (documentation flavor) with
actionlint, zizmor, yamllint, markdownlint, and cspell. See
`.github/workflows/mega-linter.yml` and `.mega-linter.yml`.

MegaLinter activation is an explicit `ENABLE_LINTERS` allowlist that matches
that advertised set. Do not use `ENABLE` with a descriptor (`REPOSITORY`,
`SPELL`, `YAML`, `MARKDOWN`, `ACTION`): every linter in the descriptor would
run, including ones this repo does not advertise (for example
`REPOSITORY_SEMGREP` and `REPOSITORY_LS_LINT`). Semgrep also defaults to
fetching its `auto` ruleset.

`ACTION_DIRECTORY` is `.github` so zizmor covers reusable workflows and
composite actions. Leave actionlint on `.github/workflows/` (it is a
workflow linter).

Locally:

```bash
actionlint .github/workflows/<file>.yml
yamllint -c .yamllint.yml .github .mega-linter.yml
markdownlint -c .markdownlint.yml README.md OWNERS.md SECURITY.md
zizmor .github
```

For a full check, exercise the workflow from a consumer repository (for example
a manual `workflow_dispatch` in dry-run mode) before relying on a scheduled run.

## Security

Report vulnerabilities per [`SECURITY.md`](SECURITY.md). Do not open a public
issue for a security report.
