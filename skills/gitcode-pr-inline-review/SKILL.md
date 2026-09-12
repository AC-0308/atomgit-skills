---
name: gitcode-pr-inline-review
description: Submit line-level GitCode PR review comments with gitcode pr comment --path --position. Use for actionable findings tied to a line in the new version of a file; use a general PR review for findings without a line anchor.
---

# GitCode PR Inline Review

Attach an actionable finding to a specific line in the PR's new file version.

## Position Semantics

`--position` is the one-based line number in the new version of the file
(`new_line`, the right side of the diff). It is not a unified-diff offset and
does not count `@@` headers or deleted lines.

```bash
gitcode pr comment <PR> \
  --path path/to/file.go \
  --position <new-file-line> \
  --body-file inline-finding.md \
  --json \
  -R owner/repo
```

The current command targets lines present on the new side of the diff. A line
that exists only on the deleted side cannot be anchored with this form; place
that finding on a nearby retained/new line or submit it as a general review
comment and identify the deleted code precisely.

## Workflow

1. Read PR metadata and the complete diff.
2. Confirm the target path exactly matches the path in the diff.
3. Open the new file version and identify its one-based source line number.
4. Confirm the target line is part of the PR diff and exists on the new side.
5. Submit one concise finding with impact and a recommended correction.
6. Read comments as JSON and verify the returned `path`, `position`, and
   `discussion_id`.
7. Submit a general review summary without duplicating the inline body.

Useful commands:

```bash
gitcode pr view <PR> -R owner/repo --json
gitcode pr diff <PR> -R owner/repo
gitcode pr comments <PR> -R owner/repo --json
gitcode pr review <PR> -R owner/repo --comment-file review-summary.md
```

## Finding Format

State severity, the defect or risk, its impact, and the expected correction.
Keep the line anchor tight and avoid posting the same finding both inline and in
the general summary.

## Rules

- Use the new file's source line number for `--position`.
- Do not estimate a unified-diff position or count hunk lines.
- Verify the first submitted comment before posting a large batch.
- Do not anchor a general design concern to an unrelated line.
- Never include credentials, tokens, private keys, or sensitive exploit details
  in review content.
