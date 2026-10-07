# Incident.io alerts and HTTP steps

---

## `http/curl-without-fail` — Important

**Pattern:** a `run:` script `curl`s an HTTP API whose success is later
read from `steps.<id>.outcome`, without `--fail` or `--fail-with-body`.

**Detect:** in a changed workflow, a `curl` that posts or fetches a
non-optional resource (Incident.io, a required API) and is missing
`--fail` / `--fail-with-body`. Flag it when a later step branches on that
step's `outcome`.

**Why it matters:** `curl` exits 0 on HTTP 4xx/5xx unless `--fail` is set.
PR #13's drift-alert steps reported "successfully sent to Incident.io"
when the API rejected the payload.

**Evidence:** PR #13. Copilot: "This `curl` does not use `--fail` (or
`--fail-with-body`), so it exits 0 even when Incident.io returns an HTTP
4xx/5xx."

---

## `incidentio/unsafe-defaults-for-external-callers` — Critical

**Pattern:** `enable_incidentio_alert` defaults to `true`, or
`incidentio_alert_token` / `incidentio_alert_source` default to
non-empty Linux Foundation-internal Secrets Manager paths.

**Detect:** in `lfx-opentofu-check.yml`, `enable_incidentio_alert` must
default to `false`, and the token/source inputs must default to empty.
Flag a default of `true` or a baked-in `ENVVAR, /path/in/sm` string.

**Why it matters:** an external LF project that calls the workflow without
overriding those inputs still enters the "Read Incident.io secrets" step,
which fails hard when the SM paths do not exist in their account.

**Evidence:** PR #13. Copilot: "an external LF project that calls this
workflow without overriding these will still satisfy the `if` condition …
and execute the Read Incident.io secrets step, which fails hard."
A human reviewer: "remove our internal tokens and make the `_token`
and `_source` empty … keep the `enable_` var as default false."
