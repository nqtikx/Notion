# WhiteBird Exchange API Guide

## Introduction

This guide describes how to integrate **WhiteBird Exchange API** for operational exchange flows.

The document is focused on:
- **OnRamp** (`fiat -> crypto`)
- **OffRamp** (`crypto -> fiat`)
- merchant and client API usage patterns
- status/compliance routing behavior
- operational tracking and webhook-related capabilities

> This guide is exchange-flow focused.  
> User onboarding, registration, KYC profile submission, and token issuance are documented separately in **WhiteBird Registration API Guide**.

---

## Table of Contents

- [1. Base principles](#1-base-principles)
- [2. OnRamp (fiat -> crypto) — Merchant API](#2-onramp-fiat---crypto--merchant-api)
- [3. OffRamp (crypto -> fiat) — Merchant API](#3-offramp-crypto---fiat--merchant-api)
- [4. Errors and status routing](#5-errors-and-status-routing)
- [5. Webhooks and callbacks](#6-webhooks-and-callbacks)

---

## 1. Base principles

### Scope
WhiteBird Exchange API supports two directions:
- **OnRamp**: `fiat -> crypto`
- **OffRamp**: `crypto -> fiat`

### API domains
- **Merchant API** — backend-to-backend, auth via `x-api-key`.
- **Client API** — user-context API:
  - normally uses `Authorization: Bearer <accessToken>`;
  - for some endpoints, anonymous mode is supported with `merchantId` in request body.

### Recommended pipeline (v2-first)
1. `assets`
2. `payment/provider` (and `payment/method` if needed)
3. `limit` (v2)
4. `quote` (v2)
5. `buy/sell`
6. `order status/current/history/reject`

### Status routing
Core statuses used by exchange routing:
- `NOT_VERIFIED`
- `PENDING`
- `VERIFIED`
- `FROZEN`
- `ARREST`

## 2. [OnRamp (fiat -> crypto) — Merchant API](./V2.md)

## 3. [OffRamp (crypto -> fiat) — Merchant API](./V2.md)

## 4. Errors and Status Routing (V2 OnRamp / OffRamp)

This page lists practical error responses for V2 merchant exchange flow endpoints:
- `/api/v2/exchange/merchant/assets`
- `/api/v2/exchange/merchant/payment/provider`
- `/api/v2/exchange/merchant/payment/method`
- `/api/v2/exchange/merchant/limit`
- `/api/v2/exchange/merchant/quote`
- `/api/v2/exchange/merchant/buy`
- `/api/v2/exchange/merchant/sell`
- `/api/v2/exchange/merchant/order`
- `/api/v2/exchange/merchant/order/current`
- `/api/v2/exchange/merchant/order/history`

---

## Errors and status routing table

| Scope | HTTP | `status` | When it occurs | Example JSON response |
|---|---:|---|---|---|
| Any merchant V2 endpoint | `401` | `Unauthorized` | Invalid or missing `x-api-key` | `{"message":"Invalid api key","code":401,"status":"Unauthorized"}` |
| Any merchant V2 endpoint | `403` | `Forbidden` | Access denied to resource/client/order | `{"message":"Access is denied","code":403,"status":"Forbidden"}` |
| Any merchant V2 endpoint | `429` | `Too Many Requests` | Rate limit exceeded | `{"message":"Too many requests","code":429,"status":"Too Many Requests"}` |
| Any merchant V2 endpoint | `415` | `Unsupported Media Type` | Unsupported request content type | `{"message":"Unsupported content type: text/plain; ...","code":415,"status":"Unsupported Media Type"}` |
| Any merchant V2 endpoint | `400` | `Bad Request` | Invalid JSON body or required parameter missing | `{"message":"Invalid request body","code":400,"status":"Bad Request"}` |
| Any merchant V2 endpoint with `@Valid` | `400` | `BAD_REQUEST` | Bean validation failed | `{"status":"BAD_REQUEST","message":"Validation errors","code":400,"fields":[{"field":"clientId","value":null,"message":"must not be null"}]}` |
| `/merchant/quote`, `/merchant/buy`, `/merchant/sell` | `400` | `QUOTE_NOT_FOUND` | Quote does not exist or is expired | `{"message":"Quote is expired: <quoteId>","code":400,"status":"QUOTE_NOT_FOUND"}` |
| `/merchant/quote`, `/merchant/buy`, `/merchant/sell` | `400` | `INVALID_QUOTE` | Wrong quote direction/type, two amounts provided, quote-client mismatch | `{"message":"Not buy quote: <quoteId>","code":400,"status":"INVALID_QUOTE"}` |
| `/merchant/quote`, `/merchant/limit` | `400` | `INVALID_ASSET` | Asset/payment type mismatch or unsupported asset for operation | `{"message":"Fiat provider payment type supports only fiat assets","code":400,"status":"INVALID_ASSET"}` |
| `/merchant/quote`, `/merchant/limit`, `/merchant/buy`, `/merchant/sell` | `400` | `INVALID_ADDRESS` | Crypto address format is invalid | `{"message":"invalid crypto address","code":400,"status":"INVALID_ADDRESS"}` |
| `/merchant/quote`, `/merchant/limit`, `/merchant/buy`, `/merchant/sell` | `400` | `DESTINATION_ADDRESS_IS_INTERNAL` | Destination crypto address belongs to internal wallet | `{"message":"destination address is internal","code":400,"status":"DESTINATION_ADDRESS_IS_INTERNAL"}` |
| `/merchant/quote` | `400` | `INVALID_PAYMENT_PROVIDER` | Payment provider is not available for this operation/client | `{"message":"Payment provider is not available for this operation","code":400,"status":"INVALID_PAYMENT_PROVIDER"}` |
| `/merchant/quote` | `400` | `INVALID_PAYMENT_TOKEN` | Payment token is missing/invalid for required provider | `{"message":"Invalid payment token","code":400,"status":"INVALID_PAYMENT_TOKEN"}` |
| `/merchant/quote` | `400` | `INVALID_PAYMENT_METHOD` | Payment method cannot be validated/used | `{"message":"Invalid payment method","code":400,"status":"INVALID_PAYMENT_METHOD"}` |
| `/merchant/quote`, `/merchant/buy`, `/merchant/sell` | `400` | `ORDER_SESSION_ALREADY_EXISTS` | Request sessionId already belongs to existing order | `{"message":"Order with sessionId: <sessionId> already exists","code":400,"status":"ORDER_SESSION_ALREADY_EXISTS"}` |
| `/merchant/limit` | `503` | `EXCHANGE_UNAVAILABLE` | Exchange backend/service unavailable | `{"message":"<service message>","code":503,"status":"EXCHANGE_UNAVAILABLE"}` |
| `/merchant/limit` | `400` | `LIMIT_VIOLATION` | Generic limit violation on calculation | `{"message":"<limit violation message>","code":400,"status":"LIMIT_VIOLATION"}` |
| `/merchant/limit` | `400` | `AMOUNT_TOO_SMALL` | Amount is below allowed threshold | `{"message":"<amount too small message>","code":400,"status":"AMOUNT_TOO_SMALL"}` |
| `/merchant/limit`, `/merchant/buy`, `/merchant/sell` | `400` | `INVALID_MIN_LIMIT` | Amount below minimum allowed limit | `{"status":"INVALID_MIN_LIMIT","message":"<limit message>","code":400,"fields":[{"field":"limit","value":"<min>","message":"<limit message>"}]}` |
| `/merchant/limit`, `/merchant/buy`, `/merchant/sell` | `400` | `INVALID_MAX_LIMIT` | Amount above maximum allowed limit | `{"status":"INVALID_MAX_LIMIT","message":"<limit message>","code":400,"fields":[{"field":"limit","value":"<max>","message":"<limit message>"}]}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `BAD_REQUEST` | Destination crypto address is required but missing | `{"message":"Destination crypto address is null","code":400,"status":"BAD_REQUEST"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `EXCHANGE_DISABLED` | Exchange pair/route currently disabled | `{"message":"exchange is disabled","code":400,"status":"EXCHANGE_DISABLED"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `ACTIVE_ORDER_FOUND` | Client already has active processing order | `{"message":"Active order is found","code":400,"status":"ACTIVE_ORDER_FOUND"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `ACTIVE_DEPOSIT_REQUEST_FOUND` | Client has active deposit request | `{"message":"Active deposit request is found","code":400,"status":"ACTIVE_DEPOSIT_REQUEST_FOUND"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `ACTIVE_WITHDRAWAL_REQUEST_FOUND` | Client has active withdrawal request | `{"message":"Active withdrawal request is found","code":400,"status":"ACTIVE_WITHDRAWAL_REQUEST_FOUND"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `INVALID_CLIENT_STATUS` | Client status is not eligible (`NOT_VERIFIED`, `PENDING`, testing incomplete, etc.) | `{"message":"invalid client status","code":400,"status":"INVALID_CLIENT_STATUS"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `PHONE_CONFIRMATION_REQUIRED` | Phone confirmation is required for client | `{"message":"Phone confirmation required","code":400,"status":"PHONE_CONFIRMATION_REQUIRED"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `PHONE_NUMBER_REQUIRED` | Client phone is blank/invalid | `{"message":"Phone number is blank","code":400,"status":"PHONE_NUMBER_REQUIRED"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `INVALID_MFA_STATUS` | Client must have MFA enabled for operation | `{"message":"Client must have MFA enabled","code":400,"status":"INVALID_MFA_STATUS"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `INVALID_INTERNAL_BALANCE` | Insufficient client internal balance | `{"message":"Insufficient client internal balance ...","code":400,"status":"INVALID_INTERNAL_BALANCE"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `INVALID_CRYPTO_BALANCE` | Insufficient system crypto liquidity for payout | `{"message":"Insufficient system crypto balance: ...","code":400,"status":"INVALID_CRYPTO_BALANCE"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `INVALID_FIAT_BALANCE` | Insufficient system fiat balance for payout | `{"message":"Insufficient system fiat balance: ...","code":400,"status":"INVALID_FIAT_BALANCE"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `INVALID_CORPORATE_CARD` | Required corporate/system payment card not found | `{"message":"No fiat card by <asset>, <provider>","code":400,"status":"INVALID_CORPORATE_CARD"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `INVALID_SCAN_STATUS` | Mempool scan is unavailable for one of assets | `{"message":"mempool scan unavailable TRX","code":400,"status":"INVALID_SCAN_STATUS"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `SINGLE_OP_MAX_VOLUME_EXCEEDED` | Single operation amount exceeds configured restriction | `{"message":"Single operation max volume exceeded, client ...","code":400,"status":"SINGLE_OP_MAX_VOLUME_EXCEEDED"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `OP_MAX_VOLUME_FOR_PERIOD_REACHED` | Period volume restriction reached | `{"message":"Period operations max volume reached, client ...","code":400,"status":"OP_MAX_VOLUME_FOR_PERIOD_REACHED"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `OP_MAX_NUM_FOR_PERIOD_REACHED` | Max operations count for period reached | `{"message":"Operations max num reached","code":400,"status":"OP_MAX_NUM_FOR_PERIOD_REACHED"}` |
| `/merchant/order` | `400` | `Bad Request` | Order id exists but business flow cannot load/process it | `{"message":"No order found: <orderId>","code":400,"status":"Bad Request"}` |
| `/merchant/order/history` | `400` | `Bad Request` | No `clientId`/`externalClientId` provided in request filter | `{"message":"Client id is required","code":400,"status":"Bad Request"}` |

---

## Routing summary by HTTP status

| HTTP | Meaning for integrator | Typical action |
|---:|---|---|
| `400` | Request/business validation failed | Fix request data, client state, limits, or route |
| `401` | API key is invalid/missing | Check `x-api-key` |
| `403` | Access denied | Check merchant-client ownership and access scope |
| `404` | Entity not found (rare for these endpoints) | Validate identifiers and endpoint path |
| `415` | Unsupported media type | Use `Content-Type: application/json` |
| `429` | Rate limit exceeded | Retry later with backoff |
| `503` | Exchange/calculation temporarily unavailable | Retry with backoff and monitoring |


## 5. [Webhooks and callbacks](./Webhooks%20Overview.md)
