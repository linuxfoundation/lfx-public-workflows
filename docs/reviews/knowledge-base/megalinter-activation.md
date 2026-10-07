# MegaLinter activation versus advertised CI

README.md and AGENTS.md advertise a small CI set. MegaLinter's `ENABLE`
variable is a *descriptor* list: every linter in that descriptor runs
unless it is also named in `DISABLE_LINTERS`.

---

## `megalinter/enable-descriptor-overbroad` — Important

**Pattern:** `.mega-linter.yml` uses `ENABLE` with a descriptor key
(`REPOSITORY`, `SPELL`, `YAML`, `MARKDOWN`, `ACTION`) instead of an
`ENABLE_LINTERS` allowlist of the advertised CI linters.

**Detect:** when the patch changes `.mega-linter.yml`,
`.github/workflows/mega-linter.yml`, or the CI / Validation section of
`README.md` / `AGENTS.md`, collect the advertised linters (actionlint,
zizmor, yamllint, markdownlint, cspell). Flag if `.mega-linter.yml` has
`ENABLE:` with descriptor names and no `ENABLE_LINTERS:`. Also flag
`ENABLE: REPOSITORY` (or an `ENABLE_LINTERS` that includes
`REPOSITORY_LS_LINT` or `REPOSITORY_SEMGREP`) when those tools are not
in the advertised set. Semgrep's MegaLinter default
`REPOSITORY_SEMGREP_RULESETS` is `auto`.

**Why it matters:** `ENABLE: REPOSITORY` activates every repository
linter in the documentation flavor. In v9.6.0 that includes ls-lint and
semgrep even when `DISABLE_LINTERS` names gitleaks, trivy, checkov, and
friends. CI then blocks on tools the PR never claimed to run, and
semgrep fetches its auto ruleset.

**Evidence:** PR #21. Copilot: "ENABLE: REPOSITORY activates every
repository linter included in the v9.6.0 documentation flavor. Because
the disable list omits REPOSITORY_LS_LINT and REPOSITORY_SEMGREP, both
run as blocking checks in addition to the PR's stated linter set;
Semgrep also defaults to fetching its auto ruleset. Use an explicit
linter allowlist (or disable those two) so CI matches the advertised
scope."

**Not a finding when:** `ENABLE_LINTERS` lists the advertised CI
linters and does not include unadvertised repository linters.
`SPELL_CSPELL` may still be non-blocking via `DISABLE_ERRORS_LINTERS`.

---

## `megalinter/zizmor-skips-composite-actions` — Important

**Pattern:** MegaLinter's `ACTION_DIRECTORY` is left at the default
`.github/workflows`, so `ACTION_ZIZMOR` never sees
`.github/actions/**/action.yml`.

**Detect:** when the patch changes `.mega-linter.yml`,
`.github/workflows/mega-linter.yml`, or `.github/actions/**`, require
`ACTION_DIRECTORY: .github` (or another path that includes composite
actions). Flag a missing key, a value of `.github/workflows`, or a
zizmor include-filter that drops `actions/`. Also flag
`ACTION_ACTIONLINT` scanning composite actions: keep actionlint on
workflows (`ACTION_ACTIONLINT_FILTER_REGEX_INCLUDE` matching
`.github/workflows/`).

**Why it matters:** MegaLinter v9.6.0 identifies ACTION files under
`ACTION_DIRECTORY` (default `.github/workflows`). This repo's security
hardening lives in both reusable workflows and
`.github/actions/helm-chart-oci-publisher/action.yml`. Local validation
is `zizmor .github`; CI that only scans workflows lets composite-action
regressions through.

**Evidence:** PR #21. Copilot: "MegaLinter's v9.6.0 ACTION_ZIZMOR
descriptor fixes files_sub_directory to .github/workflows, so this step
does not scan the composite action that this PR also hardens. … Add a
dedicated zizmor invocation or custom descriptor that also covers
.github/actions/**/action.y*ml."

**Not a finding when:** `ACTION_DIRECTORY` is `.github` (or `any`) and
zizmor receives the composite action files. actionlint may still be
limited to workflows.
