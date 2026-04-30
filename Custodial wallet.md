# CUSTODIAL_WALLET

The custodial wallet is used for operations with the client internal balance: deposit, withdrawal, buy, sell, and asset conversion.  
In the UI, these are 5 quick actions (`Deposit`, `Send`, `Buy`, `Sell`, `Conversion`), and for backend integration only merchant endpoints are described below.  
All examples below use `{{URL}}` format and `x-api-key`.

## Headers
- `x-api-key: {{apiKey}}`
- `Content-Type: application/json`

---

## 0) Wallet base data

### Step 0.1 Get available assets
**POST** `{{URL}}/api/v2/exchange/merchant/assets?destination=SDK_ACCOUNTING`

**Response**
```json
{
  "fiatAssets": [
    { "id": "BYN", "code": "BYN" },
    { "id": "RUB", "code": "RUB" },
    { "id": "USD", "code": "USD" },
    { "id": "EUR", "code": "EUR" }
  ],
  "cryptoAssets": [
    { "id": "BTC", "code": "BTC", "network": "Bitcoin" },
    { "id": "ETH", "code": "ETH", "network": "Ethereum" },
    { "id": "TRX", "code": "TRX", "network": "Tron" },
    { "id": "USDT_TRC", "code": "USDT", "network": "Tron", "protocol": "TRC-20" }
  ]
}
```

### Step 0.2 Get current balance operations
**GET** `{{URL}}/api/v2/exchange/merchant/balance/current?clientId={{clientId}}`

Required request params:
- `clientId`

**Response**
```json
{
  "fiatOperations": [
    {
      "number": 9210086,
      "accountType": "WALLET",
      "operationType": "DEPOSIT",
      "amount": 100,
      "transactionId": "fiat-transaction-id",
      "asset": "BYN",
      "status": "PROCESSING",
      "fiatProvider": "ASSIST",
      "orderIdentity": "order-id",
      "createdAt": "2026-04-30T10:00:00"
    }
  ],
  "cryptoOperations": [
    {
      "number": 9210087,
      "accountType": "WALLET",
      "operationType": "DEPOSIT",
      "submitTimeout": "DEFAULT",
      "transactionId": "crypto-transaction-id",
      "status": "PENDING",
      "depositCryptoAddress": "TCT2pKJXo233hrKWQMeCptC8My1KGvtsU4",
      "amount": 100,
      "asset": "USDT_TRC",
      "network": "Tron",
      "txHash": null,
      "createdAt": "2026-04-30T10:00:00"
    }
  ]
}
```

### Step 0.3 Get enhanced merchant account balances
**GET** `{{URL}}/api/v2/accounting/merchant/account/enhanced`

**Response**
```json
{
  "balances": [
    {
      "currency": "BTC",
      "type": "USER_BALANCE",
      "amount": 0.00005088,
      "usdRate": 77622.51,
      "usdAmount": 3.95,
      "creationDate": 1732799992688,
      "modificationDate": 1777534608348,
      "fiat": false
    },
    {
      "currency": "USDT",
      "type": "USER_BALANCE",
      "amount": 40.24507500,
      "usdRate": 1,
      "usdAmount": 40.25,
      "creationDate": 1732799992688,
      "modificationDate": 1777534606246,
      "fiat": false
    },
    {
      "currency": "USD",
      "type": "USER_BALANCE",
      "amount": 85.00,
      "usdRate": 1,
      "usdAmount": 85.00,
      "creationDate": 1732799992688,
      "modificationDate": 1732800042763,
      "fiat": true
    },
    {
      "currency": "RUB",
      "type": "USER_BALANCE",
      "amount": 4109.49,
      "usdRate": 0.012,
      "usdAmount": 49.31,
      "creationDate": 1732799992688,
      "modificationDate": 1776797970394,
      "fiat": true
    }
  ],
  "totalFiatUsdAmount": 134.31,
  "totalCryptoUsdAmount": 93.48
}
```

---

## 1) Deposit (`deposit`)

### Step 1.1 Create crypto deposit
**POST** `{{URL}}/api/v2/exchange/merchant/balance/crypto/deposit`

Required body fields:
- `clientId`
- `accountType`
- `asset.code`
- `asset.network`
- `asset.amount`

