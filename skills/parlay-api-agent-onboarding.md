---
name: parlay-api-agent-onboarding
description: Programmatically create a free ParlayAPI key for a user and set up
  line-movement webhooks and prop-line alerts.
api: ParlayAPI
generated: 2026-09-07
method: generated
source: openapi/parlay-api-openapi.json, https://parlay-api.com/llms.txt, asyncapi/parlay-api-webhooks.json
operations:
- agent_signup_v1_agent_signup_post
- agent_magic_link_v1_agent_magic_link_post
- create_webhook_v1_webhooks_post
- test_webhook_v1_webhooks__webhook_id__test_post
- create_prop_line_alert_v1_alerts_prop_line_post
- rotate_webhook_secret_v1_webhooks__webhook_id__rotate_secret_post
---

# Agent onboarding and alert wiring

1. **Sign the user up.** `POST /v1/agent/signup` with `{"email": "<user's
   email>"}` (`agent_signup_v1_agent_signup_post`) returns `{api_key, tier,
   claim_url, upgrade_url, instructions_for_agent}`; the key works immediately
   (free tier, 1,000 credits/month, no card). Idempotent on email: an existing
   account returns 409 with a login URL — use
   `agent_magic_link_v1_agent_magic_link_post` instead; the key is never
   re-disclosed. Rate limit: 5 signups/IP/hour. Get the user's consent before
   submitting their email.
2. **Create a webhook (Pro+).** `POST /v1/webhooks`
   (`create_webhook_v1_webhooks_post`) with `{url, events[], sport_filter}`.
   Event types: arb_flagged, ev_alert, line_move, live_arb, odds_drop,
   new_event, test. Deliveries are signed HMAC-SHA256 in `X-Parlay-Signature`
   (`t=<ts>,v1=<hex>`, 5-minute replay window); store the `whsec_` secret —
   it is shown exactly once.
3. **Test it.** `POST /v1/webhooks/{webhook_id}/test`
   (`test_webhook_v1_webhooks__webhook_id__test_post`) fires a test event.
4. **Add a prop-line alert.** `POST /v1/alerts/prop-line`
   (`create_prop_line_alert_v1_alerts_prop_line_post`) with sport_key,
   player_name, market_key, threshold_line and the webhook_id to notify.
5. **Rotate on leak.**
   `rotate_webhook_secret_v1_webhooks__webhook_id__rotate_secret_post` issues a
   new secret once.

Reversal paths: webhooks and alerts are deleted with their DELETE operations or
disabled via PATCH (`is_active` / `enabled` false); subscriptions cancel via
`cancel_subscription_route_billing_subscription_cancel_post`. Retry policy on
deliveries: 3 attempts (immediate, +30s, +5min), auto-disable after 5
consecutive failures.
