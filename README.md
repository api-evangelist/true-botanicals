# True Botanicals

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

True Botanicals is a clean-luxury skincare company selling clinically tested, MADE SAFE certified face and
body products direct to consumers from [truebotanicals.com](https://truebotanicals.com/). It is not a
developer-tools company: there is no developer portal, no API keys, no SDKs and no API pricing, and this
profile does not claim otherwise.

What it does have is an agent-commerce surface, served on its own domain by its Shopify storefront and
probed live on 2026-08-30:

| Surface | URL | Result |
|---|---|---|
| Agent instructions | `/llms.txt`, `/agents.md` | 200 — a real agent-instruction document, advertised from `robots.txt` and a dedicated agentic-discovery sitemap |
| UCP merchant profile | `/.well-known/ucp` | 200 — Universal Commerce Protocol `2026-08-25`, naming merchant "True Botanicals", shop 5451009 |
| MCP endpoint | `/api/ucp/mcp` | 200 — `tools/list` returns 13 catalog, cart, checkout and order tools with JSON Schema 2020-12 input schemas |
| Storefront GraphQL | `/api/2026-01/graphql.json` | 200 — keyless introspection, 424 types; `shop.name` returns "True Botanicals" |
| OAuth / OIDC discovery | `/.well-known/openid-configuration`, `/.well-known/oauth-authorization-server` | 200 — Shopify customer-accounts issuer for this shop |
| Agent card | `/.well-known/agent-card.json`, `/.well-known/agent.json` | 404 on every host — no A2A artifact was written |
| security.txt | `/.well-known/security.txt` | 404 — no vulnerability-disclosure or trust-centre programme found |

**Provenance caveat, stated once and stated plainly.** The schemas behind these surfaces are authored by
Shopify and UCP, not by True Botanicals, and every Shopify merchant gets them. They are recorded here because
they are genuinely served from `truebotanicals.com` over this merchant's own data — not as evidence of an
in-house API programme. Each artifact repeats the caveat in its own `x-provenance` block.

**The finding worth knowing.** Discovery is anonymous; invocation is not. `tools/list` needs no credential,
but every `tools/call` requires the *caller* to publish a resolvable UCP agent profile, and `get_order`
requires a bearer JWT on top. There is no idempotency key anywhere in the tool set, no test mode, and no
reversal for `complete_checkout` — carts and checkouts can be cancelled, completed purchases cannot.
