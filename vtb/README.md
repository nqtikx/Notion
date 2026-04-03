# VTB

VTB is a payment provider for bank transfer processing in RUB. The payment provider does not require card binding in Whitebird. Payment is processed through the provider banking flow after order creation.

## Available currencies:

- RUB

## Available Bank Card By Region:

- Russia

## Directions & Commission:

- Buy Crypto - not available in current configuration
- Sell Crypto 2 %

## Verify that the payment provider is available

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
        "id": "VTB",
        "name": "VTB",
        "addPaymentMethod": false,
        "config": {
            "paymentSystems": [
                {
                    "paymentSystem": null,
                    "type": "CA",
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

It is sufficient to verify that the payment provider is available via the id field.  id = VTB 

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
        "providerId": "VTB",
        "providerType": "VTB",
        "status": "ENABLED",
        "name": "VTB"
    }
]
```

If `VTB` is not returned (or returned with non-`ENABLED` status), this means `SELL` via VTB is not available for the current client/merchant/environment

## Second step

Create a quote for the selected VTB payment method

### POST api/v2/exchange/merchant/quote

### Request Header:

x-api-key 

### Request Body:

```jsx
{
  "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
  "fromAsset": {
    "code": "TRX",
    "network": "Tron",
    "amount": "50.629578"
},
  "toAsset": {
    "code": "RUB",
    "network": null
  },
  "paymentMethod": "VTB"
}
```

### Response:

```jsx
{
    "quoteId": "a66199ab-da47-469b-ae75-721f040d5f17",
    "fromAsset": {
        "code": "TRX",
        "network": "Tron",
        "amount": "50.629578"
    },
    "toAsset": {
        "code": "RUB",
        "amount": "1299.77"
    },
    "rate": 25.6721,
    "plainRate": 26.1961,
    "fee": {
        "total": 26.53,
        "service": null,
        "network": 0,
        "asset": "RUB"
    },
    "expirationDate": "2026-03-31T10:15:07+0000"
}
```

## Third step

Create a sell order using the created quote.

### GET api/v2/exchange/merchant/sell?quoteId=e50dc4e9-d61c-4617-9ccc-1eea152e7d8a

### Request Header:

x-api-key 

### Response:

```jsx
{
    "id": "f6f5524d-c933-42be-8e67-128379dfd43e",
    "type": "SELL",
    "status": "PROCESSING",
    "creationDate": "2026-03-31T11:50:47.618594",
    "modificationDate": "2026-03-31T11:50:48.370130",
    "cryptoTransaction": null,
    "expiresAtDate": "2026-03-31T12:20:45.931382",
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
    "id": "c711b777-5d13-4156-a57d-456cc307626b",
    "type": "SELL",
    "status": "PROCESSING",
    "creationDate": "2026-03-31T10:42:22.598426",
    "modificationDate": "2026-03-31T10:42:22.939084",
    "number": 211000003411,
    "exchangeOperation": {
        "inputCurrency": "TRX",
        "inputAsset": 50.629578,
        "outputCurrency": "RUB",
        "outputAsset": 1299.36,
        "exchangeFeeAssetInFiat": 26.52,
        "bonusOutputAsset": null,
        "plainRatio": 26.1879,
        "ratio": 25.664,
        "currencyPair": {
            "fromCurrency": "TRX",
            "toCurrency": "RUB"
        }
    },
    "cryptoTransaction": {
        "hash": null,
        "externalCryptoAddress": null,
        "internalCryptoAddress": "TY3GBBkVMcYra6c8gPqXwdi4msvd7HU7Mx",
        "fromAddress": null,
        "toAddress": "TY3GBBkVMcYra6c8gPqXwdi4msvd7HU7Mx",
        "status": "NEW",
        "currency": "TRX",
        "fee": null,
        "feePaymentEnabledByClient": false,
        "type": "AUTO",
        "comment": null
    },
    "fiatTransaction": {
        "status": "NEW",
        "paymentToken": "008eda7d-1b99-4f5b-90fb-291e3fbb128d",
        "post": null,
        "brand": null,
        "internalToken": "26226972800000104219",
        "orderIdentity": "d1a3fd86-5764-48dd-91c2-275a562019ea",
        "link": null,
        "providerType": "VTB",
        "paymentType": null,
        "processingBank": null,
        "resultMessage": null,
        "currency": "RUB"
    },
    "client": {
        "clientId": "d028460b-95cd-4562-ac5e-767bf87df40b"
    },
    "serverDate": "2026-03-31T10:56:46+0000",
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
    "expiresAtDate": "2026-03-31T11:12:21.361309"
}
```

For this step, the client must send crypto to the deposit address from the order response:

- **Address field:** `cryptoTransaction.toAddress`
- **Network / asset field:** `cryptoTransaction.currency` (in this example: `TRX`, i.e. Tron network)
- **Amount field:** `exchangeOperation.inputAsset` (in this example: `50.629578`)

The transfer must be made in the correct network and before the order expiration time (`expiresAtDate`).

After the transfer is sent, poll `GET /api/v2/exchange/merchant/order?orderId=**c711b777-5d13-4156-a57d-456cc307626b**` until the order reaches a final status (`COMPLETED` or `FAILED`).
