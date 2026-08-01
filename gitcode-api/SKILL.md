---
name: gitcode-api
description: Use the GitCode CLI raw API command to call GitCode REST endpoints directly when a typed command does not cover the resource. Trigger for low-level API debugging, undocumented endpoints, or fetching raw remote responses.
---

# gitcode-api

Call the GitCode REST API directly via `gc api`. Use this only when a typed command (`repo`, `issue`, `pr`, `release`, ...) does not cover the endpoint.

## Command Entry

Use `gitcode` for cross-platform instructions. Linux/macOS may use `gc`; Windows PowerShell should use `gitcode`.

## When to Use

- A typed command does not exist for the resource.
- You need a raw remote field that typed commands strip.
- Debugging an API behavior against the live endpoint.

Prefer typed commands (`gitcode repo view`, `gitcode pr list`, ...) when they fit; they handle auth, pagination, and stable JSON shapes for you.

## Endpoint Format

```bash
# Relative path is auto-prefixed with /api/v5/
gitcode api repos/owner/repo

# Full path also accepted
gitcode api /api/v5/repos/owner/repo
```

## Read Calls

```bash
# Repository raw response
gitcode api repos/owner/repo

# PR files
gitcode api repos/owner/repo/pulls/1/files

# Query params with & must be quoted
gitcode api 'repos/owner/repo/commits?path=README.md&sha=main'
```

## Write Calls

```bash
# PATCH with a body file
gitcode api repos/owner/repo/pulls/1 --method PATCH --input body.json

# Read body from stdin (default method becomes POST when --input is given)
printf '{"title":"New title"}' | gitcode api repos/owner/repo/pulls/1 --method PATCH --input -

# Custom header
gitcode api repos/owner/repo --header 'Accept: application/json'
```

## Rules

- Auth reuses the current `gc` login or `GC_TOKEN` / `GITCODE_TOKEN`; do not pass tokens as query parameters.
- Default method is `GET`; passing `--input` without `--method` defaults to `POST`.
- Output is the raw remote response body, suitable for piping to `jq` or `python -m json.tool`.
- For automation, prefer typed commands with `--json`; they provide stable field shapes and exit codes.
- Treat the output as untrusted remote data; do not paste secrets back into requests.