**Request**
```json
{
  "clientId": "b62c5c11-3f1d-4e54-95f5-4f19f2fd4e48",
  "accountType": "WALLET",
  "asset": {
    "code": "USDT_TRC",
    "network": "Tron",
    "amount": 100
  }
}
```

**Response**
```json
{
  "transactionId": "e9b08950-ed34-4e78-88ca-5e74b22a125c",
  "depositCryptoAddress": "TCT2pKJXo233hrKWQMeCptC8My1KGvtsU4"
}
```

### Step 1.2 Get fiat payment methods
**POST** `{{URL}}/api/v2/exchange/merchant/payment/method`

Required body fields:
- `clientId`

Optional filters:
- `fiatAsset`
- `orderType`
- `destination`
- `providers`
- `isCrypto`
- `countryGroup`

**Request**
```json
{
  "clientId": "b62c5c11-3f1d-4e54-95f5-4f19f2fd4e48",
  "fiatAsset": "BYN",
  "orderType": "BUY",
  "destination": "SDK_ACCOUNTING"
}
```

**Response**
```json
[
  {
    "id": "payment-token",
    "number": "**** **** **** 1111",
    "brand": "VISA",
    "providerId": "ASSIST",
    "providerType": "ASSIST",
    "status": "ENABLED",
    "isRestricted": false,
    "isCrypto": false,
    "country": "Belarus",
    "currency": "BYN",
    "supportedCurrencies": ["BYN"]
  }
]
```

### Step 1.3 Create fiat deposit
**POST** `{{URL}}/api/v2/exchange/merchant/balance/fiat/deposit`

Required body fields:
- `clientId`
- `accountType`
- `fiatProviderType`
- `asset.code`
- `asset.amount`
- one of `paymentToken` or `internalToken`

**Request**
```json
{
  "clientId": "b62c5c11-3f1d-4e54-95f5-4f19f2fd4e48",
  "accountType": "WALLET",
  "fiatProviderType": "ASSIST",
  "paymentToken": "payment-token",
  "asset": {
    "code": "BYN",
    "amount": 100
  }
}
```

**Response**
```json
{
    "fiatPaymentLink": "https://payments.t.paysecure.ru/pay/p2p",
    "creationDate": "2026-04-30T09:45:15+0000",
    "expirationMinutes": 15,
    "paymentDetails": {
        "paymentLink": "https://payments.t.paysecure.ru/pay/p2p",
        "notificationPhoneNumber": null
    }
}
```

---

## 2) Send (`withdrawal`)

### Step 2.1 Calculate crypto withdrawal
**POST** `{{URL}}/api/v2/exchange/merchant/balance/crypto/withdrawal/calculate`

Required body fields:
- `clientId`
- `asset.code`
- `asset.network`
- `asset.amount`
- `toAddress`

**Request**
```json
{
    "clientId": "{{clientId}}",
    "asset":{
        "amount":10,
        "code":"TRX",
        "network":"Tron"
    },
    "toAddress":"TCT2pKJXo233hrKWQMeCptC8My1KGvtsU4"  // destination crypto withdrawal address
}
```

**Response**
```json
{
    "id": "92db12c2-bf7a-402e-995b-3c43f1e4eb77",
    "withdrawalAmount": "10",
    "commissionAmount": "0.263",
    "receivedAmount": "9.737",
    "expirationDate": "2026-04-30T09:51:19+0000"
}
```

### Step 2.2 Create crypto withdrawal
**POST** `{{URL}}/api/v2/exchange/merchant/balance/crypto/withdrawal`

Required body fields:
- `clientId`
- `calculationId`
- `accountType`

**Request**
```json
{
    "clientId": "{{clientId}}",
    "accountType":"WALLET",
    "calculationId":"ea40bbbf-16a2-4fa2-aada-f55121c45eac",
    "comment":""  // MEMO/comment/TAG field, used as destination memo for networks like TON
}
```

**Response**
```json
{
  "transactionId": "crypto-withdrawal-transaction-id"
}
```

### Step 2.3 Calculate fiat withdrawal
**POST** `{{URL}}/api/v2/exchange/merchant/balance/fiat/withdrawal/calculate`

Required body fields:
- `clientId`
- `fiatProviderType`
- `asset.code`
- `asset.amount`

