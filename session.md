# Session API

## What Session API is for

Session API creates a pre-calculated exchange session for SDK flow and returns:

- `sessionId` for tracking;
- exchange preview (amounts/rates/fees);
- limits preview;
- ready-to-open SDK URL.

This endpoint does not create an order by itself.

---

## Create Session

### `POST /api/v3/exchange/merchant/session`

Creates a session for fiat -> crypto or crypto -> fiat scenario.

### Headers

| Header | Required | Description |
| --- | --- | --- |
| `x-api-key` | Yes | Merchant API key |
| `Content-Type: application/json` | Yes | Request content type |

### Request body

```json
{
  "fromAsset": "BYN",
  "fromAmount": 100,
  "toAsset": "USDT_TRC",
  "destinationCryptoAddress": "TYPAe4vUYtUAKgrRt7iUc2fLuVhRHfCiJG",
  "externalClientId": "external-client-id-1",
  "paymentMethod": "CA",
  "comment": "optional",
  "redirectUrl": "https://partner.app/result",
  "email": "test.user@example.com"
}
```

### Request fields

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `fromAsset` | string | Yes | Source asset id (example: `BYN`, `TRX`) |
| `fromAmount` | number | Conditionally | Source amount. Provide only one of `fromAmount` or `toAmount` |
| `toAsset` | string | Yes | Target asset id |
| `toAmount` | number | Conditionally | Target amount. Provide only one of `fromAmount` or `toAmount` |
| `paymentMethod` | string | No | Fiat payment provider id (validated for merchant) |
| `destinationCryptoAddress` | string | Yes | Destination crypto address (validated) |
| `comment` | string | No | Extra comment (used for crypto transfer details) |
| `externalClientId` | string | Yes | Partner-side client id |
| `redirectUrl` | string | No | Redirect URL added to SDK URL |
| `email` | string | No | Optional email added to SDK URL |

### Validation rules (from code)

- `fromAsset`, `toAsset`, `destinationCryptoAddress`, `externalClientId` must be provided.
- Exactly one amount must be set:
  - allowed: only `fromAmount`, or only `toAmount`;
  - rejected: both set;
  - rejected: both missing.
- Assets must be allowed for merchant.
- `destinationCryptoAddress` must be valid and must not be internal.
- If `paymentMethod` is passed, it must exist for this merchant.

### Response example

```json
{
  "sessionId": "14500600-0631-46c4-9ae1-fab9e4c798f8",
  "externalClientId": "external-client-id-2",
  "destinationCryptoAddress": "TYPAe4vUYtUAKgrRt7iUc2fLuVhRHfCiJG",
  "comment": null,
  "exchange": {
    "fromAsset": "USD",
    "fromGrossAmount": 20,
    "fromNetAmount": 19.5,
    "toAsset": "TRX",
    "toGrossAmount": 68.783069,
    "toNetAmount": 68.520069,
    "actualRate": 0.2919,
    "exchangeRate": 0.2835,
    "fees": [
      {
        "type": "PERCENT",
        "amount": -0.5,
        "asset": "USD"
      },
      {
        "type": "CRYPTO_FEE",
        "amount": -0.263,
        "asset": "TRX"
      }
    ]
  },
  "limit": {
    "min": 11.67,
    "max": 13333.33
  },
  "sdk": {
    "url": "https://sdk.dev.wbdevel.net/v2.0/?mode=LoginMode&merchantId=...&externalClientId=...&sessionId=..."
  }
}
```

---

## Session-related order tracking (v3)

Session API itself only creates session context. Orders are tracked via Order v3 API.

### Get current order

`GET /api/v3/exchange/merchant/order/current`

Query params:

- `clientId` (optional, if `externalClientId` query param is not used)
- `externalClientId` (optional alternative)

Headers:

- `x-api-key` (required)
- optional `externalClientId` header is also supported in auth flow

### Get order history filtered by session

`POST /api/v3/exchange/merchant/order/history`

Request example:

```json
{
  "externalClientId": "external-client-id-2",
  "sessionIds": ["14500600-0631-46c4-9ae1-fab9e4c798f8"]
}
```

This returns orders linked to that session id.

---

## Webhooks for session-based orders

Session-specific fields are included in order webhooks when available (`sessionId`, `externalClientId`).

Examples of relevant events:

- `order.processing`
- `order.completed`
- `order.expired`
- `order.failed`

Example:

```json
{
  "id": "webhook-id",
  "type": "order.processing",
  "createdAt": "2024-05-23T08:44:58+0000",
  "sessionId": "3c0130a4-06f2-4d18-bf39-27153caff6f5",
  "orderId": "e57e77fc-802c-4c0d-8c7c-08806c159725",
  "externalClientId": "external-client-id-5"
}
```

