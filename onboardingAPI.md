# WhiteBird Registration API Guide

## Purpose

This guide documents registration and related KYC endpoints used by merchant integrations.

Source of truth in code:
- `kyc-service/src/main/java/io/wb/kyc/core/crm/client/ClientManagementController.java`
- `kyc-service/src/main/java/io/wb/kyc/core/crm/client/ClientValidationController.java`
- `kyc-service/src/main/java/io/wb/kyc/core/crm/client/dto/RegisterClientRequest.java`
- `kyc-service/src/main/java/io/wb/kyc/core/crm/client/ClientManagementService.java`

All merchant endpoints below are backend-to-backend and require `x-api-key`.

## Table of Contents

- [1. Registration endpoint](#1-registration-endpoint)
- [2. Field usage by flow](#2-field-usage-by-flow)
- [3. Client status endpoint](#3-client-status-endpoint)
- [4. Crypto test endpoints](#4-crypto-test-endpoints)
- [5. Additional useful endpoints](#5-additional-useful-endpoints)
- [6. User-side validation endpoints](#6-user-side-validation-endpoints)
- [7. Minimal registration fields](#7-minimal-registration-fields)
- [8. Data types](#8-data-types)

## 1. Registration endpoint

### POST `/api/v2/kyc/merchant/client/register`

Creates or registers a client in registration/KYC flow.

### Tech-required fields (DTO `@NotEmpty`)

- `email`
- `phone`
- `firstNameRu`
- `lastNameRu`
- `patronymicRu`
- `firstName`
- `lastName`
- `residence`
- `placeOfBirth`
- `registrationCountry`
- `registrationRegion`
- `residenceDistrict`
- `registrationCity`
- `registrationStreet`
- `registrationHouseAndFlat`
- `identityDocType`
- `identityDocNumber`
- `identityDocIssuer`
- `postCode`
- `gender`
- `nationality`

### Optional fields (DTO level)

- `birthDate`
- `identityDocIssueDate`
- `identityDocExpireDate`
- `personalNumber`
- `notUSTaxPayer`
- `agreedWithOffer`
- `exchangeInPersonalInterests`
- `files`
- `externalClientId`
- `isPotentialDrop`

### Business-required conditions

- `personalNumber` is required when `registrationCountry` contains `112` (Belarus).
  - otherwise service throws: `Personal number must not be empty for registration country BLR`.
- `notUSTaxPayer`, `agreedWithOffer`, `exchangeInPersonalInterests` are not hard-required for DTO validation, but they affect follow-up CRM processing.

### Response

- `id` - registered `clientId`
- `status` - current client status

## 2. Field usage by flow

| Field | Register | Crypto-test POST | Agreed-offer POST | Status POST | Validate POST |
|---|---|---|---|---|---|
| `postCode` | Required (`@NotEmpty`) | Not used | Not used | Not used | Not used |
| `notUSTaxPayer` | Optional bool | Optional bool | Optional bool | Not used | Not used |
| `agreedWithOffer` | Optional bool | Optional bool | Optional bool | Not used | Not used |

Notes:
- `postCode` must be non-empty for register request validation.
- `notUSTaxPayer` and `agreedWithOffer` are accepted in multiple flows and persist when sent as `true`.

## 3. Client status endpoint

### POST `/api/v2/kyc/merchant/client/status`

Returns current client status.

Request:
- `clientId` (request body)
- `externalUserId` (optional request param)

Response:
- `String` (`ClientStatus`)

## 4. Crypto test endpoints

Crypto test is required for residents (BY rule path).  
Flow: GET questions -> POST answers.

### GET `/api/v2/kyc/merchant/client/crypto-test?clientId=...`

Request headers:
- `x-api-key`
- optional `externalClientId`

Response:
- `cryptoTestRequired`: bool
- `questions`: `TestQuestion[]` (present only when required)

If client is not resident:

```json
{
  "cryptoTestRequired": false
}
```

### POST `/api/v2/kyc/merchant/client/crypto-test`

Request body:
- `clientId`
- `exchangeInPersonalInterests`
- `agreedWithOffer`
- `notUSTaxPayer`
- `answers` (`questionId -> answerId`)

Response:

```json
{
  "accepted": true
}
```

Behavior:
- for non-resident client: returns `{ "accepted": false }`.
- for resident client: answer validation is strict; wrong set throws `Wrong answers to crypto test`.
- `notUSTaxPayer`, `agreedWithOffer`, `exchangeInPersonalInterests` persist when sent as `true`.

## 5. Additional useful endpoints

### Merchant endpoints (`x-api-key`)

1) **Get Sumsub SDK token**
- `POST /api/v2/kyc/merchant/client/sumsub/token`
- Request: `clientId`, `levelType`
- Response: `token`, `validTill`

2) **Update offer agreement flags**
- `POST /api/v2/kyc/merchant/client/agreed-offer`
- Request: `clientId`, `exchangeInPersonalInterests`, `agreedWithOffer`, `notUSTaxPayer`
- Response: `"OK"`

3) **Get personal number**
- `POST /api/v2/kyc/merchant/client/personal-number`
- Request: `clientId`
- Response: `personalNumber`

### Auth side (used together with registration in many integrations)

1) `POST /api/v2/auth/merchant/client/register` (light register)  
2) `POST /api/v2/auth/merchant/client/token/generate`

## 6. User-side validation endpoints

From `ClientValidationController` (`Bearer token`, user context):

- `POST /api/v1/kyc/client/validate`
- `POST /api/v1/kyc/client/confirm`
- `POST /api/v1/kyc/client/ico/agreement`
- `POST /api/v1/kyc/client/ico/agreements`

These endpoints do not consume `postCode`, `notUSTaxPayer`, or `agreedWithOffer`.

## 7. Minimal registration fields

### Minimum technically valid payload

Minimum for `POST /api/v2/kyc/merchant/client/register` is all DTO `@NotEmpty` fields listed in section 1.

### Additional minimum for BY

If `registrationCountry = "112"`, add:
- `personalNumber` (non-empty)

### Minimal example (non-BY)

```json
{
  "email": "user@example.com",
  "phone": "+79990000000",
  "gender": "жен",
  "firstNameRu": "Анна",
  "lastNameRu": "Иванова",
  "patronymicRu": "Ивановна",
  "firstName": "Anna",
  "lastName": "Ivanova",
  "placeOfBirth": "Moscow",
  "nationality": "643",
  "residence": "643",
  "registrationCountry": "643",
  "registrationRegion": "Moscow region",
  "residenceDistrict": "-",
  "registrationCity": "Moscow",
  "registrationStreet": "Lenina",
  "registrationHouseAndFlat": "1-1",
  "identityDocType": "9",
  "identityDocNumber": "1234567890",
  "identityDocIssuer": "MVD",
  "postCode": "101000"
}
```

### What else is required

- header `x-api-key` (merchant key)
- merchant permission for `KYC_REGISTER_EP`
- for external linkage flows, pass `externalClientId` where needed

## 8. Data types

```typescript
enum Gender {
  Men = "муж",
  Woman = "жен"
}

enum ClientStatus {
  CREATED = "CREATED",
  PENDING = "PENDING",
  VERIFIED = "VERIFIED",
  FROZEN = "FROZEN",
  ARREST = "ARREST"
}

interface TestQuestion {
  id: string;
  title: string;
  answers: TestAnswer[];
}

interface TestAnswer {
  id: number;
  title: string;
  correct: boolean;
}
```
