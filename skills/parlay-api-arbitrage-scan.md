---
name: parlay-api-arbitrage-scan
description: Scan a sport for cross-book arbitrage, positive-EV plays, and
  middles with ParlayAPI, pricing the credit cost before each call.
api: ParlayAPI
generated: 2026-09-07
method: generated
source: openapi/parlay-api-openapi.json
operations:
- quote_credits_v1_meta_quote_post
- find_arbitrage_v1_sports__sport_key__arbitrage_get
- find_positive_ev_v1_sports__sport_key__ev_get
- find_middles_v1_sports__sport_key__middles_get
- try_arbitrage_v1_try__sport_key__arbitrage_get
---

# Arbitrage and +EV scanning

1. **Rehearse keyless first.** `GET /v1/try/{sport_key}/arbitrage`
   (`try_arbitrage_v1_try__sport_key__arbitrage_get`) costs 0 credits and needs
   no key (IP-limited 60/h) — use it to validate parsing before spending.
2. **Price the call.** `POST /v1/meta/quote` (`quote_credits_v1_meta_quote_post`)
   returns the credit cost of a request before you make it. Arbitrage, EV and
   middles scans cost 3 credits each.
3. **Scan.** `GET /v1/sports/{sport_key}/arbitrage`
   (`find_arbitrage_v1_sports__sport_key__arbitrage_get`) returns cross-book
   arbitrage candidates, 3-way (home/draw/away) markets included.
4. **Positive EV.** `GET /v1/sports/{sport_key}/ev`
   (`find_positive_ev_v1_sports__sport_key__ev_get`) compares each price to a
   sharp book's no-vig fair line.
5. **Middles.** `GET /v1/sports/{sport_key}/middles`
   (`find_middles_v1_sports__sport_key__middles_get`).

Rules: scans are read-only (nothing to reverse). Watch
`X-Credits-Remaining`; a 403 `credit_limit_exceeded` means the monthly
allowance is gone — upgrade or wait for the reset. On 429 sleep for
`Retry-After` seconds with capped exponential backoff.
