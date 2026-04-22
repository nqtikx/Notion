# SDK Documentation

## Introduction
Welcome to the WhiteBird SDK documentation!

This SDK provides the ability to integrate **WhiteBird instant crypto exchange** functionality directly into your website (JS SDK).  
Authorization flow depends on the selected SDK mode.

The SDK supports interaction with WhiteBird in three modes:
- LoginMode
- AuthMode
- TokensMode

#### Presentation
- You can review the visual flow in [Figma](https://www.figma.com/design/QhTl1W0BEncjvGXRu03UiW/SDK-flow?node-id=0-1&p=f)
- Demo is available at: [SDK Demo](https://sdk.dev.wbdevel.net/v2.0/assets/sdk-demo/index.html)  
  (```MerchantId: 4f19017b-0793-4591-94ff-610bb3c4665b```)

Descriptions of each mode are available [below](#brief-description-of-modes) in this documentation.

## High-level process overview

| General WhiteBird SDK flow | Registration process |
|-|-|
| ![UserFlow](UserFlow.drawio.svg) | ![Registration](Registration.drawio.svg) |

| Verification process | Authorization process                      |
|-|--------------------------------------------|
| ![Verification](verification.drawio.svg) | ![Authorization](Authorization.drawio.svg) |

## Brief description of modes

Terminology:
- Client – the company integrating the SDK
- User – the end user of the product

### LoginMode
This mode is intended for user authentication *(authorization/registration/verification)* via WhiteBird. It allows users to log in with WhiteBird credentials, perform exchange operations, view their transaction history, and contact support. Suitable for scenarios where you need to add the ability to deposit/withdraw funds or exchange cryptocurrency to fiat and back.

### AuthMode
In this mode, the SDK is used for user authentication *(authorization/registration/verification)* via WhiteBird and provides user authorization tokens, which can be used for interaction with the WhiteBird API. In this case, the client is **not** a *user identification agent* for us, and implements their own custom UI for exchange, which makes operations through our API using the client’s authorization tokens.

### TokensMode
This mode is designed to run our SDK with access tokens. It includes all the capabilities of LoginMode but removes WhiteBird authorization.  
It implies a seamless transition from the client’s app to the WhiteBird platform and back. In this mode, the client is assumed to be a *user identification agent* for us.

## User identification agent

The main feature enabled by this is the use of SDK in TokensMode. The user will not need to log into the WhiteBird platform themselves; the client does this on their behalf, obtaining Auth tokens through **backend-to-backend** interaction over REST API, using the merchant’s API Key.

- User registration request: [register](../onboardingAPI/README.md#register-post-request)
- User token request: [generate](../onboardingAPI/README.md#generate-tokens-request)

## Adding to a website

### Connection via CDN
```html
<script src="https://sdk.dev.wbdevel.net/v2.0/integration/wbExchangeSdk-v001.js"></script>
```

### Container
- The SDK content is designed for mobile view – the container should not be less than 360px wide.
- The SDK container can be placed anywhere on the website.
- Pass the HTML element of this container in the configuration.
- !! The SDK does not modify the container’s styles, all container styling is handled by the application.
- The SDK iframe takes up all available space within the container.

### API Reference
Example initialization of SDK in ```LoginMode```:
```javascript
wbExchangeSdk.setup({
  // container for embedding SDK
  el: document.getElementById("wbExchangeSdkWrapper"),
  // client id
  merchantId: 'xxxx',
  mode: wbExchangeSdk.mode.LoginMode,
})
```

Example initialization of SDK in ```TokensMode```:
```javascript
wbExchangeSdk.setup({
  // container for embedding SDK
  el: document.getElementById("wbExchangeSdkWrapper"),
  // client id
  merchantId: 'xxxx',
  mode: wbExchangeSdk.mode.TokensMode,

  accessToken: '*****',
  refreshToken: '*****',
})
```

Example initialization of SDK in ```AuthMode```:
```javascript
wbExchangeSdk.setup({
  // container for embedding SDK
  el: document.getElementById("wbExchangeSdkWrapper"),
  // client id
  merchantId: 'xxxx',
  mode: wbExchangeSdk.mode.AuthMode,

  // callback returns user tokens for interaction with WhiteBird API
  // token payload depends on backend auth/verification state.
  onLogin: ({ email, accessToken, refreshToken, isUserVerified }) => {},
})
```

## Optional SDK configuration parameters

**externalClientId** - string, should be provided by the merchant to link WhiteBird users with merchant’s users.

**email** - string, allows pre-filling the email field to reduce user actions during WhiteBird login.

**merchantPass** - string, optional parameter used for SDK session/auth context (passed into SDK configuration).

**currencyAmount** - int, allows pre-filling the currency amount for the exchange.

**currencyFrom** - string from Currency enum. Allows pre-filling the currency to be exchanged from.

**disableCurrencyFrom** - bool, disable currencyFrom selector, also it blocks amount field (if it was provided).

**currencyTo** - string from Currency enum. Allows pre-filling the currency to be exchanged to.

**disableCurrencyTo** - bool, disable currencyTo field, also it blocks cryptoWallet field (if it was provided).

**cryptoWallet** - string, allows pre-filling the user’s wallet field.

**showBackButtonOnHomePage** - bool, shows a "back" button in our UI and reacts to it using the onExit callback.

**onExit** - () -> void, callback to handle "back" button press.

**onOrderCreated** - ({orderId, internalCryptoAddress}) -> void, callback for order creation notification for merchant.

**currencyToAmount** - int, allows pre-filling the target amount for exchange.

**disableAmount** - bool, disables amount input editing in SDK UI.

**isAuthAgent** - bool, indicates partner identification-agent mode behavior.

**redirectUrl** - string, URL used by SDK for redirect/navigation after specific flow actions.

**startAppPage** - string, allows starting SDK from a specific internal page/state.

**disableAddCard** - bool, disables add-card flow in SDK UI.

**onOrderCompleted** - ({orderId, status}) -> void, callback for order completion notification for merchant.

**refId** - string, optional external reference identifier passed through SDK context.

**debug** - bool, enables SDK debug logging in browser console.

**isTgBot** - bool, enables Telegram WebApp-specific behavior for link opening.

It is recommended to call ```wbExchangeSdk.cleanup()``` after finishing work with the SDK.

```typescript
let currencyFrom: Currency;
let currencyTo: Currency;

enum Currency {
    BYN,
    RUB,
    USD,
    EUR,
    BTC,
    ETH,
    USDT, // ERC-20
    USDC, // ERC-20
    TRX,
    USDT_TRC, // TRC-20
    TON,
    USDT_TON, // TON
    WBP, // TRC-20
}
```
## General WhiteBird SDK flow

### One-line flow
`Initialize SDK -> identify user context -> enforce compliance path -> unlock exchange/wallet actions`

### Scenarios
- **LoginMode:** user authenticates inside SDK, then SDK routes by verification status.
- **AuthMode:** user authenticates inside SDK, SDK returns auth payload via `onLogin(...)`, then status-based access applies.
- **TokensMode:** SDK starts with merchant-issued tokens, skips login screen, then applies same status-based routing.
- **Verified user path:** direct access to exchange/wallet operations.
- **Restricted user path:** verification/pending/crypto-test gates are applied before operations.

### Status routing table

| User status condition | SDK behavior | Next step |
|---|---|---|
| `NOT_VERIFIED` | Operations are blocked | Verification agreements -> SumSub |
| `ON_VERIFICATION` | Limited access | Wait AML decision |
| `VERIFIED` + `testingNeeded=true` + `testingCompleted=false` | Crypto test required | Complete crypto test |
| `VERIFIED` and no pending test | Full access | Exchange / wallet actions |

---

## Registration process

### One-line flow
`Sign up input -> agreements -> email confirmation -> phone verification rule -> registration -> auto-login`

### Scenarios
- **Standard signup:** user enters email/phone/password, accepts agreements, confirms email, then registration is finalized.
- **BY country flow:** after email confirmation, SMS phone verification is required before registration completion.
- **Non-BY flow:** registration continues after email confirmation without mandatory SMS confirmation step.
- **Successful completion:** user is automatically logged in and receives session tokens.

### Registration routing table

| Condition | Behavior | Result |
|---|---|---|
| Email code not confirmed | Registration does not continue | Stay in email confirmation step |
| `countryCode == BY` | Phone verification is required | Proceed after SMS code confirmation |
| `countryCode != BY` | Phone verification is not mandatory gate | Proceed to registration |
| Registration + login success | Tokens issued | User enters authorized flow |

---

## Verification process

### One-line flow
`Legal agreements -> SumSub KYC -> AML review -> verified status -> optional crypto test gate`

### Scenarios
- **Verification start:** user accepts required legal confirmations (`notUSTaxPayer`, `exchangeInPersonalInterests`, `agreedWithOffer`).
- **KYC execution:** SDK runs SumSub documents + liveness steps.
- **AML pending:** user is moved to `ON_VERIFICATION` and waits review outcome.
- **Approval path:** user becomes `VERIFIED`.
- **Regulatory add-on:** if crypto test is required, user must pass it before full access.

### Verification routing table

| Condition | Behavior | Result |
|---|---|---|
| Agreements not accepted | Verification cannot proceed | Stay on agreements step |
| SumSub in progress | KYC flow remains active | Continue KYC steps |
| AML pending | User is gated | `ON_VERIFICATION` waiting state |
| AML approved | Verification completed | `VERIFIED` |
| `testingNeeded=true` and not completed | Extra compliance gate | Crypto test required |

---

## Authorization process

### One-line flow
`Credentials input -> challenge checks (MFA/Captcha) -> token issuance -> status-based access`

### Scenarios
- **Primary sign-in:** user enters email/password.
- **MFA path:** if enabled, user confirms login with 2FA code.
- **Captcha path:** when required, user solves captcha before token issuance.
- **Auth success:** access/refresh tokens are issued.
- **Post-auth control:** access to operations still depends on verification and compliance status.

### Authorization routing table

| Condition | Behavior | Result |
|---|---|---|
| Invalid credentials | Auth fails | Stay on sign-in |
| MFA enabled | Request 2FA confirmation | Continue after valid code |
| Captcha required | Request captcha solve | Continue after valid captcha |
| Auth successful | Tokens issued | Authorized session |
| User not compliant yet | Access gated by status | Redirect to verification/pending/crypto-test path |
