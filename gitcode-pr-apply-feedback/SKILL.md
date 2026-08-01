---
name: gitcode-pr-apply-feedback
description: Pull GitCode PR review feedback, organize it into an actionable fix list, apply and verify fixes, push them, reply to each discussion, and resolve only completed inline discussions. Use when applying review feedback or closing review threads.
---

# Apply GitCode PR Feedback

Close the review loop without losing discussion state:

```text
Collect -> triage -> fix -> verify -> push -> reply -> resolve -> verify state
```

## 1. Collect Feedback

Read inline discussions and their machine-readable state:

```bash
gitcode pr comments <PR> -R owner/repo --json
```

Record each actionable comment's `id`, `discussion_id`, `path`, `position`, and
`resolved` fields. `position` is the target line number in the new version of
the file (`new_line`), not a unified-diff offset.

Read general PR comments as well:

```bash
gitcode pr view <PR> -R owner/repo --comments --json
```

If the PR references an issue, read that issue's comments separately:

```bash
gitcode issue comments <issue> -R owner/repo --json
```

Never copy tokens, private keys, credentials, or vulnerability details into a
working list, reply, commit, or review summary.

## 2. Triage

Order findings by impact: blockers and correctness or security defects first,
then missing tests, maintainability concerns, and optional suggestions. Group
items by file and distinguish these outcomes:

- `fix`: implement and verify now.
- `clarify`: ask the reviewer; keep the discussion unresolved.
- `defer`: explain why and keep the discussion unresolved unless the reviewer
  explicitly agrees that it is complete.
- `reject`: explain the technical reason; do not resolve the discussion merely
  because a reply was posted.

## 3. Apply And Verify Fixes

Check out the PR branch without overwriting concurrent work, apply each accepted
fix, and run the repository's required checks. For a Go repository, the minimum
usually includes:

```bash
go build ./...
go test ./...
go vet ./...
```

Follow the target repository's own contribution, testing, security, and release
rules when they require more.

## 4. Push Safely

Review the staged diff, commit intentionally, and push the branch. Use
`--force-with-lease` only when rewriting the author's branch is necessary and
authorized; never use an unconditional force push.

## 5. Reply To Inline Discussions

Use the `discussion_id` returned by `pr comments`, not the comment `id`:

```bash
gitcode pr reply <PR> \
  --discussion <discussion-id> \
  --body "Fixed and verified: <concise evidence>." \
  -R owner/repo
```

The reply should identify the change and the verification evidence. A reply by
itself does not mark the discussion as resolved.

## 6. Resolve Completed Discussions

Resolve only after all of these are true:

1. The requested change, or an agreed alternative, is implemented.
2. Required local verification passed.
3. The fix was pushed to the PR branch.
4. A reply with concise evidence was posted to the same discussion.
5. No clarification, disagreement, or follow-up remains.

```bash
gitcode pr comment resolve <PR> <discussion-id> -R owner/repo
```

Do not resolve deferred, rejected, partially fixed, failed-verification, or
awaiting-clarification findings.

## 7. Verify Remote State

Read the discussions again and confirm the target `discussion_id` reports
`resolved: true`:

```bash
gitcode pr comments <PR> -R owner/repo --json
```

If the state is still false, report the failure instead of claiming the review
feedback is complete. Finish with a PR summary that lists fixed, unresolved,
and deferred items plus verification evidence.

## Rules

- Use structured JSON to identify comment and discussion IDs.
- Fix by understanding the finding, not by blindly applying a suggestion.
- Reply to every actionable finding; use the matching discussion for inline
  feedback.
- Resolve only completed discussions and verify the resulting remote state.
- Keep unresolved work visible until the reviewer agrees it is complete.
- Do not expose secrets in commands, output, comments, commits, or logs.
