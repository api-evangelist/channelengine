---
name: Handle ChannelEngine returns and cancellations
description: Retrieve, declare, acknowledge and update marketplace returns and cancellations via the ChannelEngine Merchant API.
api: ChannelEngine Merchant API
operations: [returnGetUnhandled, returnGetReturns, returnDeclareForMerchant, returnAcknowledge, returnUpdateForMerchant, cancellationCreate]
---

# Handle ChannelEngine returns and cancellations

Process marketplace returns and cancellations end to end.

## Auth & base
- Base URL: `https://{subdomain}.channelengine.net/api`; authenticate with the `apikey` query parameter.
- Envelope responses: check `Success` / `StatusCode`; page collections with `Page` (100/page).

## Steps
1. **Find work** — call `returnGetUnhandled` for returns needing action, or `returnGetReturns` / `returnGetByMerchantOrderNo` to look up specific returns.
2. **Declare a merchant-initiated return** — call `returnDeclareForMerchant` when the merchant originates the return.
3. **Acknowledge** — call `returnAcknowledge` to confirm receipt of a channel-declared return (idempotent).
4. **Update status** — call `returnUpdateForMerchant` to move the return through its lifecycle (received, refunded, rejected).
5. **Cancel** — for order cancellations, call `cancellationCreate`; review with `cancellationGetForMerchant`.

## Rules
- Retry on 4xx/5xx; acknowledge/update operations are idempotent.
- Keep `LogId` from responses for support correlation.
- Back off on `429` using `retry-after`.
