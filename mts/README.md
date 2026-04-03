# MTS

## Available currencies:

- RUB

## Available Bank Card By Region:

- Russia

## Directions & Commission:

- Buy Crypto 2,5 %
- Sell Crypto 2 %

## Flow for binding card:

![image.png](MTS/image.png)

## First step

Verify that the payment provider is available

### POST api/v2/exchange/merchant/payment/provider

### Request Header:

x-api-key 

### Request Body:

```jsx
{
    "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
    "fiatAsset": "RUB", -- optional (available RUB)
    "orderType": "SELL"  -- optional (available SELL)
}
```

### Response:

```jsx
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

```

It is sufficient to verify that the payment provider is available via the id field.  id = MTS 

## Second step

Generate link to bind client card

### POST api/v2/exchange/merchant/payment/card/bind

### Request Header:

x-api-key 

### Request Body:

`returnUrl` - Link to the resource where the client should be redirected after binding the card 

```jsx
{
    "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
    "providerType": "MTS",
    "returnUrl": "https://www.google.com"
}
```

### Response:

```jsx
{
    "url": "https://frontnew.dev.wbdevel.net/attach-card?returnUrl=https%3A%2F%2Fwww.google.com&token=ODZjMmIxMmItYTMzMi00OWFkLWE0NDctZDAyYzBiNjIxZGM0fDE3NzQ5MDEzOTE5Nzh8MDZlYzdiOTU5ZGI5OTFkMjc2NWViMWNlN2JjMWE2NzVlNDZlZGJmNjUzMGRhYWZkNmE1MjdmNzMyMWYzZDk4Yw=="
}
```

# Third step

Open link for the client to bind their card 

Test card data:
Choose country: Russia, Kazakhstan, Kyrgyzstan, Uzbekistan
Number: 4469157300098872

                                                                Enter card details

![Снимок экрана 2026-03-30 в 22.41.40.png](MTS/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA_%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0_2026-03-30_%D0%B2_22.41.40.png)

  How can I find out the result of a card binding?  
Use webhooks from Whitebird or use the request from step four

### There are three webhooks from Whitebird:

### client.payment.method.binding

Сlient initiated card binding

```
{
  "id": "webhook-id",
  "clientId": "ed8ff528-3017-45bc-9d4d-f90e58f91bf9",
  "bindId": "856c460d-7081-433b-904d-c46e313b1225",
  "providerType": "MTS",
  "createdAt": "2025-04-21T09:00:17+0000",
  "type": "client.payment.method.binding"
}
```

### client.payment.method.bound

Сlient's card was successfully bound

```
{
  "id": "webhook-id",
  "clientId": "ed8ff528-3017-45bc-9d4d-f90e58f91bf9",
  "paymentToken": "7ee5900d-7a02-4bcf-a757-7a7b2fce462d",
  "providerType": "MTS",
  "createdAt": "2025-04-21T13:51:26+0000",
  "type": "client.payment.method.bound"

```

### client.payment.method.failed

Сlient did not bind the card or the binding was declined by the bank

```
{
  "id": "webhook-id",
  "clientId": "5646c1b7-d934-44ce-8490-938feb810910",
  "bindId": "97d009fe-80e6-426d-8ea2-9784f676e08e",
  "cardMask": "0380",
  "brand": "MASTERCARD",
  "providerType": "MTS",
  "createdAt": "2025-04-22T07:47:31+0000",
  "type": "client.payment.method.failed"
}
```

If the client has successfully bound their card, you can get the card id in Whitebird as the paymentToken value

# Fourth step

Get available payment methods for the client 

### POST api/v2/exchange/merchant/payment/method

### Request Header:

x-api-key 

### Request Body:

```jsx
{
    "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
    "fiatAsset": "RUB", -- optional (available RUB)
    "orderType": "SELL"  -- optional (available BUY and SELL)
}
```

### Response:

```jsx
{
        "id": "fa380b87-d2cd-471b-8d92-e13a2494f70d",
        "number": "4469 **** **** 8872",
        "brand": "MIR",
        "providerId": "MTS",
        "providerType": "MTS",
        "status": "ENABLED",
        "isRestricted": false,
        "isCrypto": false,
        "country": "RUSSIA"
}
```

If the card status is ENABLED, the id field value can be used for the exchange operation as the paymentToken field

# Buy Crypto Flow:

## First step

Get available payment methods for the client 

### POST api/v2/exchange/merchant/payment/method

### Request Header:

x-api-key 

### Request Body:

```jsx
{
    "clientId": "d028460b-95cd-4562-ac5e-767bf87df40b",
    "fiatAsset": "RUB", -- optional (available RUB)
    "orderType": "BUY"  -- optional (available BUY)
}
```

### Response:

```jsx
[
    {
        "id": "fa380b87-d2cd-471b-8d92-e13a2494f70d",
        "number": "4469 **** **** 8872",
        "brand": "MIR",
        "providerId": "MTS",
        "providerType": "MTS",
        "status": "ENABLED",
        "isRestricted": false,
        "isCrypto": false,
        "country": "RUSSIA"
    }
]
```

## Second step

