# Reusable workflow contract

Patterns that fire when a `workflow_call` file advertises a contract callers
cannot actually use.

---

## `contract/workflow-call-output-undeclared` — Critical

**Pattern:** a job sets `jobs.<id>.outputs.<name>` (or the README promises a
caller-visible output) without a matching
`on.workflow_call.outputs.<name>` mapping.

**Detect:** in a changed `.github/workflows/*.yml` that has
`on: workflow_call`, every `jobs.*.outputs.<name>` that callers are told to
read — README, PR body, or another job in a *different* workflow via
`needs.<caller-job>.outputs` — must have
`on.workflow_call.outputs.<name>.value: ${{ jobs.<id>.outputs.<name> }}`.
Flag a new or renamed job output with no `workflow_call.outputs` entry.

**Why it matters:** GitHub drops undeclared reusable-workflow outputs. The
caller sees an empty string, so a documented skip-apply-when-no-changes
path never fires.

**Evidence:** PR #14. Copilot:
"This only creates an output on the reusable workflow's internal
`opentofu_plan` job. Callers … cannot access it … unless
`on.workflow_call.outputs.changes` is also declared."
Fixed in `1a965aa` by adding the `workflow_call.outputs` mapping on both
`lfx-opentofu-plan.yml` and `lfx-opentofu-plan-apply.yml`.

**Not a finding when:** the output is only consumed by another job *inside
the same* reusable workflow (`needs:` within the file). Those are job
outputs, not the caller contract.

---

## `contract/declared-input-unconsumed` — Critical

**Pattern:** `on.workflow_call.inputs.<name>` (or `secrets.<name>`) is
declared and documented, but no step reads `${{ inputs.<name> }}` /
`${{ secrets.<name> }}`.

**Detect:** for each input or secret whose declaration is added or modified
in the diff of a `workflow_call` file, grep the same file for
`inputs.<name>` or `secrets.<name>` outside the `on:` block. Flag a name
that appears only in the declaration (and possibly README). Also flag a
hunk that removes the last valid consumer of an existing declared input
while leaving the declaration in place. Do not flag an input whose
declaration and consumers are both unchanged.

**Why it matters:** callers pass a value that is silently ignored. PR #13
shipped `env` on plan/apply/plan-apply that only `check` actually wrote to
`$GITHUB_ENV`.

**Evidence:** PR #13. Copilot: "The `env` input is declared … but this Set
Environment Variables step only writes `secrets.env_secret` … never
processes `inputs.env`." Fixed on the merged PR by wiring `inputs.env` into
the set-env steps.

**Not a finding when:** the input is consumed through a composite expansion
that the detect grep still sees (`inputs.<name>` in `with:` or `env:`).

---

## `contract/input-name-mismatch` — Critical

**Pattern:** README (or other caller docs) advertises an input name that is
not declared under `on.workflow_call.inputs`, or a documented name that
does not match the name the workflow actually consumes.

**Detect:** when the patch changes `README.md` or a `workflow_call` file,
every advertised caller-facing input name in README must match a key in
that file's `on.workflow_call.inputs`. Flag a documented name such as
`pre_run_commands` when the workflow declares `pre_run_script`. Do **not**
flag `${{ inputs.<wrong> }}` inside workflow YAML: actionlint already
type-checks undeclared `inputs.*` on `workflow_call` files.

**Why it matters:** GitHub evaluates a missing input to empty. Callers who
copy the README name never set the real input. PR #13's `TERRAFORM_PRE_RUN`
path was dead because docs and YAML used different names; actionlint now
catches the YAML side, not the README side.

**Evidence:** PR #13. Copilot, on plan/apply/check/plan-apply: "The
`TERRAFORM_PRE_RUN` env var references `inputs.pre_run_commands`, but the
input declared … is `pre_run_script`." PR #21. Copilot: this pattern
duplicates actionlint on workflow YAML; keep it for advertised names
actionlint does not see.

**Not a finding when:** the mismatch is only inside the workflow YAML
(`${{ inputs.* }}` vs `on.workflow_call.inputs`). CI owns that.
