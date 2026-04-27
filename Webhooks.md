# Webhooks

WhiteBird webhooks allow partner systems to receive notifications about client, order, operation, and payment method events.

The partner should expose a backend endpoint that accepts `POST` requests with a JSON payload.

---

## 1. How it works

WhiteBird sends webhook notifications as HTTP `POST` requests to the merchant webhook URL configured in the merchant account.

General flow:

1. A domain event occurs in WhiteBird: client status change, order status change, operation status change, or payment method event.
2. WhiteBird builds a JSON payload with event-specific fields.
3. WhiteBird signs the raw payload body and sends it with the `x-payload-digest` header.
4. Merchant endpoint verifies the signature and processes the event.
5. Merchant returns `2xx` to confirm successful processing.

Related webhook management endpoints:

- `PUT /api/v1/merchant/current/webhookURL` — set or update webhook URL
- `POST /api/v1/merchant/current/generate/webhookSigningHash` — rotate signing secret
- `POST /api/v1/merchant/current/webhook/server/paged` — webhook delivery history
- `POST /api/v1/merchant/current/webhook/server/{webhookId}/resend` — resend webhook

---

## 2. Common use cases

Typical partner use cases:

- Synchronize client lifecycle in partner CRM
- Track order lifecycle and update user-facing UI
- Track operation-level progress for deposits, withdrawals, and payouts
- Process payment method lifecycle events
- Trigger internal notifications, compliance checks, and support actions
- Build reconciliation pipelines between WhiteBird and partner systems

---

## 3. Supported events

### Client events

- `client.created`
- `client.pending`
- `client.verified`
- `client.frozen`

### Order events

- `order.processing`
- `order.completed`
- `order.expired`
- `order.failed`
- `order.error` — legacy / deprecated

### Input operation events

- `order.operation.in.selected`
- `order.operation.in.processing`
- `order.operation.in.completed`
- `order.operation.in.failed`
- `order.operation.in.expired`

### Output operation events

- `order.operation.out.selected`
- `order.operation.out.processing`
- `order.operation.out.completed`
- `order.operation.out.failed`
- `order.operation.out.expired`

### Payment method events

- `client.payment.method.binding`
- `client.payment.method.bound`
- `client.payment.method.failed`

---

# Client Webhooks

## `client.created`

This event is triggered when a new client is created.

```json
{
  "id": "webhook-id",
  "type": "client.created",
  "createdAt": "2024-05-23T08:30:21+0000",
  "clientId": "0d58e7ec-0369-48d7-9804-90c6b23a52be"
}
```

---

## `client.pending`

This event is triggered when a client's status is set to pending.

```json
{
  "id": "webhook-id",
  "type": "client.pending",
  "createdAt": "2024-05-23T08:30:21+0000",
  "clientId": "0d58e7ec-0369-48d7-9804-90c6b23a52be"
}
```

---

## `client.verified`

This event is triggered when a client is verified.

```json
{
  "id": "webhook-id",
  "type": "client.verified",
  "createdAt": "2024-05-23T08:30:21+0000",
  "clientId": "0d58e7ec-0369-48d7-9804-90c6b23a52be"
}
```

---

## `client.frozen`

This event is triggered when a client is frozen.

```json
{
  "id": "webhook-id",
  "type": "client.frozen",
  "createdAt": "2024-05-23T08:30:21+0000",
  "clientId": "0d58e7ec-0369-48d7-9804-90c6b23a52be"
}
```

---

# Order Webhooks

## `order.processing`

This event is triggered when a new order is created or starts processing.

```json
{
  "id": "webhook-id",
  "type": "order.processing",
  "createdAt": "2024-05-23T08:44:58+0000",
  "orderId": "3c0130a4-06f2-4d18-bf39-27153caff6f5",
  "sessionId": null,
  "externalClientId": "external-client-id-5"
}
```

---

## `order.completed`

This event is triggered when an order is completed.

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

---

## `order.expired`

This event is triggered when an order is expired.

```json
{
  "id": "webhook-id",
  "type": "order.expired",
  "createdAt": "2024-05-23T08:44:58+0000",
  "orderId": "3c0130a4-06f2-4d18-bf39-27153caff6f5",
  "sessionId": null,
  "externalClientId": "external-client-id-5"
}
```

---

## `order.failed`

This event is triggered when an order fails.

