# WhiteBird Card API Guide

## Purpose

This guide describes how to:

1. check available payment providers;
2. start card binding;
3. track binding result;
4. fetch available payment methods/tokens for exchange operations.

All merchant requests are Backend-to-Backend and authorized with header:

- `x-api-key: <merchant-api-key>`

---

## Full card binding flow

### Step 1. Get available providers

`POST /api/v2/exchange/merchant/payment/provider`

Request body:

```json
{
  "clientId": "93a8b39b-d883-4de3-9527-d92b1eabe38c",
  "fiatAsset": "BYN",
  "orderType": "BUY"
}
```

Response (array):

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
              "direction": "SELL",
              "currencies": [
                {
                  "currency": "BYN",
                  "countries": [
                    "Belarus",
                    "Azerbaijan",
                    "Armenia",
                    "Kazakhstan",
                    "Kyrgyzstan",
                    "Moldova",
                    "Tajikistan",
                    "Turkmenistan",
                    "Uzbekistan",
                    "Georgia"
                  ]
                }
              ]
            }
          ]
        }
      ]
    }
  }
]
```

Notes:

- if `addPaymentMethod=true`, card binding flow is required for that provider;
- response also may include `commissions`.

---

### Step 2. Start card binding

`POST /api/v2/exchange/merchant/payment/card/bind`

`returnUrl` - Link to the resource where the client should be redirected after binding the card 

Request body:

```json
{
  "clientId": "1a0e2c64-8a90-4144-ac05-5e66bde1ca84",
  "providerType": "ASSIST",
  "returnUrl": ""
}
```

Response:

```json
{
  "url": "https://payments.t.paysecure.ru/pay/p2p/register_ccard.cfm?..."
}
```

Open `url` in client UI to complete PSP binding page.

---

### Step 3. Track binding result

You can track result in two ways:

1. **webhooks** (recommended);
2. polling bind status endpoint.

#### Webhook events

`client.payment.method.binding` (binding started)

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

`client.payment.method.bound` (success)

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

`client.payment.method.failed` (failed/declined)

```json
{
  "id": "webhook-id",
  "clientId": "5646c1b7-d934-44ce-8490-938feb810910",
  "bindId": "97d009fe-80e6-426d-8ea2-9784f676e08e",
  "cardMask": "0380",
  "brand": "MASTERCARD",
  "providerType": "ASSIST",
  "createdAt": "2025-04-22T07:47:31+0000",
  "type": "client.payment.method.failed"
}
```

If the client has successfully bound their card, you can get the card id in Whitebird as the paymentToken value

#### Polling endpoints (optional)

- `GET /api/v2/exchange/merchant/payment/card/bind?bindId=<bindId>` - get one bind operation result;
- `GET /api/v2/exchange/merchant/payment/card/bind/history?clientId=<clientId>` with request body filter:

```json
{
  "providerTypes": ["ASSIST"],
  "statuses": ["BOUND", "FAILED", "BINDING"]
}
```

---

### Step 4. Get payment methods for exchange

`POST /api/v2/exchange/merchant/payment/method`

Request body:

```json
{
  "clientId": "93a8b39b-d883-4de3-9527-d92b1eabe38c",
  "fiatAsset": "BYN",
  "orderType": "BUY"
}
```

Response:

```json
[
  {
        "id": "fc4b130e-c3bf-4a3d-abe5-9ec5900c9868",
        "number": "**** **** **** 1111",
        "brand": "VISA",
        "providerId": "ASSIST",
        "providerType": "ASSIST",
        "status": "ENABLED",
        "isRestricted": false,
        "isCrypto": false,
        "country": "Russia"
    },
]
```

Use `id` as `paymentToken` in quote/order requests.

---

## Payment method statuses

Current statuses returned by API:

- `ENABLED`
- `CURRENCY_DISABLED`
- `DIRECTION_DISABLED`
- `UNKNOWN` (when `fiatAsset` or `orderType` is not provided)

---

## Optional: unbind card

`POST /api/v2/exchange/merchant/payment/card/unbind`

Request body:

```json
{
  "clientId": "1a0e2c64-8a90-4144-ac05-5e66bde1ca84",
  "providerType": "ASSIST",
  "cardId": "ee6693e1-c340-47cb-8b9e-29304b6d9fd8"
}
```

Response: `200 OK` (empty body).

