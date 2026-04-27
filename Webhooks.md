# Webhooks

WhiteBird webhooks notify partner systems about client, order, operation, and payment method events.

Use webhooks to keep your backend, UI, CRM, and reconciliation systems synchronized with WhiteBird state changes.



## Supported Events Overview

| Category | Events |
|---|---|
| Client | `client.created`, `client.pending`, `client.verified`, `client.frozen` |
| Order | `order.processing`, `order.completed`, `order.expired`, `order.failed`, `order.error` |
| Input Operation | `order.operation.in.selected`, `order.operation.in.processing`, `order.operation.in.completed`, `order.operation.in.failed`, `order.operation.in.expired` |
| Output Operation | `order.operation.out.selected`, `order.operation.out.processing`, `order.operation.out.completed`, `order.operation.out.failed`, `order.operation.out.expired` |
| Payment Method | `client.payment.method.binding`, `client.payment.method.bound`, `client.payment.method.failed` |

> `order.error` is a legacy / deprecated event.



## How it works

WhiteBird sends webhook notifications as HTTP `POST` requests to the merchant webhook URL configured in the merchant account.

General flow:

1. A domain event occurs in WhiteBird.
2. WhiteBird builds a JSON payload with event-specific fields.
3. WhiteBird signs the raw payload body and sends it with the `x-payload-digest` header.
4. Merchant endpoint verifies the signature and processes the event.
5. Merchant returns `2xx` to confirm successful processing.



## Related webhook management endpoints

| Method | Endpoint | Description |
|---|---|---|
| `PUT` | `/api/v1/merchant/current/webhookURL` | Set or update webhook URL |
| `POST` | `/api/v1/merchant/current/generate/webhookSigningHash` | Rotate signing secret |
| `POST` | `/api/v1/merchant/current/webhook/server/paged` | Webhook delivery history |
| `POST` | `/api/v1/merchant/current/webhook/server/{webhookId}/resend` | Resend webhook |

## Signature verification

WhiteBird signs each webhook request and sends signature in header `x-payload-digest`.

Verification rules:
- Algorithm: `HMAC-SHA1`
- Key: merchant `webhookSigningHash`
- Message: **raw HTTP request body bytes** (exactly as received)
- Encoding: hex digest
- Compare computed digest with `x-payload-digest` using constant-time compare

If signature validation fails, return `401` or `403`.

## Delivery and idempotency

- Webhooks are delivered asynchronously and may be retried.
- Treat webhook processing as idempotent.
- Use webhook `id` as deduplication key.
- Return `2xx` only after event is accepted for processing.
- Return `5xx` for temporary processing failures to allow retry.

## Common Payload Fields

| Field | Type | Description |
|---|---|---|
| `id` | string | Unique webhook event ID |
| `type` | string | Webhook event type |
| `createdAt` | string | Event creation timestamp |
| `clientId` | string | WhiteBird client ID |
| `orderId` | string | WhiteBird order ID |
| `sessionId` | string/null | SDK session ID |
| `externalClientId` | string/null | Partner client identifier |
| `bindId` | string | Payment method binding ID |
| `paymentToken` | string | Tokenized payment method identifier |
| `providerType` | string | Payment provider type |
| `cardMask` | string | Masked card digits |
| `brand` | string | Payment card brand |

> Not every webhook contains all fields. Payload fields depend on the event category.



# Event Payloads

## Client Events

| Event | Description |
|---|---|
| `client.created` | Triggered when a new client is created |
| `client.pending` | Triggered when a client status becomes pending |
| `client.verified` | Triggered when a client is verified |
| `client.frozen` | Triggered when a client is frozen |

### Example payload

```json
{
  "id": "webhook-id",
  "type": "client.verified",
  "createdAt": "2024-05-23T08:30:21+0000",
  "clientId": "0d58e7ec-0369-48d7-9804-90c6b23a52be"
}
```



## Order Events

| Event | Description |
|---|---|
| `order.processing` | Triggered when an order is created or starts processing |
| `order.completed` | Triggered when an order is completed |
| `order.expired` | Triggered when an order expires |
| `order.failed` | Triggered when an order fails |
| `order.error` | Legacy / deprecated event |

### Example payload

```json
{
  "id": "webhook-id",
  "type": "order.completed",
  "createdAt": "2024-05-23T08:44:58+0000",
  "orderId": "3c0130a4-06f2-4d18-bf39-27153caff6f5",
  "sessionId": null,
  "externalClientId": "external-client-id-5"
}
```



## Input Operation Events

| Event | Description |
|---|---|
| `order.operation.in.selected` | Triggered when an input operation is selected |
| `order.operation.in.processing` | Triggered when an input operation starts processing |
| `order.operation.in.completed` | Triggered when an input operation is completed |
| `order.operation.in.failed` | Triggered when an input operation fails |
| `order.operation.in.expired` | Triggered when an input operation expires |

### Example payload

```json
{
  "id": "webhook-id",
  "type": "order.operation.in.processing",
  "createdAt": "2024-05-23T08:44:58+0000",
  "orderId": "3c0130a4-06f2-4d18-bf39-27153caff6f5",
  "sessionId": null,
  "externalClientId": "external-client-id-5"
}
```



## Output Operation Events

| Event | Description |
|---|---|
| `order.operation.out.selected` | Triggered when an output operation is selected |
| `order.operation.out.processing` | Triggered when an output operation starts processing |
| `order.operation.out.completed` | Triggered when an output operation is completed |
| `order.operation.out.failed` | Triggered when an output operation fails |
| `order.operation.out.expired` | Triggered when an output operation expires |

### Example payload

```json
{
  "id": "webhook-id",
  "type": "order.operation.out.completed",
  "createdAt": "2024-05-23T08:44:58+0000",
  "orderId": "3c0130a4-06f2-4d18-bf39-27153caff6f5",
  "sessionId": null,
  "externalClientId": "external-client-id-5"
}
```



## Payment Method Events

| Event | Description |
|---|---|
| `client.payment.method.binding` | Triggered when a client starts binding a new payment method |
| `client.payment.method.bound` | Triggered when a client successfully binds a new payment method |
| `client.payment.method.failed` | Triggered when payment method binding fails |

### Example: `client.payment.method.binding`

```json
{
  "id": "webhook-id",
  "clientId": "ed8ff528-3017-45bc-9d4d-f90e58f91bf9",
  "bindId": "856c460d-7081-433b-904d-c46e313b1225",
  "providerType": "ASSIST",
  "createdAt": "2025-04-21T09:00:17+0000",
  "type": "client.payment.method.binding"
}
```

### Example: `client.payment.method.bound`

```json
{
  "id": "webhook-id",
  "clientId": "ed8ff528-3017-45bc-9d4d-f90e58f91bf9",
  "paymentToken": "7ee5900d-7a02-4bcf-a757-7a7b2fce462d",
  "providerType": "ASSIST",
  "createdAt": "2025-04-21T13:51:26+0000",
  "type": "client.payment.method.bound"
}
```

### Example: `client.payment.method.failed`

```json
{
  "id": "webhook-id",
  "clientId": "5646c1b7-d934-44ce-8490-938feb810910",
  "bindId": "97d009fe-80e6-426d-8ea2-9784f676e08e",
  "cardMask": "0380",
  "brand": "MASTERCARD",
  "providerType": "ALFA",
  "createdAt": "2025-04-22T07:47:31+0000",
  "type": "client.payment.method.failed"
}
```
