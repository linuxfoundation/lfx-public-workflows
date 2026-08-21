# Shared cspell flagWords

`flagwords.snippet.json` is the canonical, org-wide list of cspell
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
   `POST_COMMANDS` entry that restores the original `.cspell.json`
   once the run finishes:

   ```yaml
   SPELL_CSPELL_PRE_COMMANDS:
     - command: >-
         cp .cspell.json /tmp/cspell.json.orig &&
         curl -sf
         https://raw.githubusercontent.com/linuxfoundation/lfx-public-workflows/main/cspell/flagwords.snippet.json
         -o /tmp/flagwords.snippet.json &&
         sed -i -e '/"flagWords": \[\]/{r /tmp/flagwords.snippet.json' -e 'd}' .cspell.json
       cwd: "workspace"
   POST_COMMANDS:
     - command: cp /tmp/cspell.json.orig .cspell.json
       cwd: "workspace"
   ```

   This is a plain line-for-line text substitution (no `jq`/`python`
   JSON tooling required, since neither `jq` nor `yq` is present in
   MegaLinter's images by default): `sed` finds the exact placeholder
   line and replaces it with the fetched snippet's contents, which
   already includes the `"flagWords": [...],` key/value plus trailing
   comma. Both `curl` and `sed` are present in MegaLinter's Docker
   images out of the box, so no extra `apk add` step is needed.

   The backup/restore pair matters for local runs: MegaLinter's
   `.github/workflows/mega-linter.yml` runs against an ephemeral
   checkout in CI, but a developer running MegaLinter locally via
   `make megalinter` typically bind-mounts the actual working
   directory (see repos' Makefiles), so the `PRE_COMMANDS` substitution
   would otherwise mutate the real, tracked `.cspell.json` on disk. The
   `POST_COMMANDS` restore ensures the placeholder-only file is put
   back once the run completes, so a local run never leaves the
   substituted content sitting in the working tree ready to be
   accidentally committed. Note `POST_COMMANDS` is global (there is no
   documented per-linter `SPELL_CSPELL_POST_COMMANDS`), so it runs
   after every linter in the run, not just cspell -- that's fine here
   since its only job is restoring one file.

## Updating the shared list

Edit `flagwords.snippet.json` directly (keep it a single
`"flagWords": [...],` JSON fragment, not a full JSON document, so it drops in as-is).
Changes take effect on every consuming repo's next MegaLinter run
automatically, no per-repo update required.

For guidance on which terms to flag and what to recommend instead, see
[inclusivenaming.org](https://inclusivenaming.org/)'s word list and
definitions.
