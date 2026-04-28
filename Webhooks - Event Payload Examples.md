# Webhooks - Event Payload Examples

This page contains webhook requests sent by WhiteBird and expected merchant behavior.

## Delivery Contract

- Method: `POST`
- Content-Type: `application/json`
- Signature header: `x-payload-digest`
- Signature algorithm: `HMAC-SHA1` over raw request body bytes
- Timeout: `15s` connect / `15s` read
- Retries: up to `3` attempts with `1s` backoff on delivery failures

## Idempotency Guidance (Merchant side)

- Treat `id` as webhook event id.
- Store processed ids in a dedup store (Redis/DB).
- Recommended dedup window: at least `24h` (or your maximum replay window).
- If duplicate `id` is received, return `2xx` and skip re-processing.

---

## Event Catalog

### Client events

| Event | Purpose |
|---|---|
| `client.created` | New WhiteBird client is created |
| `client.pending` | Client status changed to pending |
| `client.verified` | Client status changed to verified |
| `client.frozen` | Client status changed to frozen |

### Order events

| Event | Purpose |
|---|---|
| `order.processing` | Order created or entered processing |
| `order.completed` | Order completed |
| `order.expired` | Order expired |
| `order.failed` | Order failed |
| `order.error` | Legacy deprecated error event |

### Input operation events

| Event | Purpose |
|---|---|
| `order.operation.in.selected` | Input operation selected |
| `order.operation.in.processing` | Input operation started |
| `order.operation.in.completed` | Input operation completed |
| `order.operation.in.failed` | Input operation failed |
| `order.operation.in.expired` | Input operation expired |

### Output operation events

| Event | Purpose |
|---|---|
| `order.operation.out.selected` | Output operation selected |
| `order.operation.out.processing` | Output operation started |
| `order.operation.out.completed` | Output operation completed |
| `order.operation.out.failed` | Output operation failed |
| `order.operation.out.expired` | Output operation expired |

### Payment method events

| Event | Purpose |
|---|---|
| `client.payment.method.binding` | Payment method binding started |
| `client.payment.method.bound` | Payment method bound successfully |
| `client.payment.method.failed` | Payment method binding failed |

### Additional events

| Event | Purpose |
|---|---|
| `SUMSUB_STATUS_CHANGED` | Sumsub status update |
| `CLIENT_EMAIL_CHANGED` | Client email changed |
| `CRM_EVENT_PROCESSED` | CRM event processed |

---

## JSON Schemas (required/optional)

### 1) Client status events schema

Applies to:
`client.created`, `client.pending`, `client.verified`, `client.frozen`

```json
{
  "type": "object",
  "required": ["id", "type", "createdAt", "clientId"],
  "properties": {
    "id": { "type": "string" },
    "type": { "type": "string" },
    "createdAt": { "type": "string", "format": "date-time-like-local" },
    "clientId": { "type": "string", "format": "uuid" }
  },
  "additionalProperties": false
}
```

### 2) Order + operation events schema

Applies to:
`order.*`, `order.operation.in.*`, `order.operation.out.*`

```json
{
  "type": "object",
  "required": ["id", "type", "createdAt", "orderId"],
  "properties": {
    "id": { "type": "string" },
    "type": { "type": "string" },
    "createdAt": { "type": "string", "format": "date-time-like-local" },
    "orderId": { "type": "string", "format": "uuid" },
    "sessionId": { "type": ["string", "null"], "format": "uuid" },
    "externalClientId": { "type": ["string", "null"] }
  },
  "additionalProperties": false
}
```

### 3) Payment method binding schema

`client.payment.method.binding`

```json
{
  "type": "object",
  "required": ["id", "type", "createdAt", "clientId", "bindId", "providerType"],
  "properties": {
    "id": { "type": "string" },
    "type": { "type": "string", "const": "client.payment.method.binding" },
    "createdAt": { "type": "string", "format": "date-time-like-local" },
    "clientId": { "type": "string", "format": "uuid" },
    "bindId": { "type": "string", "format": "uuid" },
    "providerType": { "type": "string" }
  },
  "additionalProperties": false
}
```

### 4) Payment method bound schema

`client.payment.method.bound`

```json
{
  "type": "object",
  "required": ["id", "type", "createdAt", "clientId", "paymentToken", "providerType"],
  "properties": {
    "id": { "type": "string" },
    "type": { "type": "string", "const": "client.payment.method.bound" },
    "createdAt": { "type": "string", "format": "date-time-like-local" },
    "clientId": { "type": "string", "format": "uuid" },
    "paymentToken": { "type": "string" },
    "providerType": { "type": "string" }
  },
  "additionalProperties": false
}
```

### 5) Payment method failed schema

`client.payment.method.failed`

```json
{
  "type": "object",
  "required": ["id", "type", "createdAt", "clientId", "bindId", "providerType"],
  "properties": {
    "id": { "type": "string" },
    "type": { "type": "string", "const": "client.payment.method.failed" },
    "createdAt": { "type": "string", "format": "date-time-like-local" },
    "clientId": { "type": "string", "format": "uuid" },
    "bindId": { "type": "string", "format": "uuid" },
    "cardMask": { "type": ["string", "null"] },
    "brand": { "type": ["string", "null"] },
    "providerType": { "type": "string" }
  },
  "additionalProperties": false
}
```

### 6) SUMSUB status changed schema

`SUMSUB_STATUS_CHANGED`

