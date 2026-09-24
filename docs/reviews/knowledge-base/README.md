# LFX Public Workflows local review knowledge base

Empirical review patterns for `lfx-public-workflows`, extracted from verified
past PR review comments on this repository. This directory is the **single
repo-owned home** for this repo's review pattern evidence and false-positive
decisions; the reviewer skill holds the review *method* and loads this path.

Read by the `lfx-public-workflows-learnings-reviewer` brain (the learnings
role of the local pre-PR reviewer trio). It is plain documentation: nothing
here is wired into `.github/**`.

## Scope

These entries describe **what reviewers of this repo actually caught and
developers actually fixed**. They are not a restatement of `AGENTS.md` and
not general GitHub Actions advice. The repo's written rules are the
`lfx-public-workflows-code-reviewer` brain's surface; this one is purely
empirical.

## Evidence base

Entries derive from review comments on pull requests of
`linuxfoundation/lfx-public-workflows`. Inline review threads and Copilot
review overviews (`copilot-pull-request-reviewer`) both qualify. Human
reviewers are in scope when a developer fixing commit landed. Prefer
merged PRs; a maintainer may record a pattern from an open PR once the
fix is on the branch.

## Promotion gate

An entry is in this knowledge base only if **all** of these hold:

1. A **verified review comment** raised it, with the PR recorded.
   Inline review threads and Copilot review overviews both qualify.
2. A **developer fixing commit exists** (merged PR, or an open PR whose
   fix has already landed when a maintainer is recording the pattern).
3. The condition is **still relevant** to the current workflows.
4. The condition is **mechanically detectable from a diff**.
5. No deterministic check in this repo already catches it (actionlint,
   zizmor, yamllint, MegaLinter). Patterns that zizmor now flags as
   `template-injection` stay out — CI owns them.

## Categories

| File                                                           | Patterns                                                                        |
|----------------------------------------------------------------|---------------------------------------------------------------------------------|
| [reusable-workflow-contract.md](reusable-workflow-contract.md) | workflow_call outputs; declared inputs must be consumed; advertised input names |
| [cspell-flagwords.md](cspell-flagwords.md)                     | placeholder-line guard; POST restore of `.cspell.json`                          |
| [incidentio-and-http.md](incidentio-and-http.md)               | `curl --fail-with-body`; safe Incident.io defaults                              |
| [agent-docs.md](agent-docs.md)                                 | AGENTS.md vs tree: repo CI is not `workflow_call`; non-MIT SPDX exceptions      |
| [megalinter-activation.md](megalinter-activation.md)           | ENABLE_LINTERS allowlist; ACTION_DIRECTORY covers composite actions             |
| [known-false-positives.md](known-false-positives.md)           | the floor, applied last                                                         |

## Entry format

Each entry carries a stable pattern id, **Severity**, **Detect**, **Why it
matters**, **Evidence**, and **Not a finding when**.

## Maintenance

Add an entry only with its provenance chain intact. Add a floor entry only
with the maintainer's rebuttal. Removing a pattern or a floor entry is a
human-gated decision.
