# WhiteBird flow for enable OnRamp/OffRamp

This document is a practical end-to-end guide for partner backend teams.  
It explains the exact sequence required to make OnRamp/OffRamp available for a client: full registration, KYC completion, crypto test (when required), payment method/card binding, and final readiness checks before creating quote/order requests.

## 1) Register client with full KYC data

### Endpoint

`POST {{URL}}/api/v2/kyc/merchant/client/register`

### Headers

- `x-api-key: <merchant-api-key>`
- `Content-Type: application/json`

### Request body (test example)

```json
{
  "email": "test.user.testov+29042026a@ya.ru",
  "firstNameRu": "Кирилл",
  "lastNameRu": "Воронцов",
  "patronymicRu": "Михайлович",
  "firstName": "Kirill",
  "lastName": "Vorontsov",
  "residence": "112",
  "placeOfBirth": "Республика Беларусь, г. Минск",
  "birthDate": "1992-07-17",
  "nationality": "112",
  "registrationCountry": "112",
  "registrationRegion": "Минская область",
  "registrationCity": "Минск",
  "registrationStreet": "Улица Примерная",
  "registrationHouseAndFlat": "10-15",
  "identityDocType": "3",
  "identityDocIssueDate": "2024-01-14",
  "identityDocExpireDate": "2075-08-20",
  "identityDocNumber": "DP9735284",
  "personalNumber": "1084110P146PB7",
  "postCode": "781609",
  "phone": "+375293863256",
  "gender": "муж",
  "residenceDistrict": "Ленинский",
  "identityDocIssuer": "ОВД Ленинского района г. Минска",
  "exchangeInPersonalInterests": true,
  "notUSTaxPayer": true,
  "agreedWithOffer": true
}
```

### Expected response

```json
{
  "id": "0d58e7ec-0369-48d7-9804-90c6b23a52be",
  "status": "PENDING"
}
```

Save `id` as `clientId`.

## 2) If client is `112` (Belarus) -> complete crypto test

### 2.1 Get test

`GET {{URL}}/api/v2/kyc/merchant/client/crypto-test?clientId={{clientId}}`

If response contains `"cryptoTestRequired": true`, submit answers.

### 2.2 Submit answers

`POST {{URL}}/api/v2/kyc/merchant/client/crypto-test`

```json
{
  "clientId": "0d58e7ec-0369-48d7-9804-90c6b23a52be",
  "answers": {
    "1": 2,
    "2": 4,
    "3": 9,
    "4": 11,
    "5": 14
  }
}
```

Expected success:

```json
{
  "accepted": true
}
```

## 3) Poll final KYC status

Endpoint:

`POST {{URL}}/api/v2/kyc/merchant/client/status`

```json
{
  "clientId": "0d58e7ec-0369-48d7-9804-90c6b23a52be"
}
```

Ready for exchange when status becomes `VERIFIED`.

## 4) Prepare payment context (card or account/token)

### 4.1 First step: check payment provider availability

`POST {{URL}}/api/v2/exchange/merchant/payment/provider`

Request Header:

- `x-api-key`

Request Body:

```json
{
  "clientId": "93a8b39b-d883-4de3-9527-d92b1eabe38c",
  "fiatAsset": "BYN",
  "orderType": "BUY"
}
```

Response (actual endpoint returns an array):

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

### 4.2 Second step: generate link to bind client card

`POST {{URL}}/api/v2/exchange/merchant/payment/card/bind`

Request Header:

- `x-api-key`

Request Body:

```json
{
  "clientId": "1a0e2c64-8a90-4144-ac05-5e66bde1ca84",
  "providerType": "ASSIST",
  "returnUrl": "https://www.google.com"
}
```

Response:

```json
{
  "url": "https://payments.t.paysecure.ru/pay/p2p/register_ccard.cfm?..."
}
```

### 4.3 Third step: open link and complete card binding flow

Open `url` from step 2 for the client.

Test card data:

- Number: `4111 1111 1111 1111`
- Expiration: `12/30`
- CVV: `123`

After successful card binding, client is redirected to `returnUrl`.

### 4.4 How to get card binding result

Use WhiteBird webhooks or call payment methods API.

Webhook events:

1. `client.payment.method.binding` (binding started)

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

2. `client.payment.method.bound` (binding success)

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

3. `client.payment.method.failed` (binding failed/declined)

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

If card binding is successful, use `paymentToken` from `client.payment.method.bound`.

### 4.5 Fourth step: get payment methods/tokens for operations

`POST {{URL}}/api/v2/exchange/merchant/payment/method`

Request Body:

```json
{
  "clientId": "0d58e7ec-0369-48d7-9804-90c6b23a52be",
  "fiatAsset": "BYN",
  "orderType": "BUY"
}
```

Response: 

```json
[
    {
        "id": "ee6693e1-c340-47cb-8b9e-29304b6d9fd8",
        "number": "**** **** **** 1111",
        "brand": "VISA",
        "providerId": "ASSIST",
        "providerType": "ASSIST",
        "status": "ENABLED",
        "isRestricted": false,
        "isCrypto": false,
        "country": "Russia"
    }
]
```
After bind redirect flow finishes, call `/payment/method` again and use returned `id` as payment token.

