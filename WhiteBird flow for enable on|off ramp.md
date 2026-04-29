# WhiteBird flow for enable OnRamp/OffRamp

## Goal

This document describes a practical activation flow to bring a client from registration to a state where merchant OnRamp/OffRamp operations can be executed.

## 1) Register client with full KYC data

### Endpoint

`POST {{URL}}/api/v2/kyc/merchant/client/register`

### Request body (actual structure used in current flow)

```json
{
  "email": "{{email}}",
  "firstNameRu": "{{firstNameRu}}",
  "lastNameRu": "{{lastNameRu}}",
  "patronymicRu": "{{patronymicRu}}",
  "firstName": "{{firstName}}",
  "lastName": "{{lastName}}",
  "residence": "112",
  "placeOfBirth": "{{placeOfBirth}}",
  "birthDate": "{{birthDate}}",
  "nationality": "112",
  "registrationCountry": "112",
  "registrationRegion": "{{registrationRegion}}",
  "registrationCity": "{{registrationCity}}",
  "registrationStreet": "{{registrationStreet}}",
  "registrationHouseAndFlat": "{{registrationHouseAndFlat}}",
  "identityDocType": "3",
  "identityDocIssueDate": "{{identityDocIssueDate}}",
  "identityDocExpireDate": "{{identityDocExpireDate}}",
  "identityDocNumber": "{{identityDocNumber}}",
  "personalNumber": "{{personalNumber}}",
  "postCode": "{{postCode}}",
  "phone": "{{phone}}",
  "gender": "{{gender}}",
  "residenceDistrict": "{{residenceDistrict}}",
  "identityDocIssuer": "{{identityDocIssuer}}",
  "exchangeInPersonalInterests": true,
  "notUSTaxPayer": true,
  "agreedWithOffer": true
}
```

### Notes

- Keep URL format with slash after host: `{{URL}}/api/...`.
- In current DTO (`RegisterClientRequest`) `residence` is `String`, not boolean.
- Save `id` from response as `clientId` for all next steps.

## 2) If residence/nationality is Belarus (`112`) -> complete crypto test

### 2.1 Get questions

`GET {{URL}}/api/v2/kyc/merchant/client/crypto-test?clientId={{clientId}}`

If response has `"cryptoTestRequired": true`, continue with submit.

### 2.2 Submit answers

`POST {{URL}}/api/v2/kyc/merchant/client/crypto-test`

```json
{
  "clientId": "{{clientId}}",
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

## 3) Poll client status until ready

Endpoint:

`POST {{URL}}/api/v2/kyc/merchant/client/status`

```json
{
  "clientId": "{{clientId}}"
}
```

Ready state for exchange operations:

- status is `VERIFIED`
- required crypto test is completed (when applicable)

## 4) Payment binding and operation context before first exchange

1. Get providers  
   `POST {{URL}}/api/v2/exchange/merchant/payment/provider`
2. Get payment methods/tokens  
   `POST {{URL}}/api/v2/exchange/merchant/payment/method`
3. If token is required and absent -> bind card  
   `POST {{URL}}/api/v2/exchange/merchant/payment/card/bind`

Then proceed to quote/order flow for OnRamp or OffRamp.

## 5) What is validated by WhiteBird during order creation

Core checks in exchange validation include:

- client status must be `VERIFIED`
- testing completion when required
- phone confirmation and phone presence checks
- active order/deposit/withdrawal restrictions
- payment provider/token validity when required
- limits, balances, route availability, address checks
