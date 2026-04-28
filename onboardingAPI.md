# WhiteBird Registration API Guide

## Purpose

This guide describes Backend-to-Backend registration APIs for partners that act as an identification agent for WhiteBird users.

If the partner already has the required personal identity document (PID) data, WhiteBird can receive this data from the partner instead of collecting it directly from the user.

All requests are authorized with the `x-api-key` header.

## API flows

| Flow | Endpoint sequence | Use case |
|---|---|---|
| Full KYC registration | `POST /api/v2/kyc/merchant/client/register` | Partner sends complete user KYC/PID data to WhiteBird |
| Client status check | `POST /api/v2/kyc/merchant/client/status` | Partner checks whether the client is allowed to transact |
| Crypto test for BY residents | `GET /api/v2/kyc/merchant/client/crypto-test` -> `POST /api/v2/kyc/merchant/client/crypto-test` | Endpoint returns test questions for residents that require crypto test |
| SDK light registration | `POST /api/v2/auth/merchant/client/register` | Partner creates a client without full KYC data |
| SDK token generation | `POST /api/v2/auth/merchant/client/token/generate` | Partner receives SDK access tokens for a registered client |

## Common headers

| Header | Required | Description |
|---|---|---|
| `x-api-key` | Yes | Merchant API key used for Backend-to-Backend authorization |

> `externalClientId` is not a common header for all endpoints. Depending on the endpoint, it can be passed in the request body, as a header, or as a query parameter. See each endpoint section for details.

## 1. Full KYC registration

### `POST /api/v2/kyc/merchant/client/register`

Creates a WhiteBird client from a complete KYC/PID data set.

### Request fields

#### Contact and partner identifiers

| Field | Type | Required | Description |
|---|---|---|---|
| `email` | `string` | Yes | User email. Must not be empty |
| `phone` | `string` | Yes | User phone. Must not be empty |
| `externalClientId` | `string` | No | User identifier in partner system |

#### Personal data

