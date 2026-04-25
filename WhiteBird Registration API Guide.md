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
- [4. Client API mirror](#4-client-api-mirror)
- [5. Errors and status routing](#5-errors-and-status-routing)
- [6. Webhooks and callbacks](#6-webhooks-and-callbacks)
- [7. Legacy v2 compatibility](#7-legacy-v2-compatibility)

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

### Recommended pipeline (v3-first)
1. `assets`
2. `payment/provider` (and `payment/method` if needed)
3. `limit` (v3)
4. `quote` (v3)
5. `order create`
6. `order status/current/history/reject`

### Status routing
Core statuses used by exchange routing:
- `NOT_VERIFIED`
- `PENDING`
- `VERIFIED`
- `FROZEN`
- `ARREST`

### Notes
- v3 endpoints are preferred; v2 endpoints remain for compatibility.
- Onboarding/register/token issuance are documented in a separate guide.

## 2. OnRamp (fiat -> crypto) — Merchant API
Below is a tested OnRamp flow with real request/response examples.
> All requests are merchant backend-to-backend using `x-api-key`.
### Headers
- `x-api-key: {{apiKey}}`
- `Content-Type: application/json`
---
### Step 1. Get available assets
**POST** `{{URL}}/api/v2/exchange/merchant/assets`
**Request**
```json
{
  "clientId": "{{clientId}}"
}
```
**Response**
```json
{
  "fiatAssets": [
    { "id": "BYN", "code": "BYN" },
    { "id": "EUR", "code": "EUR" },
    { "id": "RUB", "code": "RUB" },
    { "id": "USD", "code": "USD" }
  ],
  "cryptoAssets": [
    { "id": "BNB", "code": "BNB", "network": "BNB" },
    { "id": "USDC_BNB", "code": "USDC", "network": "BNB", "protocol": "BEP-20" },
    { "id": "USDT_BNB", "code": "USDT", "network": "BNB", "protocol": "BEP-20" },
    { "id": "BTC", "code": "BTC", "network": "Bitcoin" },
    { "id": "AAVE", "code": "AAVE", "network": "Ethereum", "protocol": "ERC-20" },
    { "id": "ETH", "code": "ETH", "network": "Ethereum" },
    { "id": "LINK", "code": "LINK", "network": "Ethereum", "protocol": "ERC-20" },
    { "id": "PAXG", "code": "PAXG", "network": "Ethereum", "protocol": "ERC-20" },
    { "id": "UNI", "code": "UNI", "network": "Ethereum", "protocol": "ERC-20" },
    { "id": "USDC_ERC", "code": "USDC", "network": "Ethereum", "protocol": "ERC-20" },
    { "id": "USDT_ERC", "code": "USDT", "network": "Ethereum", "protocol": "ERC-20" },
    { "id": "XAUT", "code": "XAUT", "network": "Ethereum", "protocol": "ERC-20" },
    { "id": "TON", "code": "TON", "network": "Ton" },
    { "id": "USDT_TON", "code": "USDT", "network": "Ton", "protocol": "TON" },
    { "id": "TRX", "code": "TRX", "network": "Tron" },
    { "id": "USDT_TRC", "code": "USDT", "network": "Tron", "protocol": "TRC-20" }
  ]
}
```
### Step 2. Get payment providers
**POST** {{URL}}/api/v2/exchange/merchant/payment/provider

**Request**
```json
{
  "clientId": "{{clientId}}",
  "fiatAsset": "BYN",
  "orderType": "BUY"
}
```
Response
```json
[
  {
    "id": "ASSIST",
    "name": "ASSIST",
    "addPaymentMethod": true,
    "config": {
      "paymentSystems": [
        {
          "paymentSystem": "VISA",
          "type": "PSP",
          "directions": [
            {
              "direction": "BUY",
              "currencies": [
                {
                  "currency": "BYN",
                  "countries": ["Belarus", "Azerbaijan", "Armenia", "Kazakhstan", "Kyrgyzstan", "Moldova", "Tajikistan", "Turkmenistan", "Uzbekistan", "Georgia"]
                }
              ]
            }
          ]
        }
      ]
    },
    "commissions": [
      {
        "bank": "OTHER",
        "buyCommission": "3,5-4,9",
        "sellCommission": "3,0-4,9"
      }
    ]
  },
  {
    "id": "MTS",
    "name": "MTS",
    "addPaymentMethod": true,
    "config": {
      "paymentSystems": [
        {
          "paymentSystem": "MIR",
          "type": "PSP",
          "directions": [
            {
              "direction": "SELL",
              "currencies": [
                { "currency": "RUB", "countries": ["Russia"] }
              ]
            }
          ]
        }
      ]
    },
    "commissions": [
      { "buyCommission": "2,5", "sellCommission": "2,0" }
    ]
  },
  {
    "id": "CA",
    "name": "CA",
    "addPaymentMethod": false,
    "config": {
      "paymentSystems": [
        {
          "paymentSystem": "CA",
          "type": "CA",
          "directions": [
            {
              "direction": "BUY",
              "currencies": [
                { "currency": "RUB" },
                { "currency": "BYN" },
                { "currency": "USD" },
                { "currency": "EUR" },
                { "currency": "AED" }
              ]
            }
          ]
        }
      ]
    },
    "commissions": [
      { "buyCommission": "2,5", "sellCommission": "1,5" }
    ]
  }
]
```
### Step 3. Get payment methods
**POST** {{URL}}/api/v2/exchange/merchant/payment/method

**Request**
```json
{
  "clientId": "{{clientId}}",
  "fiatAsset": "BYN",
  "orderType": "BUY"
}
```
**Response**
```json
[
  {
    "id": "8ff59a79-461a-4c6f-8f77-832cfa836c86",
    "number": "**** **** **** 1111",
    "brand": "VISA",
    "providerId": "ASSIST",
    "providerType": "ASSIST",
    "status": "ENABLED",
    "isRestricted": false,
    "isCrypto": false,
    "country": "Russia"
  },
  {
    "providerId": "BRAZINO",
    "providerType": "BRAZINO",
    "status": "ENABLED",
    "name": "BRAZINO"
  },
  {
    "providerId": "CARUSELL",
    "providerType": "CARUSELL",
    "status": "CURRENCY_DISABLED",
    "name": "CARUSELL",
    "supportedCurrencies": ["RUB"]
  }
]
```
### Step 4. Calculate limits (v3)
**POST** {{URL}}/api/v3/exchange/merchant/limit

**Request**
```json
{
  "clientId": "{{clientId}}",
  "fromAsset": "RUB",
  "fromPaymentDetails": {
    "type": "FIAT_PROVIDER",
    "provider": "MTS",
    "token": "1cf1fe09-e7dd-4486-839d-270c482f6afa"
  },
  "toAsset": "TRX",
  "toPaymentDetails": {
    "type": "CRYPTO_TRANSFER"
  }
}
```
**Response**
```json
{
  "fromMinAmount": "921.05",
  "fromMaxAmount": "368421.05",
  "toMinAmount": "33.063035",
  "toMaxAmount": "13330.244487"
}
```
### Step 5. Create quote (v3)
**POST** {{URL}}/api/v3/exchange/merchant/quote

**Request**
```json
{
  "clientId": "{{clientId}}",
  "input": {
    "type": "FIAT_PROVIDER",
    "asset": "RUB",
    "amount": 950,
    "provider": "MTS",
    "token": "{{payment_token}}"
  },
  "output": {
    "type": "CRYPTO_TRANSFER",
    "asset": "TRX",
    "toAddress": "{{walletAddress}}"
  }
}
```
**Response**
```json
{
  "id": "3f81d094-989f-4643-b754-d3c22421d8bb",
  "rate": "TRX/RUB",
  "systemRateValue": "26.9465",
  "exchangeRateValue": "26.9465",
  "actualRateValue": "27.8505",
  "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
  "creationDate": "2026-04-25T12:44:22+0000",
  "expirationDate": "2026-04-25T12:44:52+0000",
  "input": {
    "type": "FIAT_PROVIDER",
    "asset": "RUB",
    "amount": "950",
    "feeAmount": "23.75",
    "provider": "MTS",
    "token": "1cf1fe09-e7dd-4486-839d-270c482f6afa",
    "paymentType": null,
    "processingBank": null
  },
  "output": {
    "type": "CRYPTO_TRANSFER",
    "asset": "TRX",
    "amount": "34.110666",
    "feeAmount": "0.263",
    "toAddress": null,
    "comment": null
  }
}
```
### Step 6. Create order (v3)
**POST** {{URL}}/api/v3/exchange/merchant/order

**Request**
```json
{
  "quoteId": "{{quoteId}}",
  "returnUrl": "https://frontend.dev.wbdevel.net/",
  "destinationCryptoAddress": "{{walletAddress}}"
}
```
**Response**
```json
{
  "id": "37a48ec9-150f-4c3f-88f8-7e8d48a2fd3b",
  "number": 491000003970,
  "conditions": {
    "fromAsset": "RUB",
    "toAsset": "TRX",
    "fromGrossAmount": "950",
    "fromNetAmount": "926.25",
    "fromFeeAmount": "23.75",
    "toGrossAmount": "34.373666",
    "toNetAmount": "34.110666",
    "toFeeAmount": "0.263",
    "promoCode": null,
    "rate": "TRX/RUB",
    "systemRateValue": "26.9465",
    "exchangeRateValue": "26.9465",
    "actualRateValue": "27.8505"
  },
  "recalculationReason": null,
  "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
  "status": "PROCESSING",
  "failureMessage": null,
  "completionDate": null,
  "creationDate": "2026-04-25T12:44:39+0000",
  "sessionId": null,
  "input": {
    "type": "FIAT_PROVIDER",
    "asset": "RUB",
    "amount": "950",
    "transactionAmount": "950",
    "feeAmount": "23.75",
    "status": "PROCESSING",
    "failureMessage": null,
    "expirationDate": null,
    "provider": "MTS",
    "paymentType": null,
    "processingBank": null,
    "clientBank": null,
    "fromToken": "1cf1fe09-e7dd-4486-839d-270c482f6afa",
    "toToken": null,
    "link": null,
    "processorTransactionId": "6066404",
    "post": null,
    "paymentSystem": null
  },
  "output": {
    "type": "CRYPTO_TRANSFER",
    "asset": "TRX",
    "amount": "34.373666",
    "transactionAmount": "34.110666",
    "feeAmount": "0.263",
    "status": "NEW",
    "failureMessage": null,
    "expirationDate": null,
    "fromAddress": null,
    "toAddress": "TCT2pKJXo233hrKWQMeCptC8My1KGvtsU4",
    "comment": null,
    "hash": null
  }
}
```
### Step 7. Get order by ID
**GET** {{URL}}/api/v3/exchange/merchant/order/{{Order_ID}}

**Response**
```json
{
  "id": "37a48ec9-150f-4c3f-88f8-7e8d48a2fd3b",
  "number": 491000003970,
  "status": "PROCESSING",
  "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
  "conditions": {
    "fromAsset": "RUB",
    "toAsset": "TRX",
    "fromGrossAmount": "950",
    "toNetAmount": "34.110666"
  },
  "input": {
    "type": "FIAT_PROVIDER",
    "provider": "MTS",
    "status": "PROCESSING"
  },
  "output": {
    "type": "CRYPTO_TRANSFER",
    "asset": "TRX",
    "status": "NEW"
  }
}
```
### Step 8. Get current order (optional)
**GET** {{URL}}/api/v3/exchange/merchant/order/current?clientId={{clientId}}

**Response**
```json
{
  "id": "37a48ec9-150f-4c3f-88f8-7e8d48a2fd3b",
  "status": "PROCESSING",
  "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562"
}
```
### Step 9. Get order history (optional)
**POST** {{URL}}/api/v3/exchange/merchant/order/history

**Request**
```json
{
  "clientIds": [
    "{{clientId}}"
  ]
}
```
**Response**
```json
{
  "content": [
    {
      "id": "37a48ec9-150f-4c3f-88f8-7e8d48a2fd3b",
      "status": "PROCESSING"
    },
    {
      "id": "2cac7fd6-83e0-4d81-9532-85fb1bc14ca2",
      "status": "COMPLETED"
    },
    {
      "id": "7d8d3767-f403-4727-a9a0-9be71c563ac5",
      "status": "FAILED"
    }
  ],
  "totalElements": 33,
  "totalPages": 4,
  "number": 0,
  "size": 10
}
```
## Notes
For new integrations, use v3 (limit, quote, order) as the primary path.
payment/provider and payment/method are required to resolve provider and token context.
History may contain operations from different directions (including OffRamp/internal balance operations).

## 3. OffRamp (crypto -> fiat) — Merchant API

_To be completed._

## 4. Client API mirror

_To be completed._

## 5. Errors and status routing

_To be completed._

## 6. Webhooks and callbacks

_To be completed._

## 7. Legacy v2 compatibility

_To be completed._
