---
name: Verify a user's identity with CLEAR
description: Create a CLEAR1 verification session, hand the user off to CLEAR's hosted UI, then retrieve the verified result and traits.
api: openapi/clear-verification-openapi.json
operations:
  - create_verification_session
  - get_verification
  - search_verification_sessions
generated: '2026-07-18'
method: generated
source: https://docs.clearme.com/docs/standard-integration
---

# Verify a user's identity with CLEAR

CLEAR1's identity verification is a two-call standard integration: create a session, then read the result.

## Auth
- Base URL: `https://verified.clearme.com/v1`
- Send `Authorization: Bearer <API_KEY>` on every request (HTTPS only).
- Use a **sandbox** API key while testing; switch to a production key to go live (issued by CLEAR support).

## Steps

1. **Create the session** — call `create_verification_session` (`POST /verification_sessions`) with your `project_id` (from the Console) and an optional `redirect_url` carrying a server-generated `state` UUID. Set `"sandbox": true` while testing. Save `id` (to retrieve results) and `token` (to launch the UI).
2. **Hand off to CLEAR's hosted UI** — direct the user to `https://verified.clearme.com/verify?token=<token>` (redirect, webview, or kiosk). CLEAR collects the government ID, selfie/liveness, and any project-configured data.
3. **Know when it's done** — prefer a webhook (`event_verification_session_completed_v1`); or use the redirect `state`; or poll. A redirect alone is never proof — always confirm via the API.
4. **Retrieve the result** — call `get_verification` (`GET /verification_sessions/{id}`). Check `status` (`success | failed | expired | awaiting_user_input | canceled`) and inspect `checks[]`. Read verified attributes from `traits` (use `traits.document`; do **not** use the deprecated `verified_info`).
5. **Reveal SPII only when required** — SSN and ID images are `REDACTED` by default; add `?reveal_sensitive_data=true` only when your workflow and compliance posture require it.

## Conventions & errors
- Persist CLEAR's `user_id` (persistent identifier) and the `verification_id` for returning-user matching and audits; avoid storing raw PII.
- No idempotency-key header is documented — do not assume safe retries of create.
- Errors use `{error_type, message, fields[]}`; `429 rate_limit_exceeded` signals rate limiting. See `errors/clear-problem-types.yml` and identity-check exit codes in `errors/clear-reason-codes.yml`.

## Sandbox
- OTP is not sent; enter `123456`. Enter phone `408-222-2222` to simulate a returning user. See `sandbox/clear-sandbox.yml` for synthetic test identities.

## Search existing sessions
- Use `search_verification_sessions` (`GET /verification_sessions/search`) with filters (status, email, custom fields via `custom_<name>`) instead of the deprecated `list_verifications`.