---

## Common errors

| Error code | Description |
| --- | --- |
| `INVALID_AMOUNTS` | Both amounts are set, or both are missing |
| `INVALID_DESTINATION_CRYPTO_ADDRESS` | Destination crypto address is invalid |
| `DESTINATION_ADDRESS_IS_INTERNAL` | Destination crypto address is internal |
| `INVALID_PAYMENT_PROVIDER` | Payment provider not found for merchant |
| `Unauthorized` | Invalid API key / access validation failed |

<aside>
🛠

Base URL for the test environment: https://api.dev.wbdevel.net/

</aside>

# Create Session

### **POST** `/api/v3/exchange/merchant/session`

Creates an  session for fiat ↔ crypto or crypto ↔ fiat operations and returns an SDK URL to complete the order.

---

## Headers

| Header | Required | Description |
| --- | --- | --- |
| `x-api-key` | ✅ Yes | Merchant API key |

---

## Request Body

```json
{
	"fromAsset":"BYN",
	"fromAmount":"100",
	"toAsset":"USDT_TRC",
	"destinationCryptoAddress":"TYPAe4vUYtUAKgrRt7iUc2fLuVhRHfCiJG",
	"externalClientId":"externalClientId",
}
```

### Request Parameters

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `fromAsset` | string | ✅ | Source asset |
| `fromAmount` | string | ✅ | Amount to exchange |
| `toAsset` | string | ✅ | Target asset |
| `paymentMethod` | string | ❌ | Payment provider |
| `destinationCryptoAddress` | string | ✅ | Destination crypto wallet address |
| `comment` | string | ❌ | Optional comment for TON |
| `externalClientId` | string | ✅ | External client identifier |
| `redirectUrl` | string | ❌ | Redirect URL after completion |
| `email` | string | ❌ | email for autofill when opening SDK |

---

## Supported Assets

### Fiat

- `BYN`
- `RUB`
- `USD`
- `EUR`

### Crypto

- `BTC`
- `ETH`
- `TRX`
- `TON`
- `USDT_ERC`
- `USDC_ERC`
- `USDT_TRC`
- `USDT_TON`

---

## Supported Payment Providers

### Belarusian payment providers

- `ALFA` - BYN, RUB, USD, EUR
- `ASSIST` - BYN, RUB, USD, EUR
- `STATUSBANK` - BYN, RUB, USD, EUR
- `CA` - BYN, RUB, USD, EUR

### **Russian payment** providers

- `SBER` - only RUB
- `CARUSELL` - only RUB
- `MTS` - only RUB
- `VTB` - only RUB
- `CA` - BYN, RUB, USD, EUR

### **Tajikistan payment** providers

- `CORTI_MILLI` - only RUB

---

## Errors & Exceptions

| Error Code | Description |
| --- | --- |
| `INVALID_AMOUNTS` | Amount not specified or two asset amounts provided |
| `INVALID_DESTINATION_CRYPTO_ADDRESS` | Invalid destination crypto address |
| `DESTINATION_ADDRESS_IS_INTERNAL` | Destination address is internal |
| `NoSuchCurrencyException` | Requested currency is not supported |
| `DIRECTION_BLOCKED` | Exchange direction is blocked |
| `Unauthorized` | Invalid api key |
| `INVALID_PAYMENT_PROVIDER` | Payment provider not found |

---

## Response

```json
{
    "sessionId": "14500600-0631-46c4-9ae1-fab9e4c798f8",
    "externalClientId": "external-client-id-2",
    "destinationCryptoAddress": "TYPAe4vUYtUAKgrRt7iUc2fLuVhRHfCiJG",
    "comment": null,
    "exchange": {
        "fromAsset": "USD",
        "fromGrossAmount": "20",
        "fromNetAmount": "19.5",
        "toAsset": "TRX",
        "toGrossAmount": "68.783069",
        "toNetAmount": "68.520069",
        "actualRate": "0.2919",
        "exchangeRate": "0.2835",
        "fees": [
            {
                "type": "PERCENT",
                "amount": "-0.5",
                "asset": "USD"
            },
            {
                "type": "CRYPTO_FEE",
                "amount": "-0.263",
                "asset": "TRX"
            }
        ]
    },
    "limit": {
        "min": "11.67",
        "max": "13333.33"
    },
    "sdk": {
        "url": "https://sdk.dev.wbdevel.net/v2.0/?mode=LoginMode&merchantId=...&externalClientId=...&sessionId=..."
    }
}
```

