---
name: google-oauth-validate
description: Use when running the Gemini / Antigravity OAuth + probe + SMS-validation flow for a single Google member account. Mirrors auto_chrome stage 3. Consumes google-login-playbook and oauth-token-harvest. Produces a verified credential record ready for credentials.json.
---

# Google OAuth + Antigravity Validation (per-member)

Business playbook that replicates `auto_chrome/src/3_local_oauth.js` using
stealth-chrome-mcp tools and Node fetch. Pipeline orchestration (batching,
concurrency, host↔member pairing, credentials.json read/write) stays in
the caller's code — this skill is the **per-member** recipe.

## Prerequisites

- MCP server env: `CLIENT_ID` / `CLIENT_SECRET` set (Antigravity OAuth creds,
  same as `auto_chrome/src/3_local_oauth.js` CLIENT_ID constant if unset).
- `HTTPS_PROXY` set if your environment needs it (MCP server already routes
  Node fetch through it).
- Account object per `google-login-playbook`.

## The Six-Step Per-Member Sequence

```
1. chrome_launch({ tags: { email: member.email } })
       → { sessionId }

2. google_login({ sessionId, account: member, smsBehavior: "auto" })
       → { status: "ok", finalUrl }
    # If status != "ok", record to failed.json and return

3. google_oauth_get_code({
       sessionId,
       scopes: ANTIGRAVITY_SCOPES,    // see below
       account: member,
   })
       → { code, redirectUri }

4. oauth_exchange_code({ code, redirectUri })
       → tokens { accessToken, refreshToken, expiresIn, scope, idToken, tokenType }

5. Get project_id via HTTP (see "Antigravity HTTP calls" below)
   Then probe `streamGenerateContent` — three outcomes:
     - 200 → verified, skip to step 6
     - 403 with validation_url → drive validation flow, re-probe, then step 6
     - Other → save as "saved_unverified" or mark failed

6. chrome_close({ sessionId })
   Persist credential record (shape below) to credentials.json via upsert.
```

## ANTIGRAVITY_SCOPES (full set)

Copy verbatim — do NOT subset. Must match what `auto_chrome`'s existing
credentials were issued against so the `refresh_token` stays compatible:

```
[
  "https://www.googleapis.com/auth/cloud-platform",
  "https://www.googleapis.com/auth/userinfo.email",
  "https://www.googleapis.com/auth/userinfo.profile",
  "https://www.googleapis.com/auth/cclog",
  "https://www.googleapis.com/auth/experimentsandconfigs"
]
```

## Antigravity HTTP Calls

These are plain `fetch` calls from the agent's Node runtime — no MCP
tool needed. All use the access token from step 4.

### Get project_id

```
POST https://cloudcode-pa.googleapis.com/v1internal:loadCodeAssist
Authorization: Bearer <accessToken>
Content-Type: application/json
User-Agent: geminicli-oauth/1.0

Body: {"metadata":{"ideType":"ANTIGRAVITY","platform":"PLATFORM_UNSPECIFIED","pluginType":"GEMINI"}}
```

Response shape:

- 200 with `cloudaicompanionProject` field → use it as `project_id`
- 200 without that field but with `allowedTiers[].id` → fall through to
  `onboardUser` (same endpoint family, body includes `tierId` from
  allowedTiers[isDefault=true]). Poll up to 6 times with 2s delay until
  `done: true` in response, then extract `cloudaicompanionProject`.
- Non-200 → `project_id = ''` (some accounts never get one; probe may
  still succeed)

See `getProjectId` in `auto_chrome/src/3_local_oauth.js` for the full
implementation you should replicate.

### Probe streamGenerateContent

```
POST https://cloudcode-pa.googleapis.com/v1internal:streamGenerateContent?alt=sse
Authorization: Bearer <accessToken>
Content-Type: application/json
User-Agent: antigravity/1.20.5 windows/amd64

Body: {
    "model": "gemini-3-pro-high",
    "request": {
        "contents": [{"role":"user","parts":[{"text":"ping"}]}],
        "generationConfig": {"thinkingConfig":{"includeThoughts":true}}
    },
    "project": "<project_id>",
    "requestId": "agent-<uuid>",
    "userAgent": "antigravity",
    "requestType": "agent"
}
```

