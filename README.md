# JoopJS

**Enterprise-grade, framework-agnostic TypeScript SDK for financial applications.**

[![npm version](https://img.shields.io/badge/npm-v2.0.2-blue)](https://www.npmjs.com/package/joopjs)
[![license](https://img.shields.io/badge/license-MIT-green)](./LICENSE)
[![node](https://img.shields.io/badge/node-%3E%3D18-brightgreen)](https://nodejs.org)
[![tests](https://img.shields.io/badge/tests-2168%20passed-success)](#)
[![playground](https://img.shields.io/badge/playground-live-orange)](https://kundan-leo.github.io/JoopJs/demo/)
[![github](https://img.shields.io/badge/github-kundan--leo%2FJoopJs-black?logo=github)](https://github.com/kundan-leo/JoopJs)

> Author: **Kundan Singh**

JoopJS provides 80+ production-ready services covering banking, finance, security, authentication, and encryption — all as plain TypeScript classes with no framework dependencies. Works with **React 18+**, **Angular 17+**, **Vue 3+**, or any plain TypeScript/JavaScript project.

---

## Table of Contents

- [Install](#install)
- [Quick Start](#quick-start)
- [Service Catalogue](#service-catalogue)
  - [Banking](#banking-26-services)
  - [Finance](#finance-13-services)
  - [Security](#security-12-services)
  - [Auth & Sessions](#auth--sessions-10-services)
  - [Encryption](#encryption-8-services)
  - [Device](#device-7-services)
  - [Network & API](#network--api-11-services)
  - [Workers & State](#workers--state-8-services)
  - [Observability](#observability-6-services)
  - [UI & Forms](#ui--forms-9-services)
  - [Utilities](#utilities-21-services)
  - [India & RegTech](#india--regtech-2-modules)
- [Framework Integration](#framework-integration)
- [Reactive Observables](#reactive-observables)
- [TypeScript Types](#typescript-types)
- [Sub-path Imports](#sub-path-imports)
- [Contributing](#contributing)
- [License](#license)

---

## Install

```bash
npm install joopjs
# or
yarn add joopjs
pnpm add joopjs
```

**Requirements:** Node.js ≥18, TypeScript ≥5.0 (optional but recommended).

No mandatory peer dependencies — React / Angular / Vue bindings are opt-in.

---

## Quick Start

### 1. Initialise

```typescript
import { createJoop } from 'joopjs';

const joop = createJoop({
  env: 'production',
  appId: 'my-banking-app',
  currency: 'USD',
  locale: 'en',
});
```

### 2. Use a service

```typescript
import { JoopDigitalWalletService } from 'joopjs';

const wallet = new JoopDigitalWalletService();

// Create a wallet
const w = wallet.createWallet('user-001', { currency: 'USD', label: 'Primary' });

// Top up and pay
wallet.topUp(w.id, 1000);
wallet.pay(w.id, 49.99, 'merchant-001', 'Coffee subscription');

// Check balance
console.log(wallet.getBalance(w.id)); // 950.01

// React to balance changes
const unsub = wallet.balance$().subscribe(({ walletId, balance }) => {
  console.log(`Wallet ${walletId}: $${balance}`);
});
unsub(); // stop listening
```

### 3. Auth flow

```typescript
import { JoopAuthService, JoopJwtService } from 'joopjs';

const auth = new JoopAuthService();
const jwt  = new JoopJwtService({ secret: process.env.JWT_SECRET!, expiryMs: 3_600_000 });

const session = await auth.login('alice@bank.com', 'password123');
const token   = jwt.sign({ userId: session.userId, role: 'admin' });
const claims  = jwt.verify(token); // { userId, role, iat, exp }
```

### 4. Encrypt sensitive data

```typescript
import { JoopGcmService } from 'joopjs';

const gcm = new JoopGcmService();
const key = await gcm.generateKey();

const { ciphertext, iv } = await gcm.encrypt('4111111111111111', key);
const pan = await gcm.decrypt(ciphertext, key, iv); // '4111111111111111'
```

---

## Service Catalogue

All services follow the same pattern:
```typescript
const svc = new JoopXxxService();  // plain class, no DI required
```

### Banking (26 services)

| Service | Key Methods |
|---|---|
| `JoopDigitalWalletService` | `createWallet`, `topUp`, `pay`, `transfer`, `freeze`, `getBalance`, `balance$` |
| `JoopLoanServicingService` | `createLoan`, `recordPayment`, `getSchedule`, `getOutstandingBalance`, `markDefaulted` |
| `JoopFxForwardService` | `setSpotRate`, `createForward`, `settleForward`, `getMarkToMarket`, `getExposure` |
| `JoopLedgerService` | `addAccount`, `postEntry`, `getBalance`, `getTrialBalance` |
| `JoopReconciliationService` | `createSession`, `autoMatch`, `manualMatch`, `getSummary` |
| `JoopAmlService` | `addRule`, `checkTransaction`, `flag`, `alert$` |
| `JoopLimitManagementService` | `setLimit`, `checkLimit`, `recordUsage` |
| `JoopStandingOrderService` | `create`, `execute`, `pause`, `resume`, `cancel`, `getDue` |
| `JoopPaymentOrchestratorService` | `submit`, `retry`, `getStatus`, `cancel` |
| `JoopOpenBankingClient` | `requestConsent`, `getAccounts`, `getTransactions`, `initiatePayment` |
| `JoopCardManagementService` | `issueCard`, `freeze`, `setSpendingLimits`, `setControls`, `checkTransaction` |
| `JoopBeneficiaryService` | `add`, `validate`, `getAll`, `remove` |
| `JoopDisputeService` | `file`, `uploadEvidence`, `resolve`, `getStats` |
| `JoopRemittanceService` | `setExchangeRate`, `getQuote`, `initiate`, `updateStatus` |
| `JoopInsuranceService` | `addPolicy`, `fileClaim`, `renewPolicy`, `recordPremiumPayment` |
| `JoopVirtualAccountService` | `create`, `recordCredit`, `isFullyCollected`, `close` |
| `JoopMandateService` | `create`, `execute`, `pause`, `resume`, `cancel`, `getDue` |
| `JoopSplitPaymentService` | `createExpense`, `addParticipant`, `settle`, `getBalances` |
| `JoopChequebookService` | `requestChequebook`, `activateChequebook`, `issueLeaf`, `markCleared` |
| `JoopBillPaymentService` | `addBiller`, `createBill`, `pay`, `getUpcoming` |
| `JoopStatementGeneratorService` | `addTransaction`, `generate` |
| `JoopScheduledPaymentService` | `schedule`, `cancel`, `getUpcoming` |
| `JoopNotificationCenterService` | `send`, `markRead`, `getUnread` |
| `JoopPaymentUriService` | `generate`, `parse` (UPI · SEPA · QR) |
| `JoopReceiptService` | `generate`, `getById` |
| `JoopStatementParser` | `parse` (CSV · OFX · QIF formats) |

**Example — Loan with amortisation schedule:**
```typescript
import { JoopLoanServicingService } from 'joopjs';

const loans = new JoopLoanServicingService();
const loan  = loans.createLoan({
  borrowerName: 'Alice Smith', borrowerId: 'u-001',
  principalAmount: 120_000, annualInterestRatePercent: 9.5,
  tenureMonths: 36, currency: 'USD',
});

console.log(`EMI: $${loan.emiAmount}`);          // auto-calculated
loans.recordPayment(loan.id, loan.emiAmount);    // interest-first allocation

const schedule = loans.getSchedule(loan.id);    // JoopInstallment[]
console.log(schedule[0].status);                // 'paid'
```

---

### Finance (13 services)

| Service | Key Methods |
|---|---|
| `JoopMutualFundService` | `registerFund`, `invest`, `redeem`, `updateNav`, `createSip`, `executeSip` |
| `JoopBudgetService` | `createBudget`, `addCategory`, `recordSpend`, `getReport`, `getAlerts` |
| `JoopPortfolioService` | `createPortfolio`, `addHolding`, `updatePrice`, `getSummary`, `rebalance` |
| `JoopTaxCalculatorService` | `setTaxProfile`, `addSlab`, `calculate`, `calcCapitalGainsTax` |
| `JoopCashFlowService` | `addCashFlow`, `forecast`, `getSummary` |
| `JoopSavingsGoalService` | `create`, `contribute`, `getProgress`, `getSuggestedContribution` |
| `JoopNetWorthService` | `addAsset`, `addLiability`, `getSnapshot`, `getHistory` |
| `JoopCurrencyExchangeService` | `setRate`, `convert`, `convertMultiple`, `getRateHistory` |
| `JoopLoanEligibilityService` | `check`, `getMaxAmount` |
| `JoopInterestCalculatorService` | `simple`, `compound`, `emi`, `rule72` |
| `JoopCreditScoreService` | `calculate`, `getFactors`, `simulate` |
| `JoopLoyaltyService` | `addPoints`, `redeem`, `getBalance`, `getTier` |
| `JoopTransactionCategorizerService` | `categorize`, `batchCategorize`, `addRule` |

**Example — Mutual Fund SIP:**
```typescript
import { JoopMutualFundService } from 'joopjs';

const mf = new JoopMutualFundService();
mf.registerFund({ id: 'EQ001', name: 'Equity Growth', category: 'equity', currentNav: 45.50, currency: 'USD' });

const holding = mf.invest('EQ001', 'inv-001', 5000);
console.log(holding.units);         // 5000 / 45.50 = 109.89 units

const sip = mf.createSip('EQ001', 'inv-001', 500, 'monthly', Date.now());
mf.executeSip(sip.id);              // invests 500, advances next execution date
```

---

### Security (12 services)

| Service | Key Methods |
|---|---|
| `JoopSanctionsScreeningService` | `loadList`, `screen`, `disableList`, `enableList`, `hit$` |
| `JoopAmlService` | `addRule`, `checkTransaction`, `flag`, `alert$` |
| `JoopRiskEngineService` | `addFactor`, `evaluate`, `setThreshold`, `score$` |
| `JoopFraudDetectionService` | `addRule`, `assess`, `alert$` |
| `JoopKycStateMachineService` | `submit`, `verify`, `getStatus`, `status$` |
| `JoopBehavioralBiometricsService` | `startSession`, `recordEvent`, `analyzeSession`, `setBaseline` |
| `JoopThreatIntelligenceService` | `addIndicator`, `checkIp`, `isBlocked`, `blockIndicator` |
| `JoopCompliancePolicyService` | `addPolicy`, `recordAudit`, `getPolicyStatus`, `getRecentFailures` |
| `JoopPIIScannerService` | `scan`, `redact`, `addPattern` |
| `JoopSecureClipboardService` | `write`, `read`, `clear`, `clearAfter` |
| `JoopAntiTamperService` | `protect`, `verify`, `onTamper$` |
| `JoopCertPinningService` | `pin`, `verify`, `addCertificate` |

**Example — Sanctions screening:**
```typescript
import { JoopSanctionsScreeningService } from 'joopjs';

const sanctions = new JoopSanctionsScreeningService();
sanctions.loadList('ofac', ofacEntities);

// React to hits before screening
sanctions.hit$().subscribe(hit => alertComplianceTeam(hit));

const result = sanctions.screen({ name: customer.fullName, country: customer.country });
// result.status: 'clear' | 'hit' | 'review'
// result.matches[]: [{ entity, matchType: 'exact'|'alias'|'fuzzy', score }]
if (result.status !== 'clear') throw new Error('Blocked by sanctions');
```

---

### Auth & Sessions (10 services)

| Service | Key Methods |
|---|---|
| `JoopAuthService` | `login`, `logout`, `refresh`, `getCurrentUser`, `session$` |
| `JoopJwtService` | `sign`, `verify`, `decode`, `revoke`, `isRevoked`, `renew` |
| `JoopOtpService` | `generate`, `verify`, `generateTotpSecret`, `verifyTotp` |
| `JoopMfaService` | `enroll`, `challenge`, `respond`, `unenroll`, `status$` |
| `JoopPKCEService` | `buildAuthorizationUrl`, `exchangeCode`, `refreshTokens`, `parseIdToken` |
| `JoopOIDCClient` | `loadDiscovery`, `buildLoginUrl`, `processCallback`, `endSession` |
| `JoopSsoService` | `registerProvider`, `getLoginUrl`, `handleCallback`, `mapRole` |
| `JoopIdleService` | `start`, `reset`, `stop`, `idle$` |
| `JoopMultiTabSyncService` | `broadcast`, `listen`, `sync$` |
| `JoopConcurrentSessionService` | `register`, `invalidateOthers`, `getActiveSessions` |

---

### Encryption (8 services)

| Service | Key Methods |
|---|---|
| `JoopGcmService` | `generateKey`, `encrypt`, `decrypt`, `encryptObject`, `decryptObject`, `exportKey`, `importKey` |
| `JoopX25519Service` | `generateKeyPair`, `deriveSharedSecret`, `deriveAesKey` |
| `JoopE2EService` | `initSession`, `encrypt`, `decrypt` |
| `JoopRsaService` | `generateKeyPair`, `encrypt`, `decrypt`, `sign`, `verify`, `exportPublicKey` |
| `JoopHashService` | `sha256`, `sha512`, `hmac`, `hashPassword`, `verifyPassword` |
| `JoopKeyVault` | `store`, `retrieve`, `revoke`, `rotate`, `list` |
| `JoopSecureStorage` | `set`, `get`, `remove`, `clear` |
| `JoopRandomService` | `bytes`, `hex`, `uuid`, `token`, `integer`, `nonce` |

**Example — End-to-end encrypted messaging:**
```typescript
import { JoopX25519Service, JoopGcmService } from 'joopjs';

const dh  = new JoopX25519Service();
const gcm = new JoopGcmService();

// Both parties generate key pairs and exchange public keys
const alice = dh.generateKeyPair();
const bob   = dh.generateKeyPair();

// Derive identical shared secret on both sides
const aliceShared = dh.deriveSharedSecret(alice.privateKey, bob.publicKey);
const sessionKey  = await dh.deriveAesKey(aliceShared);

// Encrypt / decrypt messages
const enc = await gcm.encrypt('Transfer $500 to Bob', sessionKey);
const msg = await gcm.decrypt(enc.ciphertext, sessionKey, enc.iv);
```

---

### Device (7 services)

| Service | Key Methods |
|---|---|
| `JoopDeviceFingerprintService` | `collect`, `collectFlat`, `hash` |
| `JoopBiometricAuthService` | `isAvailable`, `enroll`, `authenticate`, `revoke` |
| `JoopCardScannerService` | `scan`, `parse`, `validate` |
| `JoopPushService` | `register`, `unregister`, `onMessage$` |
| `JoopAppLifecycleService` | `onForeground$`, `onBackground$`, `onTerminate$` |
| `JoopScreenSecurityService` | `isDevtoolsOpen`, `activate`, `deactivate` |
| `JoopClientInfoService` | `collect`, `collectFlat` |

---

### Network & API (11 services)

| Service | Key Methods |
|---|---|
| `JoopHttpClient` | `get`, `post`, `put`, `patch`, `delete`, `upload` |
| `JoopHttpClientExtended` | `getWithRetry`, `postWithCache`, `batchRequests` |
| `JoopInterceptorPipeline` | `add`, `remove`, `execute` |
| `JoopRequestSigningService` | `sign`, `verify`, `signRequest` |
| `JoopRequestQueue` | `enqueue`, `flush`, `pause`, `resume` |
| `JoopOfflineQueueService` | `enqueue`, `flush`, `onOnline$` |
| `JoopPollingService` | `start`, `stop`, `result$` |
| `JoopIdempotencyService` | `execute`, `isDuplicate`, `clear` |
| `JoopWebhookVerifier` | `verify`, `addSecret` |
| `JoopCacheService` | `get`, `set`, `getOrSet`, `invalidate`, `stats` |
| `JoopWebSocketService` | `connect`, `send`, `message$`, `disconnect` |

---

### Workers & State (8 services)

| Service | Key Methods |
|---|---|
| `JoopStore` / `createStore` | `getState`, `dispatch`, `subscribe`, `undo`, `redo` |
| `JoopWorkflowEngine` | `register`, `start`, `transition`, `getState` |
| `JoopSyncEngine` | `push`, `pull`, `sync`, `conflict$` |
| `JoopWorkerPool` | `run`, `terminate`, `stats` |
| `JoopCompressionService` | `compress`, `decompress`, `compressString` |
| `JoopEventBus` / `joopEventBus` | `emit`, `on`, `off`, `once` |
| `JoopCircuitBreaker` | `execute`, `getState`, `reset` |
| `CRDTCounter` / `CRDTSet` / `CRDTText` | Conflict-free replicated data types |

---

### Observability (6 services)

| Service | Key Methods |
|---|---|
| `JoopAuditLog` | `log`, `query`, `getFailures` |
| `JoopPerformanceService` | `mark`, `measure`, `getEntries`, `clearMarks` |
| `JoopErrorReporter` | `report`, `captureException`, `flush` |
| `JoopWebVitals` | `collect`, `report`, `vitals$` |
| `JoopSessionRecorder` | `start`, `stop`, `replay`, `export` |
| `JoopCorrelationService` | `generate`, `set`, `get`, `clear` |

---

### UI & Forms (9 services)

| Service | Key Methods |
|---|---|
| `JoopThemeService` | `apply`, `toggle`, `currentTheme$` |
| `JoopI18nService` | `setLocale`, `t`, `formatCurrency`, `formatDate`, `locale$` |
| `JoopFeatureFlagService` | `enable`, `disable`, `isEnabled`, `change$` |
| `JoopFormValidator` | `validate`, `addRule`, `getErrors` |
| `JoopFormBuilder` | `field`, `group`, `array`, `build` |
| `JoopAlertService` | `success`, `error`, `warn`, `info`, `alert$` |
| `JoopPaginationService` | `setPage`, `nextPage`, `prevPage`, `state$` |
| `JoopA11yService` | `announce`, `setFocusTrap`, `releaseFocusTrap` |
| `JoopVirtualScroll` | `setItems`, `getVisibleRange`, `scroll$` |

---

### Utilities (21 services)

| Service | Key Methods |
|---|---|
| `JoopConfigService` | `set`, `get`, `getAll`, `reset` |
| `JoopLogger` | `debug`, `info`, `warn`, `error`, `setLevel` |
| `JoopHealthService` | `register`, `check`, `status$` |
| `JoopRateLimiter` | `check`, `record`, `block`, `getStatus` |
| `JoopAnalyticsService` | `track`, `identify`, `page`, `flush` |
| `JoopExportService` | `toCSV`, `toJSON`, `toXLSX` |
| `JoopGeoService` | `getCurrentPosition`, `watch`, `distanceTo` |
| `JoopNotificationService` | `show`, `schedule`, `cancel`, `permission$` |
| `JoopAiClient` | `chat`, `embed`, `complete` (OpenAI · Anthropic · Gemini · Ollama) |
| `JoopRouterService` | `navigate`, `addGuard`, `currentRoute$` |
| `JoopPluginService` | `register`, `execute`, `list` |
| ... and more | `JoopMediaService`, `JoopWatermarkService`, `JoopConsentService`, `JoopDeeplinkRouter`, `JoopNativeBridgeService` |

---

### India & RegTech (2 modules)

```typescript
import {
  validatePan, maskPan, getPanDetails,
  validateAadhaar, maskAadhaar, formatAadhaar,
  validateIfsc, getIfscBank, getIfscDetails,
  validateUpi, getUpiDetails, generateUpiLink,
  validateGst, maskGst, getGstState, getGstDetails,
} from 'joopjs/india';

validatePan('ABCDE1234F');          // true
maskPan('ABCDE1234F');              // 'ABCDE****F'
generateUpiLink({ payeeVpa: 'merchant@bank', amount: 100 }); // upi://pay?...
```

For NRIC (Singapore) and MyKad (Malaysia) — import from `'joopjs'` directly.

---

## Framework Integration

### React

```tsx
import { useEffect, useState } from 'react';
import { JoopDigitalWalletService } from 'joopjs';

const walletSvc = new JoopDigitalWalletService();   // singleton outside component

function BalanceCard({ walletId }: { walletId: string }) {
  const [balance, setBalance] = useState(0);

  useEffect(() => {
    const unsub = walletSvc.balance$().subscribe(({ walletId: id, balance }) => {
      if (id === walletId) setBalance(balance);
    });
    return unsub;   // auto-cleanup on unmount
  }, [walletId]);

  return <div>Balance: ${balance}</div>;
}
```

### Angular

```typescript
// app.config.ts (standalone)
import { ApplicationConfig } from '@angular/core';
import { provideJoop } from 'joopjs/angular';
import { createJoop } from 'joopjs';

export const appConfig: ApplicationConfig = {
  providers: [
    provideJoop(createJoop({ env: 'production', appId: 'my-app' })),
  ],
};
```

```typescript
// component.ts
import { inject } from '@angular/core';
import { JOOP_AUTH } from 'joopjs/angular';

@Component({ ... })
export class LoginComponent {
  private auth = inject(JOOP_AUTH);

  async login(email: string, password: string) {
    const session = await this.auth.login(email, password);
    console.log('Logged in:', session.userId);
  }
}
```

### Vue

```typescript
// main.ts
import { createApp } from 'vue';
import { createJoopVue } from 'joopjs/vue';
import { createJoop } from 'joopjs';
import App from './App.vue';

createApp(App)
  .use(createJoopVue(createJoop({ env: 'production', appId: 'my-app' })))
  .mount('#app');
```

```vue
<!-- BalanceCard.vue -->
<script setup lang="ts">
import { useJoopWallet } from 'joopjs/vue';
const { balance, topUp } = useJoopWallet('wallet-001');
</script>

<template>
  <div>Balance: {{ balance }}</div>
  <button @click="topUp(100)">Add $100</button>
</template>
```

---

## Reactive Observables

JoopJS uses its own lightweight reactive primitives (`JoopSubject`, `JoopBehaviorSubject`) — **not RxJS**. They are lighter, tree-shakeable, and have no RxJS dependency.

```typescript
import { JoopSubject, JoopBehaviorSubject } from 'joopjs';

// Event stream — no initial value
const alerts$ = new JoopSubject<string>();

// Stateful stream — holds current value, replays to new subscribers
const status$ = new JoopBehaviorSubject<'idle' | 'processing' | 'done'>('idle');

// Subscribe
const unsub = status$.subscribe(value => console.log('Status:', value));
// → immediately logs 'idle' (BehaviorSubject replays current value)

// Emit
status$.next('processing');   // logs 'processing'

// Read current value synchronously
status$.getValue();            // 'processing'

// Stop listening
unsub();
```

**Critical rule:** Always use `.next(value)` to emit — **never `.emit(value)`** (that method does not exist).

---

## TypeScript Types

All public types use the `Joop*` prefix and are exported from the same entry as their service:

```typescript
// Status unions
type JoopWalletStatus      = 'active' | 'frozen' | 'closed';
type JoopLoanAccountStatus = 'active' | 'closed' | 'defaulted' | 'written-off';
type JoopRepaymentStatus   = 'pending' | 'paid' | 'partial' | 'overdue' | 'waived';
type JoopForwardType       = 'buy' | 'sell';
type JoopSipStatus         = 'active' | 'paused' | 'cancelled' | 'completed';
type JoopFundCategory      = 'equity' | 'debt' | 'hybrid' | 'money-market' | 'index' | 'etf';
type JoopSanctionMatchType = 'exact' | 'alias' | 'fuzzy';
type JoopSanctionsList     = 'ofac' | 'un' | 'eu' | 'uk' | 'custom';
type JoopAccountType       = 'asset' | 'liability' | 'equity' | 'revenue' | 'expense';

// Data conventions
// Amounts  → number  (never a Money class)
// Currency → string  (ISO-4217: 'USD', 'EUR', 'AED')
// Dates    → number  (Unix epoch milliseconds: Date.now())
// IDs      → string
```

---

## Sub-path Imports

Import from a specific sub-path for optimal tree-shaking:

```typescript
import { JoopGcmService }          from 'joopjs/encryption';
import { JoopAuthService }         from 'joopjs/auth';
import { JoopCacheService }        from 'joopjs/cache';
import { JoopWorkflowEngine }      from 'joopjs/workflow';
import { JoopAiClient }            from 'joopjs/ai';
import { validatePan, maskPan }    from 'joopjs/india';
```

**Available sub-paths:**
`encryption` · `auth` · `api` · `core` · `session` · `banking` · `security` · `device` · `observability` · `theme` · `i18n` · `ui` · `native-bridge` · `deeplink` · `cache` · `network` · `analytics` · `validation` · `utilities` · `forms` · `pwa` · `router` · `ai` · `state` · `workers` · `workflow` · `sync` · `platform` · `react` · `angular` · `vue` · `india`

---

## Contributing

See [CONTRIBUTING.md](./docs/CONTRIBUTING.md) for development setup, coding conventions, and the pull request process.

---

## License

[MIT](./LICENSE) © Kundan Singh
