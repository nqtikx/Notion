# Errors and Status Routing (V2 OnRamp / OffRamp)

This section lists practical error responses for V2 merchant exchange flow endpoints:
- `/api/v2/exchange/merchant/assets`
- `/api/v2/exchange/merchant/payment/provider`
- `/api/v2/exchange/merchant/payment/method`
- `/api/v2/exchange/merchant/limit`
- `/api/v2/exchange/merchant/quote`
- `/api/v2/exchange/merchant/buy`
- `/api/v2/exchange/merchant/sell`
- `/api/v2/exchange/merchant/order`
- `/api/v2/exchange/merchant/order/current`
- `/api/v2/exchange/merchant/order/history`

### Errors and status routing table

| Scope | HTTP | `status` | When it occurs | Example JSON response |
|---|---:|---|---|---|
| Any merchant V2 endpoint | `401` | `Unauthorized` | Invalid or missing `x-api-key`; missing/mismatched `externalClientId` when required | `{"message":"Invalid api key","code":401,"status":"Unauthorized"}` |
| Any merchant V2 endpoint | `403` | `Forbidden` | Spring Security access denied (e.g. order does not belong to actor) | `{"message":"Order does not belong to actor","code":403,"status":"Forbidden"}` |
| Any merchant V2 endpoint | `429` | `Too Many Requests` | Rate limit exceeded | `{"message":"Too many requests","code":429,"status":"Too Many Requests"}` |
| Any merchant V2 endpoint | `415` | `Unsupported Media Type` | Unsupported request content type | `{"message":"Unsupported content type: text/plain; ...","code":415,"status":"Unsupported Media Type"}` |
| Any merchant V2 endpoint | `400` | `Bad Request` | Invalid JSON body or required parameter missing | `{"message":"Invalid request body","code":400,"status":"Bad Request"}` |
| Any merchant V2 endpoint with `@Valid` | `400` | `BAD_REQUEST` | Bean validation failed | `{"status":"BAD_REQUEST","message":"Validation errors","code":400,"fields":[{"field":"clientId","value":null,"message":"must not be null"}]}` |
| `/merchant/payment/method`, `/merchant/order/current`, `/merchant/order/history`, `/merchant/quote`, `/merchant/limit` (and other endpoints that validate merchant-client relation) | `400` | `CLIENT_NOT_FOUND` | `clientId` does not exist for current merchant or merchant-client relation is missing | `{"message":"Client not found","code":400,"status":"CLIENT_NOT_FOUND"}` |
| `/merchant/quote`, `/merchant/limit`, `/merchant/buy`, `/merchant/sell` | `400` | `CURRENCY_NOT_FOUND` | Unknown or invalid asset/currency provided in exchange request | `{"message":"No such currency: <value>","code":400,"status":"CURRENCY_NOT_FOUND"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `QUOTE_NOT_FOUND` | Quote not found or already expired when creating order | `{"message":"Quote is expired: <quoteId>","code":400,"status":"QUOTE_NOT_FOUND"}` |
| `/merchant/quote` | `400` | `INVALID_QUOTE` | Both `from.amount` and `to.amount` provided, or invalid pair direction (not buy and not sell) | `{"message":"Two asset amounts are not supported","code":400,"status":"INVALID_QUOTE"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `INVALID_QUOTE` | Wrong quote type for the operation, quote without client, or quote not belonging to actor | `{"message":"Not buy quote: <quoteId>","code":400,"status":"INVALID_QUOTE"}` |
| `/merchant/quote`, `/merchant/buy`, `/merchant/sell` | `400` | `INVALID_ASSET` | Asset/payment type mismatch or unsupported asset for operation/client | `{"message":"Fiat provider payment type supports only fiat assets","code":400,"status":"INVALID_ASSET"}` |
| `/merchant/quote`, `/merchant/limit`, `/merchant/buy`, `/merchant/sell` | `400` | `INVALID_ADDRESS` | Crypto address format is invalid | `{"message":"invalid crypto address","code":400,"status":"INVALID_ADDRESS"}` |
| `/merchant/quote`, `/merchant/limit`, `/merchant/buy`, `/merchant/sell` | `400` | `DESTINATION_ADDRESS_IS_INTERNAL` | Destination crypto address belongs to internal wallet | `{"message":"destination address is internal","code":400,"status":"DESTINATION_ADDRESS_IS_INTERNAL"}` |
| `/merchant/quote`, `/merchant/buy`, `/merchant/sell` | `400` | `INVALID_PAYMENT_PROVIDER` | Payment provider is not available for this operation/client | `{"message":"Payment provider is not available for this operation","code":400,"status":"INVALID_PAYMENT_PROVIDER"}` |
| `/merchant/quote`, `/merchant/buy`, `/merchant/sell` | `400` | `INVALID_PAYMENT_TOKEN` | Payment token is missing/invalid for required provider | `{"message":"Invalid payment token","code":400,"status":"INVALID_PAYMENT_TOKEN"}` |
| `/merchant/quote`, `/merchant/buy`, `/merchant/sell` | `400` | `INVALID_PAYMENT_METHOD` | Payment method cannot be validated/used | `{"message":"Invalid payment method","code":400,"status":"INVALID_PAYMENT_METHOD"}` |
| `/merchant/quote`, `/merchant/buy`, `/merchant/sell` | `400` | `ORDER_SESSION_ALREADY_EXISTS` | Provided `sessionId` already belongs to an existing order | `{"message":"Order with sessionId: <sessionId> already exists","code":400,"status":"ORDER_SESSION_ALREADY_EXISTS"}` |
| `/merchant/limit` | `503` | `EXCHANGE_UNAVAILABLE` | Exchange backend/calculation service unavailable | `{"message":"<service message>","code":503,"status":"EXCHANGE_UNAVAILABLE"}` |
| `/merchant/limit` | `400` | `LIMIT_VIOLATION` | Generic limit violation on calculation | `{"message":"<limit violation message>","code":400,"status":"LIMIT_VIOLATION"}` |
| `/merchant/limit` | `400` | `AMOUNT_TOO_SMALL` | Amount is below allowed threshold | `{"message":"<amount too small message>","code":400,"status":"AMOUNT_TOO_SMALL"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `INVALID_MIN_LIMIT` | Amount below minimum allowed limit | `{"status":"INVALID_MIN_LIMIT","message":"<limit message>","code":400,"fields":[{"field":"limit","value":"<min>","message":"<limit message>"}]}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `INVALID_MAX_LIMIT` | Amount above maximum allowed limit | `{"status":"INVALID_MAX_LIMIT","message":"<limit message>","code":400,"fields":[{"field":"limit","value":"<max>","message":"<limit message>"}]}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `BAD_REQUEST` | Destination crypto address is required but missing (and not present in quote) | `{"message":"Destination crypto address is null","code":400,"status":"BAD_REQUEST"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `EXCHANGE_DISABLED` | Exchange pair/route currently disabled | `{"message":"exchange is disabled","code":400,"status":"EXCHANGE_DISABLED"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `ACTIVE_ORDER_FOUND` | Client already has active processing order | `{"message":"Active order is found","code":400,"status":"ACTIVE_ORDER_FOUND"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `ACTIVE_DEPOSIT_REQUEST_FOUND` | Client has active deposit request | `{"message":"Active deposit request is found","code":400,"status":"ACTIVE_DEPOSIT_REQUEST_FOUND"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `ACTIVE_WITHDRAWAL_REQUEST_FOUND` | Client has active withdrawal request | `{"message":"Active withdrawal request is found","code":400,"status":"ACTIVE_WITHDRAWAL_REQUEST_FOUND"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `INVALID_CLIENT_STATUS` | Client status is not eligible (`NOT_VERIFIED`, `PENDING`, `FROZEN`, `ARREST`, testing incomplete) | `{"message":"invalid client status","code":400,"status":"INVALID_CLIENT_STATUS"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `PHONE_CONFIRMATION_REQUIRED` | Phone confirmation is required for client | `{"message":"Phone confirmation required","code":400,"status":"PHONE_CONFIRMATION_REQUIRED"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `PHONE_NUMBER_REQUIRED` | Client phone is blank/invalid | `{"message":"Phone number is blank","code":400,"status":"PHONE_NUMBER_REQUIRED"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `INVALID_MFA_STATUS` | Client must have MFA enabled for deposit/withdrawal operation | `{"message":"Client must have MFA enabled","code":400,"status":"INVALID_MFA_STATUS"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `INVALID_INTERNAL_BALANCE` | Insufficient client internal balance | `{"message":"Insufficient client internal balance ...","code":400,"status":"INVALID_INTERNAL_BALANCE"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `INVALID_CRYPTO_BALANCE` | Insufficient system crypto liquidity for payout | `{"message":"Insufficient system crypto balance: ...","code":400,"status":"INVALID_CRYPTO_BALANCE"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `INVALID_FIAT_BALANCE` | Insufficient system fiat balance for payout | `{"message":"Insufficient system fiat balance: ...","code":400,"status":"INVALID_FIAT_BALANCE"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `INVALID_CORPORATE_CARD` | Required corporate/system payment card not found | `{"message":"No fiat card by <asset>, <provider>","code":400,"status":"INVALID_CORPORATE_CARD"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `INVALID_SCAN_STATUS` | Mempool scan is unavailable for one of assets | `{"message":"mempool scan unavailable TRX","code":400,"status":"INVALID_SCAN_STATUS"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `SINGLE_OP_MAX_VOLUME_EXCEEDED` | Single operation amount exceeds configured restriction | `{"message":"Single operation max volume exceeded, client ...","code":400,"status":"SINGLE_OP_MAX_VOLUME_EXCEEDED"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `OP_MAX_VOLUME_FOR_PERIOD_REACHED` | Period volume restriction reached | `{"message":"Period operations max volume reached, client ...","code":400,"status":"OP_MAX_VOLUME_FOR_PERIOD_REACHED"}` |
| `/merchant/buy`, `/merchant/sell` | `400` | `OP_MAX_NUM_FOR_PERIOD_REACHED` | Max operations count for period reached | `{"message":"Operations max num reached","code":400,"status":"OP_MAX_NUM_FOR_PERIOD_REACHED"}` |
| `/merchant/order` | `400` | `Bad Request` | Order id exists but business flow cannot load/process it | `{"message":"No order found: <orderId>","code":400,"status":"Bad Request"}` |

### Routing summary by HTTP status

| HTTP | Meaning for integrator | Typical action |
|---:|---|---|
| `400` | Request/business validation failed | Fix request data, client state, limits, or route |
| `401` | API key is invalid/missing or `externalClientId` mismatch | Check `x-api-key` and `externalClientId` |
| `403` | Spring Security access denied | Check actor permissions and resource ownership |
| `415` | Unsupported media type | Use `Content-Type: application/json` |
| `429` | Rate limit exceeded | Retry later with backoff |
| `503` | Exchange/calculation temporarily unavailable | Retry with backoff and monitoring |