**Request**
```json
{
  "clientId": "{{clientId}}",
  "fiatProviderType": "ASSIST",
  "paymentToken": "{{payment_token}}",
  "asset": {
    "code": "BYN",
    "amount": 100
  }
}
```

**Response**
```json
{
    "id": null,
    "withdrawalAmount": "100",
    "commissionAmount": "1.5",
    "receivedAmount": "98.5",
    "expirationDate": null
}
```

### Step 2.4 Create fiat withdrawal
**POST** `{{URL}}/api/v2/exchange/merchant/balance/fiat/withdrawal`

Required body fields:
- `clientId`
- `accountType`
- `fiatProviderType`
- `asset.code`
- `asset.amount`
- one of `paymentToken` or `internalToken`

**Request**
```json
{
    "clientId": "{{clientId}}",
    "accountType": "WALLET",
    "fiatProviderType": "ASSIST",
    "paymentToken": "{{payment_token}}",
    "asset": {
        "code": "BYN",
        "amount": 100
    }
}
```

**Response**
```json
{
  "transactionId": "fiat-withdrawal-transaction-id"
}
```

---

## 3) Buy (`buy`) — merchant V3 flow

### Step 3.1 Create quote
**POST** `{{URL}}/api/v3/exchange/merchant/quote`

Required body fields:
- `input.type`
- `input.asset`
- `output.type`
- `output.asset`
- at least one amount: `input.amount` or `output.amount`

**Request**
```json
{
    "clientId": "{{clientId}}",
    "input":{
        "type":"FIAT_PROVIDER",  // operation type: INTERNAL_BALANCE / FIAT_PROVIDER / CRYPTO_TRANSFER
        "asset":"BYN",              // asset: BYN RUB EUR USD BTC ETH USDT_ERC USDC_USDC TRX USDT_TRC TON USDT_TON
        "amount":50,                // amount
        "provider": "ASSIST",       // provider
        "token": "{{payment_token}}"       // payment token id
    },
    "output":{
        "type":"INTERNAL_BALANCE",
        "asset":"TRX"
    }
}
```

**Response**
```json
{
    "id": "3cf9f5b7-1013-4769-b396-9eb28e6b408d",
    "rate": "TRX/BYN",
    "systemRateValue": "0.9768",
    "exchangeRateValue": "0.9768",
    "actualRateValue": "1.0469",
    "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
    "creationDate": "2026-04-30T11:28:17+0000",
    "expirationDate": "2026-04-30T11:28:47+0000",
    "input": {
        "type": "FIAT_PROVIDER",
        "asset": "BYN",
        "amount": "50",
        "feeAmount": "3.35",
        "provider": "ASSIST",
        "token": "fc4b130e-c3bf-4a3d-abe5-9ec5900c9868",
        "paymentType": "P2P",
        "processingBank": "BELARUSBANK"
    },
    "output": {
        "type": "INTERNAL_BALANCE",
        "asset": "TRX",
        "amount": "47.757985",
        "feeAmount": "0"
    }
}
```

### Step 3.2 Create buy order
**POST** `{{URL}}/api/v3/exchange/merchant/order`

**Request**
```json
{
    "quoteId":"47b2985a-2fe3-427c-9a18-6b16736c460e"
}

// exchange operation is processed immediately
```

