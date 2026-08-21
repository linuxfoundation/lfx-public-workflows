# Shared cspell flagWords

`flagwords.json.snippet` is the canonical, org-wide list of cspell
[`flagWords`](https://cspell.org/configuration/dictionaries/#flagwords)
for LFX repos: terms we want cspell to actively reject (usually
non-inclusive language, per the guidance at
[inclusivenaming.org](https://inclusivenaming.org/)), each paired with
a recommended replacement (e.g. `"master: controller, primary, main,
leader, parent"`).

Rather than copy-pasting and drifting this list across every repo's
`.cspell.json`, repos pull it in at MegaLinter run time via
`SPELL_CSPELL_PRE_COMMANDS`, and keep only repo-specific `words` /
`dictionaries` / `ignorePaths` locally.

The file is deliberately named `flagwords.json.snippet` rather than
`flagwords.snippet.json`: it's a `"flagWords": [...],` fragment, not a
complete JSON document, so a `.json` extension would make tooling
(editors, linters) treat it as JSON and flag it as invalid. The
`.json.snippet` extension keeps the content's origin clear while
excluding it from JSON validation.

## Repo setup

1. In the repo's `.cspell.json`, add an empty `flagWords` placeholder
   (must be on its own line, exactly `"flagWords": [],`):

   ```json
   {
     "language": "en",
     "dictionaries": ["companies", "filetypes", "fullstack", "softwareTerms"],
     "words": ["lfx", "lfxv", "linuxfoundation"],
     "flagWords": [],
     "ignorePaths": [".cspell.json", "CODEOWNERS", "LICENSE", "LICENSE-docs", "megalinter-reports", "styles"]
   }
   ```

2. In `.mega-linter.yml`, add a pre-command that fetches this file,
   backs up the local `.cspell.json`, and substitutes the snippet in
   for the placeholder line before cspell runs. Also add a
   `SPELL_CSPELL_POST_COMMANDS` entry that restores the original
   `.cspell.json` once cspell finishes:

   ```yaml
   SPELL_CSPELL_PRE_COMMANDS:
     - command: >-
         cp .cspell.json /tmp/cspell.json.orig &&
         curl -sSf
         https://raw.githubusercontent.com/linuxfoundation/lfx-public-workflows/main/cspell/flagwords.json.snippet
         -o /tmp/flagwords.json.snippet &&
         test "$(grep -c '^[[:space:]]*"flagWords"[[:space:]]*:[[:space:]]*\[\],[[:space:]]*$' .cspell.json)" -eq 1 &&
         sed -i -e '/^[[:space:]]*"flagWords"[[:space:]]*:[[:space:]]*\[\],[[:space:]]*$/{r /tmp/flagwords.json.snippet' -e 'd}' .cspell.json
       cwd: "workspace"
       continue_if_failed: false
   SPELL_CSPELL_POST_COMMANDS:
     - command: cp /tmp/cspell.json.orig .cspell.json
       cwd: "workspace"
   ```

   This is a plain line-for-line text substitution (no `jq`/`python`
   JSON tooling required, since neither `jq` nor `yq` is present in
   MegaLinter's images by default): `sed` finds the exact placeholder
   line and replaces it with the fetched snippet's contents, which
   already includes the `"flagWords": [...],` key/value plus trailing
   comma. Both `curl` and `sed` are present in MegaLinter's Docker
   images out of the box, so no extra `apk add` step is needed. The
   `test "$(grep -c ...)" -eq 1` guard runs before the `sed` and fails
   the command if the placeholder line is missing, duplicated, or
   already substituted (e.g. from a prior local run whose restore step
   didn't run): `sed`'s address-based `{...}` block otherwise exits
   successfully even when it matches zero (or more than one) lines, so
   without this check a missing/reformatted placeholder would silently
   leave `flagWords: []` in place and let cspell run with the shared
   policy disabled, despite `continue_if_failed: false`.

   `continue_if_failed: false` and `curl -sSf` (rather than the default
   `-sf`) are both deliberate: MegaLinter's `PRE_COMMANDS` default to
   `continue_if_failed: true`, so without the explicit override, a
   failed fetch (network hiccup, moved file, etc.) or a failed `sed`
   substitution would silently leave the local placeholder-only
   `flagWords: []` in place and let the run continue -- CI would pass
   with the shared policy quietly not applied. `-sSf` keeps `curl`'s
   error output in the logs (`-s` alone suppresses it) so a failure is
   visible instead of just producing an empty/missing file.

   The backup/restore pair matters for local runs: MegaLinter's
   `.github/workflows/mega-linter.yml` runs against an ephemeral
   checkout in CI, but a developer running MegaLinter locally via
   `make megalinter` typically bind-mounts the actual working
   directory (see repos' Makefiles), so the `PRE_COMMANDS` substitution
   would otherwise mutate the real, tracked `.cspell.json` on disk.
   `SPELL_CSPELL_POST_COMMANDS` runs immediately after cspell finishes
   (scoped to that linter specifically, unlike the global
   `POST_COMMANDS`, which only runs once *all* linters finish and could
   fire even when cspell itself didn't run) and restores the
   placeholder-only file right away, so a local run never leaves the
   substituted content sitting in the working tree ready to be
   accidentally committed.

## Updating the shared list

Edit `flagwords.json.snippet` directly (keep it a single
`"flagWords": [...],` JSON fragment, not a full JSON document, so it drops in as-is).
Changes take effect on every consuming repo's next MegaLinter run
automatically, no per-repo update required.

For guidance on which terms to flag and what to recommend instead, see
[inclusivenaming.org](https://inclusivenaming.org/)'s word list and
definitions.