```json
{
  "type": "object",
  "required": ["type", "createdAt", "clientId", "processName", "levelType", "reviewResult", "actionId"],
  "properties": {
    "type": { "type": "string", "const": "SUMSUB_STATUS_CHANGED" },
    "createdAt": { "type": "string", "format": "date-time-like-local" },
    "clientId": { "type": "string", "format": "uuid" },
    "processName": { "type": "string" },
    "levelType": { "type": "string" },
    "reviewResult": { "type": "string" },
    "actionId": { "type": "string" }
  },
  "additionalProperties": false
}
```

### 7) Client email changed schema

`CLIENT_EMAIL_CHANGED`

```json
{
  "type": "object",
  "required": ["type", "createdAt", "clientId", "email"],
  "properties": {
    "type": { "type": "string", "const": "CLIENT_EMAIL_CHANGED" },
    "createdAt": { "type": "string", "format": "date-time-like-local" },
    "clientId": { "type": "string", "format": "uuid" },
    "email": { "type": "string", "format": "email" }
  },
  "additionalProperties": false
}
```

### 8) CRM event processed schema

`CRM_EVENT_PROCESSED`

```json
{
  "type": "object",
  "required": ["type", "createdAt", "clientId", "topic"],
  "properties": {
    "type": { "type": "string", "const": "CRM_EVENT_PROCESSED" },
    "createdAt": { "type": "string", "format": "date-time-like-local" },
    "clientId": { "type": "string", "format": "uuid" },
    "topic": { "type": "string" }
  },
  "additionalProperties": true
}
```

> Note: implementation currently uses `type` both as event discriminator and as CRM payload field internally; API consumers should rely on `type=CRM_EVENT_PROCESSED` and `topic`.

---

## Signature Verification Examples (`x-payload-digest`)

Use raw HTTP body bytes exactly as received.

### Node.js (crypto)

```js
import crypto from 'crypto';

function verifySignature(rawBodyBuffer, signingSecret, receivedDigest) {
  const expected = crypto
    .createHmac('sha1', signingSecret)
    .update(rawBodyBuffer)
    .digest('hex');

  const a = Buffer.from(expected, 'utf8');
  const b = Buffer.from(receivedDigest || '', 'utf8');
  if (a.length !== b.length) return false;
  return crypto.timingSafeEqual(a, b);
}
```

### Java

```java
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.nio.charset.StandardCharsets;

public static boolean verify(byte[] rawBody, String secret, String receivedDigest) throws Exception {
    Mac mac = Mac.getInstance("HmacSHA1");
    mac.init(new SecretKeySpec(secret.getBytes(StandardCharsets.UTF_8), "HmacSHA1"));
    byte[] digestBytes = mac.doFinal(rawBody);
    StringBuilder sb = new StringBuilder();
    for (byte b : digestBytes) sb.append(String.format("%02x", b));
    String expected = sb.toString();
    return java.security.MessageDigest.isEqual(
            expected.getBytes(StandardCharsets.UTF_8),
            (receivedDigest == null ? "" : receivedDigest).getBytes(StandardCharsets.UTF_8)
    );
}
```

### Python

```python
import hmac
import hashlib

def verify_signature(raw_body: bytes, secret: str, received_digest: str | None) -> bool:
    expected = hmac.new(secret.encode("utf-8"), raw_body, hashlib.sha1).hexdigest()
    return hmac.compare_digest(expected, received_digest or "")
```

---

## Failure / Retry Behavior

### Bad signature example (merchant side)

- Merchant verifies `x-payload-digest`.
- If invalid, merchant returns `401` or `403`.
- WhiteBird retries according to retry policy (up to 3 attempts total).

Example response:

```http
HTTP/1.1 401 Unauthorized
Content-Type: application/json

{"error":"invalid signature"}
```

### 4xx / 5xx behavior

- Any network failure or non-success delivery path is treated as failed send and retried.
- Response body is stored (trimmed to 1000 chars) for diagnostics.
- If all retries fail, webhook remains persisted with last failure code/message.

### Retry sequence example

1. Attempt #1 at `T0` -> fails (`500` or timeout)
2. Attempt #2 at `T0 + 1s` -> fails
3. Attempt #3 at `T0 + 2s` -> success or final failure

---

## Minimal payload examples

### `client.created`

```json
{
  "id": "webhook-id",
  "type": "client.created",
  "createdAt": "2024-05-23T08:30:21",
  "clientId": "0d58e7ec-0369-48d7-9804-90c6b23a52be"
}
```

### `order.failed`

```json
{
  "id": "webhook-id",
  "type": "order.failed",
  "createdAt": "2024-05-23T08:44:58",
  "orderId": "3c0130a4-06f2-4d18-bf39-27153caff6f5",
  "sessionId": null,
  "externalClientId": "external-client-id-5"
}
```

### `client.payment.method.failed`

```json
{
  "id": "webhook-id",
  "type": "client.payment.method.failed",
  "createdAt": "2025-04-22T07:47:31",
  "clientId": "5646c1b7-d934-44ce-8490-938feb810910",
  "bindId": "97d009fe-80e6-426d-8ea2-9784f676e08e",
  "cardMask": "0380",
  "brand": "MASTERCARD",
  "providerType": "ALFA"
}
```

### `SUMSUB_STATUS_CHANGED`

```json
{
  "type": "SUMSUB_STATUS_CHANGED",
  "createdAt": "2025-04-27T16:10:00",
  "clientId": "0d58e7ec-0369-48d7-9804-90c6b23a52be",
  "processName": "applicantReviewed",
  "levelType": "KYC",
  "reviewResult": "GREEN",
  "actionId": "sumsub-action-id"
}
```
