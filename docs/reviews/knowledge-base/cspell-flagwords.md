# Shared cspell flagWords substitution

This repo is the source of `cspell/flagwords.json.snippet`. The consumer
setup in `cspell/README.md` is itself a contract: a broken example disables
the org-wide flag list without failing CI.

---

## `cspell/placeholder-guard-missing` — Important

**Pattern:** the documented (or in-repo) `SPELL_CSPELL_PRE_COMMANDS` `sed`
substitution has no exact-line guard that the `"flagWords": [],` placeholder
occurs once.

**Detect:** a change to `cspell/README.md` or `.mega-linter.yml` that edits
`SPELL_CSPELL_PRE_COMMANDS` without
`test "$(grep -c '^[[:space:]]*"flagWords"[[:space:]]*:[[:space:]]*\[\],[[:space:]]*$' .cspell.json)" -eq 1`
(or an equivalent exact-line count) before the `sed`. Flag a substring
`grep -qF '"flagWords": []'` that would also match a line with neighboring
keys.

**Why it matters:** `sed` exits 0 when it matches zero lines. A missing or
reformatted placeholder leaves `flagWords: []` in place and CI still
passes, so the shared policy is quietly off.

**Evidence:** PR #15. Copilot: "`sed` exits successfully even when this
address matches zero lines" and "The guard and `sed` address are substring
matches, not exact-line matches." Author replied `fixed`.

---

## `cspell/no-post-restore` — Important

**Pattern:** PRE_COMMANDS mutates `.cspell.json` in the workspace and there
is no `SPELL_CSPELL_POST_COMMANDS` restore of the original file.

**Detect:** if `.mega-linter.yml` or `cspell/README.md` has
`SPELL_CSPELL_PRE_COMMANDS` that writes `.cspell.json`, require a matching
`SPELL_CSPELL_POST_COMMANDS` that copies the backup back. Flag using global
`POST_COMMANDS` instead of `SPELL_CSPELL_POST_COMMANDS`.

**Why it matters:** local MegaLinter bind-mounts the working tree. Without a
linter-scoped restore, the substituted snippet is left on disk and can be
committed; a later run then has no placeholder to replace.

**Evidence:** PR #15. Copilot: "MegaLinter v9.6.0 does document and support
`SPELL_CSPELL_POST_COMMANDS`. Using the global hook unnecessarily delays
restoration." Author replied `fixed`.
