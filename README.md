# LFX Shared Public Workflows

The public repository for shared workflows for projects under the `linuxfoundation/` GH org.

## Current Workflows

- **`license-header-check.yml`**

  Reusable workflow for checking license headers in source code files.

- **`lfx-opentofu-{plan,apply,check,plan-apply}`**

  Reusable workflows wrapping <https://github.com/dflook/terraform-github-actions> OpenTofu actions with extra setup
  glue for AWS login and secrets handling.

  The standalone `plan` and `apply` workflows are intended for a plan-on-PR/merge-on-apply workflow, `check` is for reoccurring drift detection jobs, and `plan-apply` is for manual triggers outside of the PR process (e.g. for retrying or resolving drift).

  Most inputs are shared across all four workflows:

  | Input                     | Type   | Required | Default             | Description                                          |
  | ------------------------- | ------ | -------- | ------------------- | ---------------------------------------------------- |
  | `environment`             | string | yes      |                     | Name of the GHA environment to use                   |
  | `opentofu_workspace`      | string |          | (environment input) | Name of the OpenTofu workspace to use                |
  | `opentofu_variables`      | string |          |                     | Variables to pass to OpenTofu                        |
  | `opentofu_backend_config` | string |          |                     | Backend configuration for OpenTofu                   |
  | `opentofu_plan_label`     | string |          | (environment input) | Label for the OpenTofu plan output in PR comment     |
  | `opentofu_var_file`       | string |          |                     | List of var file paths, one per line                 |
  | `opentofu_path`           | string |          | `.`                 | Path to the OpenTofu directory                       |
  | `oidc_role_arn`           | string | yes      |                     | ARN of the IAM role to assume with OIDC              |
  | `oidc_audience`           | string |          | `sts.amazonaws.com` | OIDC audience to authenticate against                |
  | `oidc_region`             | string |          | `us-east-2`         | Default region for the AWS client                    |
  | `env`                     | string |          |                     | Additional environment variables                     |
  | `secrets_manager_keys`    | string |          |                     | List of keys to fetch from AWS Secrets Manager       |
  | `artifact_name`           | string |          |                     | Name of an artifact to download before running       |
  | `artifact_path`           | string |          |                     | Path to download the artifact to                     |
  | `pre_run_script`          | string |          |                     | Shell commands passed through as `TERRAFORM_PRE_RUN` |
  | `secrets.env_secret`      | secret |          |                     | Additional secret environment variables              |

  A few inputs only apply to specific workflows:

  | Input                         | Applies to           | Type    | Default | Description                                               |
  | ----------------------------- | -------------------- | ------- | ------- | --------------------------------------------------------- |
  | `opentofu_add_github_comment` | `plan`, `plan-apply` | string  | `true`  | Post the plan output as a comment on the PR               |
  | `opentofu_changes`            | `apply`              | boolean | `true`  | Set to `false` to skip applying (e.g. a no-op plan)        |
  | `enable_incidentio_alert`     | `check`              | boolean | `false` | Enable Incident.io alert creation on drift detection      |
  | `incidentio_alert_token`      | `check`              | string  |         | Incident.io alert token source (as `ENVVAR, /path/in/sm`) |
  | `incidentio_alert_source`     | `check`              | string  |         | Incident.io alert source (as `ENVVAR, /path/in/sm`)       |

  Both `plan` and `plan-apply` also expose a `changes` job output (`true`/`false`, from the underlying
  `dflook/tofu-plan` action) indicating whether the plan has any changes to apply. `plan-apply` uses this
  internally to skip its apply job on a no-op plan. For the standalone `plan`/`apply` split, pass the `plan`
  job's `changes` output through as the `apply` workflow's `opentofu_changes` input to skip applying when
  nothing changed.

## License

Copyright The Linux Foundation and each contributor to LFX.

This project’s source code is licensed under the MIT License. A copy of the license is available in LICENSE.

This project’s documentation is licensed under the Creative Commons Attribution 4.0 International License \(CC-BY-4.0\).
A copy of the license is available in LICENSE-docs.
