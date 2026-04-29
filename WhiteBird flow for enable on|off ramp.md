# WhiteBird flow for enable OnRamp/OffRamp

## Goal

Step-by-step flow from client registration to first OnRamp/OffRamp operation, with concrete request bodies.

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

### 4.1 Get providers

`POST {{URL}}/api/v2/exchange/merchant/payment/provider`

```json
{
  "clientId": "0d58e7ec-0369-48d7-9804-90c6b23a52be",
  "fiatAsset": "BYN",
  "orderType": "BUY"
}
```

### 4.2 Get payment methods/tokens

`POST {{URL}}/api/v2/exchange/merchant/payment/method`

```json
{
  "clientId": "0d58e7ec-0369-48d7-9804-90c6b23a52be",
  "fiatAsset": "BYN",
  "orderType": "BUY"
}
```

### 4.3 If token required and missing -> bind card

`POST {{URL}}/api/v2/exchange/merchant/payment/card/bind`

```json
{
  "clientId": "0d58e7ec-0369-48d7-9804-90c6b23a52be",
  "providerType": "ASSIST",
  "returnUrl": "https://partner.example.com/payment/callback"
}
```

Expected response:

```json
{
  "url": "https://psp.example.com/bind?token=..."
}
```

After bind redirect flow finishes, call `/payment/method` again and use returned `id` as payment token.

## 5) What is still required to enable first OnRamp/OffRamp

In practice, after KYC is `VERIFIED`, the remaining setup is:

1. Provider and payment method resolution.
2. Either:
   - card binding (for providers with `addPaymentMethod=true`), or
   - use existing account/method token returned by `/payment/method` (for providers where card binding is not required).

No separate merchant API endpoint for "create current account" was found in `exchange-core` merchant exchange API.

## 6) WhiteBird runtime validations during order creation

Main checks include:

- client status must be `VERIFIED`;
- required crypto test must be completed;
- phone confirmation / phone validity checks;
- active order/deposit/withdrawal restrictions;
- payment provider/token checks;
- limits, balances, route availability, crypto address checks.
