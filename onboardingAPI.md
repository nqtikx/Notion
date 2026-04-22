# WhiteBird Registration API Guide

## Purpose

For integration with partners who have a large user base with a full set of personal identity document (PID) information.

##### Description
To carry out transactions on our platform, we need to collect PID information from users. If a partner already has the required user data, we can delegate the data collection task from users to the partner. This interaction will be formalized by an agreement, making the partner our “identification agent.”

In this case, the user registration request in our system serves as the protocol for receiving user data from the "identification agent."

Request authorization is performed through the **x-api-key** header. All requests are Backend-to-Backend.

### Sections:
- **[Registration](#register-post-request)**
- **[Client status](#client-status)**
- **[Crypto test (for BY users)](#crypto-test-requests)**
- **[Token generation (for SDK)](#generate-tokens-request)**

### Register POST request

Protocol for receiving a complete set of user data and creating an account.

If the identity document does not contain registration address information, this data should be obtained from other documents confirming the user’s residence (utility bills, bank statements, etc.). It is also acceptable to obtain this information verbally from the user.

#### POST /api/v2/kyc/merchant/client/register

#### Params:
- **email** - string(255), required
- **phone** - string(100/255), required
- **gender** - Gender, required
- **firstNameRu** - string(255), First name in Russian (if present in PID)
- **lastNameRu** - string(255), Last name in Russian (if present in PID)
- **patronymicRu** - string(255), Middle name/Patronymic in Russian (if present in PID)
- **firstName** - string(255), First name in Latin characters (if present in PID)
- **lastName** - string(255), Last name in Latin characters (if present in PID)

- **placeOfBirth** - string(unlimited), Place of birth, required
- **birthDate** - "YYYY-MM-DD", Date of birth
- **nationality** - CountryCode, Nationality by birth, required

- **residence** - CountryCode, Country of issued PID, required
- **identityDocType** - DocType, Type of PID from the list, required
- **identityDocIssueDate** - "YYYY-MM-DD", Issue date of PID
- **identityDocExpireDate** - "YYYY-MM-DD", Expiry date of PID
- **identityDocNumber** - string(50), Series and number of PID, required
- **identityDocIssuer** - string(unlimited), Authority that issued the PID, required
- **personalNumber** - string(50), Identification number from PID

> Registration address data may differ depending on the country. Fill in the fields that correspond to the address format in the user’s registration country.

- **registrationCountry** - CountryCode, Registration country, required
- **registrationRegion** - string(unlimited), Registration region, required
- **residenceDistrict** - string(unlimited), Registration district, required
- **registrationCity** - string(unlimited), Registration city (must include settlement name), required
- **registrationStreet** - string(unlimited), Registration street, required
- **registrationHouseAndFlat** - string(100), House, building, and apartment number, required
- **postCode** - string(20), Postal code, required

- **notUSTaxPayer** - bool, Not a U.S. taxpayer
- **agreedWithOffer** - bool, User’s consent to WhiteBird offer
- **exchangeInPersonalInterests** - bool, Confirmation that exchange is for personal interests
- **externalClientId** - string, Optional - client identification number of the user in partner's system

> Core KYC fields are required. Some fields are country-specific and validated according to backend rules.

#### Response:
- **id** - string(255), registered clientId
- **status** - string, current client status

Request examples:

``` json
// BY user example
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
   "exchangeInPersonalInterests": true
}

// RU user example
{
   "email":"test.user.testov+15112024@ya.ru",
   "phone": "-",
   "gender":"жен",
   "firstNameRu":"Джон",
   "lastNameRu":"До",
   "patronymicRu":"Иванович",
   "firstName": "-",
   "lastName": "-",
   "placeOfBirth":"Russian Federation, Jewish Autonomous Region, Birobidzhan",
   "birthDate":"1994-05-09",
   "nationality":"643",
   "residence":"643",
   "identityDocType":"9",
   "identityDocIssueDate":"2020-01-02",
   "identityDocExpireDate":"2030-01-02",
   "identityDocNumber":"9992129425",
   "identityDocIssuer":"MIA of Russia, Jewish Autonomous Region",
   "registrationCountry":"643",
   "registrationRegion":"Jewish Autonomous Region",
   "residenceDistrict":"Smidovich district",
   "registrationCity":"Nikolaevka settlement",
   "registrationStreet":"Komsomolskaya street",
   "registrationHouseAndFlat":"23-30",
   "postCode": "-",
   "notUSTaxPayer": true,
   "agreedWithOffer": true,
   "exchangeInPersonalInterests": true
}
```

### Client status

Request to get the client’s current status. The only valid status for transactions is VERIFIED.

#### POST /api/v2/kyc/merchant/client/status

#### Params:
- **clientId** - string(255)
#### Response:
- **String** - ClientStatus

### Crypto test requests

All users registered with Belarusian documents are required to pass the crypto test. This is a regulator requirement and cannot be bypassed. When using SDK, the test is built-in.

The test consists of 5 simple questions. Use GET crypto-test to retrieve them, then send the user’s answers in POST crypto-test.

#### GET /api/v2/kyc/merchant/client/crypto-test?clientId=

#### Response:
- **cryptoTestRequired** - bool, whether the user must pass the test
- **questions** - TestQuestion[], questions with answer options

#### POST /api/v2/kyc/merchant/client/crypto-test

#### Request:
- **clientId** - string(255)
- **exchangeInPersonalInterests** - bool
- **agreedWithOffer** - bool
- **notUSTaxPayer** - bool
- **answers** - object, “questionId”(long): answerId(long)

### Generate tokens request

Request to obtain client tokens for SDK use.  
Not used in On/Off ramp API.

#### POST /api/v2/auth/merchant/client/token/generate

#### Params:
- **clientId** - string(255)
- **externalClientId** - string, Optional - client identification number of the user in partner's system

#### Response:
- **token** - string(unlimited)
- **refreshToken** - string(unlimited)

> For SDK initialization, map `token` from this response to `accessToken` in `wbExchangeSdk.setup(...)`.

### Simple register request

Request to obtain clientId for token/generate use but without KYC data.

#### POST /api/v2/auth/merchant/client/register

#### Params:
- **email** - string(255), required
- **phone** - string(100/255), required
- **merchantId** - string(255), Optional - x-api-key
- **externalClientId** - string, Optional - client identification number of the user in partner's system
- **agreedWithOffer** - bool

#### Response:
- **id** - string(255), registered clientId
- **status** - string, current client status

#### Data types

```typescript
enum Gender {
    Men = "муж",    // male
    Woman = "жен"   // female
}

CountryCode = string // ISO country code

enum DocType {
    PassportBY = "3",           // Belarusian passport
    ResidencePermitBY = "6",    // Belarusian residence permit
    RefugeeCertificateBY = "7", // Belarusian refugee certificate
    ForeignPassport = "9",      // Foreign passport
    IDCardBY = "15",            // Belarusian ID card
    ForeignBiometricResidencePermitBY = "16", // Biometric residence permit for foreign citizens in Belarus
    RefugeeBiometricResidencePermitBY = "17", // Biometric residence permit for stateless persons in Belarus
    Other = "99"                // Other
}

enum ClientStatus {
    CREATED = "CREATED",    // AML documents not provided
    PENDING = "PENDING",    // Awaiting AML check
    VERIFIED = "VERIFIED",  // Allowed to make transactions
    FROZEN = "FROZEN",      // Final status (duplicate account)
    ARREST = "ARREST"       // Final status (dirty crypto used)
}

interface TestQuestion {
    id: string;
    title: string;
    answers: TestAnswer[]
}

interface TestAnswer {
    id: number;
    title: string;
    correct: boolean;
}
```
