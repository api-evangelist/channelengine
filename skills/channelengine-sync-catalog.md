---
name: Sync product catalog and stock to ChannelEngine
description: Upsert products and keep stock and prices in sync across marketplaces via the ChannelEngine Merchant API.
api: ChannelEngine Merchant API
operations: [productCreate, productGetByFilter, offerStockUpdate, offerStockPriceUpdate, listedProductGetByFilter]
---

# Sync product catalog and stock to ChannelEngine

Keep your ChannelEngine catalog and availability current.

## Auth & base
- Base URL: `https://{subdomain}.channelengine.net/api`; authenticate with the `apikey` query parameter.
- Product writes are an **upsert** keyed on your own `MerchantProductNo` — idempotent, so re-sending the same payload converges to the same state.

## Steps
1. **Upsert products** — call `productCreate` with your product content. Omitting a default attribute (name, description, price, images, optionally stock) **clears** it, so always send the full default-attribute set on upsert. Batch large loads into chunks of ~10,000 and do not run overlapping batches concurrently.
2. **Verify** — call `productGetByFilter` (or `productGetByMerchantProductNo`) to confirm content, and `listedProductGetByFilter` to see what is actually listed on channels.
3. **Update availability** — call `offerStockUpdate` to set absolute stock, or `offerStockPriceUpdate` to set stock and price together. Absolute-value sets are idempotent and safe to retry.

## Rules
- Prefer **incremental** updates; keep a manual full re-export path for recovery.
- On `200` from bulk product/offer calls, inspect `ValidationErrors` — individual items can fail while the request succeeds.
- Respect `x-rate-limit-remaining` / `retry-after`; back off on `429`.
