# WhiteBird Card API Guide

## Purpose
Requests for partners to retrieve the list of a client's available payment methods, as well as the ability to add new payment methods.

### Description
To perform operations, a partner can act as a [paymentProvider](../paymentProviderAPI). This is a deep integration of payment methods in order to avoid PSP fees.

A partner can also use the existing payment systems that are already integrated with WhiteBird. In this case, the user adds payment cards in the partner's application via the API described below.

Requests are authorized via the **x-api-key** header. All requests are Backend-to-Backend.

### Sections:
- **[Get list of available providers](#providers-request)**
- **[Add a card](#card-bind-request)**
- **[Get list of cards](#payment-methods-request)**

### Providers request

> List of available payment methods with their characteristics

[Documentation](../exchangeAPI#providers-request)

---

### Card bind request

#### POST /api/v2/exchange/merchant/payment/card/bind
> Add a new card via the selected provider on the PSP page (maximum of 5 cards can be added)

#### params:
- **clientId** - string(255), registered client ID
- **providerType** - [PaymentProviderId](../exchangeAPI#PaymentProvider-interfaces.), provider used to add the card
- **returnUrl** - string, the page to which the PSP should redirect the user after the card is added

#### response:
- **url** - string, link to the PSP page for adding a card

> A list of cards is available for the test environment.

---

### Payment methods request

#### POST /api/v2/exchange/merchant/payment/method
> Retrieve the user's available payment methods.

#### params:
- **clientId** - string(255), registered client ID
- **fiatAsset** - [CurrencyCode](../exchangeAPI#common-interfaces.), card currency
- **orderType** - "BUY" | "SELL", operation type, since payment methods may have direction restrictions

#### response:
- **[PaymentMethod](#interfaces)[]**

---

#### Interfaces
```typescript
interface PaymentMethod {
    providerType: PaymentProviderId,
    id: "string",                   // payment card token
    number: "string",               // card mask, last 4 digits
    brand: PaymentSystemType,       // payment system type, from exchangeAPI
    status: PaymentMethodStatus     // card status
}

enum PaymentMethodStatus {
    ACTIVE = "ACTIVE",
    // ... ???
}
```

