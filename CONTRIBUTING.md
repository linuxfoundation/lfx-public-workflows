# Contributing to LFX Shared Public Workflows

## Pinning policy

All third-party GitHub Actions and container images referenced by workflows in
this repository MUST be pinned to an immutable reference, annotated with a
human-readable version.

### GitHub Actions

Pin every `uses:` reference to a full-length (40-character) commit SHA, with a
trailing comment naming the release tag:

```yaml
uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1
```

Do not use moving references: no `@v4`, `@main`, `@master`, or other branch or
tag refs. A tag or branch can be repointed to different code after review; a
commit SHA cannot.

### Container images

Pin container images a workflow runs (for example via `docker run` or
`uses: docker://`) to a `@sha256:` digest, with a trailing comment naming the
version:

```yaml
CLEANUP_IMAGE: ghcr.io/snok/container-retention-policy@sha256:884037f146d6ef574b61fbcc59bef490f887322e6b9dc2eefcf364cb4e9b4112 # 3.1.0
```

Pinning an action by SHA does not pin a container image that action runs
internally; when a workflow invokes such an image directly, pin the image by
digest.

### Calling these reusable workflows

Consumers MUST pin the `uses:` reference to a full commit SHA of this
repository, annotated with the release version:

```yaml
uses: linuxfoundation/lfx-public-workflows/.github/workflows/ghcr-image-cleanup.yaml@<full-commit-sha> # v1.0.0
```

### Why

Immutable pins protect the supply chain: every run executes exactly the
reviewed code. The version comment keeps the pin readable and lets tooling
(for example Dependabot or Renovate) bump the SHA while preserving the
annotation.