To create a quota, use id **`api/v2/exchange/merchant/payment/method`** where MTS is enabled

### POST api/v2/exchange/merchant/quote

### Request Header:

x-api-key 

### Request Body:

```jsx
{
  "clientId": "d028460b-95cd-4562-ac5e-767bf87df40b",
  "fromAsset": {
    "code": "RUB",
    "network": null,
    "amount": "922"
  },
  "toAsset": {
    "code": "TRX",
    "network": "Tron"
  },
  "paymentMethod": "MTS",
  "paymentMethodToken": "fa380b87-d2cd-471b-8d92-e13a2494f70d",
  "destinationCryptoAddress": "TCT2pKJXo233hrKWQMeCptC8My1KGvtsU4"
}
```

### Response:

```jsx
{
    "quoteId": "620ece6b-bb48-4f68-9d6f-09496e94170f",
    "fromAsset": {
        "code": "RUB",
        "amount": "922"
    },
    "toAsset": {
        "code": "TRX",
        "network": "Tron",
        "amount": "33.326158"
    },
    "rate": 27.666,
    "plainRate": 26.7631,
    "fee": {
        "total": 23.05,
        "service": null,
        "network": 0.263,
        "asset": "RUB"
    },
    "expirationDate": "2026-03-31T06:42:05+0000"
}
```

## Third step

Create an order specifying `returnUrl` and `failUrl` (optional) after quote creation

### GET api/v2/exchange/merchant/buy?quoteId=c6d3c3b2-78ce-4374-8272-9bd2a9160a5d&returnUrl=https://www.google.com&failUrl=https://www.google.com

### Request Header:

x-api-key 

### Response:

```jsx
{
    "id": "1492954d-a1c5-40f9-9dde-abac06278504",
    "type": "BUY",
    "status": "PROCESSING",
    "creationDate": "2026-03-31T07:02:47.976606",
    "modificationDate": "2026-03-31T07:02:48.319782"
}
```

## Fourth step

### **Payment confirmation step (MTS Bank)**

In production flow, after order creation the client must complete payment in the **MTS Bank mobile app**:

**“Transfers abroad” → “Belarus” → “RUB” → Enter amount → “WhiteBird” → Enter order number → Confirm transfer.**

![Снимок экрана 2026-03-31 в 10.50.08.png](MTS/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA_%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0_2026-03-31_%D0%B2_10.50.08.png)

## **Check order status**

### GET /api/v2/exchange/merchant/order?orderId=1492954d-a1c5-40f9-9dde-abac06278504

### Request Header:

x-api-key 

### Response:

```jsx
{
    "id": "1492954d-a1c5-40f9-9dde-abac06278504",
    "type": "BUY",
    "status": "COMPLETED",
    "creationDate": "2026-03-31T07:02:47.976606",
    "modificationDate": "2026-03-31T07:04:56.869242",
    "number": 831000003399,
    "exchangeOperation": {
        "inputCurrency": "RUB",
        "inputAsset": 922,
        "outputCurrency": "TRX",
        "outputAsset": 33.326158,
        "exchangeFeeAssetInFiat": 23.05,
        "bonusOutputAsset": null,
        "plainRatio": 26.7631,
        "ratio": 27.666,
        "currencyPair": {
            "fromCurrency": "RUB",
            "toCurrency": "TRX"
        }
    },
    "cryptoTransaction": {
        "hash": "6dca5fb9350096325e030c0e2ab582be0644a5c571a160d356f5c15645ccf73b",
        "externalCryptoAddress": "TCT2pKJXo233hrKWQMeCptC8My1KGvtsU4",
        "internalCryptoAddress": "TPLBm3TBPd6PgGEBKPACKSsL4UEQTPnCtU",
        "fromAddress": "TPLBm3TBPd6PgGEBKPACKSsL4UEQTPnCtU",
        "toAddress": "TCT2pKJXo233hrKWQMeCptC8My1KGvtsU4",
        "status": "CONFIRMED",
        "currency": "TRX",
        "fee": "0.268",
        "feePaymentEnabledByClient": false,
        "type": "AUTO",
        "comment": null
    },
    "fiatTransaction": {
        "status": "APPROVED",
        "paymentToken": "fa380b87-d2cd-471b-8d92-e13a2494f70d",
        "post": null,
        "brand": null,
        "internalToken": null,
        "orderIdentity": "7307742",
        "link": null,
        "providerType": "MTS",
        "paymentType": null,
        "processingBank": null,
        "resultMessage": null,
        "currency": "RUB"
    },
    "client": {
        "clientId": "d028460b-95cd-4562-ac5e-767bf87df40b"
    },
    "serverDate": "2026-03-31T07:06:18+0000",
    "exchangeType": "SELL",
    "operationType": "FIAT_TO_CRYPTO",
    "orderType": "DEFAULT",
    "completionDate": "2026-03-31T07:04:56+0000",
    "resultMessage": null,
    "submitByResident": null,
    "merchantName": "wb",
    "merchantBonus": null,
    "promoCodeDetails": null,
    "fromSource": "EXT",
    "toSource": "EXT",
    "expiresAtDate": null
}
```
