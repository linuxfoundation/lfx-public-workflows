# Agent docs versus the tree

`AGENTS.md` is the rulebook future agents follow. When it describes the
repository as something the tree is not, later local reviews will *quote
the stale claim* instead of catching it.

---

## `docs/agents-every-workflow-is-reusable` — Important

**Pattern:** `AGENTS.md` says every workflow here is reusable
(`on: workflow_call`) or every file is a consumer-facing workflow or
composite action, while `.github/workflows/` contains a file triggered by
something else (`pull_request`, `push`, `schedule`, `workflow_dispatch`).

**Detect:** when the patch changes `AGENTS.md` or any
`.github/workflows/*.yml`, classify each workflow at `target_sha` as
reusable (`workflow_call`) or repo CI (any other `on` / `"on":` trigger).
Flag if Overview or Repository layout still claims every workflow is
`workflow_call` (phrases such as "Everything here is a reusable workflow"
or "each triggered by `on: workflow_call`"), or if a repo-CI workflow
exists that those sections do not name as this repository's CI.

**Why it matters:** agents treat `AGENTS.md` as ground truth. A stale
overview hides this repo's own CI and sends later edits toward
`workflow_call` contracts that do not apply.

**Evidence:** PR #21. Copilot: "This overview is already false after this
PR adds `.github/workflows/mega-linter.yml`, which is triggered by
`pull_request` and is neither reusable nor a composite action. Since this
file guides future agents, distinguish repository CI from the reusable
assets instead of claiming every file is consumer-facing." Fixed in
`481558e` by naming MegaLinter as this repository's CI in Overview and
Repository layout.

**Not a finding when:** Overview and Repository layout name each
non-`workflow_call` workflow as this repository's CI, and no remaining
sentence claims every workflow is reusable.

---

## `docs/agents-blanket-mit-header` — Important

**Pattern:** `AGENTS.md` tells agents that every workflow, composite
action, and script starts with the two-line LFX MIT header, while a
tracked YAML file under `.github/` uses a different
`SPDX-License-Identifier` (or carries upstream copyright that the MIT
header would replace).

**Detect:** when the patch changes `AGENTS.md` or any `.github/**/*.yml`,
collect `SPDX-License-Identifier` values at `target_sha`. Flag if License
headers still states a blanket MIT rule ("Every workflow, composite
action, and script file starts with" the MIT SPDX block) without naming
each non-MIT file as an exception. Also flag a hunk that rewrites
`.github/actions/helm-chart-oci-publisher/action.yml` to MIT or drops the
upstream Heimdall copyright.

**Why it matters:** a blanket MIT instruction will relicense the
vendor-derived Helm publisher on the next agent pass and drop required
upstream attribution.

**Evidence:** PR #21. Copilot: "This blanket MIT-header rule contradicts
`.github/actions/helm-chart-oci-publisher/action.yml:1-4`, which retains
upstream copyright and an `Apache-2.0` SPDX identifier. Qualify the rule
by license (or document that action as an exception) so an agent does not
incorrectly relicense or remove attribution from the existing composite
action." Fixed in `481558e` by scoping MIT to new LFX-owned YAML and
naming the Apache-2.0 exception.

**Not a finding when:** License headers names each non-MIT YAML file and
its SPDX identifier, and does not instruct relicensing it.