**Response**
```json
{
    "id": "d938165d-2158-4f4e-8bf1-9ef6c5806fdc",
    "number": 721000004148,
    "conditions": {
        "fromAsset": "BYN",
        "toAsset": "TRX",
        "fromGrossAmount": "50",
        "fromNetAmount": "46.65",
        "fromFeeAmount": "3.35",
        "toGrossAmount": "47.757985",
        "toNetAmount": "47.757985",
        "toFeeAmount": "0",
        "promoCode": null,
        "rate": "TRX/BYN",
        "systemRateValue": "0.9768",
        "exchangeRateValue": "0.9768",
        "actualRateValue": "1.0469"
    },
    "recalculationReason": null,
    "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
    "status": "PROCESSING",
    "failureMessage": null,
    "completionDate": null,
    "creationDate": "2026-04-30T11:29:29+0000",
    "sessionId": null,
    "input": {
        "type": "FIAT_PROVIDER",
        "asset": "BYN",
        "amount": "50",
        "transactionAmount": "50",
        "feeAmount": "3.35",
        "status": "PROCESSING",
        "failureMessage": null,
        "expirationDate": null,
        "provider": "ASSIST",
        "paymentType": "P2P",
        "processingBank": "BELARUSBANK",
        "clientBank": null,
        "fromToken": "fc4b130e-c3bf-4a3d-abe5-9ec5900c9868",
        "toToken": "97fe9aa7-7805-438f-8c5e-aea24b4f9dc4",
        "link": "https://payments.t.paysecure.ru/pay/p2p/cc2mc.cfm?merchant_id=...&orderNumber=...&customerNumber=...&orderCurrency=BYN&orderAmount=50.0&checkValue=...&signature=...&tokenFrom=...&tokenTo=...",
        "processorTransactionId": "c4e50d3e83bd4fccb8bf8b742470475f",
        "post": null,
        "paymentSystem": null
    },
    "output": {
        "type": "INTERNAL_BALANCE",
        "asset": "TRX",
        "amount": "47.757985",
        "transactionAmount": "47.757985",
        "feeAmount": "0",
        "status": "NEW",
        "failureMessage": null,
        "expirationDate": null
    }
}
```

---

## 4) Sell (`sell`) — merchant V3 flow

### Step 4.1 Create quote
**POST** `{{URL}}/api/v3/exchange/merchant/quote`

Required body fields:
- `input.type`
- `input.asset`
- `output.type`
- `output.asset`
- at least one amount: `input.amount` or `output.amount`

**Request**
```json
{
    "clientId": "{{clientId}}",
    "input":{
        "type":"INTERNAL_BALANCE",  // operation type: INTERNAL_BALANCE / FIAT_PROVIDER / CRYPTO_TRANSFER
        "asset":"TRX",              // asset: BYN RUB EUR USD BTC ETH USDT_ERC USDC_USDC TRX USDT_TRC TON USDT_TON
        "amount":100                  // amount
    },
    "output":{
        "type":"FIAT_PROVIDER",
        "asset":"BYN",
        "provider": "ASSIST",                                // provider
        "token": "{{payment_token}}"      // payment token id
    }
}

// amount can be provided in input or output
// systemRateValue   - rate without fee
// exchangeRateValue - rate with fee
// actualRateValue   - currently not used in UI logic
// feeAmount         - fee amount
// expirationDate    - quote lifetime
```

**Response**
```json
{
    "id": "a95bf590-c029-47b2-bf95-adbcf50a11bb",
    "rate": "TRX/BYN",
    "systemRateValue": "0.9765",
    "exchangeRateValue": "0.9765",
    "actualRateValue": "0.9208",
    "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
    "creationDate": "2026-04-30T11:38:25+0000",
    "expirationDate": "2026-04-30T11:38:55+0000",
    "input": {
        "type": "INTERNAL_BALANCE",
        "asset": "TRX",
        "amount": "100",
        "feeAmount": "0"
    },
    "output": {
        "type": "FIAT_PROVIDER",
        "asset": "BYN",
        "amount": "92.08",
        "feeAmount": "5.57",
        "provider": "ASSIST",
        "token": "fc4b130e-c3bf-4a3d-abe5-9ec5900c9868",
        "paymentType": "P2P",
        "processingBank": "BELARUSBANK"
    }
}
```

### Step 4.2 Create sell order
**POST** `{{URL}}/api/v3/exchange/merchant/order`

**Request**
```json
{
    "quoteId":"a95bf590-c029-47b2-bf95-adbcf50a11bb"
}

// exchange operation is processed immediately
```

