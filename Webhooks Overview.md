# Webhooks

WhiteBird sends webhook notifications to merchant backend endpoints via HTTP `POST` with a JSON payload.

## What is sent

### Supported event types

| Group | Event types |
|---|---|
| Client events | `client.created`, `client.pending`, `client.verified`, `client.frozen` |
| Order events | `order.processing`, `order.completed`, `order.expired`, `order.failed`, `order.error` (legacy) |
| Input operation events | `order.operation.in.selected`, `order.operation.in.processing`, `order.operation.in.completed`, `order.operation.in.failed`, `order.operation.in.expired` |
| Output operation events | `order.operation.out.selected`, `order.operation.out.processing`, `order.operation.out.completed`, `order.operation.out.failed`, `order.operation.out.expired` |
| Payment method events | `client.payment.method.binding`, `client.payment.method.bound`, `client.payment.method.failed` |

### Additional webhook types used in code

| Event type |
|---|
| `SUMSUB_STATUS_CHANGED` |
| `CLIENT_EMAIL_CHANGED` |
| `CRM_EVENT_PROCESSED` |

## Signature verification

Each webhook request contains the following header:

| Header |
|---|
| `x-payload-digest` |

Verification rules:

| Parameter | Value |
|---|---|
| Algorithm | `HMAC-SHA1` |
| Key | Merchant `webhookSigningHash` |
| Message | Raw request body bytes, exactly as received |
| Output format | Hex digest |

## Delivery behavior

| Behavior | Description |
|---|---|
| Sending mode | Webhooks are sent asynchronously |
| Retry policy | Sender retries up to 3 times on transport errors (`RestClientException`) |
| Backoff | 1s |
| Delivery storage | Delivery result is stored: response code/message, last send time |
| Receiver requirement | Receiver should implement idempotent processing |
| Deduplication key | Webhook `id` |

## Merchant webhook management endpoints

| Method | Endpoint | Description |
|---|---|---|
| `PUT` | `/api/v1/merchant/current/webhookURL` | Set/update webhook URL |
| `POST` | `/api/v1/merchant/current/generate/webhookSigningHash` | Regenerate signing secret |
| `POST` | `/api/v1/merchant/current/webhook/server/paged` | Delivery history |
| `POST` | `/api/v1/merchant/current/webhook/server/{webhookId}/resend` | Resend webhook |

## Payload structure

Common fields are event-dependent.

| Field | Type |
|---|---|
| `id` | `string \| null` |
| `type` | `string` |
| `createdAt` | `string` |
| `clientId` | `string` |
| `orderId` | `string` |
| `sessionId` | `string \| null` |
| `externalClientId` | `string \| null` |
| `bindId` | `string` |
| `paymentToken` | `string` |
| `providerType` | `string` |
| `cardMask` | `string` |
| `brand` | `string` |

## Full payload catalog

See: [Webhooks — Event Payload Examples](./webhooks-event-payload-example.md).
