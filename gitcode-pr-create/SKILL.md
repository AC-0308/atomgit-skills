---
name: gitcode-pr-create
description: Guide creation of GitCode pull requests using GitCode CLI with branch checks, diff review, body-file support, fork PR support, and JSON result parsing. Trigger when users want to open, draft, or update a GitCode PR.
---

# gitcode-pr-create

Create a GitCode PR with evidence and a clean description.

## Command Entry

Use `gitcode` for cross-platform instructions. Linux/macOS may use `gc`; Windows PowerShell should use `gitcode`.

## Workflow

1. Verify current branch and working tree.
2. Confirm remote and target repository.
3. Review diff and test evidence.
4. Draft PR body in a file.
5. Create PR with `--body-file` and `--json`.

## Local Checks

```bash
git status --short --branch
git remote -v
git log --oneline --decorate -5
git diff --stat origin/main...HEAD
```

## PR Body Template

```markdown
## Summary
- ...

## Verification
- [ ] command/result

## Risk
- ...

## Related
- Closes #...
```

## Create

```bash
gitcode pr create -R owner/repo --base main --head feature-branch --title "feat: summary" --body-file pr.md --json
gitcode pr create -R owner/repo --title "fix: summary" --body-file pr.md --draft --json
echo "PR body" | gitcode pr create -R owner/repo --title "docs: update" --body-file - --json
```

Fork:

```bash
gitcode pr create -R upstream/repo --fork myfork/repo --head feature-branch --base main --title "feat: summary" --body-file pr.md --json
```

## Update Existing PR

```bash
gitcode pr edit 123 -R owner/repo --body-file pr.md --json
gitcode pr edit 123 -R owner/repo --title "new title" --json
gitcode pr ready 123 -R owner/repo --ready --yes --json
```

## Rules

- `pr create` supports `--body-file`; prefer it for multi-line descriptions.
- Use `--json` to capture the created PR URL/number.
- Do not create from `main` unless the repository workflow explicitly allows it.
- Use SSH for push/clone operations.
- Do not include secrets in the PR body.
