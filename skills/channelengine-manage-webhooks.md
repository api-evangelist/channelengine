---
name: Subscribe to ChannelEngine webhooks
description: Register and manage webhook subscriptions so an agent reacts to order, product, return and shipment changes.
api: ChannelEngine Merchant API
operations: [webhooksCreate, webhooksGetAll, webhooksUpdate, webhooksDelete]
---

# Subscribe to ChannelEngine webhooks

Get near-real-time notifications instead of polling.

## Auth & base
- Base URL: `https://{subdomain}.channelengine.net/api`; authenticate with the `apikey` query parameter.

## Steps
1. **List existing** — call `webhooksGetAll` to see current subscriptions.
2. **Create** — call `webhooksCreate` with your HTTPS callback URL and the events you want. Available events: Orders create, Orders change, Products change, Product bundles change, Returns change, Shipments change, Notifications create.
3. **Update** — call `webhooksUpdate` to change the callback URL or subscribed events.
4. **Delete** — call `webhooksDelete` to remove a subscription.

## Rules
- Serve an HTTPS endpoint that responds quickly (2xx) and process asynchronously.
- Treat webhook delivery as at-least-once; make handlers idempotent (dedupe on the resource id).
- On a webhook, call the relevant read operation (e.g. `orderGetByFilter`, `productGetByFilter`) to fetch full current state.
