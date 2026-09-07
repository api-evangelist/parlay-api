---
name: parlay-api-find-best-odds
description: Fetch live odds for a sport and find the best price per outcome
  across 30+ sportsbooks using ParlayAPI.
api: ParlayAPI
generated: 2026-09-07
method: generated
source: openapi/parlay-api-openapi.json
operations:
- list_sports_v1_sports_get
- get_odds_v1_sports__sport_key__odds_get
- best_line_alias_v1_sports__sport_key__best_line_get
- get_consensus_v1_sports__sport_key__consensus_get
---

# Find the best odds for a game

1. **Discover the sport key.** `GET /v1/sports` (`list_sports_v1_sports_get`)
   lists all 92 sport keys (e.g. `americanfootball_nfl`, `basketball_nba`).
2. **Pull the market.** `GET /v1/sports/{sport_key}/odds?markets=h2h`
   (`get_odds_v1_sports__sport_key__odds_get`). Moneyline only costs 1 credit;
   `markets=h2h,spreads,totals` costs 2; any `player_*` props market costs 5.
   Auth: `X-API-Key` header (or `?apiKey=` for the-odds-api compatibility).
3. **Best price per outcome.** `GET /v1/sports/{sport_key}/best-line`
   (`best_line_alias_v1_sports__sport_key__best_line_get`) returns the best
   available price per outcome across books.
4. **Sanity-check against consensus.** `GET /v1/sports/{sport_key}/consensus`
   (`get_consensus_v1_sports__sport_key__consensus_get`) gives the cross-book
   average; a price far off consensus may be stale — check
   `/v1/bookmakers/{key}/freshness`.

Rules: read `X-Credits-Cost` / `X-Credits-Remaining` on every response; on 429
honor `Retry-After` before retrying (immediate retries waste credits). Errors
come back as `{"error", "message", "request_id", "docs_url"}` — not
problem+json. Keyless rehearsal: the same shapes are served at
`/v1/sandbox/sports/{sport_key}/odds` with no key.
