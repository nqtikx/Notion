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

## 3) KYC progression via Sumsub/CRM events

### 3.1 Sumsub webhook events (local test sequence)

Endpoint:

`POST http://localhost:8088/api/v1/kyc/sumsub/event`

Headers:

- `Content-Type: application/json`
- `x-payload-digest: ...` (required outside local profile)

Send in this order:

1. `applicantCreated`
2. `applicantPending`
3. `applicantReviewed`

Bodies:

```json
{
  "type": "applicantCreated",
  "applicantId": "",
  "externalUserId": "",
  "inspectionId": "test-inspection",
  "correlationId": "test-correlation"
}
```

```json
{
  "type": "applicantPending",
  "applicantId": "",
  "externalUserId": "",
  "inspectionId": "test-inspection",
  "correlationId": "test-correlation"
}
```

```json
{
  "type": "applicantReviewed",
  "applicantId": "",
  "externalUserId": "",
  "inspectionId": "test-inspection",
  "correlationId": "test-correlation",
  "reviewResult": {}
}
```

Response from Sumsub webhook handler:

```json
{
  "status": "OK"
}
```

### 3.2 CRM sync step

Your current test flow uses:

`POST http://localhost:8088/api/v1/crm/sync`

```json
[
  {
    "status": "Verified",
    "residence": "false",
    "group": "false",
    "identity": ""
  }
]
```

Important code note:

- In current `kyc-service` source, direct REST controller for `/api/v1/crm/sync` was not found.
- CRM sync processing is implemented via `CrmSyncProcessor` listener (`UPDATE_PROFILE_STATUS_QUEUE_NAME`).
- Keep this HTTP step only if your local environment exposes it via another module/gateway/mock.

## 4) Poll client status until ready

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

## 5) Payment binding and operation context before first exchange

1. Get providers  
   `POST {{URL}}/api/v2/exchange/merchant/payment/provider`
2. Get payment methods/tokens  
   `POST {{URL}}/api/v2/exchange/merchant/payment/method`
3. If token is required and absent -> bind card  
   `POST {{URL}}/api/v2/exchange/merchant/payment/card/bind`

Then proceed to quote/order flow for OnRamp or OffRamp.

## 6) What is validated by WhiteBird during order creation

Core checks in exchange validation include:

- client status must be `VERIFIED`
- testing completion when required
- phone confirmation and phone presence checks
- active order/deposit/withdrawal restrictions
- payment provider/token validity when required
- limits, balances, route availability, address checks

Note on MFA:

- MFA check is enforced in v3 order flow for deposit/withdrawal scenarios.
- In legacy v2 buy/sell flow, MFA is not universally enforced for every case.
