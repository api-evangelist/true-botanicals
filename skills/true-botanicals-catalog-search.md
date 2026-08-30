---
name: true-botanicals-catalog-search
description: Search and resolve products in the True Botanicals storefront over its UCP/MCP endpoint, without transacting.
api: True Botanicals UCP Agent Commerce (MCP)
endpoint: https://truebotanicals.com/api/ucp/mcp
operations:
  - search_catalog
  - lookup_catalog
  - get_product
generated: '2026-08-30'
method: generated
source: mcp/true-botanicals-tools-list.json (live tools/list, 2026-08-30)
---

# Search the True Botanicals catalog

Read-only. Nothing here creates a cart, a checkout, or a charge.

## Before the first call

You need a UCP agent profile of your own. Every `tools/call` against this endpoint carries
`meta.ucp-agent.profile`, and the store fetches that URI. If it is missing you get HTTP 422 with JSON-RPC
`-32001` / `invalid_profile_url`; if it does not resolve you get the same `-32001` with `profile_unreachable`.
Publish the profile at a URL that answers 200 to an anonymous fetcher before you start.

`tools/list` itself needs no profile and no credential — use it to confirm the tool set is still 13 tools.

## Steps

1. **Search.** Call `search_catalog` with `catalog.query` set to the buyer's intent in plain language
   ("vitamin C serum", "cleanser for oily skin"). Set `catalog.context.address_country` and
   `catalog.context.currency` — pricing and availability depend on them, and llms.txt says so explicitly.
   Narrow with `catalog.filters.categories`, `catalog.filters.available` and `catalog.filters.price.min` /
   `.max`. Page with `catalog.pagination.cursor` and `catalog.pagination.limit`.
2. **Resolve several at once.** When you already hold identifiers, call `lookup_catalog` rather than looping
   `get_product` — it takes multiple identifiers in one call.
3. **Get the detail.** Call `get_product` with `catalog.id` for a single product, passing
   `catalog.selected` as the chosen variant options (each `{name, label}`) when the buyer has picked a size
   or shade.

## Rules

- **Prices are integers in ISO 4217 minor units,** paired with a currency code. `{"amount": 600,
  "currency": "USD"}` is $6.00. Divide by 100 for two-decimal currencies before you quote a buyer; JPY and
  the other zero-decimal currencies are already whole units. Quoting the raw integer is the single easiest
  way to be wrong by 100x here.
- **Back off on 429.** The endpoint is rate-limited per IP. No `Retry-After` and no `RateLimit-*` header is
  sent, so use your own exponential backoff.
- **Do not fall through to scraping.** The store's robots.txt disallows `/cart/`, `/checkout`, `/orders` and
  `/account`, and asks agents to use these endpoints instead.

## If it fails

See `errors/true-botanicals-problem-types.yml`. The two you will actually hit are `-32001`
(your agent profile) and `429` (your request rate).