| Field | Type | Required | Description |
|---|---|---|---|
| `gender` | `Gender` | Yes | User gender. See [Data dictionaries](#data-dictionaries) |
| `firstNameRu` | `string` | Yes | First name in Russian/Cyrillic form |
| `lastNameRu` | `string` | Yes | Last name in Russian/Cyrillic form |
| `patronymicRu` | `string` | Yes | Patronymic/middle name in Russian/Cyrillic form. Must not be empty |
| `firstName` | `string` | Yes | First name in Latin form. Must not be empty |
| `lastName` | `string` | Yes | Last name in Latin form. Must not be empty |
| `placeOfBirth` | `string` | Yes | Place of birth |
| `birthDate` | `string` | No | Date of birth in `YYYY-MM-DD` format |
| `nationality` | `CountryCode` | Yes | Nationality/citizenship code |
| `residence` | `CountryCode` | Yes | Country of residence / country related to issued PID |

#### Identity document

| Field | Type | Required | Description |
|---|---|---|---|
| `identityDocType` | `DocType` | Yes | Identity document type |
| `identityDocIssueDate` | `string` | No | Issue date in `YYYY-MM-DD` format |
| `identityDocExpireDate` | `string` | No | Expiry date in `YYYY-MM-DD` format |
| `identityDocNumber` | `string` | Yes | Identity document number / series and number |
| `identityDocIssuer` | `string` | Yes | Authority that issued the document |
| `personalNumber` | `string` | Conditional | Required when `registrationCountry` contains `112` (Belarus). Optional for other countries |

#### Registration address

Registration address format differs by country. Fill in values that correspond to the registration country format.

| Field | Type | Required | Description |
|---|---|---|---|
| `registrationCountry` | `CountryCode` | Yes | Registration country |
| `registrationRegion` | `string` | Yes | Registration region/state |
| `residenceDistrict` | `string` | Yes | Registration district. Mapped internally to `registrationDistrict` |
| `registrationCity` | `string` | Yes | City/locality/settlement |
| `registrationStreet` | `string` | Yes | Street |
| `registrationHouseAndFlat` | `string` | Yes | House, building and apartment |
| `postCode` | `string` | Yes | Postal code. Must not be empty |

#### Consents and flags

| Field | Type | Required | Description |
|---|---|---|---|
| `notUSTaxPayer` | `boolean` | No | Confirms that user is not a U.S. taxpayer |
| `agreedWithOffer` | `boolean` | No | Confirms user agreement with WhiteBird offer |
| `exchangeInPersonalInterests` | `boolean` | No | Confirms exchange is performed in user personal interests |
| `files` | `string[]` | No | Optional file references saved with KYC client data |
| `isPotentialDrop` | `boolean` | No | Optional risk flag passed to CRM processing |

### Request example (Belarus user)

```json
{
  "email": "test.user.testov+15112024@ya.ru",
  "phone": "+375297778899",
  "gender": "муж",
  "firstNameRu": "Джон",
  "lastNameRu": "До",
  "patronymicRu": "Иванович",
  "firstName": "John",
  "lastName": "Doe",
  "placeOfBirth": "Republic of Belarus, city Minsk",
  "birthDate": "1994-01-05",
  "nationality": "112",
  "residence": "112",
  "identityDocType": "3",
  "identityDocIssueDate": "2020-01-02",
  "identityDocExpireDate": "2030-01-02",
  "identityDocNumber": "HB2129425",
  "identityDocIssuer": "Central ROVD of Minsk",
  "personalNumber": "3029120H059PB9",
  "registrationCountry": "112",
  "registrationRegion": "Minsk region",
  "residenceDistrict": "-",
  "registrationCity": "Minsk",
  "registrationStreet": "Kriptomanov street",
  "registrationHouseAndFlat": "30/1-3",
  "postCode": "220000",
  "notUSTaxPayer": true,
  "agreedWithOffer": true,
  "exchangeInPersonalInterests": true,
  "externalClientId": "partner-client-123"
}
```

### Request example (foreign passport)

```json
{
  "email": "test.user.testov+15112024@ya.ru",
  "phone": "-",
  "gender": "жен",
  "firstNameRu": "Джон",
  "lastNameRu": "До",
  "patronymicRu": "Иванович",
  "firstName": "-",
  "lastName": "-",
  "placeOfBirth": "Russian Federation, Jewish Autonomous Region, Birobidzhan",
  "birthDate": "1994-05-09",
  "nationality": "643",
  "residence": "643",
  "identityDocType": "9",
  "identityDocIssueDate": "2020-01-02",
  "identityDocExpireDate": "2030-01-02",
  "identityDocNumber": "9992129425",
  "identityDocIssuer": "MIA of Russia, Jewish Autonomous Region",
  "registrationCountry": "643",
  "registrationRegion": "Jewish Autonomous Region",
  "residenceDistrict": "Smidovich district",
  "registrationCity": "Nikolaevka settlement",
  "registrationStreet": "Komsomolskaya street",
  "registrationHouseAndFlat": "23-30",
  "postCode": "-",
  "notUSTaxPayer": true,
  "agreedWithOffer": true,
  "exchangeInPersonalInterests": true
}
```

### Response

| Field | Type | Description |
|---|---|---|
| `id` | `string` | Registered WhiteBird client id |
| `status` | `KycClientStatus` | Current KYC client status |

```json
{
  "id": "0d58e7ec-0369-48d7-9804-90c6b23a52be",
  "status": "PENDING"
}
```

### Notes

- Java validation requires all `@NotEmpty` fields from the tables above.
- Date strings are parsed as ISO local dates (`YYYY-MM-DD`).
- `personalNumber` is additionally validated for Belarus registration country (`112`).
- If all consent flags are `true`, WhiteBird schedules CRM user update after registration.

## 2. Client status

### `POST /api/v2/kyc/merchant/client/status`

Returns current client status.

### Request fields

| Field | Type | Required | Description |
|---|---|---|---|
| `clientId` | `string` | Yes | WhiteBird client id returned by registration |

### Optional query parameters

| Field | Type | Required | Description |
|---|---|---|---|
| `externalUserId` | `string` | No | Partner-side client id for additional client lookup |

### Request example

```json
{
  "clientId": "0d58e7ec-0369-48d7-9804-90c6b23a52be"
}
```

### Response example

```json
"VERIFIED"
```

## 3. Crypto test

For non-resident clients this flow returns `cryptoTestRequired=false` and does not update test answers.

### `GET /api/v2/kyc/merchant/client/crypto-test?clientId={clientId}`

Returns whether the crypto test is required and, if required, returns the questions.

### Query parameters

| Field | Type | Required | Description |
|---|---|---|---|
| `clientId` | `string` | Yes | WhiteBird client id |

### Optional headers

| Header | Required | Description |
|---|---|---|
| `externalClientId` | No | Partner-side client id for additional client lookup |

### Response example: test is required

```json
{
  "cryptoTestRequired": true,
  "questions": [
    {
      "id": "1",
      "title": "Question text",
      "answers": [
        {
          "id": 10,
          "title": "Answer text",
          "correct": true
        }
      ]
    }
  ]
}
```

### Response example: test is not required

```json
{
  "cryptoTestRequired": false
}
```

### `POST /api/v2/kyc/merchant/client/crypto-test`

Submits answers to the crypto test.

### Request fields

| Field | Type | Required | Description |
|---|---|---|---|
| `clientId` | `string` | Yes | WhiteBird client id |
| `answers` | `object` | Yes | Map of question id to answer id. Example: `{ "1": 10 }` |
| `notUSTaxPayer` | `boolean` | No | Can be set together with test submission |
| `agreedWithOffer` | `boolean` | No | Can be set together with test submission |
| `exchangeInPersonalInterests` | `boolean` | No | Can be set together with test submission |

### Optional headers

| Header | Required | Description |
|---|---|---|
| `externalClientId` | No | Partner-side client id for additional client lookup |

### Request example

```json
{
  "clientId": "0d58e7ec-0369-48d7-9804-90c6b23a52be",
  "answers": {
    "1": 10,
    "2": 20,
    "3": 30,
    "4": 40,
    "5": 50
  },
  "notUSTaxPayer": true,
  "agreedWithOffer": true,
  "exchangeInPersonalInterests": true
}
```

### Response example

```json
{
  "accepted": true
}
```

### Notes

- If the client is not a Belarus resident, response is `{ "accepted": false }` and no test update is needed.
- Wrong answers cause validation error: `Wrong answers to crypto test`.

## 4. SDK light registration

### `POST /api/v2/auth/merchant/client/register`

Registers a merchant client without sending full KYC data.

### Request fields

| Field | Type | Required | Description |
|---|---|---|---|
| `email` | `string` | Yes | User email. Must not be empty |
| `phone` | `string` | Yes | User phone. Must not be empty |
| `externalClientId` | `string` | No | User identifier in partner system |
| `agreedWithOffer` | `boolean` | No | Whether user agreed with WhiteBird offer during registration |
| `merchantId` | `string` | No | Used by OTP registration flow; not required for B2B `x-api-key` flow |

### Request example

```json
{
  "email": "client@example.com",
  "phone": "+375297778899",
  "externalClientId": "partner-client-123",
  "agreedWithOffer": true
}
```

### Response example

```json
{
  "id": "0d58e7ec-0369-48d7-9804-90c6b23a52be",
  "status": "NOT_VERIFIED"
}
```

## 5. SDK token generation

### `POST /api/v2/auth/merchant/client/token/generate`

Generates access tokens for merchant client.

### Request fields

| Field | Type | Required | Description |
|---|---|---|---|
| `clientId` | `string` | Yes | WhiteBird client id |
| `externalClientId` | `string` | No | User identifier in partner system |

### Request example

```json
{
  "clientId": "0d58e7ec-0369-48d7-9804-90c6b23a52be",
  "externalClientId": "partner-client-123"
}
```

### Response fields

| Field | Type | Description |
|---|---|---|
| `token` | `string` | Access token |
| `refreshToken` | `string` | Refresh token, can be `null` |

### Response example

```json
{
  "token": "access-token",
  "refreshToken": "refresh-token"
}
```

## 6. Optional KYC support endpoints

### `POST /api/v2/kyc/merchant/client/agreed-offer`

Updates consent flags for an already registered client.

Optional header: `externalClientId`.

| Field | Type | Required | Description |
|---|---|---|---|
| `clientId` | `string` | Yes | WhiteBird client id |
| `notUSTaxPayer` | `boolean` | No | Confirms user is not a U.S. taxpayer |
| `agreedWithOffer` | `boolean` | No | Confirms user agreement with WhiteBird offer |
| `exchangeInPersonalInterests` | `boolean` | No | Confirms exchange is performed in user personal interests |

Response:

```json
"OK"
```

### `POST /api/v2/kyc/merchant/client/personal-number`

Returns stored personal number for a registered client.

Optional header: `externalClientId`.

| Field | Type | Required | Description |
|---|---|---|---|
| `clientId` | `string` | Yes | WhiteBird client id |

Response:

```json
{
  "personalNumber": "3029120H059PB9"
}
```

## Data dictionaries

```typescript
type Gender = string // Examples used in existing integrations: "муж", "жен"

type CountryCode = string // Numeric ISO 3166-1 country code, e.g. "112" for Belarus, "643" for Russia

type DocType = string // Examples used in existing integrations: "3" (BY passport), "9" (foreign passport)

enum KycClientStatus {
  NOT_VERIFIED = "NOT_VERIFIED",
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
  id: number; // serialized from Java Long
  title: string;
  correct: boolean;
}
```

## Common validation errors

### Missing required field

If a field marked as required is missing or empty, the request fails Java bean validation.

### Missing Belarus personal number

For Belarus registration country (`112`), `personalNumber` is required.

Error message: `Personal number must not be empty for registration country BLR`

### Wrong crypto test answers

Error message: `Wrong answers to crypto test`
