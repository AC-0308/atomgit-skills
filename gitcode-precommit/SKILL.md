---
name: gitcode-precommit
<<<<<<< HEAD
description: Use GitCode CLI `precommit check` to verify a repository's pre-commit configuration and local environment before committing or submitting code. Detects `.pre-commit-config.yaml`, verifies/auto-installs the pre-commit tool, ensures the git hook is initialized, and optionally runs the hooks. Trigger before committing or opening a PR, or when a user asks to check pre-commit readiness. Requires GitCode CLI v0.6.0+.
=======
description: Use GitCode CLI precommit commands to check a repository's pre-commit configuration and local environment before committing. Trigger when users need to verify pre-commit is installed, initialize hooks, or run checks before commit.
>>>>>>> 30dda26 (chore: align skills with GitCode CLI v0.7.0)
---

# gitcode-precommit

<<<<<<< HEAD
Make sure pre-commit is configured and the local environment can run it before code is committed.
=======
Check and prepare the pre-commit environment for a repository before committing.
>>>>>>> 30dda26 (chore: align skills with GitCode CLI v0.7.0)

## Command Entry

Use `gitcode` for cross-platform instructions. Linux/macOS may use `gc`; Windows PowerShell should use `gitcode`.

<<<<<<< HEAD
Requires GitCode CLI **v0.6.0+** (the `precommit` command was added in v0.6.0). Confirm with `gitcode version` or `gitcode help precommit check`.

## What It Checks

`gitcode precommit check` runs in the current git repository and:

1. Detects a `.pre-commit-config.yaml` (or `.yml`) in the repo root. No config means nothing to check (exit 0).
2. Verifies the `pre-commit` tool is installed.
3. Verifies the git `pre-commit` hook is initialized (worktree-aware).
4. With `--run`, executes `pre-commit run --all-files`.

When the config is present but the tool or hook is missing, **install and
initialize it** rather than just reporting the gap. The command does this
automatically in an interactive terminal; in a non-interactive (non-TTY)
environment — which is the usual case for an agent — pass `--yes` to authorize
the install/init. Use `--no-install` only when the intent is strictly to
diagnose without changing the environment.

## Check

```bash
# Verify the environment is ready
gitcode precommit check

# Verify and actually run the hooks
gitcode precommit check --run

# Machine-readable result
gitcode precommit check --json
```

## Install / Initialize When Missing

When the repo has a pre-commit config but the tool is not installed or the hook
is not initialized, run the check in install-authorizing mode so it installs
pre-commit and/or runs `pre-commit install` before continuing. In an
interactive terminal `--yes` is not required; in automation it is.

```bash
# Install + initialize if missing, then run the hooks (non-interactive / agent)
gitcode precommit check --run --yes

# Install + initialize if missing, no hook run
gitcode precommit check --yes

# Diagnose only, never modify the environment (explicit opt-out)
gitcode precommit check --no-install --json
```

The `actions_taken` array reports what was done, e.g.
`["installed pre-commit via pipx", "ran pre-commit install"]`.

## Flags

| Flag | Effect |
| --- | --- |
| `--run` | After verifying, run `pre-commit run --all-files`. |
| `--no-install` | Only diagnose; never install the tool or hook. Mutually exclusive with `--yes`. |
| `--yes`, `-y` | Authorize environment changes (install/init) in a non-interactive environment. |
| `--json` | Emit a structured result to stdout. |

## JSON Result

```json
{
  "config_found": true,
  "tool_installed": true,
  "tool_version": "3.7.0",
  "hook_installed": true,
  "actions_taken": ["installed pre-commit via pipx", "ran pre-commit install"],
  "run_result": "passed",
  "run_output": "",
  "ok": true
}
```

- Read `ok` together with `config_found`: `ok: true, config_found: false` means "no config, skipped", not "verified ready".
- `run_output` carries the `pre-commit run` output only when `run_result` is `failed`.

## Exit Codes

- `0` ready, or no config (nothing to check).
- `1` not ready / checks failed / not a git repository / non-interactive change refused.
- `2` usage error (e.g. `--no-install` together with `--yes`).

## Cross-platform Install

When auto-installing, the tool tries available installers in order and falls through on failure: `pipx` → `python3 -m pip install --user` → `python -m pip install --user`, plus the `py` launcher on Windows. If no Python tooling is available it prints platform-specific manual instructions instead of installing Python.

## Rules

- Run from inside the target git repository.
- When the repo has a pre-commit config but the tool/hook is missing, install
  and initialize it: pass `--yes` in non-interactive automation so the command
  is authorized to install pre-commit and run `pre-commit install`. Only use
  `--no-install` when the task is explicitly diagnose-only.
- Use `--json` when a script or agent needs to parse readiness; key off `ok`,
  `run_result`, and `actions_taken`.
- This command checks/sets up the local pre-commit environment; it does not
  replace the repository's own test suite or CI.
=======
## Check Flow

`precommit check` runs four steps:

1. Detect `.pre-commit-config.yaml` (or `.yml`) at the repo root. No config means "nothing to check" (exit `0`).
2. Detect whether the local `pre-commit` tool is installed.
3. Detect whether the git pre-commit hook is initialized.
4. Optionally run `pre-commit run --all-files` with `--run`.

## Commands

```bash
# Check whether the environment is ready
gitcode precommit check

# Check and actually run pre-commit
gitcode precommit check --run

# Diagnose only; never install or initialize
gitcode precommit check --no-install

# Allow auto install/initialize in non-interactive environments
gitcode precommit check --yes

# Machine-readable output
gitcode precommit check --json
```

## Result Classification

The `--json` `reason` field is stable and machine-readable; branch on it:

- `no_config` — repo has no pre-commit config (`ok=true`, normal skip).
- `tool_missing` — `pre-commit` tool not installed (and not authorized to install).
- `hook_missing` — git pre-commit hook not initialized.
- `run_failed` — environment ready but `pre-commit run` failed.
- `install_failed` — auto-install authorized but no usable tool produced; see `install_failure_categories` (`permission` / `network` / `toolchain`).
- `not_in_repo` — current directory is not inside a git repo.
- empty — environment fully ready.

## Rules

- `--no-install` and `--yes` are mutually exclusive.
- In a TTY, missing tooling may auto-install; in non-interactive environments pass `--yes` to allow modifications, otherwise it only diagnoses and errors.
- Exit codes: `0` ready or no config; `1` not ready / check failed / not a git repo / non-interactive without `--yes`; `2` usage error.
- Do not run `--run` blindly in CI without first confirming the config; treat `run_failed` as a real signal, not an environment issue.
>>>>>>> 30dda26 (chore: align skills with GitCode CLI v0.7.0)
