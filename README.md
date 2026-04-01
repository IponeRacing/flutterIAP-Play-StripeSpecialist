# Flutter IAP · Play · Stripe Specialist

> A Claude Code sub-agent that implements native in-app purchases, subscriptions,
> and payments in Flutter — without RevenueCat or any third-party SDK.

[![License: MIT](https://img.shields.io/badge/License-MIT-purple.svg)](LICENSE)
[![Flutter](https://img.shields.io/badge/Flutter-3.27%2B-02569B?logo=flutter)](https://flutter.dev)
[![StoreKit 2](https://img.shields.io/badge/StoreKit-2-black?logo=apple)](https://developer.apple.com/storekit/)
[![Play Billing](https://img.shields.io/badge/Play_Billing-v7-34A853?logo=google-play)](https://developer.android.com/google/play/billing)
[![Stripe](https://img.shields.io/badge/Stripe-v11-635BFF?logo=stripe)](https://stripe.com)

---

## What it does

This agent handles every payment flow in Flutter using **raw native APIs** — no RevenueCat,
no third-party entitlement service:

| Platform | Technology | Use case |
|----------|-----------|----------|
| 🍎 iOS / macOS | Apple StoreKit 2 | IAP, subscriptions, consumables |
| 🤖 Android | Google Play Billing v7 | IAP, subscriptions, upgrades |
| 🌐 Web / Cross-platform | Stripe v11 | Checkout, SaaS billing, non-IAP |

It covers the full purchase lifecycle:
- Product loading and paywall UI
- Purchase initiation (consumable / non-consumable / subscription)
- Pending purchase handling (family sharing, parental approval)
- Server-side JWS verification (Apple) + PurchaseToken (Google)
- Subscription upgrade/downgrade with `ReplacementMode`
- Entitlement management and offline caching
- Restore purchases flow
- EU DMA 2026 compliance

---

## Installation

### Project-level (one project)

```bash
mkdir -p your-project/.claude/agents
cp agents/flutter-payment-specialist.md your-project/.claude/agents/
```

### Global (all Claude Code sessions)

```bash
mkdir -p ~/.claude/agents
cp agents/flutter-payment-specialist.md ~/.claude/agents/
```

---

## Usage examples

```
Implement in-app purchases for my Flutter word game:
- "hints_pack_10" consumable (10 hints for $0.99)
- "remove_ads" non-consumable ($2.99)
- "premium_monthly" subscription ($4.99/month)

Use StoreKit 2 on iOS and Play Billing v7 on Android.
Add server-side JWS verification. Handle pending purchases.
```

```
Add a Stripe payment sheet to my Flutter web app for a $9.99
one-time purchase. The backend is Node.js Express.
```

```
My Android app crashes with: enablePendingPurchases() was not called.
Fix the Google Play Billing initialization.
```

---

## Key patterns covered

### 🍎 StoreKit 2 (iOS 15+)
- `InAppPurchaseStoreKitPlatformAddition` — enable SK2, set delegate
- `jwsRepresentation` as `serverVerificationData` — server-side JWS verification
- Swift `Transaction.updates` listener bridge for real-time subscription changes
- `Transaction.currentEntitlements` equivalent via `restorePurchases()`

### 🤖 Google Play Billing v7
- `enablePendingPurchases()` — **mandatory**, prevents `IllegalArgumentException` crash
- `ReplacementMode` — replaces the removed `ProrationMode`
- Suspended subscription detection and "Fix payment" UI
- `PurchaseToken` verification via Google Play Developer API v3

### 🔄 Cross-platform
- `purchaseStream` subscription from `main()` — never inside a widget
- `completePurchase()` on every status, including errors
- `storekit_duplicate_product_object` duplicate transaction guard
- `restorePurchases()` with settings button

### 💳 Stripe (web/cross-platform)
- `PaymentSheet` with Apple Pay + Google Pay support
- Backend `PaymentIntent` creation (Node.js reference included)
- `SetupIntent` for subscription billing

---

## Packages used

```yaml
in_app_purchase: ^3.2.0
in_app_purchase_storekit: ^0.3.18
in_app_purchase_android: ^0.3.6
flutter_stripe: ^11.0.0
```

---

## Anti-patterns prevented

| ❌ Wrong | ✅ Correct |
|----------|-----------|
| Deliver content without server verification | Always verify via Apple/Google API |
| Skip `completePurchase()` on error | Call for ALL statuses (purchased, restored, error) |
| Subscribe to `purchaseStream` inside a widget | Subscribe in `main()` before first frame |
| Use `ProrationMode` (removed in Play v7) | Use `ReplacementMode` |
| Test on emulator | Test on real device with sandbox account |
| `buyNonConsumable` for coins | `buyConsumable` for consumables |
| Ignore `PurchaseStatus.pending` | Show "waiting for approval" UI |

---

## Compatibility

| Flutter | Dart | iOS | Android |
|---------|------|-----|---------|
| 3.27+ | 3.6+ | 15.0+ (SK2) | API 24+, compileSdk 35+ |

---

## License

MIT — [EngageEngine](https://engageengine.ch)