---

## Webhooks

### **order.processing**

Triggered when a new order is created.

```json
{
	"id":"webhook-id",
	"type":"order.processing",
	"createdAt":"2024-05-23T08:44:58+0000",
	"sessionId":"3c0130a4-06f2-4d18-bf39-27153caff6f5",
	"orderId":"e57e77fc-802c-4c0d-8c7c-08806c159725",
	"externalClientId": "external-client-id-5"
}

```

---

### **order.completed**

Triggered when the order is successfully completed.

```json
{
	"id":"webhook-id",
	"type":"order.completed",
	"createdAt":"2024-05-23T08:44:58+0000",
	"sessionId":"3c0130a4-06f2-4d18-bf39-27153caff6f5",
	"orderId":"e57e77fc-802c-4c0d-8c7c-08806c159725",
	"externalClientId": "external-client-id-5"
}
```

---

### **order.expired**

Triggered when the order or session expires.

```json
{
	"id":"webhook-id",
	"type":"order.expired",
	"createdAt":"2024-05-23T08:44:58+0000",
	"sessionId":"3c0130a4-06f2-4d18-bf39-27153caff6f5",
	"orderId":"e57e77fc-802c-4c0d-8c7c-08806c159725",
	"externalClientId": "external-client-id-5"
}
```

---

### **order.error**

Triggered when the order fails.

```json
{
	"id":"webhook-id",
	"type":"order.failed",
	"createdAt":"2024-05-23T08:44:58+0000",
	"sessionId":"3c0130a4-06f2-4d18-bf39-27153caff6f5",
	"orderId":"e57e77fc-802c-4c0d-8c7c-08806c159725",
	"externalClientId": "external-client-id-5"
}
```

---

# Get Limits For Session

### POST `/api/v2/exchange/merchant/buy-limit`

## Headers

| Header | Required | Description |
| --- | --- | --- |
| `x-api-key` | ✅ Yes | Merchant API keyRequest |

---

## Request Body

```json
{
	"asset":"RUB",
	"paymentMethod":"CARUSELL"
}
```

### Request Parameters

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `asset` | string | ✅ | Source asset (only fiat asset) |
| `paymentMethod` | string | ✅ | Payment provider |

---

## Response

```json
{
    "asset": {
        "id": "RUB",
        "code": "RUB"
    },
    "min": 947.37,
    "max": 3735526.32
}
```

---

## Errors & Exceptions

| Error Code | Description |
| --- | --- |
| `NoSuchCurrencyException` | Requested currency is not supported |
| `DIRECTION_BLOCKED` | Exchange direction is blocked |
| `Unauthorized` | Invalid api key |
| `INVALID_PAYMENT_PROVIDER` | Payment provider not found |

---

# Get Current Session Order

### GET `/api/v3/exchange/merchant/oreder/current`

## Headers

| Header | Required | Description |
| --- | --- | --- |
| `x-api-key` | ✅ Yes | Merchant API key |

---

## RequestParam

| Param | Required | Description |
| --- | --- | --- |
| `externalClientId` | ✅ Yes |  External client identifier  |

---

## Response

```json
        {
            "id": "5289b6f0-4945-4b74-b243-4fad013eed50",
            "number": 9210086,
            "rate": "TRX/BYN",
            "systemRateValue": "1.2",
            "exchangeRateValue": "1.2",
            "actualRateValue": "1.24409",
            "promoCode": null,
            "recalculationReason": "NONE",
            "clientId": "ad372b6f-9eb8-48fc-9278-51c7a3639aef",
            "status": "COMPLETED",
            "failureMessage": null,
            "completionDate": "2025-12-22T18:02:35+0000",
            "creationDate": "2025-12-22T17:59:18+0000",
            "input": {
                "type": "FIAT_PROVIDER",
                "asset": "BYN",
                "amount": "50",
                "transactionAmount": "50",
                "feeAmount": "1.45",
                "status": "COMPLETED",
                "failureMessage": null,
                "expirationDate": null,
                "provider": "CA",
                "paymentType": null,
                "processingBank": null,
                "fromToken": "only internal token should be set",
                "toToken": "3fe6f272-2608-4800-9859-56897a4261da",
                "link": "191e0716-5e84-4e92-8e62-23b04cec63cb"
            },
            "output": {
                "type": "CRYPTO_TRANSFER",
                "asset": "TRX",
                "amount": "40.190074",
                "transactionAmount": "40.190074",
                "feeAmount": "0",
                "status": "COMPLETED",
                "failureMessage": null,
                "expirationDate": null,
                "fromAddress": "TTUhGjn4LZKZC35Sk6T7e731dEE8HqBWwQ",
                "toAddress": "TYPAe4vUYtUAKgrRt7iUc2fLuVhRHfCiJG",
                "comment": null,
                "hash": "b733486d285253130a61b2b2cf07dbd6d76528f56195e6937cdfbb68d42e4759"
            }
        }
```

