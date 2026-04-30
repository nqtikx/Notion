# CUSTODIAL_WALLET

Кастодиальный кошелек нужен для работы с внутренним балансом клиента: пополнение, вывод, обмен и просмотр текущих операций в одном интерфейсе.  
В UI это реализовано через 5 быстрых кнопок (`Пополнить`, `Отправить`, `Купить`, `Продать`, `Конвертация`) и набор API вызовов для расчета, создания операций и контроля статусов.  
Ниже — flow в формате endpoint-first с привязкой к фактическим контроллерам.

## Что доступно в CUSTODIAL_WALLET

- `Пополнить` (`deposit`)
- `Отправить` (`withdrawal`)
- `Купить` (`buy`)
- `Продать` (`sell`)
- `Конвертация` (`conversion`)

Базовый запрос для текущих операций по клиенту (merchant-side):

- `GET https://api.dev.wbdevel.net/api/v2/exchange/merchant/balance/current?clientId=<clientId>`

---

## 1) Пополнение (`deposit`)

Перед формой пополнения подгружаются справочники/контекст:

- `GET https://api.dev.wbdevel.net/api/v2/exchange/client/assets?merchantId=11111111-1111-1111-1111-111111111111&destination=SDK_ACCOUNTING`
- `GET https://api.dev.wbdevel.net/api/v2/accounting/client/account/enhanced`
- `GET https://api.dev.wbdevel.net/api/v2/exchange/client/balance/current`

### 1.1 Crypto deposit

Flow:

- (предрасчет в UI) `POST https://api.dev.wbdevel.net/api/v3/exchange/client/quote`
- создание депозита: `POST https://api.dev.wbdevel.net/api/v2/exchange/client/balance/crypto/deposit`

Пример ответа:

```json
{
  "transactionId": "e9b08950-ed34-4e78-88ca-5e74b22a125c",
  "depositCryptoAddress": "0x3b172141906157c335B9F52BD9BfDBdEe8f59f16"
}
```

### 1.2 Fiat deposit

Flow:

- методы оплаты: `POST https://api.dev.wbdevel.net/api/v2/exchange/client/payment/method`
- создание депозита: `POST https://api.dev.wbdevel.net/api/v2/exchange/client/balance/fiat/deposit`

### 1.3 Merchant-side crypto deposit endpoint

- `POST {{WB_URL}}/api/v2/exchange/merchant/balance/crypto/deposit`

Тело запроса:

```json
{
  "clientId": "b62c5c11-3f1d-4e54-95f5-4f19f2fd4e48",
  "transactionId": "827fe5dd-651b-4283-b333-15675e41a3bb"
}
```

Код-заметка: для `merchant/balance/crypto/deposit` фактический DTO включает параметры депозита (asset/accountType и т.д.), а `transactionId` используется в deprecated endpoint `POST /api/v2/exchange/merchant/balance/crypto/deposit/submit`.

---

## 2) Отправить (`withdrawal`)

### 2.1 Crypto withdrawal calculate

- `POST {{WB_URL}}/api/v2/exchange/merchant/balance/crypto/withdrawal/calculate`

Тело запроса:

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

### 2.2 Fiat withdrawal calculate

- `POST {{WB_URL}}/api/v2/exchange/merchant/balance/fiat/withdrawal/calculate`

Код-заметка: для fiat используется отдельный endpoint `.../fiat/withdrawal/calculate` (не `crypto`).

---

## 3) Купить (`buy`)

Для merchant API по коду используется **V2 flow**:

- quote: `POST {{WB_URL}}/api/v2/exchange/merchant/quote`
- order create (buy): `GET {{WB_URL}}/api/v2/exchange/merchant/buy?quoteId=...`

То есть для раздела `Купить` ориентируемся на V2 OnRamp flow (calculation/quote + buy).

---

## 4) Продать (`sell`)

Для merchant API по коду также используется **V2 flow**:

- quote: `POST {{WB_URL}}/api/v2/exchange/merchant/quote`
- order create (sell): `GET {{WB_URL}}/api/v2/exchange/merchant/sell?quoteId=...`

То есть для раздела `Продать` ориентируемся на V2 OffRamp flow (calculation/quote + sell).

---

## 5) Operation details (общий для 3/4 и применимо для 1/2)

История/детали балансовых операций:

- `POST {{WB_URL}}/api/v2/exchange/merchant/balance/operation?page=0&size=20&sort=creationDate,desc`

Тело запроса:

```json
{
  "clientId": "3e1469fa-8d35-441c-87b1-a007aeba2562",
  "transactionTypes": ["DEPOSIT"],
  "transactionStatuses": [],
  "from": null,
  "to": null
}
```

Поддерживаемые значения:

- `transactionTypes`: `DEPOSIT`, `WITHDRAWAL`
- `transactionStatuses`: `PROCESSING`, `DECLINED`, `PROCESSED`
- `currencies` (optional): `BTC`, `ETH`, `USDT`, `USDC`, `TRX`, `USDT_TRC`, `TON`, `USDT_TON`, `BYN`, `RUB`, `EUR`, `USD`
- `from`, `to`: дата в формате `yyyy-MM-dd`

---

## 6) Конвертация (`conversion`)

Для конвертации в кастодиальном кошельке используется тот же exchange-flow с внутренним балансом (`INTERNAL_BALANCE` / `type: user_balance`):

- limit (по сценарию): `POST {{WB_URL}}/api/v2/exchange/merchant/limit`
- quote: `POST {{WB_URL}}/api/v2/exchange/merchant/quote`
- order (buy/sell в зависимости от направления): `GET {{WB_URL}}/api/v2/exchange/merchant/buy` или `GET {{WB_URL}}/api/v2/exchange/merchant/sell`

Дополнительный accounting endpoint, который используется для расширенного баланса:

- `https://api.dev.wbdevel.net/api/v2/accounting/merchant/account/enhanced`

