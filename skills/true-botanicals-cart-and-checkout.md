---
name: true-botanicals-cart-and-checkout
description: Build a cart and prepare a checkout at True Botanicals over UCP/MCP, stopping at the buyer-approval boundary.
api: True Botanicals UCP Agent Commerce (MCP)
endpoint: https://truebotanicals.com/api/ucp/mcp
operations:
  - create_cart
  - get_cart
  - update_cart
  - cancel_cart
  - create_checkout
  - get_checkout
  - update_checkout
  - complete_checkout
  - cancel_checkout
  - get_order
generated: '2026-08-30'
method: generated
source: mcp/true-botanicals-tools-list.json (live tools/list, 2026-08-30)
---

# Cart and checkout at True Botanicals

This skill writes. Read `## The approval boundary` before anything else.

## The approval boundary

True Botanicals states, in both `robots.txt` and `llms.txt`, that checkout, payment and order placement must
not be completed automatically — no scripted form fills, no browser automation, no end-to-end flow that
finalizes payment without an explicit, contemporaneous human approval step.

Everything up to `complete_checkout` is reversible: `cancel_cart` unwinds a cart and `cancel_checkout`
unwinds a checkout. `complete_checkout` is not. There is **no refund, void or reverse tool** in this tool
set, and `get_order` is read-only, so once it succeeds you have no programmatic way back — reversal becomes
a human path through https://truebotanicals.com/policies/refund-policy. No cancellation window is published
anywhere, so do not promise the buyer one.

If you cannot obtain approval at the moment of payment, `llms.txt` directs you to route the purchase through
the Shop skill at https://shop.app/SKILL.md instead. That is Shopify's surface, not True Botanicals'.

## Steps

1. **Cart.** `create_cart` with `cart.line_items` (each `{id, quantity, item}`) and
   `cart.context.address_country` / `.currency`. Keep the returned cart id — it is a
   `gid://shopify/Cart/...` URI and it is your only handle.
2. **Adjust.** `update_cart` with the id and the changed `cart` object; `get_cart` to re-read totals;
   `cancel_cart` to abandon. Add promotion codes as `cart.discounts.codes`.
3. **Checkout.** `create_checkout`, passing `checkout.cart_id` to carry the cart across, plus
   `checkout.buyer.email` and any `checkout.attribution` you are entitled to pass. Keep the checkout id.
4. **Fulfillment.** `update_checkout` with `checkout.fulfillment.methods` to set the destination and
   shipping method. This store declares one method combination — `["shipping"]` — and no multi-destination
   support, in its `/.well-known/ucp` profile.
5. **Payment.** Select an instrument under `checkout.payment.instruments`. The store declares three payment
   handlers: Google Pay (`com.google.pay`, merchant `True Botanicals`), Shopify card
   (`dev.shopify.card`, Visa/Mastercard/Amex/Discover/Diners), and Shop Pay (`dev.shopify.shop_pay`).
6. **Stop. Ask the human.** Present the totals in major units and get explicit approval.
7. **Complete.** Only then `complete_checkout`. Treat a success as final.
8. **Order.** `get_order` needs a bearer JWT — without one it returns HTTP 403 and JSON-RPC `-32000`
   `AuthenticationRequired`. Mint one per https://shopify.dev/docs/agents/get-started/authentication.

## Rules

- **Every call carries `meta.ucp-agent.profile`,** a fetchable URI for your own UCP agent profile.
  Unresolvable means HTTP 422 / `-32001` before any business logic runs.
- **Prices are ISO 4217 minor-unit integers.** Convert before quoting.
- **There is no idempotency key.** Nothing in these schemas accepts an `Idempotency-Key` header or an
  idempotency field. Retrying `create_cart` or `create_checkout` after a timeout creates a *second* object.
  Retry `update_*` against a retained id instead, and reconcile with `get_cart` / `get_checkout` before
  ever re-creating.
- **There is no test mode.** No sandbox, no test payment instrument, no simulation flag is published. Any
  `complete_checkout` you send is a real purchase.
- **Correlate with `x-request-id`,** returned on every response, when you need support.
