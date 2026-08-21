# Shared cspell flagWords

`flagwords.snippet.json` is the canonical, org-wide list of cspell
[`flagWords`](https://cspell.org/configuration/dictionaries/#flagwords)
for LFX repos: terms we want cspell to actively reject (usually
non-inclusive language), each paired with a recommended replacement
(e.g. `"master: controller, primary, main, leader, parent"`).

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

2. In `.mega-linter.yml`, add a pre-command that fetches this file and
   substitutes it in for the placeholder line before cspell runs:

   ```yaml
   SPELL_CSPELL_PRE_COMMANDS:
     - command: >-
         curl -sf
         https://raw.githubusercontent.com/linuxfoundation/lfx-public-workflows/main/cspell/flagwords.snippet.json
         -o /tmp/flagwords.snippet.json &&
         sed -i -e '/"flagWords": \[\]/{r /tmp/flagwords.snippet.json' -e 'd}' .cspell.json
       cwd: "workspace"
   ```

   This is a plain line-for-line text substitution (no `jq`/`python`
   JSON tooling required, since neither `jq` nor `yq` is present in
   MegaLinter's images by default): `sed` finds the exact placeholder
   line and replaces it with the fetched snippet's contents, which
   already includes the `"flagWords": [...],` key/value plus trailing
   comma. Both `curl` and `sed` are present in MegaLinter's Docker
   images out of the box, so no extra `apk add` step is needed.

## Updating the shared list

Edit `flagwords.snippet.json` directly (keep it a single `"flagWords":
[...]," ` JSON fragment, not a full JSON document, so it drops in as-is).
Changes take effect on every consuming repo's next MegaLinter run
automatically, no per-repo update required.