## 5) What is still required to enable first OnRamp/OffRamp

In practice, after KYC is `VERIFIED`, the remaining setup is:

1. Provider and payment method resolution.
2. Either:
   - card binding (for providers with `addPaymentMethod=true`), or
   - use existing account/method token returned by `/payment/method` (for providers where card binding is not required).

## 6) WhiteBird runtime validations during order creation

Main checks include:

- client status must be `VERIFIED`;
- required crypto test must be completed;
- phone confirmation / phone validity checks;
- active order/deposit/withdrawal restrictions;
- payment provider/token checks;
- limits, balances, route availability, crypto address checks.

---

## 7) Errors and status routing for this flow

This table covers requests used in this document:

- `/api/v2/kyc/merchant/client/register`
- `/api/v2/kyc/merchant/client/crypto-test` (GET, POST)
- `/api/v2/kyc/merchant/client/status`
- `/api/v2/exchange/merchant/payment/provider`
- `/api/v2/exchange/merchant/payment/card/bind`
- `/api/v2/exchange/merchant/payment/method`

### Errors and status routing table

| Scope | HTTP | `status` | When it occurs | Example JSON response |
|---|---:|---|---|---|
| Any endpoint from this flow | `401` | `Unauthorized` | Invalid/missing `x-api-key`; `externalClientId` mismatch when merchant requires it | `{"message":"Invalid api key","code":401,"status":"Unauthorized"}` |
| Any endpoint from this flow | `429` | `Too Many Requests` | Rate limit exceeded | `{"message":"Too many requests","code":429,"status":"Too Many Requests"}` |
| Any endpoint from this flow | `415` | `Unsupported Media Type` | Request sent with unsupported `Content-Type` | `{"message":"Unsupported content type: text/plain; ...","code":415,"status":"Unsupported Media Type"}` |
| Any endpoint from this flow | `400` | `Bad Request` | Invalid JSON body, missing request param, or generic runtime validation error | `{"message":"Invalid request body","code":400,"status":"Bad Request"}` |
| Any endpoint with `@Valid` request body | `400` | `BAD_REQUEST` | Bean validation failed (`@NotNull`, `@NotBlank`, `@Email`, etc.) | `{"status":"BAD_REQUEST","message":"Validation errors","code":400,"fields":[{"field":"clientId","value":null,"message":"must not be null"}]}` |
| `POST /api/v2/kyc/merchant/client/register` | `400` | `Bad Request` | Personal number is missing for BLR registration country | `{"message":"Personal number must not be empty for registration country BLR","code":400,"status":"Bad Request"}` |
| `POST /api/v2/kyc/merchant/client/crypto-test` | `400` | `Bad Request` | Wrong crypto test answers | `{"message":"Wrong answers to crypto test","code":400,"status":"Bad Request"}` |
| KYC endpoints with merchant-client lookup (`crypto-test`, `status`) | `400` | `Bad Request` | Client not found in KYC context (validation/runtime failure) | `{"message":"Client not found","code":400,"status":"Bad Request"}` |
| `/api/v2/exchange/merchant/payment/provider` | `400` | `INVALID_PAYMENT_PROVIDER` | Payment provider is unknown/not available for current request | `{"message":"Payment provider not found","code":400,"status":"INVALID_PAYMENT_PROVIDER"}` |
| `/api/v2/exchange/merchant/payment/provider`, `/api/v2/exchange/merchant/payment/method`, `/api/v2/exchange/merchant/payment/card/bind` | `400` | `CLIENT_NOT_FOUND` | `clientId` does not belong to merchant, or merchant-client relation missing | `{"message":"Client not found","code":400,"status":"CLIENT_NOT_FOUND"}` |
| `/api/v2/exchange/merchant/payment/card/bind` | `400` | `CARD_BINDING_FORBIDDEN` | Card binding operation is forbidden by business rules/provider constraints | `{"message":"<provider/business restriction message>","code":400,"status":"CARD_BINDING_FORBIDDEN"}` |
| `/api/v2/exchange/merchant/payment/method` | `400` | `INVALID_PAYMENT_METHOD` | Payment method cannot be validated for the selected operation | `{"message":"Invalid payment method","code":400,"status":"INVALID_PAYMENT_METHOD"}` |
| `/api/v2/exchange/merchant/payment/method` | `400` | `INVALID_PAYMENT_TOKEN` | Payment token is missing/invalid for selected provider | `{"message":"Invalid payment token","code":400,"status":"INVALID_PAYMENT_TOKEN"}` |

### Routing summary by HTTP status

| HTTP | Meaning for integrator | Typical action |
|---:|---|---|
| `400` | Request/business validation failed | Fix request data, client context, or provider/method parameters |
| `401` | API key invalid/missing or external client access mismatch | Check `x-api-key` and `externalClientId` usage |
| `415` | Unsupported media type | Use `Content-Type: application/json` |
| `429` | Rate limit exceeded | Retry later with backoff |