---

# Get Current Session Orders By ExternalClientId

### POST `/api/v3/exchange/merchant/oreder/history`

## Headers

| Header | Required | Description |
| --- | --- | --- |
| `x-api-key` | ✅ Yes | Merchant API keyRequest |

---

## Request Body

```json
{
    "externalClientId": "5289b6f0-4945-4b74-b243-4fad013eed50",
    "sessionIds": ["5289b6f0-4945-4b74-b243-4fad013eed50"]
}
```

---

## Response

```json
{
    "content": [
        {
            "id": "5289b6f0-4945-4b74-b243-4fad013eed50",
            "number": 9210086,
            "rate": "TRX/BYN",
            "systemRateValue": "1.2",
            "exchangeRateValue": "1.2",
            "actualRateValue": "1.24409",
            "promoCode": null,
            "recalculationReason": "NONE",
            "clientId": "ad372b6f-9eb8-48fc-9278-51c7a3639aef",
            "status": "COMPLETED",
            "failureMessage": null,
            "completionDate": "2025-12-22T18:02:35+0000",
            "creationDate": "2025-12-22T17:59:18+0000",
            "input": {
                "type": "FIAT_PROVIDER",
                "asset": "BYN",
                "amount": "50",
                "transactionAmount": "50",
                "feeAmount": "1.45",
                "status": "COMPLETED",
                "failureMessage": null,
                "expirationDate": null,
                "provider": "CA",
                "paymentType": null,
                "processingBank": null,
                "fromToken": "only internal token should be set",
                "toToken": "3fe6f272-2608-4800-9859-56897a4261da",
                "link": "191e0716-5e84-4e92-8e62-23b04cec63cb"
            },
            "output": {
                "type": "CRYPTO_TRANSFER",
                "asset": "TRX",
                "amount": "40.190074",
                "transactionAmount": "40.190074",
                "feeAmount": "0",
                "status": "COMPLETED",
                "failureMessage": null,
                "expirationDate": null,
                "fromAddress": "TTUhGjn4LZKZC35Sk6T7e731dEE8HqBWwQ",
                "toAddress": "TYPAe4vUYtUAKgrRt7iUc2fLuVhRHfCiJG",
                "comment": null,
                "hash": "b733486d285253130a61b2b2cf07dbd6d76528f56195e6937cdfbb68d42e4759"
            }
        }
    ],
    "pageable": {
        "sort": {
            "unsorted": false,
            "sorted": true,
            "empty": false
        },
        "pageNumber": 0,
        "pageSize": 10,
        "offset": 0,
        "paged": true,
        "unpaged": false
    },
    "totalElements": 1,
    "totalPages": 1,
    "last": true,
    "numberOfElements": 1,
    "size": 10,
    "number": 0,
    "sort": {
        "unsorted": false,
        "sorted": true,
        "empty": false
    },
    "first": true,
    "empty": false
}
```

---

# Get Available Payment Providers

### POST `/api/v2/exchange/merchant/payment/provider`

## Headers

| Header | Required | Description |
| --- | --- | --- |
| `x-api-key` | ✅ Yes | Merchant API keyRequest |

---

## Request Body

```json
{}    //require
```

---

## Response

```json
[
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
                                {
                                    "currency": "RUB",
                                    "countries": [
                                        "Russia"
                                    ]
                                }
                            ]
                        }
                    ]
                }
            ]
        },
        "commissions": [
            {
                "buyCommission": "2,5",
                "sellCommission": "2,0"
            }
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
                                {
                                    "currency": "RUB"
                                },
                                {
                                    "currency": "BYN"
                                },
                                {
                                    "currency": "USD"
                                },
                                {
                                    "currency": "EUR"
                                }
                            ]
                        },
                        {
                            "direction": "SELL",
                            "currencies": [
                                {
                                    "currency": "RUB"
                                },
                                {
                                    "currency": "BYN"
                                },
                                {
                                    "currency": "USD"
                                },
                                {
                                    "currency": "EUR"
                                }
                            ]
                        }
                    ]
                }
            ]
        },
        "commissions": [
            {
                "buyCommission": "2,5",
                "sellCommission": "1,5"
            }
        ]
    }
]
```

---