```json
{
  "id": "webhook-id",
  "type": "order.failed",
  "createdAt": "2024-05-23T08:44:58+0000",
  "orderId": "3c0130a4-06f2-4d18-bf39-27153caff6f5",
  "sessionId": null,
  "externalClientId": "external-client-id-5"
}
```

---

## `order.error`

This event is legacy / deprecated.

```json
{
  "id": "webhook-id",
  "type": "order.error",
  "createdAt": "2024-05-23T08:44:58+0000",
  "orderId": "3c0130a4-06f2-4d18-bf39-27153caff6f5",
  "sessionId": null,
  "externalClientId": "external-client-id-5"
}
```

---

# Input Operation Webhooks

## `order.operation.in.selected`

This event is triggered when an input operation is selected.

```json
{
  "id": "webhook-id",
  "type": "order.operation.in.selected",
  "createdAt": "2024-05-23T08:44:58+0000",
  "orderId": "3c0130a4-06f2-4d18-bf39-27153caff6f5",
  "sessionId": null,
  "externalClientId": "external-client-id-5"
}
```

---

## `order.operation.in.processing`

This event is triggered when an input operation starts processing.

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

---

## `order.operation.in.completed`

This event is triggered when an input operation is completed.

```json
{
  "id": "webhook-id",
  "type": "order.operation.in.completed",
  "createdAt": "2024-05-23T08:44:58+0000",
  "orderId": "3c0130a4-06f2-4d18-bf39-27153caff6f5",
  "sessionId": null,
  "externalClientId": "external-client-id-5"
}
```

---

## `order.operation.in.failed`

This event is triggered when an input operation fails.

```json
{
  "id": "webhook-id",
  "type": "order.operation.in.failed",
  "createdAt": "2024-05-23T08:44:58+0000",
  "orderId": "3c0130a4-06f2-4d18-bf39-27153caff6f5",
  "sessionId": null,
  "externalClientId": "external-client-id-5"
}
```

---

## `order.operation.in.expired`

This event is triggered when an input operation is expired.

```json
{
  "id": "webhook-id",
  "type": "order.operation.in.expired",
  "createdAt": "2024-05-23T08:44:58+0000",
  "orderId": "3c0130a4-06f2-4d18-bf39-27153caff6f5",
  "sessionId": null,
  "externalClientId": "external-client-id-5"
}
```

---

# Output Operation Webhooks

## `order.operation.out.selected`

This event is triggered when an output operation is selected.

```json
{
  "id": "webhook-id",
  "type": "order.operation.out.selected",
  "createdAt": "2024-05-23T08:44:58+0000",
  "orderId": "3c0130a4-06f2-4d18-bf39-27153caff6f5",
  "sessionId": null,
  "externalClientId": "external-client-id-5"
}
```

---

## `order.operation.out.processing`

This event is triggered when an output operation starts processing.

```json
{
  "id": "webhook-id",
  "type": "order.operation.out.processing",
  "createdAt": "2024-05-23T08:44:58+0000",
  "orderId": "3c0130a4-06f2-4d18-bf39-27153caff6f5",
  "sessionId": null,
  "externalClientId": "external-client-id-5"
}
```

---

## `order.operation.out.completed`

This event is triggered when an output operation is completed.

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

---

## `order.operation.out.failed`

This event is triggered when an output operation fails.

```json
{
  "id": "webhook-id",
  "type": "order.operation.out.failed",
  "createdAt": "2024-05-23T08:44:58+0000",
  "orderId": "3c0130a4-06f2-4d18-bf39-27153caff6f5",
  "sessionId": null,
  "externalClientId": "external-client-id-5"
}
```

---

## `order.operation.out.expired`

This event is triggered when an output operation is expired.

```json
{
  "id": "webhook-id",
  "type": "order.operation.out.expired",
  "createdAt": "2024-05-23T08:44:58+0000",
  "orderId": "3c0130a4-06f2-4d18-bf39-27153caff6f5",
  "sessionId": null,
  "externalClientId": "external-client-id-5"
}
```

---

# Payment Method Webhooks

## `client.payment.method.binding`

This event is triggered when a client starts binding a new payment method.

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

---

## `client.payment.method.bound`

This event is triggered when a client has successfully bound a new payment method.

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

---

## `client.payment.method.failed`

This event is triggered when a client's payment method binding fails.

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

---
