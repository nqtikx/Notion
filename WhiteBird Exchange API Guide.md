# WhiteBird Exchange API Guide

## Introduction

This guide describes how to integrate **WhiteBird Exchange API** for operational exchange flows.

The document is focused on:
- **OnRamp** (`fiat -> crypto`)
- **OffRamp** (`crypto -> fiat`)
- merchant and client API usage patterns
- status/compliance routing behavior
- operational tracking and webhook-related capabilities

> This guide is exchange-flow focused.  
> User onboarding, registration, KYC profile submission, and token issuance are documented separately in **WhiteBird Registration API Guide**.

---

## Table of Contents

- [1. Base principles](#1-base-principles)
- [2. OnRamp (fiat -> crypto) — Merchant API](#2-onramp-fiat---crypto--merchant-api)
- [3. OffRamp (crypto -> fiat) — Merchant API](#3-offramp-crypto---fiat--merchant-api)
- [4. Client API mirror](#4-client-api-mirror)
- [5. Errors and status routing](#5-errors-and-status-routing)
- [6. Webhooks and callbacks](#6-webhooks-and-callbacks)
- [7. Legacy v2 compatibility](#7-legacy-v2-compatibility)

---

## 1. Base principles

### Scope
WhiteBird Exchange API supports two directions:
- **OnRamp**: `fiat -> crypto`
- **OffRamp**: `crypto -> fiat`

### API domains
- **Merchant API** — backend-to-backend, auth via `x-api-key`.
- **Client API** — user-context API:
  - normally uses `Authorization: Bearer <accessToken>`;
  - for some endpoints, anonymous mode is supported with `merchantId` in request body.

### Recommended pipeline (v2-first)
1. `assets`
2. `payment/provider` (and `payment/method` if needed)
3. `limit` (v2)
4. `quote` (v2)
5. `buy/sell`
6. `order status/current/history/reject`

### Status routing
Core statuses used by exchange routing:
- `NOT_VERIFIED`
- `PENDING`
- `VERIFIED`
- `FROZEN`
- `ARREST`

## 2. [OnRamp (fiat -> crypto) — Merchant API](./V2.md)

## 3. [OffRamp (crypto -> fiat) — Merchant API](./V2.md)

## 4. Errors and status routing

_To be completed._

## 5. [Webhooks and callbacks](./Webhooks%20Overview.md)
