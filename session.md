# Create Session

### **POST** `/api/v3/exchange/merchant/session`

Session API is an entry point for SDK-oriented exchange flow.  
It prepares a pre-calculated exchange context (rates, fees, limits), returns `sessionId` for tracking, and builds ready-to-open SDK URL with required parameters.  
Session itself does not create an order; order is created later inside order flow.

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
	"fromAmount":100,
	"toAsset":"USDT_TRC",
	"destinationCryptoAddress":"TCT2pKJXo233hrKWQMeCptC8My1KGvtsU4",
	"externalClientId":"externalClientId",
}
```

### Request Parameters

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `fromAsset` | string | ✅ | Source asset |
| `fromAmount` | number | ⚠️ Conditionally | Source amount. Provide only one of `fromAmount` or `toAmount` |
| `toAsset` | string | ✅ | Target asset |
| `toAmount` | number | ⚠️ Conditionally | Target amount. Provide only one of `fromAmount` or `toAmount` |
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

## Session-related order tracking (v3)

Session API only creates session context. Orders are tracked via Order v3 API.

### GET `/api/v3/exchange/merchant/order/current`

Query params:

- `externalClientId` (optional), or
- `clientId` (optional alternative)

Header:

- `x-api-key` (required)

### POST `/api/v3/exchange/merchant/order/history`

Request example:

```json
{
  "externalClientId": "external-client-id-2",
  "sessionIds": ["14500600-0631-46c4-9ae1-fab9e4c798f8"]
}
```

---

## Response

```json
{
    "sessionId": "14500600-0631-46c4-9ae1-fab9e4c798f8",
    "externalClientId": "external-client-id-2", 
    "destinationCryptoAddress": "TCT2pKJXo233hrKWQMeCptC8My1KGvtsU4",
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

