# Known false positives — the floor, applied last

Claims a reviewer has raised on this repository that are **wrong**, each
with the maintainer's rebuttal. Apply this file **after** everything else:
if a candidate finding matches an entry here, drop it silently.

---

## 1. The Chart.lock `yq` filter should be split for readability

**The claim:** the `yq` one-liner that selects non-OCI Helm repositories
in `.github/actions/helm-chart-oci-publisher/action.yml` is too complex
and should be broken into multiple steps.

**Why it is wrong:** the filter is the entire rule — "dependencies whose
repository does not contain `oci://`". Splitting it does not change
behavior and was refused on the PR that introduced it.

**Threads:** PR #7. Copilot asked to split the filter; a maintainer
replied `disagree`.

---

## 2. Markdown files must have a license header

**The claim:** `README.md`, `SECURITY.md`, `OWNERS.md`, `AGENTS.md` or
other documentation is missing the MIT SPDX header used on YAML.

**Why it is wrong:** markdown files in this repo do not carry a license
header. Documentation is licensed CC-BY-4.0 via `LICENSE-docs`. The
written rule is in `AGENTS.md` ("Markdown files in this repo do not carry
a license header").
