# GitCode CLI Skills

This repository publishes standalone skills for operating GitCode with the latest GitCode CLI.

Current baseline: GitCode CLI `v0.6.0`.

## Cross-platform command rule

Every skill in this repository is written to work on Windows and Linux.

- Prefer `gitcode` in reusable instructions because it works on Windows PowerShell and Linux.
- On Linux/macOS, `gc` and `gitcode` are equivalent when installed from PyPI, wheel, DEB, or RPM.
- On Windows PowerShell, `gc` is a built-in alias for `Get-Content`; use `gitcode`, `gc.exe`, or `python -m gc_cli`.
- Code download and sync workflows should use SSH by default. Ensure `git@gitcode.com` SSH access works before clone/sync operations.

## Skill Design

The skills are split into two layers:

1. Core command skills: broad command-family references for direct GitCode CLI operations.
2. Workflow skills: task-oriented procedures built on top of the core commands.

This keeps the set mostly orthogonal:

- Use a workflow skill when the user asks for an end-to-end task such as creating a PR, reviewing a PR, publishing a release, onboarding, or security review.
- Use a core command skill when the user asks for direct command operations or when no workflow skill fits.
- Each skill is self-contained and can work without loading another skill.

## Core Command Skills

| Skill | Use When | Boundary |
| --- | --- | --- |
| `gitcode-auth` | Login, logout, token source, credential troubleshooting, auth status. | Does not manage repo/issue/PR workflows. |
| `gitcode-repo` | View, list, clone, create, fork, delete, stats, or sync repositories. | Does not explain project build/test conventions; use onboarding for that. |
| `gitcode-issue` | Direct issue commands: create, list, view, edit, close, reopen, comments, labels, relations. | For guided issue writing, use `gitcode-issue-create`. |
| `gitcode-pr` | Direct PR commands: create, list, view, edit, checkout, comment, ready/WIP, close, reopen, merge, sync. | For guided PR creation or engineering review, use workflow skills. |
| `gitcode-review` | PR/commit comment mechanics, review submission, approval command usage. | For full engineering review, use `gitcode-pr-review`. |
| `gitcode-release` | Direct release commands: list, view, create, edit, upload, download, delete. | For release planning and notes, use `gitcode-release-helper`. |
| `gitcode-commit` | Commit view, diff, patch, and commit comments. | Does not perform full PR review. |
| `gitcode-precommit` | Check pre-commit config and local environment before committing (`precommit check`, v0.6.0+). | Checks the local pre-commit environment; does not replace the repo's test suite or CI. |
| `gitcode-label-milestone` | Label and milestone management for issues/PRs/releases. | Does not triage an issue queue by itself. |
| `gitcode-regression` | Real-command smoke tests after CLI install, upgrade, packaging, or behavior changes. | Does not replace a target repository's own test suite. |

## Workflow Skills

| Skill | Use When | Boundary |
| --- | --- | --- |
| `gitcode-repo-onboarding` | A user wants to understand, clone, build, test, or contribute to a GitCode repository. | Uses repo commands plus local inspection; does not change remote state unless asked. |
| `gitcode-security-check` | Security audit, sensitive information review, secret scan, auth/config risk review. | Reports findings; does not rewrite history or rotate secrets without explicit request. |
| `gitcode-issue-create` | Turn a user report or request into a high-quality GitCode issue. | Focuses on pre-create drafting and duplicate search. |
| `gitcode-issue-review` | Analyze an existing issue before implementation. | Produces readiness, risks, and acceptance criteria; does not triage queues. |
| `gitcode-issue-triage` | Batch organize issue queues with labels, priorities, duplicates, and comments. | Does not do implementation planning for one issue in depth. |
| `gitcode-pr-create` | Prepare and open a PR with branch checks, diff review, body file, and JSON result. | Focuses on PR creation, not review. |
| `gitcode-pr-review` | Perform an engineering PR review with findings, verification, and approval/comment flow. | Focuses on review, not PR creation. |
| `gitcode-pr-inline-review` | Submit inline code review comments on specific PR diff lines using `gc pr comment --path --position`. | Focuses on line-level comment submission; use alongside `gitcode-pr-review` for full review workflows. |
| `gitcode-pr-apply-feedback` | Pull PR review feedback (inline comments, discussion, issue), organize into fix list, apply locally, verify, push, and reply. | Closes the review loop: receive → fix → reply. |
| `gitcode-release-helper` | Plan and publish a release: notes, tag checks, assets, verification. | Uses release commands but adds release process guidance. |
| `gitcode-pipeline-analyzer` | Analyze PR pipelines: parse pipeline comments, fetch failed task logs from openLiBing, inspect stage/job duration, build batch CI reports. | Works with GitCode PRs that have pipeline bot comments and openLiBing task links. |
| `pipeline-bisect` | Diagnose pipeline regressions: compare good/bad runs, extract error logs from artifacts, trace to the triggering commit or PR. | Focuses on root-cause bisection when a previously-passing pipeline suddenly fails. |
| `gitcode-label-stats` | Statistics of PR label distribution across repositories in a GitCode organization. Supports multiple labels, monthly breakdown, and ratio analysis. | Reports label adoption metrics; does not create, edit, or delete labels. |

## Overlap Policy

Some workflows intentionally repeat the exact command snippets they need. That repetition is deliberate so every skill can be installed and used independently.

When two skills could apply, prefer the narrower workflow skill:

- Creating a PR: `gitcode-pr-create`, not `gitcode-pr`
- Reviewing a PR: `gitcode-pr-review`, not `gitcode-review`
- Publishing a release: `gitcode-release-helper`, not `gitcode-release`
- Writing a new issue: `gitcode-issue-create`, not `gitcode-issue`

## Install

Copy any skill directory into the local skill directory of your AI client. Each skill is self-contained.

Codex example:

```bash
mkdir -p ~/.codex/skills/gitcode-pr
cp -R gitcode-pr/SKILL.md ~/.codex/skills/gitcode-pr/SKILL.md
```

Claude example:

```bash
mkdir -p ~/.claude/skills/gitcode-pr
cp -R gitcode-pr/SKILL.md ~/.claude/skills/gitcode-pr/SKILL.md
```

## Verify CLI

```bash
gitcode version
gitcode auth status
gitcode help --json
gitcode schema
```

Use `gitcode schema "pr create"` or `gitcode help pr create` when a skill needs to confirm a flag in the installed CLI.
