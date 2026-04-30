# CUSTODIAL_WALLET

Кастодиальный кошелек нужен для операций с внутренним балансом клиента: пополнение, вывод, покупка, продажа и конвертация активов.  
В интерфейсе это 5 быстрых кнопок (`Пополнить`, `Отправить`, `Купить`, `Продать`, `Конвертация`), а на backend — набор `v2` merchant/client endpoint’ов для расчета и запуска операций.  
Ниже описан flow в стиле `V2.md`: endpoint, request, response.

## Headers
- `x-api-key: {{apiKey}}` (для merchant endpoint’ов)
- `Content-Type: application/json`

---

## 0) Base data for wallet screen

### Step 0.1 Current balance operations (merchant)
**GET** `{{URL}}/api/v2/exchange/merchant/balance/current?clientId={{clientId}}`

**Response**
```json
{
  "fiatOperations": [],
  "cryptoOperations": []
}
```

### Step 0.2 Assets for SDK accounting
**GET** `{{URL}}/api/v2/exchange/client/assets?merchantId=11111111-1111-1111-1111-111111111111&destination=SDK_ACCOUNTING`

**Response**
```json
{
  "fiatAssets": [
    { "id": "BYN", "code": "BYN" },
    { "id": "RUB", "code": "RUB" }
  ],
  "cryptoAssets": [
    { "id": "TRX", "code": "TRX", "network": "Tron" },
    { "id": "USDT_TRC", "code": "USDT", "network": "Tron", "protocol": "TRC-20" }
  ]
}
```

### Step 0.3 Enhanced accounts (client / merchant accounting)
**GET** `{{URL}}/api/v2/accounting/client/account/enhanced`

**Response**
```json
{
  "accounts": []
}
```

**GET** `{{URL}}/api/v2/accounting/merchant/account/enhanced`

**Response**
```json
{
  "accounts": []
}
```

### Step 0.4 Current balance operations (client)
**GET** `{{URL}}/api/v2/exchange/client/balance/current`

**Response**
```json
{
  "fiatOperations": [],
  "cryptoOperations": []
}
```

---

## 1) Пополнение (`deposit`)

### Step 1.1 Crypto quote before deposit
**POST** `{{URL}}/api/v3/exchange/client/quote`

**Request**
```json
{
  "fromAsset": "BYN",
  "fromAmount": 100,
  "toAsset": "USDT_TRC"
}
```

**Response**
```json
{
  "id": "quote-id",
  "actualRateValue": 0.315,
  "input": { "asset": "BYN", "amount": 100 },
  "output": { "asset": "USDT_TRC", "amount": 31.5 }
}
```

### Step 1.2 Create crypto deposit (client)
**POST** `{{URL}}/api/v2/exchange/client/balance/crypto/deposit`

**Request**
```json
{
  "accountType": "WALLET",
  "asset": {
    "amount": 100,
    "code": "USDT_TRC",
    "network": "Tron"
  }
}
```

**Response**
```json
{
  "transactionId": "e9b08950-ed34-4e78-88ca-5e74b22a125c",
  "depositCryptoAddress": "0x3b172141906157c335B9F52BD9BfDBdEe8f59f16"
}
```

### Step 1.3 Get fiat payment methods
**POST** `{{URL}}/api/v2/exchange/client/payment/method`

**Request**
```json
{
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
    "providerType": "ASSIST",
    "status": "ENABLED"
  }
]
```

### Step 1.4 Create fiat deposit (client)
**POST** `{{URL}}/api/v2/exchange/client/balance/fiat/deposit`

**Request**
```json
{
  "accountType": "WALLET",
  "asset": {
    "amount": 100,
    "code": "BYN"
  },
  "fiatProviderType": "ASSIST",
  "paymentMethodId": "payment-token"
}
```

**Response**
```json
{
  "transactionId": "fiat-tx-id",
  "paymentLink": "https://payment-provider/link"
}
```

### Step 1.5 Merchant crypto deposit endpoint (separate)
**POST** `{{URL}}/api/v2/exchange/merchant/balance/crypto/deposit`

**Request**
```json
{
  "clientId": "b62c5c11-3f1d-4e54-95f5-4f19f2fd4e48",
  "accountType": "WALLET",
  "asset": {
    "amount": 100,
    "code": "USDT_TRC",
    "network": "Tron"
  }
}
```

**Response**
```json
{
  "transactionId": "827fe5dd-651b-4283-b333-15675e41a3bb",
  "depositCryptoAddress": "TCT2pKJXo233hrKWQMeCptC8My1KGvtsU4"
}
```

---

## 2) Отправить (`withdrawal`)

### Step 2.1 Crypto withdrawal calculate (merchant)
**POST** `{{URL}}/api/v2/exchange/merchant/balance/crypto/withdrawal/calculate`