Outcomes:
- **200 OK** — verified. Consume SSE (or just close stream — you only
  care about HTTP status).
- **403** — parse response body; if it contains `validation_url` in the
  `details` array, extract it and proceed to validation flow.
- **Anything else** — treat as unverified.

See `probeAntigravity` in `auto_chrome/src/3_local_oauth.js` for parsing
details. The `validation_url` appears in
`response.error.details[].violations[].description` or similar — use
`extractValidationUrl` helper from `3_sub2api.js`.

## Validation Flow (probe returned 403 + validation_url)

The validation page is an in-browser flow that typically asks for phone
verification. Drive it by:

1. Open a new page on the existing session: the MCP tool doesn't expose
   page.goto directly, so use `chrome_evaluate({ sessionId, script: "..." })`
   for custom page work, OR call the `completeValidationFlow` helper from
   `auto_chrome/src/3_sub2api.js` via a Node import (since the agent
   runtime has access to the repo).

2. `completeValidationFlow` returns `true` on success. On false, record
   `validation_flow_stuck` and skip.

3. If the flow re-encounters a `signin` page (cookies wiped mid-flow),
   call `google_login` again with the same account — should short-circuit
   fast since cookies are already warm.

4. After validation succeeds, re-call `getProjectId` and `probeAntigravity`
   — probe should now return 200.

**Do NOT** call `chrome_clear_google_cookies` between login and validation
— the validation flow needs the live Google session.

## Credential Record Shape

Write to `auto_chrome/src/credentials.json` (or a path you configure).
Upsert by `name`:

```json
{
    "name": "<host>_<member-localpart>",
    "email": "<google email from /userinfo or account.email>",
    "host_email": "<host.email>",
    "member_email": "<member.email>",
    "refresh_token": "<tokens.refreshToken>",
    "access_token": "<tokens.accessToken>",
    "token_type": "Bearer",
    "scope": "<tokens.scope>",
    "expires_in": <tokens.expiresIn>,
    "expires_at": <Date.now() + expires_in * 1000>,
    "project_id": "<from getProjectId, or empty string>",
    "verified_at": "<ISO8601 if probe_ok>",
    "probe_status": <http status from probe>,
    "probe_error": "<first 500 chars of error body, or null>",
    "updated_at": "<ISO8601>"
}
```

See existing `auto_chrome/src/credentials.json` entries for reference.

## Skip-Already-Verified Logic

Before launching Chrome for a member, check credentials.json:
- If there's a record with matching `name` AND `verified_at` is set AND
  you weren't passed `--reauth=<email>` or `--reauth-all` → skip.
- This matches the behavior of `auto_chrome/src/3_local_oauth.js`
  `processMember` early-exit.

## Error Recovery

| Signal | Action |
|--------|--------|
| `google_login.status == "rejected"` | Record failed.json entry with reason; skip member |
| `google_login.status == "stuck"` | Close session, launch fresh, retry once. Still stuck → failed |
| `OAUTH_TOKEN_EXCHANGE_FAILED: invalid_grant` | Re-run `google_oauth_get_code` to get fresh code (single-use) |
| Probe 403 + no validation_url | Save as `saved_unverified` — token is still good, account just not eligible yet |
| `completeValidationFlow` returns false | Screenshot the vpage, save `validation_flow_stuck` to failed.json |
| Chrome dies mid-flow | The server catches `CHROME_PROTOCOL_ERROR`. Restart session and retry member once. |

## Concurrency

MCP caps sessions at 5. For batch processing, launch up to 5 sessions in
parallel:

```
Promise.all(members.slice(0, 5).map(m => processMember(m)))
```

`auto_chrome` uses N=3 — conservative against WSLg resource limits and
Chrome start-up concurrency. Match that unless benchmarks show more works.

## Relationship to auto_chrome stage 3

This skill + MCP replaces `auto_chrome/src/3_local_oauth.js` for the
per-member dance. The pipeline file's `main()` (host/member pairing,
CLI args, summary stats) stays as caller code. Eventually the caller can
be an agent loop consuming members.txt rather than a Node script; that's
a separate refactor.
