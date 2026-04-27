# Webhooks - Event Payload Examples

This page contains example webhook requests from WhiteBird and expected merchant responses.

Each event example includes:

- Purpose
- Webhook request body
- Merchant response example

---

## Common request headers

```http
Content-Type: application/json
x-payload-digest: <hmac_sha1_hex_digest>
```

---

## Common merchant response

Return `2xx` after successful acceptance.

```http
HTTP/1.1 200 OK
Content-Type: application/json
```

Optional response body:

```json
{
  "ok": true
}
```

---

## Client events

| Event | Purpose |
|---|---|
| `client.created` | New WhiteBird client is created |
| `client.pending` | Client status changed to pending |
| `client.verified` | Client status changed to verified |
| `client.frozen` | Client status changed to frozen |

---

### `client.created`

Purpose: new WhiteBird client is created.

```json
{
  "id": "webhook-id",
  "type": "client.created",
  "createdAt": "2024-05-23T08:30:21+0000",
  "clientId": "0d58e7ec-0369-48d7-9804-90c6b23a52be"
}
```

Response:

```json
{
  "ok": true
}
```

---

### `client.pending`

Purpose: client status changed to pending.

```json
{
  "id": "webhook-id",
  "type": "client.pending",
  "createdAt": "2024-05-23T08:30:21+0000",
  "clientId": "0d58e7ec-0369-48d7-9804-90c6b23a52be"
}
```

---

### `client.verified`

Purpose: client status changed to verified.

```json
{
  "id": "webhook-id",
  "type": "client.verified",
  "createdAt": "2024-05-23T08:30:21+0000",
  "clientId": "0d58e7ec-0369-48d7-9804-90c6b23a52be"
}
```

---

### `client.frozen`

Purpose: client status changed to frozen.

```json
{
  "id": "webhook-id",
  "type": "client.frozen",
  "createdAt": "2024-05-23T08:30:21+0000",
  "clientId": "0d58e7ec-0369-48d7-9804-90c6b23a52be"
}
```

---

## Order events

| Event | Purpose |
|---|---|
| `order.processing` | Order created or entered processing |
| `order.completed` | Order completed |
| `order.expired` | Order expired |
| `order.failed` | Order failed |
| `order.error` | Legacy error event |

---

### `order.processing`

Purpose: order created or entered processing.

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

### `order.completed`

Purpose: order completed.

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

### `order.expired`

Purpose: order expired.

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

### `order.failed`

Purpose: order failed.

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

### `order.error`

Purpose: legacy error event.

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

## Input operation events

| Event | Purpose |
|---|---|
| `order.operation.in.selected` | Input operation selected |
| `order.operation.in.processing` | Input operation started |
| `order.operation.in.completed` | Input operation completed |
| `order.operation.in.failed` | Input operation failed |
| `order.operation.in.expired` | Input operation expired |

---

### `order.operation.in.selected`

Purpose: input operation selected.

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

### `order.operation.in.processing`

Purpose: input operation started.

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

### `order.operation.in.completed`

Purpose: input operation completed.

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

### `order.operation.in.failed`

Purpose: input operation failed.

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

### `order.operation.in.expired`

Purpose: input operation expired.

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

## Output operation events

| Event | Purpose |
|---|---|
| `order.operation.out.selected` | Output operation selected |
| `order.operation.out.processing` | Output operation started |
| `order.operation.out.completed` | Output operation completed |
| `order.operation.out.failed` | Output operation failed |
| `order.operation.out.expired` | Output operation expired |

---

### `order.operation.out.selected`

Purpose: output operation selected.

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

### `order.operation.out.processing`

Purpose: output operation started.

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

### `order.operation.out.completed`

Purpose: output operation completed.

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

### `order.operation.out.failed`

Purpose: output operation failed.

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

### `order.operation.out.expired`

Purpose: output operation expired.

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

## Payment method events

| Event | Purpose |
|---|---|
| `client.payment.method.binding` | Payment method binding started |
| `client.payment.method.bound` | Payment method bound successfully |
| `client.payment.method.failed` | Payment method binding failed |

---

### `client.payment.method.binding`

Purpose: payment method binding started.

```json
{
  "id": "webhook-id",
  "type": "client.payment.method.binding",
  "createdAt": "2025-04-21T09:00:17+0000",
  "clientId": "ed8ff528-3017-45bc-9d4d-f90e58f91bf9",
  "bindId": "856c460d-7081-433b-904d-c46e313b1225",
  "providerType": "ASSIST"
}
```

---

### `client.payment.method.bound`

Purpose: payment method bound successfully.

```json
{
  "id": "webhook-id",
  "type": "client.payment.method.bound",
  "createdAt": "2025-04-21T13:51:26+0000",
  "clientId": "ed8ff528-3017-45bc-9d4d-f90e58f91bf9",
  "paymentToken": "7ee5900d-7a02-4bcf-a757-7a7b2fce462d",
  "providerType": "ASSIST"
}
```

---

### `client.payment.method.failed`

Purpose: payment method binding failed.

```json
{
  "id": "webhook-id",
  "type": "client.payment.method.failed",
  "createdAt": "2025-04-22T07:47:31+0000",
  "clientId": "5646c1b7-d934-44ce-8490-938feb810910",
  "bindId": "97d009fe-80e6-426d-8ea2-9784f676e08e",
  "cardMask": "0380",
  "brand": "MASTERCARD",
  "providerType": "ALFA"
}
```

---

## Additional webhook examples

| Event | Purpose |
|---|---|
| `SUMSUB_STATUS_CHANGED` | Sumsub status update |
| `CLIENT_EMAIL_CHANGED` | Client email changed |
| `CRM_EVENT_PROCESSED` | CRM event processed |

---

### `SUMSUB_STATUS_CHANGED`

Purpose: Sumsub status update.

```json
{
  "type": "SUMSUB_STATUS_CHANGED",
  "createdAt": "2025-04-27T16:10:00+0000",
  "clientId": "0d58e7ec-0369-48d7-9804-90c6b23a52be",
  "processName": "applicantReviewed",
  "levelType": "KYC",
  "reviewResult": "GREEN",
  "actionId": "sumsub-action-id"
}
```

---

### `CLIENT_EMAIL_CHANGED`

Purpose: client email changed.

```json
{
  "type": "CLIENT_EMAIL_CHANGED",
  "createdAt": "2025-04-27T16:10:00+0000",
  "clientId": "0d58e7ec-0369-48d7-9804-90c6b23a52be",
  "email": "new-email@example.com"
}
```

---

### `CRM_EVENT_PROCESSED`

Purpose: CRM event processed.

```json
{
  "type": "CRM_EVENT_PROCESSED",
  "createdAt": "2025-04-27T16:10:00+0000",
  "clientId": "0d58e7ec-0369-48d7-9804-90c6b23a52be",
  "topic": "crm-topic",
  "eventType": "crm-event-type"
}
```
