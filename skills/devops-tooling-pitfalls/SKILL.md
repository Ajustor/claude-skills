---
name: devops-tooling-pitfalls
description: Use when tooling misbehaves silently — Azure DevOps PR description rejected over 4000 characters, accents corrupted by az CLI, VssPropertyValidationException, test count lower in CI than locally, squash-merged commits "missing" from main, `## ` lines vanished from commit messages, Container Apps cron job never firing, Keycloak "invalid_redirect_uri", Query Store without permissions, heredoc eating backslashes, CRLF turned into LF.
---

# DevOps and tooling field pitfalls

Empirical gotchas, each reproduced: **symptom → cause → fix**.

## Azure DevOps

- **PR description capped at 4000 characters.** `PATCH …/pullrequests/{id}` fails with `InvalidArgumentValueException: A description for a pull request must not be longer than 4000 characters` and leaves the **old** description in place — reviewers read the stale one. → Aim for ~3500; put the full detail in the commit message (no limit). After sending, check the returned body contains key passages. Count after normalizing line endings: `Get-Content -Raw` keeps `\r\n` (2 chars/line), Python sees 1.
- **`az repos pr create/update` corrupts non-ASCII characters on Windows**, and `az` stdout (cp1252) shows mojibake even when stored data is correct. → Create the PR in one REST `POST …/pullrequests?api-version=7.1` with `{sourceRefName, targetRefName, title, description, workItemRefs}`, body written to a UTF-8 file and sent as explicit UTF-8 bytes (`[Text.Encoding]::UTF8.GetBytes(...)`, or `curl --data-binary @file.json`). To verify, check code points (`0xe9` for "é", not `0xc3 0xa9`), not the glyphs a cp1252 terminal prints; in Python read `sys.stdin.buffer` and decode UTF-8.
- **`VssPropertyValidationException — You must pass a valid patch document`**: PowerShell `ConvertTo-Json` serializes a one-element array as an object. → Always `-AsArray`.
- **`System.Parent` missing from a work item GET** — the field is absent, not empty, without `$expand`. → `?$expand=relations` and look for `System.LinkTypes.Hierarchy-Reverse`. In PowerShell, `"…/$id?`$expand=…"` builds a broken URL; concatenate explicitly.
- **CI shows fewer tests than `dotnet test`.** xUnit truncates long `[Theory]` display names; two cases differing after the cut get the same name and Azure DevOps merges them. → Diff a local `.trx` against `_apis/test/Runs/<id>/results`; keep theory arguments short.
- **Squash merge:** branch commits are never ancestors of `main` (`git merge-base --is-ancestor` is false). Verify by content (grep symbols, list files), not SHAs. Only the PR title/description survive as history.

## Git

- **`## Heading` lines disappear from commit messages** after `rebase --continue` or `commit --amend` (even with `GIT_EDITOR=true`): default `cleanup=strip` treats them as comments. → `--cleanup=whitespace` (`verbatim` leaves a trailing blank line). Detect with `git range-diff old-base..old-tip base..HEAD` — `git diff` never compares messages.
- **`git add a b; git commit`**: a failed `add` (e.g. old path after `git mv`) does not stop `;` and a partial commit goes out. `git add … | grep -v "LF will be replaced" && git commit` never commits because `grep -v` exits 1 on empty output. → `git add`, then `git status --short`, then `git commit -F msg.txt` as separate steps.

## Azure

- **A suspended Container Apps job looks healthy.** `az containerapp job show` shows `runningState: null` and `provisioningState: Succeeded` exactly like a working job; config diff is clean, cron is armed. Neither an ARM PATCH nor `terraform apply` resumes it (azurerm has no suspension attribute). → Check the Activity Log for `Microsoft.App/jobs/suspend/action` (filter out `listSecrets/action` noise; `az` caps results **before** the JMESPath filter, so query short windows). In `ContainerAppSystemLogs_CL`, a live job emits `SuccessfulCreate` / `SawCompletedJob`; a suspended one emits none. Check suspension before blaming the cron expression.
- **Query Store without `VIEW DATABASE STATE`:** the ARM Query Performance Insight API works with the Reader role — `{db}/topQueries?api-version=2014-04-01&$filter=observedMetric eq 'cpu'` (top 5, 24 h), then `/topQueries/{id}/queryText` and `/statistics` (hourly; cpu/io in % of capacity). Other parameters are silently ignored; extra `$filter` terms return 500 after ~46 s; newer preview API versions may not exist in your region.
- **`az rest` on Windows goes through cmd.exe**, which splits the URL at `&`. → `Invoke-RestMethod` with a token from `az account get-access-token`.

## Keycloak

- **`Authority` ≠ issuer by design.** With `frontendUrl` set to a public domain, tokens carry the public issuer while the API reaches Keycloak on an internal address (403 from outside). Never "fix" `Authority` to the public URL — it drives server-to-server OIDC discovery. To publish a Keycloak URL (docs, Swagger), read it from `ValidIssuers[0]`: it is exercised on every token validation, so it is known-correct.
- **Keycloak 26.7.3 (not 26.6.1) rejects a `redirect_uri` containing an OIDC parameter in its query**: log "Redirect URI rejected: contains forbidden OIDC parameter 'code' in query", then `error="invalid_redirect_uri"`. Hits pages that receive `code`/`state` from **another** provider and then init `keycloak-js`, which uses the current URL. → Persist them to `sessionStorage` and strip them with `history.replaceState` before Keycloak init.
- **Upgrading a Docker Keycloak does nothing** if a volume is mounted on the whole `/opt/keycloak`: it freezes the installed version. → Use a fresh volume (keeping the old one allows rollback).

## Shell and scripting

- **Agent Bash tool: a heredoc eats one backslash level even with a quoted delimiter** (`<< 'EOF'`): `\\n` arrives as `\n`, so a Python patch needle no longer matches source containing `"\n"`, though `grep` looks identical. → Build backslashes with `chr(92)`, or patch line by line; compare with `repr()`.
- **Python `open(p).read()` + `write(..., newline='')` converts CRLF to LF.** On `core.autocrlf=true` repositories, restore CRLF on touched files.
