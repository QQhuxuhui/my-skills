---
name: oauth-token-harvest
description: Use when harvesting Google OAuth refresh tokens (access_token + refresh_token) for a given client_id and scopes via stealth-chrome-mcp. Covers the 3-step login → authorize → exchange flow and the refresh loop. Consumes google-login-playbook for the login step.
---

# OAuth Token Harvest (stealth-chrome-mcp)

Agent playbook for collecting Google OAuth tokens from a real user session
driven by the `stealth-chrome-mcp` server. Pairs with `google-login-playbook`
for the login half.

## Prerequisites

- MCP server configured with `CLIENT_ID` / `CLIENT_SECRET` in env (or pass
  them per-call to each tool). See MCP server `~/.claude.json` entry.
- Valid Google account with TOTP secret if 2FA enabled (see
  google-login-playbook for account shape).

## Three-Step Flow

```
1. google_login({ sessionId, account })
      → { status: "ok", finalUrl: "...myaccount.google.com/..." }

2. google_oauth_get_code({
         sessionId,
         scopes: [...],
         account,          // needed for TOTP re-challenge during consent
      })
      → { code, redirectUri }

3. oauth_exchange_code({ code, redirectUri })
      → { accessToken, refreshToken, expiresIn, scope, idToken, tokenType }
```

The `refreshToken` is what you persist. `accessToken` expires in
~3600s; use the refresh token to mint new access tokens as needed
(see "Refreshing Later" below).

## Choosing Scopes

Pass scopes as an array of full URLs. Common sets:

- **Generic GCP / Cloud access:**
  `["https://www.googleapis.com/auth/cloud-platform"]`
- **Antigravity / Gemini family (full set used by auto_chrome stage 3):**
  See `auto_chrome/src/3_local_oauth.js` `SCOPES` constant — 5 scopes
  including `cloud-platform`, `userinfo.email`, `userinfo.profile`,
  `cclog`, `experimentsandconfigs`.

Pick the minimal set your downstream API needs. More scopes = more
consent prompts and higher chance of user rejection in interactive flows.

## CLIENT_ID / CLIENT_SECRET

Both tools accept per-call `clientId` / `clientSecret` arguments. If
omitted, they fall back to the MCP server's env `CLIENT_ID` /
`CLIENT_SECRET`. Convention:

- Development / one-off: set env on the MCP server launcher once.
- Per-account-type flows (e.g. different OAuth apps for Gemini vs.
  Workspace): pass explicit args per tool call.

The `redirectUri` returned by `google_oauth_get_code` is what you must
pass verbatim to `oauth_exchange_code` — port is chosen dynamically by
the MCP server's callback listener. Do NOT construct your own
`redirectUri` string; use the returned value.

## Refreshing Later

After initial harvest, refresh the access token like this (no MCP needed
— plain HTTP from your backend):

```
POST https://oauth2.googleapis.com/token
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token
refresh_token=<stored>
client_id=<yours>
client_secret=<yours>
```

Response: `{ access_token, expires_in, scope, token_type }`. No new
`refresh_token` — the original stays valid until Google rotates it
(typically only on password change or explicit revoke).

`oauth_exchange_code` is ONLY for the initial `code → refresh_token` swap.
Don't use it for refresh.

## Common Errors

| Error code | Meaning | Recovery |
|------------|---------|----------|
| `OAUTH_CODE_NOT_RECEIVED` | Browser never reached `localhost:PORT/callback` | Pass `account` to `google_oauth_get_code` so consent poller can handle TOTP re-challenge and account-selection buttons |
| `OAUTH_TOKEN_EXCHANGE_FAILED: invalid_grant` | Code was already exchanged (single-use) or expired | Re-run `google_oauth_get_code` to get a fresh code |
| `OAUTH_TOKEN_EXCHANGE_FAILED: redirect_uri_mismatch` | `redirectUri` passed to exchange didn't match what authorize used | Always use the `redirectUri` returned by `google_oauth_get_code` — don't build your own |
| `PRECONDITION_FAILED: clientId/clientSecret missing` | No env, no per-call arg | Set env on MCP server or pass args |

## Why You Need `account` in google_oauth_get_code

Google's consent flow often shows:

1. **Account selector** ("Choose an account") — poller clicks the matching
   `account.email` entry
2. **TOTP re-challenge** — poller detects TOTP input and auto-fills from
   `account.totp_secret`
3. **Consent buttons** ("Continue", "Allow") — poller auto-clicks

Without `account`, the poller runs but can't fill TOTP or pick the right
account, so it stalls until timeout. Always pass `account` unless you're
sure neither challenge will appear (rare).

## Example — Full Token Harvest

```
const launch = await chrome_launch({ tags: { email: account.email } });

const login = await google_login({
    sessionId: launch.sessionId,
    account,
    smsBehavior: "auto",
});
if (login.status !== "ok") { /* handle per google-login-playbook */ }

const oauth = await google_oauth_get_code({
    sessionId: launch.sessionId,
    scopes: ["https://www.googleapis.com/auth/cloud-platform"],
    account,
});

const tokens = await oauth_exchange_code({
    code: oauth.code,
    redirectUri: oauth.redirectUri,
});

await chrome_close({ sessionId: launch.sessionId });

// Persist tokens.refreshToken.
```