**Request**
```json
{
  "clientId": "bab43583-fb31-4685-b470-33c6b89e1718",
  "accountType": "WALLET",
  "asset": {
    "amount": 10,
    "code": "TRX",
    "network": "Tron"
  },
  "toAddress": "TKFLbWh9oivTF7AFZTpCeVQn1fGx9iiJxM"
}
```

**Response**
```json
{
  "fromAmount": 10,
  "feeAmount": 0.1,
  "toAmount": 9.9
}
```

### Step 2.2 Fiat withdrawal calculate (merchant)
**POST** `{{URL}}/api/v2/exchange/merchant/balance/fiat/withdrawal/calculate`

**Request**
```json
{
  "clientId": "bab43583-fb31-4685-b470-33c6b89e1718",
  "accountType": "WALLET",
  "asset": {
    "amount": 100,
    "code": "RUB"
  },
  "fiatProviderType": "CA"
}
```

**Response**
```json
{
  "fromAmount": 100,
  "feeAmount": 2.5,
  "toAmount": 97.5
}
```

---

## 3) Купить (`buy`) — V2 flow

### Step 3.1 Create quote
**POST** `{{URL}}/api/v2/exchange/merchant/quote`

**Request**
```json
{
  "clientId": "{{clientId}}",
  "fromAsset": "BYN",
  "fromAmount": 100,
  "toAsset": "USDT_TRC",
  "paymentMethod": "ASSIST"
}
```

**Response**
```json
{
  "id": "quote-id",
  "rate": 0.315,
  "plainRate": 0.31,
  "fee": {
    "total": 2.5
  }
}
```

### Step 3.2 Create buy order
**GET** `{{URL}}/api/v2/exchange/merchant/buy?quoteId={{quoteId}}`

**Response**
```json
{
  "id": "order-id",
  "status": "PROCESSING",
  "fiatPaymentLink": "https://provider/link"
}
```

---

## 4) Продать (`sell`) — V2 flow

### Step 4.1 Create quote
**POST** `{{URL}}/api/v2/exchange/merchant/quote`

**Request**
```json
{
  "clientId": "{{clientId}}",
  "fromAsset": "USDT_TRC",
  "fromAmount": 100,
  "toAsset": "RUB",
  "paymentMethod": "CA"
}
```

**Response**
```json
{
  "id": "quote-id",
  "rate": 95.2,
  "plainRate": 96.1,
  "fee": {
    "total": 1.5
  }
}
```

### Step 4.2 Create sell order
**GET** `{{URL}}/api/v2/exchange/merchant/sell?quoteId={{quoteId}}`

**Response**
```json
{
  "id": "order-id",
  "status": "PROCESSING",
  "depositCryptoAddress": "TKFLbWh9oivTF7AFZTpCeVQn1fGx9iiJxM"
}
```

---

## 5) Operation details (общий для deposit/withdraw/buy/sell)

### Step 5.1 Get operation history/details
**POST** `{{URL}}/api/v2/exchange/merchant/balance/operation?page=0&size=20&sort=creationDate,desc`

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
      "transactionId": "tx-id",
      "transactionType": "DEPOSIT",
      "transactionStatus": "PROCESSING",
      "currency": "USDT_TRC",
      "creationDate": "2026-04-30T10:00:00Z"
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
- `from`, `to`: `yyyy-MM-dd`

---

## 6) Конвертация (`conversion`) — V2-style wallet flow

Для кастодиального кошелька конвертация идет через внутренний баланс (`INTERNAL_BALANCE` / `user_balance`) и тот же quote/order паттерн.

### Step 6.1 Get limit
**POST** `{{URL}}/api/v2/exchange/merchant/limit`

**Request**
```json
{
  "clientId": "{{clientId}}",
  "fromAsset": "USDT_TRC",
  "toAsset": "BTC"
}
```

**Response**
```json
{
  "asset": {
    "id": "USDT_TRC",
    "code": "USDT"
  },
  "min": 10,
  "max": 50000
}
```

### Step 6.2 Create quote
**POST** `{{URL}}/api/v2/exchange/merchant/quote`

**Request**
```json
{
  "clientId": "{{clientId}}",
  "fromAsset": "USDT_TRC",
  "fromAmount": 100,
  "toAsset": "BTC",
  "paymentMethod": "INTERNAL_BALANCE"
}
```

**Response**
```json
{
  "id": "quote-id",
  "rate": 0.000015,
  "plainRate": 0.0000152
}
```

### Step 6.3 Create order
**GET** `{{URL}}/api/v2/exchange/merchant/buy?quoteId={{quoteId}}`

**Response**
```json
{
  "id": "order-id",
  "status": "PROCESSING"
}
```

### Step 6.4 Enhanced accounting balance
**GET** `{{URL}}/api/v2/accounting/merchant/account/enhanced`

**Response**
```json
{
  "accounts": []
}
```

