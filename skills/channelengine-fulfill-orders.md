---
name: Fulfill new ChannelEngine orders
description: Poll for new marketplace orders, acknowledge them, and ship them through the ChannelEngine Merchant API.
api: ChannelEngine Merchant API
operations: [orderGetNew, orderAcknowledge, shipmentCreate, shipmentGetShipmentLabelCarriers, shipmentShippingLabel]
---

# Fulfill new ChannelEngine orders

Use the ChannelEngine Merchant API to pick up marketplace orders and fulfill them.

## Auth & base
- Base URL: `https://{subdomain}.channelengine.net/api` (use your tenant subdomain).
- Authenticate with the `apikey` query parameter on every call (Settings > Merchant API keys). Treat the key as a secret.
- Responses are wrapped in an envelope: check `Success`, `StatusCode`, and `Message`; collections add `Content`, `Count`, `TotalCount`, `ItemsPerPage`.

## Steps
1. **Get new orders** — call `orderGetNew` to retrieve orders not yet acknowledged. Page with the `Page` parameter (100 per page) until `Count < ItemsPerPage`.
2. **Acknowledge** — for each order, call `orderAcknowledge` so ChannelEngine stops returning it as new. This is idempotent — safe to retry.
3. **Create the shipment** — call `shipmentCreate` (or `shipmentCreateForChannelMethod` to use a channel shipping method). Only include order lines that exist on the original order — an unknown line yields a 404.
4. **Labels (optional)** — call `shipmentGetShipmentLabelCarriers` for available carriers, then `shipmentShippingLabel` to retrieve the label.

## Rules
- Honor rate limits: on `429`, wait `retry-after` seconds; watch `x-rate-limit-remaining`.
- Build retries on 4xx/5xx; on `200` for bulk calls still inspect `ValidationErrors`.
- Keep the returned `LogId` for support correlation.
