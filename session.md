# Session API

Session API is an entry point for SDK-oriented exchange flow.  
It prepares a pre-calculated exchange context (rates, fees, limits), returns `sessionId` for tracking, and builds ready-to-open SDK URL with required parameters.  
Session itself does not create an order; order is created later inside order flow.

---

# Create Session

### **POST** `/api/v3/exchange/merchant/session`

Creates a session for fiat ↔ crypto or crypto ↔ fiat operations and returns an SDK URL.

## Headers

| Header | Required | Description |
| --- | --- | --- |
| `x-api-key` | ✅ Yes | Merchant API key |
| `Content-Type` | ✅ Yes | `application/json` |

### Request body

```json
{
	"fromAsset":"BYN",
	"fromAmount":"100",
	"toAsset":"USDT_TRC",
	"destinationCryptoAddress":"TCT2pKJXo233hrKWQMeCptC8My1KGvtsU4",
	"externalClientId":"externalClientId",
}
```


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


## Request params (body fields)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `fromAsset` | string | ✅ | Source asset |
| `fromAmount` | number | ⚠️ Conditionally | Only one of `fromAmount` or `toAmount` |
| `toAsset` | string | ✅ | Target asset |
| `toAmount` | number | ⚠️ Conditionally | Only one of `fromAmount` or `toAmount` |
| `paymentMethod` | string | ❌ | Payment provider id |
| `destinationCryptoAddress` | string | ✅ | Destination crypto wallet address |
| `comment` | string | ❌ | Optional crypto transfer comment |
| `externalClientId` | string | ✅ | External client identifier |
| `redirectUrl` | string | ❌ | Redirect URL for SDK |
| `email` | string | ❌ | Optional email for SDK |

### Response

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

## Errors

| Error Code | Description |
| --- | --- |
| `INVALID_AMOUNTS` | Amount not specified or both amounts provided |
| `INVALID_DESTINATION_CRYPTO_ADDRESS` | Invalid destination crypto address |
| `DESTINATION_ADDRESS_IS_INTERNAL` | Destination address is internal |
| `INVALID_PAYMENT_PROVIDER` | Payment provider not found |
| `Unauthorized` | Invalid API key / access denied |

---

## Webhooks (code-aligned)

Possible session-related order events:

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

## `POST /api/v2/exchange/merchant/buy-limit`

### Headers

| Header | Required | Description |
| --- | --- | --- |
| `x-api-key` | ✅ Yes | Merchant API key |
| `Content-Type` | ✅ Yes | `application/json` |

### Request body

```json
{
  "asset": "RUB",
  "paymentMethod": "CARUSELL"
}
```

### Request params (body fields)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `asset` | string | ✅ Yes | Fiat asset code |
| `paymentMethod` | string | ✅ Yes | Payment provider |
| `clientId` | string | ❌ | Merchant client id |

### Response

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

### Errors

| Error Code | Description |
| --- | --- |
| `INVALID_PAYMENT_PROVIDER` | Payment provider not found |
| `Unauthorized` | Invalid API key / access denied |

---

## `GET /api/v3/exchange/merchant/order/current`

### Headers

| Header | Required | Description |
| --- | --- | --- |
| `x-api-key` | ✅ Yes | Merchant API key |

### Request params

| Param | Required | Description |
| --- | --- | --- |
| `externalClientId` | ⚠️ Conditionally required (required if externalClientId is not provided) | External client id (alternative to `clientId`) |
| `clientId` | ⚠️ Conditionally required (required if сlientId is not provided) | Client id (alternative to `externalClientId`) |

### Response

```json
{
  "id": "5289b6f0-4945-4b74-b243-4fad013eed50",
  "number": 9210086,
  "conditions": {
    "rate": "TRX/BYN",
    "systemRateValue": 1.2,
    "exchangeRateValue": 1.2,
    "actualRateValue": 1.24409
  },
  "sessionId": "14500600-0631-46c4-9ae1-fab9e4c798f8",
  "status": "COMPLETED",
  "input": {
    "type": "FIAT_PROVIDER",
    "asset": "BYN",
    "amount": 50
  },
  "output": {
    "type": "CRYPTO_TRANSFER",
    "asset": "TRX",
    "amount": 40.190074
  }
}
```

