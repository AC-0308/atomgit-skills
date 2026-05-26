# GitCode CLI Skills

This repository publishes standalone skills for operating GitCode with the latest GitCode CLI.

Current baseline: GitCode CLI `v0.5.7`.

## Cross-platform command rule

Every skill in this repository is written to work on Windows and Linux.

- Prefer `gitcode` in reusable instructions because it works on Windows PowerShell and Linux.
- On Linux/macOS, `gc` and `gitcode` are equivalent when installed from PyPI, wheel, DEB, or RPM.
- On Windows PowerShell, `gc` is a built-in alias for `Get-Content`; use `gitcode`, `gc.exe`, or `python -m gc_cli`.
- Code download and sync workflows should use SSH by default. Ensure `git@gitcode.com` SSH access works before clone/sync operations.

## Skills

- `gc-auth`: authentication and token troubleshooting
- `gc-repo`: repository view/list/create/fork/clone/delete/sync/stats
- `gc-issue`: issue create/list/view/edit/comment/label/relation workflows
- `gc-pr`: pull request create/list/view/edit/checkout/merge/sync workflows
- `gc-review`: PR and commit review workflows
- `gc-release`: release create/edit/upload/download/delete workflows
- `gc-commit`: commit view/diff/patch/comment workflows
- `gc-label-milestone`: label and milestone management
- `gc-regression`: real-command smoke and regression validation
- `gitcode-repo-onboarding`: onboard to an existing GitCode repository
- `gitcode-security-check`: repository security and sensitive information review
- `gitcode-issue-create`: guided issue creation
- `gitcode-issue-review`: pre-development issue analysis
- `gitcode-issue-triage`: issue queue triage
- `gitcode-pr-create`: guided PR creation
- `gitcode-pr-review`: engineering PR review
- `gitcode-release-helper`: release planning and publishing

## Install

Copy any skill directory into the local skill directory of your AI client. Each skill is self-contained.

Codex example:

```bash
mkdir -p ~/.codex/skills/gc-pr
cp -R gc-pr/SKILL.md ~/.codex/skills/gc-pr/SKILL.md
```

Claude example:

```bash
mkdir -p ~/.claude/skills/gc-pr
cp -R gc-pr/SKILL.md ~/.claude/skills/gc-pr/SKILL.md
```

## Verify CLI

```bash
gitcode version
gitcode auth status
gitcode help --json
gitcode schema
```

Use `gitcode schema "pr create"` or `gitcode help pr create` when a skill needs to confirm a flag in the installed CLI.
