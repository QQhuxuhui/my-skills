---
name: google-login-playbook
description: Use when driving a Google account login via the stealth-chrome-mcp server. Explains the account object shape, smsBehavior options, error recovery, and session reuse strategy. Applies whenever an agent needs to invoke `google.login` as part of a larger workflow (OAuth harvesting, dashboard automation, family group operations, etc.).
---

# Google Login Playbook (stealth-chrome-mcp)

This skill teaches agents how to drive Google account login using the
`stealth-chrome-mcp` MCP server. The server wraps the battle-tested login
state machine from the `auto_chrome` project; your job is only to compose
the tool calls correctly.

## Account Object Shape

All tools that accept `account` expect this exact shape:

```json
{
  "email": "user@gmail.com",
  "password": "the-password",
  "totp_secret": "BASE32SECRET",
  "fa_secret": "BASE32SECRET",
  "recovery_email": "backup@gmail.com"
}
```

- `email` and `password` are required.
- `totp_secret` (or `fa_secret`, a compatibility alias) is a base32 TOTP
  seed. Required for accounts with 2-step verification enabled.
- `recovery_email` is optional; supplied when Google falls back to
  "enter your recovery email" challenge.

Internally the MCP server normalizes: `password → pass`, and
`totp_secret` wins over `fa_secret` if both present.

## Happy-Path Flow (one account)

```
1. chrome_launch({ tags: { email: "user@gmail.com" } })
      → { sessionId, debugPort, dataDir }
2. google_login({ sessionId, account, smsBehavior: "auto" })
      → { status: "ok", finalUrl: "https://myaccount.google.com/..." }
3. ... your business tools (oauth, navigation, evaluate, ...)
4. chrome_close({ sessionId })
```

Always close sessions when done. The server enforces a hard cap (default
5) — leaking sessions will cause subsequent launches to fail with
`CONCURRENCY_LIMIT_EXCEEDED`.

## smsBehavior Options

The login state machine may hit a phone-verification step. `smsBehavior`
controls how that's handled:

| Value | Use when | Behavior |
|-------|----------|----------|
| `auto` (default) | You want fully automated end-to-end | MCP uses its configured SMS provider (env `SMS_PROVIDER` + credentials). Costs SMS credits. |
| `skip` | Exploratory / you don't want to burn SMS | Returns `{ status: "sms_needed" }` immediately when challenge appears. |
| `manual` | Debugging with human-in-the-loop | Keeps Chrome open and polls for up to 5 min waiting for human to type code in the browser window. |

Default `auto` is the right choice for batch automation. Use `skip` when
probing account status without intending to complete verification.

## Status Values & Recovery

`google_login` always returns a `status` field:

| status | meaning | typical recovery |
|--------|---------|------------------|
| `ok` | Fully authenticated, cookies set | Proceed to next tool |
| `rejected` | Google refused the attempt — possibly wrong password, "Couldn't sign you in", browser flagged | Check `reason` field (e.g. `wrong_password`). Inspect `screenshot` (base64 PNG) before retrying. |
| `stuck` | Login state machine deadlooped (page doesn't change) | Often a transient Chrome issue. Close session, launch fresh, retry. |
| `sms_needed` | SMS challenge appeared with `smsBehavior=skip` | Either switch to `auto` or flag account for manual handling |
| `timeout` | Exceeded `timeoutMs` (default 180s) | Inspect `screenshot` before retrying; may indicate Google added a new challenge step |

On any non-`ok` status, the response includes a `screenshot` field (base64
PNG of the current page at failure). Decode and inspect before making
retry decisions — Google changes challenge UX frequently and blind
retries waste SMS and trigger rate limits.

## Session Reuse Strategy

**Do NOT share one session across multiple accounts.** Google cookies
persist; the next `google_login` on the same session will either
short-circuit to the wrong account or confuse the state machine.

The correct pattern per-account:

```
chrome_launch → google_login → (tool chain) → chrome_close
```

Within a single account's workflow, you CAN reuse one session across
multiple tools — `google_login` puts the session in authenticated state
and subsequent tools (like `google.oauth_get_code`) see an authenticated
page. Just don't hop accounts without closing.

If you need to log a second account in the same session for any reason,
call `chrome_clear_google_cookies({ sessionId })` first — it wipes
Google-domain cookies and storage without restarting Chrome.

## When Login Is Unexpectedly Fast

If `google_login` returns `ok` in under 2 seconds and `finalUrl` points
to `myaccount.google.com`, Google auto-accepted a cached session (cookies
from the dataDir). This is a feature of persistent dataDirs — default
isolated mode uses fresh `/tmp/stealth-chrome-mcp/sess-*` every time, so
this shouldn't normally happen unless you pass a custom `dataDir`.

## Concurrency

Session hard cap: 5 by default (`MAX_SESSIONS` env override). Same-session
tool calls are serialized by an internal mutex — safe to interleave.
Different-session calls run in parallel. For batch processing multiple
accounts, launch up to 5 sessions and process them in parallel via your
agent framework.
