---
name: gitcode-actions
description: Use GitCode CLI Actions commands to inspect CI/CD pipeline runs, jobs, logs, and artifacts. Trigger when users need to view pipeline status, job details, download logs or artifacts, or triage build failures for a GitCode repository.
---

# gitcode-actions

Inspect GitCode Actions pipelines (runs, jobs, logs, artifacts). All commands are read-only and use the Actions v8 API.

## Command Entry

Use `gitcode` for cross-platform instructions. Linux/macOS may use `gc`; Windows PowerShell should use `gitcode`.

## Pipeline Runs

```bash
# Recent runs
gitcode actions run list -R owner/repo --json

# Filter by status / event / branch / executor
gitcode actions run list -R owner/repo --status FAILED --json
gitcode actions run list -R owner/repo --event Push --branch main --json
gitcode actions run list -R owner/repo --executor dev --json

# Filter by workflow name or id, or by PR number
gitcode actions run list -R owner/repo --workflow "CI" --json
gitcode actions run list -R owner/repo --pr 42 --json

# Full pagination
gitcode actions run list -R owner/repo --paginate --per-page 100 --json

# Run detail (run-id comes from `run list` -> workflow_run_id)
gitcode actions run view <run-id> -R owner/repo --json
```

`--status` values: `COMPLETED` / `RUNNING` / `FAILED` / `CANCELED` / `IGNORED` / `PAUSED` / `SUSPEND`.
`--event` values: `MR` / `Push` / `Manual`.

## Jobs and Logs

```bash
# Jobs under a run (job-id comes from here -> id)
gitcode actions job list <run-id> -R owner/repo --json

# Job detail with steps
gitcode actions job view <run-id> <job-id> -R owner/repo --json

# Download job log archive (ZIP; use --output, not stdout in a TTY)
gitcode actions job log <run-id> <job-id> -R owner/repo --output job-log.zip
```

Job logs are returned as a ZIP archive (one log file per step), not plain text. Always use `--output` in an interactive terminal; raw stdout is allowed only when piped/redirected.

## Artifacts

```bash
# List artifacts (optionally for a specific run)
gitcode actions artifact list -R owner/repo --json
gitcode actions artifact list -R owner/repo --run <run-id> --name build --json

# Artifact detail (artifact-id from `artifact list` -> id)
gitcode actions artifact view <artifact-id> -R owner/repo --json

# Download artifact as ZIP
gitcode actions artifact download <artifact-id> -R owner/repo --output artifact.zip
```

## Rules

- Prefer `--json` for parsing; empty results return `[]`.
- Pagination flags: `--limit` / `--per-page` / `--page`; `--paginate` fetches all pages up to `--limit` and is mutually exclusive with `--page`.
- These commands are read-only; the CLI does not trigger or rerun pipelines.
- Binary outputs (job logs, artifact downloads) must use `--output` or redirection; the CLI refuses to write binary to an interactive TTY.
- Exit codes follow the standard scheme: `0` success, `2` usage, `3` not found, `4` auth, `5` conflict.
