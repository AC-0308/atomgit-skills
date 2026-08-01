---
name: gitcode-regression
description: Run GitCode CLI real-command smoke and regression checks across Windows and Linux. Trigger before or after GitCode CLI upgrades, release validation, package verification, or command behavior changes.
---

# gitcode-regression

Validate GitCode CLI behavior with real commands.

## Command Entry

Use `gitcode` for cross-platform checks. On Linux/macOS also verify `gc` when installed from PyPI, wheel, DEB, or RPM. On Windows PowerShell, `gc` is `Get-Content`; use `gitcode` and optionally `gc.exe`.

## Minimum Smoke Check

```bash
gitcode version
gitcode version --json
gitcode help --json
gitcode schema
gitcode schema "repo clone"
gitcode auth status
```

Linux/macOS entrypoint parity:

```bash
gc version
gitcode version
gc help --json
gitcode help --json
```

Windows PowerShell entrypoint check:

```powershell
gitcode version
gitcode help --json
gc.exe version
python -m gc_cli version
```

## Read-Only Repository Checks

Use a safe test repository:

```bash
gitcode repo view infra-test/gctest1 --json
gitcode issue list -R infra-test/gctest1 --state open --json
gitcode pr list -R infra-test/gctest1 --state open --json
gitcode release list -R infra-test/gctest1 --json
gitcode label list -R infra-test/gctest1 --json
gitcode milestone list -R infra-test/gctest1 --json
```

## SSH Code Transfer Checks

```bash
ssh -T git@gitcode.com
gitcode repo clone infra-test/gctest1 --git-protocol ssh --depth 1
```

For sync commands, use only repositories explicitly approved for testing.

## Write-Path Checks

Only run write commands against a safe test repository:

```bash
gitcode issue create -R infra-test/gctest1 --title "test: cli regression" --body "temporary test" --dry-run --json
gitcode repo delete infra-test/nonexistent --dry-run
gitcode release delete v0.0.0-test -R infra-test/gctest1 --dry-run
```

## Exit Code Checks

The CLI defines stable exit codes for automation. Verify each:

```bash
gitcode issue view 1 -R infra-test/gctest1   # success -> exit 0
gitcode issue view abc                        # usage error -> exit 2
gitcode issue view 999999 -R infra-test/gctest1  # not found -> exit 3
gitcode auth status                           # unauthenticated -> exit 4
```

| Code | Meaning | Typical scenario |
|------|---------|------------------|
| 0 | Success | Normal completion |
| 1 | Generic error | API error, network error |
| 2 | Usage error | Missing/invalid args |
| 3 | Not found | issue/pr/repo does not exist |
| 4 | Auth error | Not logged in or invalid token |
| 5 | Conflict | PR merge conflict |

`auth status --json` still writes JSON to stdout when unauthenticated (`logged_in:false`), with exit code `4`.

## Report Format

```markdown
## GitCode CLI Regression

- CLI version: <output>
- OS / shell: <output>
- Entrypoints: gitcode OK, gc OK/N/A
- Auth: OK/failed with reason
- Read-only commands: passed/failed
- SSH transfer: passed/failed
- Write dry-runs: passed/failed
- Exit codes: 0/2/3/4 verified (note any deviation)
- Risks or skipped checks: ...
```

## Rules

- Prefer real commands over assumptions.
- Record failures with command, exit code, and important stderr.
- Do not run destructive commands without `--dry-run` or explicit user approval.
- Do not use production repositories for write tests unless the user explicitly authorizes them.
