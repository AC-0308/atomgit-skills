---
name: gitcode-delivery-notify
description: Report key AI delivery milestones (issue verified, PR created, CI result, review verdict, merged, blocked) to the user's personal Feishu/Lark via `gc lark send --to-self`. Must obtain interactive consent before reporting anything. Trigger during GitCode delivery workflows when a milestone is reached and the user wants personal Feishu notifications.
---

# gitcode-delivery-notify

Send concise delivery-milestone summaries to the user's own Feishu/Lark during a GitCode delivery loop (issue → PR → CI → review → merge). Each report goes to the user personally via `gc lark send --to-self` (bot → user P2P; no group needed).

This skill is a cross-cutting orchestrator. It does not replace `gitcode-pr-create`, `gitcode-pr-review`, etc. It only decides when and what to summarize, then delegates the send to `gc lark send`.

## Command Entry

Use `gitcode` for cross-platform instructions. Linux/macOS may use `gc`; Windows PowerShell should use `gitcode`.

## Prerequisites

1. GitCode CLI installed (`gitcode version`).
2. `lark-cli` installed and logged in. If not ready:

   ```bash
   gitcode lark install
   lark-cli config init --new          # interactive, browser
   lark-cli auth login --recommend    # interactive, browser
   gitcode lark doctor                # expect installed:true, login_ready:true
   ```

   If `gitcode lark doctor` reports `login_ready:false`, do not attempt sends — fix the login first.

## Consent (REQUIRED before any send)

This skill MUST get the user's consent before reporting. Never push messages to the user's Feishu without it.

### Consent source

Order of precedence:
1. Persisted consent file: `~/.config/gc/skills/lark-notify-consent.json`.
2. Interactive prompt this session (only if `CanPrompt` is true).
3. None → DO NOT send (skip silently, do not error).

### Consent file shape

```json
{
  "version": 1,
  "target": "self",
  "chat_id": "",
  "milestones": ["issue_verified", "pr_created", "ci", "review", "merged", "blocked"],
  "confirm_each": false
}
```

- `target`: `self` (recommended, `--to-self`) or `chat` (use `chat_id`).
- `chat_id`: only used when `target == "chat"`; the user's app bot must be a member of that group (internal groups easiest; external groups need the app's "对外共享" capability — out of scope here).
- `milestones`: which events to report (subset of the keys below).
- `confirm_each`: `true` → ask before each send; `false` → auto-send opted-in milestones.

### First-run / reconfigure prompt

If the consent file is missing and `CanPrompt` is true, ask the user (multi-select):

> Which delivery milestones should I report to your Feishu? (multi-select)
> - [ ] issue_verified — issue accepted/verified
> - [ ] pr_created — PR opened
> - [ ] ci — CI run finished
> - [ ] review — review verdict reached
> - [ ] merged — PR merged / issue closed
> - [ ] blocked — blocked or needs your input
> Target: self (default) or a chat-id?
> Confirm each send before sending? (y/n)

Then write the consent file (`mkdir -p ~/.config/gc/skills` first). The user can say "修改通知设置" / "reconfigure delivery notify" at any time to re-trigger this prompt.

### Non-interactive mode

If `--no-interactive` is set or `CanPrompt` is false and no persisted consent exists: skip all sends silently. Do not block the delivery. Log a one-line note to stderr that notifications were skipped.

## Milestones and summary templates

Keep summaries SHORT: key id, title, URL, one-line conclusion. Never paste full PR bodies, review comment dumps, or logs. Templates (fill the `{placeholders}`):

### issue_verified
```
✅ Issue #{number} verified
{title}
{one_line_conclusion}
```

### pr_created
```
🆕 PR #{number} created
{title}
{html_url}
```

### ci
```
🔍 CI run #{run_number} {overall}
{job}: {conclusion}  (one line per job, or "all passed")
{run_url}
```

### review
```
👁️ Review PR #{number}: {approved|changes-requested}
{top_finding}
{html_url}
```

### merged
```
🎉 Merged PR #{number} ({method})
issue #{issue_number} closed
```

### blocked
```
🛑 Blocked at {stage}
{blocker}
Need from you: {input}
```

## Send

For each opted-in milestone reached:

```bash
gitcode lark send --to-self --markdown "<summary>" --json
```

For `target == "chat"`:

```bash
gitcode lark send --chat-id <chat_id> --as bot --markdown "<summary>" --json
```

- `--json` so the agent can confirm `ok:true` and capture `message_id`.
- If `confirm_each` is true, show the summary to the user and get a yes before running the send command.
- If `gitcode lark send` exits non-zero (e.g. exit 4 auth/permission), do NOT retry-spam; log once and continue the delivery. Do not fail the delivery on a notification error.

## Dedup and throttle

- Track sent milestone keys this session: `{milestone}:{id}` (e.g. `pr_created:428`, `ci:428:50`). Do not re-send the same key.
- For `ci`: if the same run transitions multiple times in quick succession, send only the final conclusion.
- For `review`: send once per verdict state change.

## Security

- Summaries contain ids/titles/URLs/conclusions only — no tokens, no full bodies, no review comment dumps, no logs.
- `gc lark send` scans message text for the current `GC_TOKEN`/`GITCODE_TOKEN` value and refuses to send if found. Do not try to bypass this.
- Feishu credentials stay in `lark-cli`'s OS keychain; this skill and `gc` never read or print them.
- Never include secrets in summaries. If a PR title/body/review finding looks sensitive, summarize it generically rather than quoting.

## Workflow integration

Load this skill at the start of a delivery session. At each milestone the delivery loop reaches, check consent, check dedup, build the short summary from the relevant `gitcode` command output (`gitcode issue view`, `gitcode pr view --json`, `gitcode actions job list`, `gitcode pr review`, merge result), then send.

Typical milestone signals:
- `gitcode issue view N --json` shows `status/verified` label → `issue_verified`.
- `gitcode pr create --json` returns `number`/`html_url` → `pr_created`.
- `gitcode actions job list <run-id> -R owner/repo --json` all `COMPLETED` → `ci`.
- review command returns a verdict → `review`.
- `gitcode pr merge` success + PR `state:merged` → `merged`.
- a gate cannot proceed (needs human input, external approval, etc.) → `blocked`.

## Rules

- Never send without consent (persisted or interactive this session).
- Non-interactive + no consent → skip silently; never block the delivery.
- Summaries are short and contain no secrets or full content.
- Dedup by milestone key; do not re-send the same event.
- Notification failures are best-effort: log once, continue the delivery.
- Use `gitcode` (not `gc`) in instructions so Windows PowerShell works.
- Prefer `--to-self`; only use `--chat-id` when the user explicitly configured a group and the bot is a member.
- This skill does not edit existing skills; it is a standalone cross-cutting orchestrator.
