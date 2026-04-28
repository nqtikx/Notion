# Webhooks

WhiteBird sends webhook notifications to merchant backend endpoints via HTTP `POST` with JSON payload.

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

| Group | Event types |
|---|---|
| Client | `client.created`, `client.pending`, `client.verified`, `client.frozen` |
| Order | `order.processing`, `order.completed`, `order.expired`, `order.failed`, `order.error` *(legacy)* |
| Input operation | `order.operation.in.selected`, `order.operation.in.processing`, `order.operation.in.completed`, `order.operation.in.failed`, `order.operation.in.expired` |
| Output operation | `order.operation.out.selected`, `order.operation.out.processing`, `order.operation.out.completed`, `order.operation.out.failed`, `order.operation.out.expired` |
| Payment method | `client.payment.method.binding`, `client.payment.method.bound`, `client.payment.method.failed` |

Additional webhook types used in code:
- `SUMSUB_STATUS_CHANGED`
- `CLIENT_EMAIL_CHANGED`
- `CRM_EVENT_PROCESSED`

---

## 2) Signature verification (step-by-step)

Every webhook request includes header:
- `x-payload-digest`

This header is a signature of the request body.

### What exactly is signed

| Item | Value |
|---|---|
| Algorithm | `HMAC-SHA1` |
| Secret key | Merchant `webhookSigningHash` |
| Data to sign | Raw HTTP body bytes (exact bytes from request, before JSON parsing/reformatting) |
| Result format | Lowercase hex string |

### Verification algorithm

1. Read raw request body bytes as-is.
2. Compute `HMAC-SHA1(rawBody, webhookSigningHash)`.
3. Convert result to hex.
4. Compare computed value with header `x-payload-digest`.
5. Use constant-time compare to avoid timing attacks.

### If signature is invalid

- Return `401` or `403`.
- Do **not** execute business logic.
- Log reason (`invalid signature`) with webhook `id` if available.

### Which status to return: `401` vs `403`

Use one simple rule in integration:

- Return **`401 Unauthorized`** when auth/signature data is missing or malformed:
  - header `x-payload-digest` is missing
  - header is empty
  - header is not valid hex format
- Return **`403 Forbidden`** when signature is present but verification failed:
  - computed HMAC does not match received digest

If you do not need this distinction, always return `401` for any signature error (also acceptable).

### Step-by-step flow for sequence diagram

1. **Receive webhook request** (`POST`).
2. **Read raw body bytes** (exactly as received).
3. **Read header** `x-payload-digest`.
4. **Validate header presence/format**:
   - if missing/invalid -> return `401`, stop.
5. **Compute signature**:
   - `expected = HMAC_SHA1(rawBody, webhookSigningHash)` in hex.
6. **Compare signatures** (constant-time compare):
   - if mismatch -> return `403`, stop.
7. **Parse JSON payload**.
8. **Extract `id`** and check dedup storage.
9. **If duplicate `id`** -> return `200` (already processed), stop.
10. **Execute business logic once**.
11. **Store `id` as processed**.
12. **Return `200`**.

### If signature is valid

- Continue normal webhook handling.
- Prefer idempotent flow (see section 3).

### Typical integration mistakes to avoid

- Verifying parsed/re-serialized JSON instead of raw body bytes.
- Trimming spaces/newlines before verification.
- Using wrong secret (after regeneration old secret is invalid).
- Case-insensitive string compare without constant-time logic.

---

## 3) Delivery behavior (what merchant should expect)

Webhook sending is asynchronous. Delivery is "at least once", so duplicates are possible.

### Sender behavior

| Behavior | Description |
|---|---|
| Sending mode | Asynchronous |
| Retry policy | Up to 3 attempts on transport failures (`RestClientException`) |
| Backoff | 1 second between attempts |
| Timeouts | 15s connect / 15s read |
| Delivery log | Response code, response message, and last send timestamp are stored |

### Merchant requirements (critical)

| Requirement | Why |
|---|---|
| Handle webhook idempotently | Same event can be sent again on retry |
| Use webhook `id` as dedup key | Stable unique event identifier |
| Return `2xx` for already processed events | Prevent unnecessary retries |

### Recommended processing flow on merchant side

1. Verify signature (`x-payload-digest`).
2. Extract `id` from payload.
3. Check dedup storage (Redis/DB) by `id`.
4. If already processed: return `200 OK`.
5. If new: run business logic once, save `id` as processed, return `200 OK`.

Recommended dedup retention window: at least `24h` (better `48-72h` depending on your ops policy).

### What triggers retries

- Network errors / timeouts.
- Any transport failure represented as `RestClientException`.

### What does not trigger retries (in normal flow)

- Successful delivery with valid HTTP response.

---

## 4) Merchant webhook management endpoints

| Method | Endpoint | Description |
|---|---|---|
| `PUT` | `/api/v1/merchant/current/webhookURL` | Set/update webhook URL |
| `POST` | `/api/v1/merchant/current/generate/webhookSigningHash` | Regenerate signing secret |
| `POST` | `/api/v1/merchant/current/webhook/server/paged` | Delivery history |
| `POST` | `/api/v1/merchant/current/webhook/server/{webhookId}/resend` | Resend previously stored webhook |

Note:
- Management endpoints do not create business webhook events manually.
- Business events are generated by WhiteBird domain events and sent automatically.

---

## 5) Common payload fields

Common fields are event-dependent.

| Field | Type | Meaning |
|---|---|---|
| `id` | `string` | Webhook event ID |
| `type` | `string` | Event type |
| `createdAt` | `string` | Event creation timestamp |
| `clientId` | `string` | WhiteBird client ID |
| `orderId` | `string` | WhiteBird order ID |
| `sessionId` | `string` | SDK session ID |
| `externalClientId` | `string` | Partner-side client ID |
| `bindId` | `string` | Payment method bind ID |
| `paymentToken` | `string` | Tokenized payment method ID |
| `providerType` | `string` | Payment provider type |
| `cardMask` | `string` | Masked card digits |
| `brand` | `string` | Card brand |