### Errors

| Error Code | Description |
| --- | --- |
| `Unauthorized` | Invalid API key / access denied |
| `Invalid external client id` | External client does not belong to merchant |

---

## `POST /api/v3/exchange/merchant/order/history`

### Headers

| Header | Required | Description |
| --- | --- | --- |
| `x-api-key` | ✅ Yes | Merchant API key |
| `Content-Type` | ✅ Yes | `application/json` |

### Request params (body fields)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `externalClientId` | string | ❌ | External client id |
| `sessionIds` | UUID[] | ❌ | Filter by session ids |
| `clientIds` / `clientId` | string[] / string | ✅ Yes | Alternative client filter |
| `statuses` | string[] | ❌ | Order status filter |
| `creationDateFrame` | object | ❌ | Creation date filter |

### Response

```json
{
  "content": [
    {
      "id": "5289b6f0-4945-4b74-b243-4fad013eed50",
      "number": 9210086,
      "sessionId": "14500600-0631-46c4-9ae1-fab9e4c798f8",
      "status": "COMPLETED"
    }
  ],
  "totalElements": 1,
  "totalPages": 1,
  "number": 0,
  "size": 10
}
```

### Errors

| Error Code | Description |
| --- | --- |
| `Unauthorized` | Invalid API key / access denied |
| `Client id is required` | Neither external client id nor client id provided |

---

## `POST /api/v2/exchange/merchant/payment/provider`

### Headers

| Header | Required | Description |
| --- | --- | --- |
| `x-api-key` | ✅ Yes | Merchant API key |
| `Content-Type` | ✅ Yes | `application/json` |

### Request params (body fields)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `clientId` | string | ❌ | Merchant client id |
| `fiatAsset` | string | ❌ | Fiat asset filter |
| `orderType` | string | ❌ | `BUY` / `SELL` filter |
| `destination` | string | ❌ | Merchant destination filter |
| `providers` / `isCrypto` / `countryGroup` | varies | ❌ | Optional filters |

### Response

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
                  "countries": ["Russia"]
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
  }
]
```

### Errors

| Error Code | Description |
| --- | --- |
| `Unauthorized` | Invalid API key / access denied |
| `Invalid external user id` | External client id validation failed |

---

### How these values are calculated:

- `plainRate` = base market rate (bid/ask) before final quote adjustments.
- `rate` (final client rate):
  - for SELL: `rate = toAmount / fromGrossAmount`
  - for BUY: `rate = fromGrossAmount / toAmount`
- percentage fee for any step: `fee = amount * percent / 100`
- reverse fee (when target amount is fixed): `fee = amount / (1 - percent) - amount`
- total fee by side:
  - `fromFeeAmount = fromPaymentFeeAmount + fromExchangeFeeAmount`
  - `toFeeAmount = toPaymentFeeAmount + toExchangeFeeAmount`
  - if input is FIAT: `fee.total = fromFeeAmount`
  - if input is CRYPTO: `fee.total = toFeeAmount`
- network fee in response:
  - if input is CRYPTO: `fee.network = fromPaymentFeeAmount`
  - if input is FIAT: `fee.network = toPaymentFeeAmount`
- `fee.service = null` (service part is not returned as separate value in this response format).

### Variables:

- `fromGrossAmount`: how much the client gives before deductions.
- `fromAmount`: amount left from input side after input-side fees.
- `toAmount`: final amount the client receives after output-side fees.
- `amount`: current amount used in fee formula on current calculation step.
- `percent`: fee percent configured for this fee rule (for example `2.5` means `2.5%`).
- `fromPaymentFeeAmount` / `toPaymentFeeAmount`: payment/transfer part of fee (card processor or blockchain/network).
- `fromExchangeFeeAmount` / `toExchangeFeeAmount`: exchange service part of fee.
