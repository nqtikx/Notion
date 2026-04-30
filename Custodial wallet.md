# CUSTODIAL_WALLET

Кастодиальный кошелек нужен для операций с внутренним балансом клиента: пополнение, вывод, покупка, продажа и конвертация активов.  
В интерфейсе это 5 быстрых действий (`Пополнить`, `Отправить`, `Купить`, `Продать`, `Конвертация`), а для backend-интеграции ниже описаны только merchant endpoint’ы.  
Все примеры ниже используют формат `{{URL}}` и `x-api-key`.

## Headers
- `x-api-key: {{apiKey}}`
- `Content-Type: application/json`

---

## 0) Wallet base data

### Step 0.1 Get available assets
**POST** `{{URL}}/api/v2/exchange/merchant/assets?destination=SDK_ACCOUNTING`

**Request**
```json
{}
```

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

## 1) Пополнение (`deposit`)

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
  "fiatPaymentLink": "https://payment-provider/link",
  "creationDate": "2026-04-30T10:00:00",
  "expirationMinutes": 30,
  "paymentDetails": {
    "paymentLink": "https://payment-provider/link",
    "notificationPhoneNumber": null
  }
}
```

---

## 2) Отправить (`withdrawal`)

### Step 2.1 Calculate crypto withdrawal
**POST** `{{URL}}/api/v2/exchange/merchant/balance/crypto/withdrawal/calculate`

Required body fields:
- `clientId`
- `asset.code`
- `asset.network`
- `asset.amount`

**Request**
```json
{
  "clientId": "bab43583-fb31-4685-b470-33c6b89e1718",
  "asset": {
    "code": "TRX",
    "network": "Tron",
    "amount": 10
  },
  "toAddress": "TKFLbWh9oivTF7AFZTpCeVQn1fGx9iiJxM"
}
```

**Response**
```json
{
  "id": "calculation-id",
  "withdrawalAmount": 10,
  "commissionAmount": 0.1,
  "receivedAmount": 9.9,
  "expirationDate": "2026-04-30T10:15:00"
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
  "clientId": "bab43583-fb31-4685-b470-33c6b89e1718",
  "calculationId": "calculation-id",
  "accountType": "WALLET"
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
  "clientId": "bab43583-fb31-4685-b470-33c6b89e1718",
  "fiatProviderType": "CA",
  "paymentToken": "payment-token",
  "asset": {
    "code": "RUB",
    "amount": 100
  }
}
```

**Response**
```json
{
  "id": "calculation-id",
  "withdrawalAmount": 100,
  "commissionAmount": 2.5,
  "receivedAmount": 97.5,
  "expirationDate": "2026-04-30T10:15:00"
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
  "clientId": "bab43583-fb31-4685-b470-33c6b89e1718",
  "accountType": "WALLET",
  "fiatProviderType": "CA",
  "paymentToken": "payment-token",
  "asset": {
    "code": "RUB",
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

## 3) Купить (`buy`) — merchant V2 flow

### Step 3.1 Create quote
**POST** `{{URL}}/api/v2/exchange/merchant/quote`

Required body fields:
- `fromAsset.code`
- `fromAsset.amount` or `toAsset.amount` (only one side amount should be fixed)
- `toAsset.code`

**Request**
```json
{
  "clientId": "{{clientId}}",
  "fromAsset": {
    "code": "BYN",
    "amount": 100
  },
  "toAsset": {
    "code": "USDT_TRC",
    "network": "Tron"
  },
  "paymentMethod": "INTERNAL_BALANCE"
}
```

**Response**
```json
{
  "quoteId": "quote-id",
  "fromAsset": {
    "code": "BYN",
    "amount": 100
  },
  "toAsset": {
    "code": "USDT_TRC",
    "network": "Tron",
    "amount": 31.5
  },
  "rate": 0.315,
  "plainRate": 0.31,
  "fee": {
    "total": 2.5,
    "service": null,
    "network": 0.1,
    "asset": "BYN"
  },
  "expirationDate": "2026-04-30T10:15:00"
}
```

### Step 3.2 Create buy order
**GET** `{{URL}}/api/v2/exchange/merchant/buy?quoteId={{quoteId}}`

Required request params:
- `quoteId`

**Response**
```json
{
  "id": "order-id",
  "type": "BUY",
  "status": "PROCESSING",
  "creationDate": "2026-04-30T10:00:00",
  "modificationDate": "2026-04-30T10:00:00",
  "cryptoTransaction": null,
  "expiresAtDate": "2026-04-30T10:15:00",
  "fiatPaymentLink": null
}
```

---

## 4) Продать (`sell`) — merchant V2 flow

### Step 4.1 Create quote
**POST** `{{URL}}/api/v2/exchange/merchant/quote`

Required body fields:
- `fromAsset.code`
- `fromAsset.amount` or `toAsset.amount` (only one side amount should be fixed)
- `toAsset.code`

**Request**
```json
{
  "clientId": "{{clientId}}",
  "fromAsset": {
    "code": "USDT_TRC",
    "network": "Tron",
    "amount": 100
  },
  "toAsset": {
    "code": "RUB"
  },
  "paymentMethod": "INTERNAL_BALANCE"
}
```

**Response**
```json
{
  "quoteId": "quote-id",
  "fromAsset": {
    "code": "USDT_TRC",
    "network": "Tron",
    "amount": 100
  },
  "toAsset": {
    "code": "RUB",
    "amount": 9520
  },
  "rate": 95.2,
  "plainRate": 96.1,
  "fee": {
    "total": 1.5,
    "service": null,
    "network": 0.1,
    "asset": "RUB"
  },
  "expirationDate": "2026-04-30T10:15:00"
}
```

### Step 4.2 Create sell order
**GET** `{{URL}}/api/v2/exchange/merchant/sell?quoteId={{quoteId}}`

Required request params:
- `quoteId`

**Response**
```json
{
  "id": "order-id",
  "type": "SELL",
  "status": "PROCESSING",
  "creationDate": "2026-04-30T10:00:00",
  "modificationDate": "2026-04-30T10:00:00",
  "cryptoTransaction": null,
  "expiresAtDate": "2026-04-30T10:15:00",
  "depositCryptoAddress": "TKFLbWh9oivTF7AFZTpCeVQn1fGx9iiJxM"
}
```

---

## 5) Operation details

### Step 5.1 Get balance operation history/details
**POST** `{{URL}}/api/v2/exchange/merchant/balance/operation?page=0&size=20&sort=creationDate,desc`

Required body fields:
- `clientId`

Optional filters:
- `transactionTypes`
- `transactionStatuses`
- `currencies`
- `accountTypes`
- `creationDateFrame`
- `completionDateFrame`
- `balanceOperationId`
- `from` / `to` (deprecated; use `creationDateFrame`)

**Request**
```json
{
  "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
  "transactionTypes": ["DEPOSIT"],
  "transactionStatuses": [],
  "currencies": [],
  "from": null,
  "to": null
}
```

**Response**
```json
{
  "content": [
    {
      "id": "balance-operation-id",
      "transactionId": "transaction-id",
      "number": "9210086",
      "type": "DEPOSIT",
      "status": "PROCESSING",
      "post": null,
      "providerType": "ASSIST",
      "paymentSystem": "VISA",
      "transactionHash": null,
      "externalCryptoAddress": null,
      "asset": "BYN",
      "amount": 100,
      "requestedAmount": 100,
      "grossAmount": 100,
      "netAmount": 97.5,
      "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
      "userId": "user-identity",
      "creationDate": "2026-04-30T10:00:00",
      "completionDate": null
    }
  ],
  "totalElements": 1,
  "totalPages": 1,
  "number": 0,
  "size": 20
}
```

Supported values:
- `transactionTypes`: `DEPOSIT`, `WITHDRAWAL`
- `transactionStatuses`: `PROCESSING`, `DECLINED`, `PROCESSED`
- `currencies`: `BTC`, `ETH`, `USDT`, `USDC`, `TRX`, `USDT_TRC`, `TON`, `USDT_TON`, `BYN`, `RUB`, `EUR`, `USD`
- `from`, `to`: `yyyy-MM-dd` (deprecated fields)

---

## 6) Конвертация (`conversion`) — merchant V2 flow

Для кастодиального кошелька конвертация идет через внутренний баланс (`USER_BALANCE` / `INTERNAL_BALANCE`) и стандартный V2 quote/order flow.

### Step 6.1 Get limit
**POST** `{{URL}}/api/v2/exchange/merchant/limit`

Required body fields:
- `fromAsset.code`
- `toAsset.code`
- `paymentMethod`

**Request**
```json
{
  "clientId": "{{clientId}}",
  "fromAsset": {
    "code": "USDT_TRC",
    "network": "Tron"
  },
  "toAsset": {
    "code": "BTC",
    "network": "Bitcoin"
  },
  "paymentMethod": "INTERNAL_BALANCE"
}
```

**Response**
```json
{
  "asset": {
    "code": "USDT_TRC",
    "network": "Tron"
  },
  "min": 10,
  "max": 50000
}
```

### Step 6.2 Create quote
**POST** `{{URL}}/api/v2/exchange/merchant/quote`

Required body fields:
- `fromAsset.code`
- `fromAsset.amount` or `toAsset.amount` (only one side amount should be fixed)
- `toAsset.code`

**Request**
```json
{
  "clientId": "{{clientId}}",
  "fromAsset": {
    "code": "USDT_TRC",
    "network": "Tron",
    "amount": 100
  },
  "toAsset": {
    "code": "BTC",
    "network": "Bitcoin"
  },
  "paymentMethod": "INTERNAL_BALANCE"
}
```

**Response**
```json
{
  "quoteId": "quote-id",
  "fromAsset": {
    "code": "USDT_TRC",
    "network": "Tron",
    "amount": 100
  },
  "toAsset": {
    "code": "BTC",
    "network": "Bitcoin",
    "amount": 0.0015
  },
  "rate": 0.000015,
  "plainRate": 0.0000152,
  "fee": {
    "total": 1.5,
    "service": null,
    "network": 0.000001,
    "asset": "BTC"
  },
  "expirationDate": "2026-04-30T10:15:00"
}
```

### Step 6.3 Create order
**GET** `{{URL}}/api/v2/exchange/merchant/buy?quoteId={{quoteId}}`

Required request params:
- `quoteId`

**Response**
```json
{
  "id": "order-id",
  "type": "BUY",
  "status": "PROCESSING",
  "creationDate": "2026-04-30T10:00:00",
  "modificationDate": "2026-04-30T10:00:00",
  "cryptoTransaction": null,
  "expiresAtDate": "2026-04-30T10:15:00",
  "fiatPaymentLink": null
}
```

