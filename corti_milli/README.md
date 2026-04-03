# CORTI_MILLI

CORTI_MILLI is a payment provider for bank transfer processing in RUB. The payment provider does not require card binding in Whitebird. Payment is processed through the provider banking flow after order creation.

## Available currencies:

- RUB

## Available Bank Card By Region:

- Russia

## Directions & Commission:

- Buy Crypto - not available in current configuration
- Sell Crypto 2 %

# Sell Crypto Flow:

## First step

Get available payment methods for the client 

### POST api/v2/exchange/merchant/payment/method

### Request Header:

x-api-key 

### Request Body:

```jsx
{
    "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
    "fiatAsset": "RUB", // optional (available RUB)
    "orderType": "SELL"  // optional (available SELL)
}
```

### Response:

```jsx
[
    {
        "providerId": "CORTI_MILLI",
        "providerType": "CORTI_MILLI",
        "status": "ENABLED",
        "name": "CORTI_MILLI"
    }
]
```

If `CORTI_MILLI` is not returned (or returned with non-`ENABLED` status), `SELL` via CORTI_MILLI is not available for the current client/merchant/environment.

## Second step

Create a quote for the selected CORTI_MILLI payment method

### POST api/v2/exchange/merchant/quote

### Request Header:

x-api-key 

### Request Body:

```jsx
{
  "clientId": "YOUR_CLIENT_ID",
  "fromAsset": {
    "code": "TRX",
    "network": "Tron",
    "amount": "51.204483"
  },
  "toAsset": {
    "code": "RUB",
    "network": null
  },
  "paymentMethod": "CORTI_MILLI",
  "paymentMethodToken": "5058270370320789"
}
```

### Response:

```jsx
{
    "quoteId": "a7e8c187-4e68-4c1f-bb11-0cdfeb43a0a2",
    "fromAsset": {
        "code": "TRX",
        "network": "Tron",
        "amount": "51.204483"
    },
    "toAsset": {
        "code": "RUB",
        "amount": "1284.36"
    },
    "rate": 25.083,
    "plainRate": 25.8587,
    "fee": {
        "total": 39.72,
        "service": null,
        "network": 0,
        "asset": "RUB"
    },
    "expirationDate": "2026-03-31T14:07:05+0000"
}
```

## Third step

Create a sell order using the created quote.

### GET api/v2/exchange/merchant/sell?quoteId=420d72d9-715c-41d9-93e8-1d1cb20924e3

### Request Header:

x-api-key 

### Response:

```jsx
{
    "id": "78cc592e-cab0-4199-a70a-d0d4aab3ec6c",
    "type": "SELL",
    "status": "PROCESSING",
    "creationDate": "2026-03-31T14:50:42.625369",
    "modificationDate": "2026-03-31T14:50:43.861568",
    "cryptoTransaction": null,
    "expiresAtDate": "2026-03-31T15:20:39.089773",
    "depositCryptoAddress": "TDtMfvu2zafgdwKRhm7JEUqFf3SA9FgRW1"
}
```

## Fourth step

 Get order details and provide transfer instructions to the client.

### GET /api/v2/exchange/merchant/order?orderId=c711b777-5d13-4156-a57d-456cc307626b

### Request Header:

x-api-key 

### Response:

```jsx
{
    "id": "78cc592e-cab0-4199-a70a-d0d4aab3ec6c",
    "type": "SELL",
    "status": "PROCESSING",
    "creationDate": "2026-03-31T14:50:42.625369",
    "modificationDate": "2026-03-31T14:50:43.861568",
    "number": 741000003422,
    "exchangeOperation": {
        "inputCurrency": "TRX",
        "inputAsset": 51.204483,
        "outputCurrency": "RUB",
        "outputAsset": 1278.23,
        "exchangeFeeAssetInFiat": 39.53,
        "bonusOutputAsset": null,
        "plainRatio": 25.7352,
        "ratio": 24.9632,
        "currencyPair": {
            "fromCurrency": "TRX",
            "toCurrency": "RUB"
        }
    },
    "cryptoTransaction": {
        "hash": null,
        "externalCryptoAddress": null,
        "internalCryptoAddress": "TDtMfvu2zafgdwKRhm7JEUqFf3SA9FgRW1",
        "fromAddress": null,
        "toAddress": "TDtMfvu2zafgdwKRhm7JEUqFf3SA9FgRW1",
        "status": "NEW",
        "currency": "TRX",
        "fee": null,
        "feePaymentEnabledByClient": false,
        "type": "AUTO",
        "comment": null
    },
    "fiatTransaction": {
        "status": "NEW",
        "paymentToken": "5058270370320789",
        "post": null,
        "brand": null,
        "internalToken": "26226972800000104219",
        "orderIdentity": "47f6f80f-3813-4be1-8402-a5f06d2b5033",
        "link": null,
        "providerType": "CORTI_MILLI",
        "paymentType": null,
        "processingBank": null,
        "resultMessage": null,
        "currency": "RUB"
    },
    "client": {
        "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562"
    },
    "serverDate": "2026-03-31T14:51:52+0000",
    "exchangeType": "BUY",
    "operationType": "CRYPTO_TO_FIAT",
    "orderType": "DEFAULT",
    "completionDate": null,
    "resultMessage": null,
    "submitByResident": null,
    "merchantName": "wb",
    "merchantBonus": null,
    "promoCodeDetails": null,
    "fromSource": "EXT",
    "toSource": "EXT",
    "expiresAtDate": "2026-03-31T15:20:39.089773"
}
```

For this step, the client must send crypto to the deposit address from the order response:

- **Address field:** `cryptoTransaction.toAddress`
- **Network / asset field:** `cryptoTransaction.currency` (in this example: `TRX`, i.e. Tron network)
- **Amount field:** `exchangeOperation.inputAsset` (in this example: `51.204483`)

The transfer must be made in the correct network and before the order expiration time (`expiresAtDate`).

After the transfer is sent, poll `GET /api/v2/exchange/merchant/order?orderId=**78cc592e-cab0-4199-a70a-d0d4aab3ec6c**` until the order reaches a final status (`COMPLETED` or `FAILED`).
