# Webhooks

WhiteBird sends webhook notifications to merchant backend endpoints via HTTP `POST` with JSON payload.

## Quick start (4 steps)

1. Set merchant webhook URL: `PUT /api/v1/merchant/current/webhookURL`.
2. Get/signing secret (`webhookSigningHash`) and store it securely.
3. Verify `x-payload-digest` using `HMAC-SHA1(rawBody, webhookSigningHash)`.
4. Return `2xx` as soon as webhook is accepted/processed.

This page contains:
- supported event groups and types
- signature verification rules
- delivery behavior
- webhook management endpoints
- common payload field map

For full payload examples, see:
- [Webhooks - Event Payload Examples](./Webhooks%20-%20Event%20Payload%20Examples.md)

---

## 1) Supported event types

| Event | When triggered | Purpose | Key fields |
|---|---|---|---|
| `client.created` | Client registration event | Notify merchant about new client | `id`, `clientId`, `createdAt` |
| `client.pending` | Client status -> `PENDING` | Notify that client is pending verification | `id`, `clientId`, `createdAt` |
| `client.verified` | Client status -> `VERIFIED` | Notify successful client verification | `id`, `clientId`, `createdAt` |
| `client.frozen` | Client status -> `FROZEN` | Notify client freeze/restriction | `id`, `clientId`, `createdAt` |
| `order.processing` | Order created (v1) / order status `PROCESSING` (v2) | Notify processing start | `id`, `orderId`, `sessionId`, `externalClientId`, `createdAt` |
| `order.completed` | Final success (`CONFIRMED` v1 / `COMPLETED` v2) | Notify successful completion | `id`, `orderId`, `sessionId`, `externalClientId`, `createdAt` |
| `order.expired` | Order status -> `EXPIRED` | Notify expiration by time/rules | `id`, `orderId`, `sessionId`, `externalClientId`, `createdAt` |
| `order.failed` | Order v2 status -> `FAILED` | Notify failed completion (current flow) | `id`, `orderId`, `sessionId`, `externalClientId`, `createdAt` |
| `order.error` *(legacy)* | Order v1 status -> `DECLINED` / `REJECTED` / `ARREST` | Backward-compatible error event | `id`, `orderId`, `sessionId`, `externalClientId`, `createdAt` |
| `order.operation.in.selected` | Input operation selected | Notify selected input operation | `id`, `orderId`, `sessionId`, `externalClientId`, `createdAt` |
| `order.operation.in.processing` | Input operation processing | Notify in-progress input operation | `id`, `orderId`, `sessionId`, `externalClientId`, `createdAt` |
| `order.operation.in.completed` | Input operation completed | Notify successful input operation | `id`, `orderId`, `sessionId`, `externalClientId`, `createdAt` |
| `order.operation.in.failed` | Input operation failed | Notify failed input operation | `id`, `orderId`, `sessionId`, `externalClientId`, `createdAt` |
| `order.operation.in.expired` | Input operation expired | Notify expired input operation | `id`, `orderId`, `sessionId`, `externalClientId`, `createdAt` |
| `order.operation.out.selected` | Output operation selected | Notify selected output operation | `id`, `orderId`, `sessionId`, `externalClientId`, `createdAt` |
| `order.operation.out.processing` | Output operation processing | Notify in-progress output operation | `id`, `orderId`, `sessionId`, `externalClientId`, `createdAt` |
| `order.operation.out.completed` | Output operation completed | Notify successful output operation | `id`, `orderId`, `sessionId`, `externalClientId`, `createdAt` |
| `order.operation.out.failed` | Output operation failed | Notify failed output operation | `id`, `orderId`, `sessionId`, `externalClientId`, `createdAt` |
| `order.operation.out.expired` | Output operation expired | Notify expired output operation | `id`, `orderId`, `sessionId`, `externalClientId`, `createdAt` |
| `client.payment.method.binding` | Payment method binding started | Notify binding flow start | `id`, `clientId`, `bindId`, `providerType`, `createdAt` |
| `client.payment.method.bound` | Payment method successfully bound | Notify successful binding | `id`, `clientId`, `paymentToken`, `providerType`, `createdAt` |
| `client.payment.method.failed` | Payment method binding failed | Notify failed binding with diagnostics | `id`, `clientId`, `bindId`, `cardMask`, `brand`, `providerType`, `createdAt` |
| `SUMSUB_STATUS_CHANGED` | Sumsub webhook processed | Notify KYC/AML status update | `id` *(can be `null`)*, `clientId`, `processName`, `levelType`, `reviewResult`, `actionId`, `createdAt` |
| `CLIENT_EMAIL_CHANGED` | Client email changed | Sync updated client email | `id` *(can be `null`)*, `clientId`, `email`, `createdAt` |
| `CRM_EVENT_PROCESSED` | Type exists in codebase; no active sender path in current flow | Reserved/technical event type | `id` *(can be `null`)*, `clientId`, `topic`, `createdAt` |

---

## 2) Signature verification (step-by-step)

Every webhook request includes header:
- `x-payload-digest`

Payload date format used by sender serialization:
- `createdAt`: `yyyy-MM-dd'T'HH:mm:ss` (no timezone suffix)

<img width="512" height="722" alt="webhook flow" src="https://github.com/user-attachments/assets/84314fc4-d678-41c5-ac69-841d9012915e" />

## 3) Delivery behavior

Webhook sending is asynchronous. Delivery is "at least once", so duplicates are possible.

### Sender behavior

| Behavior | Description |
|---|---|
| Sending mode | Asynchronous |
| Retry policy | Up to 3 attempts on `RestClientException` (including `RestClientResponseException`) |
| Backoff | 1 second between attempts |
| Timeouts | 15s connect / 15s read |
| Delivery log | Response code, response message, and last send timestamp are stored |

### Recommended processing flow on merchant side

1. Verify signature (`x-payload-digest`).
2. Extract `id` from payload.
3. Check dedup storage (Redis/DB) by `id`.
4. If already processed: return `200 OK`.
5. If new: run business logic once, save `id` as processed, return `200 OK`.

### What triggers retries

- Exceptions matching `RestClientException` retry policy.
- This includes network/connect/read failures and `RestClientResponseException`.

### What does not trigger retries (in normal flow)

- Requests completed without `RestClientException`.

---

## 4) Merchant webhook management endpoints

| Method | Endpoint | Description |
|---|---|---|
| `PUT` | `/api/v1/merchant/current/webhookURL` | Set/update webhook URL |
| `POST` | `/api/v1/merchant/current/generate/webhookSigningHash` | Regenerate signing secret |
| `POST` | `/api/v1/merchant/current/webhook/server/paged` | Delivery history |
| `POST` | `/api/v1/merchant/current/webhook/server/{webhookId}/resend` | Resend previously stored webhook |