**Response**
```json
{
    "id": "2bf54839-b540-452b-9014-3ba9d32a1e93",
    "number": 161000004149,
    "conditions": {
        "fromAsset": "TRX",
        "toAsset": "BYN",
        "fromGrossAmount": "100",
        "fromNetAmount": "100",
        "fromFeeAmount": "0",
        "toGrossAmount": "97.65",
        "toNetAmount": "92.08",
        "toFeeAmount": "5.57",
        "promoCode": null,
        "rate": "TRX/BYN",
        "systemRateValue": "0.9765",
        "exchangeRateValue": "0.9765",
        "actualRateValue": "0.9208"
    },
    "recalculationReason": null,
    "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
    "status": "PROCESSING",
    "failureMessage": null,
    "completionDate": null,
    "creationDate": "2026-04-30T11:40:23+0000",
    "sessionId": null,
    "input": {
        "type": "INTERNAL_BALANCE",
        "asset": "TRX",
        "amount": "100",
        "transactionAmount": "100",
        "feeAmount": "0",
        "status": "COMPLETED",
        "failureMessage": null,
        "expirationDate": null
    },
    "output": {
        "type": "FIAT_PROVIDER",
        "asset": "BYN",
        "amount": "97.65",
        "transactionAmount": "92.08",
        "feeAmount": "5.57",
        "status": "NEW",
        "failureMessage": null,
        "expirationDate": null,
        "provider": "ASSIST",
        "paymentType": "P2P",
        "processingBank": "BELARUSBANK",
        "clientBank": null,
        "fromToken": "97fe9aa7-7805-438f-8c5e-aea24b4f9dc4",
        "toToken": "fc4b130e-c3bf-4a3d-abe5-9ec5900c9868",
        "link": null,
        "processorTransactionId": "f42bb8b78d814be48f0256a96c2208ac",
        "post": null,
        "paymentSystem": null
    }
}
```

---

## 5) Operation history/details

### Step 5.1 Get order history/details
**POST** `{{URL}}/api/v3/exchange/merchant/order/history?page=0&size=20&sort=creationDate,desc`

**Request**
```json
{
    "clientIds": [
        "{{clientId}}"
    ]
}
// "operationTypes": [], // FIAT_PROVIDER / CRYPTO_TRANSFER / INTERNAL_BALANCE
// "statuses": [],       // PROCESSING / EXPIRED / COMPLETED / FAILED
// "assets": [],         // assets: BYN RUB EUR USD BTC ETH USDT_ERC USDC_USDC TRX USDT_TRC TON USDT_TON
// "completionDateFrame":{    // completion date range
//     "start":"2024-08-25T00:00:00+0300",
//     "end":"2026-09-02T00:00:00+0300"
// },
// "creationDateFrame":{      // creation date range
//     "start":"2024-08-25T00:00:00+0300",
//     "end":"2026-09-02T00:00:00+0300"
// }
```

**Response**
```json
{
  "content": [
    {
      "id": "2bf54839-b540-452b-9014-3ba9d32a1e93",
      "number": 161000004149,
      "conditions": {
        "fromAsset": "TRX",
        "toAsset": "BYN",
        "fromGrossAmount": "100",
        "fromNetAmount": "100",
        "fromFeeAmount": "0",
        "toGrossAmount": "97.65",
        "toNetAmount": "92.08",
        "toFeeAmount": "5.57",
        "promoCode": null,
        "rate": "TRX/BYN",
        "systemRateValue": "0.9765",
        "exchangeRateValue": "0.9765",
        "actualRateValue": "0.9208"
      },
      "recalculationReason": null,
      "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
      "status": "PROCESSING",
      "failureMessage": null,
      "completionDate": null,
      "creationDate": "2026-04-30T11:40:23+0000",
      "sessionId": null,
      "input": {
        "type": "INTERNAL_BALANCE",
        "asset": "TRX",
        "amount": "100",
        "transactionAmount": "100",
        "feeAmount": "0",
        "status": "COMPLETED",
        "failureMessage": null,
        "expirationDate": null
      },
      "output": {
        "type": "FIAT_PROVIDER",
        "asset": "BYN",
        "amount": "97.65",
        "transactionAmount": "92.08",
        "feeAmount": "5.57",
        "status": "NEW",
        "failureMessage": null,
        "expirationDate": null,
        "provider": "ASSIST",
        "paymentType": "P2P",
        "processingBank": "BELARUSBANK",
        "clientBank": null,
        "fromToken": "97fe9aa7-7805-438f-8c5e-aea24b4f9dc4",
        "toToken": "fc4b130e-c3bf-4a3d-abe5-9ec5900c9868",
        "link": null,
        "processorTransactionId": "f42bb8b78d814be48f0256a96c2208ac",
        "post": null,
        "paymentSystem": null
      }
    }
  ],
  "totalElements": 1,
  "totalPages": 1,
  "number": 0,
  "size": 20
}
```

---

## 6) Conversion (`conversion`) — merchant V3 flow

