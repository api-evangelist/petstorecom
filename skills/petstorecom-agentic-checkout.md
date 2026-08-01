---
name: Petstore.com agent-driven shopping and checkout
description: >-
  Search Petstore.com's catalog, build a cart, and drive a buyer-approved checkout
  over the store's UCP/MCP endpoint. Payment always requires contemporaneous human
  approval.
api: mcp/petstorecom-mcp.yml
protocol: ucp
transport: mcp
endpoint: https://petstore.com/api/ucp/mcp
operations:
  - search_catalog
  - create_cart
  - create_checkout
  - update_checkout
  - complete_checkout
source: https://petstore.com/agents.md
method: generated
---

# Petstore.com — agent-driven shopping and checkout

Petstore.com is a Shopify store that implements the Universal Commerce Protocol
(UCP) over MCP. Use the MCP endpoint for catalog, cart, and checkout. **Do not
finalize payment without explicit, contemporaneous buyer approval.**

## Prerequisites
- MCP endpoint: `POST https://petstore.com/api/ucp/mcp` (`Content-Type: application/json`, JSON-RPC 2.0).
- Calling tools requires a UCP agent-profile URI; buyer identity uses Shopify
  Customer Account OIDC (see `authentication/petstorecom-authentication.yml`).
- Pass buyer `context.address_country` and `context.currency` for accurate pricing.

## Steps
1. **Discover.** `GET https://petstore.com/.well-known/ucp` to confirm supported
   UCP versions (`2026-04-08`, `2026-01-23`), capabilities, and payment handlers.
2. **Search.** Call `search_catalog` with the buyer's intent to find matching products.
3. **Build a cart.** Call `create_cart` to add the desired items.
4. **Start checkout.** Call `create_checkout` to begin the purchase flow.
5. **Fulfill.** Call `update_checkout` to set the shipping address and method.
6. **Complete — with approval.** Call `complete_checkout` only after the buyer has
   explicitly approved payment at that moment.

## Rules
- **Buyer-approval invariant:** never complete checkout/payment/order placement
  automatically. If you cannot get contemporaneous approval, route the purchase
  through Shop Pay via `https://shop.app/SKILL.md` instead.
- **Rate limits:** the MCP endpoint is rate-limited per IP — back off on HTTP 429.
- **Read-only alternative:** to browse without transacting, use the storefront
  JSON endpoints (`/products/{handle}.json`, `/collections/{handle}/products.json`,
  `/search?q={query}&type=product`) — no authentication required.

## Errors
Errors are JSON-RPC 2.0 error objects, e.g. `-32001 "UCP discovery failed"` when a
required agent profile URI is missing. See `conventions/petstorecom-conventions.yml`.
