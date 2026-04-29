
---

## 6. Full Onboarding Flow Before OnRamp/OffRamp

This section describes the complete integration flow required before production exchange operations.

### 6.1 Registration with full profile data (KYC registration)

Use KYC merchant endpoints in this sequence:

1. `POST /api/v2/kyc/merchant/client/register`  
   Create WB client with full profile and document data.
2. `POST /api/v2/kyc/merchant/client/status`  
   Poll current client status after registration.
3. Optional helpers:
   - `POST /api/v2/kyc/merchant/client/personal-number`
   - `POST /api/v2/kyc/merchant/client/agreed-offer`

Alternative lightweight onboarding:

- `POST /api/v2/auth/merchant/client/register` (light registration)
- `POST /api/v2/auth/merchant/client/token/generate` (SDK token for user-context SDK flows)

Use light registration only when full KYC data will be collected later through SDK/user flow.

### 6.2 KYC, statuses, and when OnRamp/OffRamp become available

#### Status and KYC checks

- Integration checks status via `POST /api/v2/kyc/merchant/client/status`.
- Exchange execution validation requires client status `VERIFIED`.
- For residents where testing is required:
  - `GET /api/v2/kyc/merchant/client/crypto-test`
  - `POST /api/v2/kyc/merchant/client/crypto-test`
  - Until required test is completed, exchange validation can fail with invalid client status.

#### What happens after registration

- Partner backend creates/registers client in WhiteBird via API.
- If KYC process must continue via Sumsub from partner UX:
  - request temporary token via `POST /api/v2/kyc/merchant/client/sumsub/token`;
  - run Sumsub flow in partner UI/app using that token;
  - continue polling `.../client/status` until final state needed by business flow.

#### Responsibility split

- Partner side:
  - call registration/status/crypto-test/sumsub-token endpoints;
  - orchestrate user journey in own UI;
  - control when to proceed to payment binding and quote/order creation.
- WhiteBird side:
  - performs KYC status updates and verification checks;
  - enforces exchange-time validations (status, testing, phone, MFA, limits, balances, route availability).

### 6.3 Payment method binding and payment context for operations

Before creating buy/sell orders, resolve provider/token context:

1. `POST /api/v2/exchange/merchant/payment/provider`  
   Get providers available for current client/asset/direction.
2. `POST /api/v2/exchange/merchant/payment/method`  
   Get available payment methods/tokens.
3. If token is missing and provider requires payment method, bind card/payment method:
   - `POST /api/v2/exchange/merchant/payment/card/bind`
   - then re-check methods and use returned token.

Then execute exchange flow:

- OnRamp: `assets -> provider -> method -> limit/quote/order flow`
- OffRamp: `assets -> provider -> method -> limit/quote/order flow`

Operational prerequisites enforced during order validation include:

- client status must be `VERIFIED`;
- testing must be completed when required;
- phone confirmation/phone number checks;
- MFA requirement for deposit/withdrawal operations;
- valid payment method/provider token when required.