For custodial wallet, conversion goes through internal balance (`USER_BALANCE` / `INTERNAL_BALANCE`) and standard V3 quote/order flow.


### Step 6.1 Check limits
**POST** `{{URL}}/api/v3/exchange/merchant/limit`

**Request**
```json
{
    "clientId": "{{clientId}}",
    "fromAsset": "USDT_TRC",
    "fromPaymentDetails": {
        "type": "INTERNAL_BALANCE"
    },
    "toAsset": "TRX",
    "toPaymentDetails": {
        "type": "INTERNAL_BALANCE"
    }
}
```

**Response**
```json
{
    "fromMinAmount": "0.00515231",
    "fromMaxAmount": "6.43319589",
    "toMinAmount": "947.37",
    "toMaxAmount": "1182894.74"
}
```

### Step 6.2 Create quote
**POST** `{{URL}}/api/v3/exchange/merchant/quote`

Required body fields:
- `input.type`
- `input.asset`
- `output.type`
- `output.asset`
- at least one amount: `input.amount` or `output.amount`

**Request**
```json
{
    "clientId": "{{clientId}}",
    "input":{
        "type":"INTERNAL_BALANCE",  // operation type: INTERNAL_BALANCE / FIAT_PROVIDER / CRYPTO_TRANSFER
        "asset":"USDT_TRC",              // asset: BYN RUB EUR USD BTC ETH USDT_ERC USDC_USDC TRX USDT_TRC TON USDT_TON
        "amount":5                  // amount
    },
    "output":{
        "type":"INTERNAL_BALANCE",
        "asset":"TRX"
    }
}

// amount can be provided in input or output
// systemRateValue   - rate without fee
// exchangeRateValue - rate with fee
// actualRateValue   - currently not used in UI logic
// feeAmount         - fee amount
// expirationDate    - quote lifetime
```

**Response**
```json
{
    "id": "601b24b6-c7c3-4205-8396-79903f76f25e",
    "rate": "TRX/USDT_TRC",
    "systemRateValue": "0.3255",
    "exchangeRateValue": "0.3255",
    "actualRateValue": "0.3305",
    "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
    "creationDate": "2026-04-30T11:05:37+0000",
    "expirationDate": "2026-04-30T11:06:07+0000",
    "input": {
        "type": "INTERNAL_BALANCE",
        "asset": "USDT_TRC",
        "amount": "5",
        "feeAmount": "0"
    },
    "output": {
        "type": "INTERNAL_BALANCE",
        "asset": "TRX",
        "amount": "15.130568",
        "feeAmount": "0.230415"
    }
}
```

### Step 6.3 Create swap operation
**POST** `{{URL}}/api/v3/exchange/merchant/order`

**Request**
```json
{
    "quoteId":""
}
// exchange operation is processed immediately
```

**Response**
```json
{
    "id": "08d5b13c-5a5b-470b-b244-6a6becb7888b",
    "number": 821000004152,
    "conditions": {
        "fromAsset": "USDT_TRC",
        "toAsset": "TRX",
        "fromGrossAmount": "5",
        "fromNetAmount": "5",
        "fromFeeAmount": "0",
        "toGrossAmount": "15.346839",
        "toNetAmount": "15.116636",
        "toFeeAmount": "0.230203",
        "promoCode": null,
        "rate": "TRX/USDT_TRC",
        "systemRateValue": "0.3258",
        "exchangeRateValue": "0.3258",
        "actualRateValue": "0.3308"
    },
    "recalculationReason": null,
    "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
    "status": "COMPLETED",
    "failureMessage": null,
    "completionDate": "2026-04-30T13:10:38+0000",
    "creationDate": "2026-04-30T13:10:35+0000",
    "sessionId": null,
    "input": {
        "type": "INTERNAL_BALANCE",
        "asset": "USDT_TRC",
        "amount": "5",
        "transactionAmount": "5",
        "feeAmount": "0",
        "status": "COMPLETED",
        "failureMessage": null,
        "expirationDate": null
    },
    "output": {
        "type": "INTERNAL_BALANCE",
        "asset": "TRX",
        "amount": "15.116636",
        "transactionAmount": "15.116636",
        "feeAmount": "0.230203",
        "status": "COMPLETED",
        "failureMessage": null,
        "expirationDate": null
    }
}
```

