# Webhooks

WhiteBird sends webhook notifications to merchant backend endpoints via HTTP `POST` with JSON payload.

## What is sent

Supported event groups:
- Client events
- Order events
- Order operation events
- Payment method events

Supported event types:
- `client.created`, `client.pending`, `client.verified`, `client.frozen`
- `order.processing`, `order.completed`, `order.expired`, `order.failed`, `order.error` (legacy)
- `order.operation.in.selected`, `order.operation.in.processing`, `order.operation.in.completed`, `order.operation.in.failed`, `order.operation.in.expired`
- `order.operation.out.selected`, `order.operation.out.processing`, `order.operation.out.completed`, `order.operation.out.failed`, `order.operation.out.expired`
- `client.payment.method.binding`, `client.payment.method.bound`, `client.payment.method.failed`

Additional webhook types used in code:
- `SUMSUB_STATUS_CHANGED`
- `CLIENT_EMAIL_CHANGED`
- `CRM_EVENT_PROCESSED`

## Signature verification

Each webhook request contains header:
- `x-payload-digest`

Verification rules:
- Algorithm: `HMAC-SHA1`
- Key: merchant `webhookSigningHash`
- Message: raw request body bytes (exactly as received)
- Output format: hex digest

## Delivery behavior

- Webhooks are sent asynchronously.
- Sender retries up to 3 times on transport errors (`RestClientException`) with 1s backoff.
- Delivery result is stored (response code/message, last send time).
- Receiver should implement idempotent processing (deduplicate by webhook `id`).

## Merchant webhook management endpoints

- `PUT /api/v1/merchant/current/webhookURL` — set/update webhook URL
- `POST /api/v1/merchant/current/generate/webhookSigningHash` — regenerate signing secret
- `POST /api/v1/merchant/current/webhook/server/paged` — delivery history
- `POST /api/v1/merchant/current/webhook/server/{webhookId}/resend` — resend webhook

## Payload structure

Common fields (event-dependent):
- `id` (`string | null`)
- `type` (`string`)
- `createdAt` (`string`)
- `clientId` (`string`)
- `orderId` (`string`)
- `sessionId` (`string | null`)
- `externalClientId` (`string | null`)
- `bindId` (`string`)
- `paymentToken` (`string`)
- `providerType` (`string`)
- `cardMask` (`string`)
- `brand` (`string`)

## Full payload catalog

See: **Webhooks — Event Payload Examples** (separate page).
