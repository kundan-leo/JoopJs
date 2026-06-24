# Changelog

**Author:** Kundan Singh

All notable changes to **JoopJS** are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).
Versioning follows [Semantic Versioning](https://semver.org/).

---

## [2.1.0] — 2026-06-25 — Angular DI & Federation-Safe Core

**0 new services · 21 new tests · 2189 tests total**

### Added
- **`joopjs/angular` — real Angular 17+ DI.** Alongside the existing string tokens (kept for back-compat):
  - `JOOP_INSTANCE` — typed `InjectionToken<JoopInstance>`, plus `provideJoopAngular()` and `injectJoop()`.
  - `joopSignal(obs, opts?)` — bridges a `JoopObservable` to an Angular `Signal`, seeded from the current value and torn down automatically via `DestroyRef`.
  - `joopAuthGuard(opts?)` — functional `CanActivateFn` backed by `JoopAuthService`, returning a redirect `UrlTree` when unauthenticated.
  - `joopAuthInterceptor(opts?)` — `HttpInterceptorFn` that attaches the access token from `JoopTokenService` (configurable header/scheme).
- **`joopjs/core` — federation-safe singletons.** `globalSingleton()`, `joopCopyCount()`, and `silenceDuplicateCopyWarning()`. Module-level singletons (`logger`, `configService`, `joopEventBus`, `clipboard`, `clientProfile`) now resolve through a version-keyed `globalThis` registry, so duplicate bundled copies in a micro-frontend converge on one instance instead of fragmenting state. A one-time console warning fires when more than one copy is detected.
- **`setSubjectErrorHandler()`** in the events module to route subscriber errors.
- **`scripts/check-core-boundary.mjs`** + `npm run check:boundary` — CI guard enforcing that core never imports a framework (framework imports allowed only in `angular`/`react`/`vue` sub-paths).

### Changed
- **`JoopSubject` / `JoopBehaviorSubject` emission is now error-isolated.** `next()` snapshots its listeners and wraps each in try/catch, so one throwing subscriber no longer aborts delivery to the rest, and a subscribe/unsubscribe triggered mid-emission can't disturb the in-flight notification.
- `@angular/common` and `@angular/router` added as **optional** peer dependencies (for `joopAuthGuard` / `joopAuthInterceptor`); `tsup` externalizes them so they are never bundled.

### Deprecated
- `joopjs/angular`: the string tokens (`JOOP`, `JOOP_AUTH`, …), `provideJoop()`, `JoopModule`, and `fromJoop()` are superseded by `JOOP_INSTANCE` + `provideJoopAngular()` + `joopSignal()`. They remain functional but will be removed in a future major. (`toJoop()` is **not** deprecated — it has no signal equivalent for bringing existing RxJS streams into JoopJS.)

## [2.0.6] — 2026-06-22 — Patch Release

**0 new services · 0 new tests · 2168 tests total**

### Fixed
- README version badge updated from `v2.0.5` → `v2.0.6`

### Notes
- Patch bump to correctly reflect the published npm version after the 2.0.5 AI Rules Setup release.
- All 21 smoke tests pass against the npm-published package (wallet, loans, AML, sanctions, mutual funds, GCM encryption, setup-ai files).

## Version History

| Version | Codename | New Services | Tests | Total Tests | Sub-paths Added |
|---|---|---|---|---|---|
| [2.1.0](#210--angular-di--federation-safe-core) | Angular DI & Federation-Safe Core | — | +21 | **2189** | — |
| [2.0.6](#206--patch-release) | Patch Release | — | — | **2168** | — |
| [2.0.5](#205--ai-rules-setup) | AI Rules Setup | — | — | **2168** | — |
| [2.0.0](#200--vue-composables) | Vue Composables | — | +39 | **1080** | `vue` |
| [1.9.0](#190--banking-depth) | Banking Depth | 4 | +100 | **1041** | — |
| [1.8.0](#180--framework-bindings) | Framework Bindings | — | +25 | 941 | `react`, `angular` |
| [1.7.0](#170--mobile-core-services) | Mobile Core Services | 5 | +48 | **916** | — |
| [1.6.0](#160--platform-foundation) | Platform Foundation | 1 | +15 | 868 | `platform` |
| [1.5.0](#150--ai-era-enterprise-pack) | AI-Era Enterprise Pack | 21 | +198 | 853 | `ai`, `state`, `workers`, `workflow`, `sync` |
| [1.4.0](#140--platform-services-pack) | Platform Services Pack | 16 | +160 | 655 | `forms`, `pwa`, `router` |
| [1.3.0](#130--enterprise-feature-pack) | Enterprise Feature Pack | 15 | +143 | 495 | `cache`, `network`, `analytics`, `validation` |
| [1.2.0](#120--framework-agnostic-services-pack) | Framework-Agnostic Services Pack | 9 | +94 | 352 | `theme`, `i18n`, `ui`, `native-bridge`, `deeplink` |
| [1.1.0](#110--enterprise--banking-pack) | Enterprise & Banking Pack | 22 | +134 | 258 | `banking`, `observability`, `security`, `device`, `dev` |
| [1.0.0](#100--initial-release) | Initial Release | 14 | 124 | 124 | `encryption`, `storage`, `auth`, `api`, `core`, `session` |

---

## Sub-path Exports Reference

| Import path | Key exports |
|---|---|
| `joopjs` | Everything (main entry) |
| `joopjs/encryption` | `JoopGcmService`, `JoopRsaService`, `JoopAesService`, `JoopX25519Service`, `JoopE2EService`, `JoopCryptoUtils` |
| `joopjs/storage` | `JoopScopeMapService`, `JoopDataStorage`, `JoopIndexedDbService` |
| `joopjs/auth` | `JoopAuthService`, `JoopTokenService`, `JoopPKCEService`, `JoopOtpService`, `JoopBiometricService`, `JoopMfaService`, `JoopOIDCClient` |
| `joopjs/api` | `JoopHttpClient`, `JoopRequestService`, `JoopInterceptorPipeline`, `JoopCircuitBreaker`, `JoopOfflineQueue`, `JoopPollingService`, `JoopFileSaverService`, `JoopResponseMapper`, `JoopExportService`, `JoopWebhookVerifier`, `JoopIdempotencyService` |
| `joopjs/core` | `JoopConfigService`, `JoopLogger`, `JoopPluginService`, `JoopEnvironmentService`, `JoopHealthService`, `JoopFeatureFlagService`, `JoopRateLimiter`, `JoopConsentService` |
| `joopjs/session` | `JoopSessionTimeoutService`, `JoopMultiTabSync`, `JoopConcurrentSessionService`, `JoopIdleService` |
| `joopjs/banking` | IBAN, currency, masking, transaction ID, Luhn/card, `JoopReceiptService`; v1.9.0+: `JoopFraudDetection`, `JoopOpenBankingClient`, `JoopPaymentOrchestrator`, `JoopStatementParser` |
| `joopjs/observability` | `JoopCorrelationService`, `JoopAuditLog`, `JoopPerformanceService`, `JoopErrorReporter`, `JoopWebVitals`, `JoopSessionRecorder` |
| `joopjs/security` | `JoopSecureStorage`, `JoopBehavioralBiometrics`, `JoopRiskEngine`, `JoopPIIScanner`, `JoopSecretSharing`, `JoopDifferentialPrivacy`, `JoopWatermarkService` |
| `joopjs/device` | `JoopDeviceFingerprintService`, `JoopKeyVault`, `JoopBiometricAuth`, `JoopPushService`, `JoopAppLifecycle`, `JoopNetworkMonitor` |
| `joopjs/dev` | `JoopMockService` |
| `joopjs/theme` | `JoopThemeService` |
| `joopjs/i18n` | `JoopI18nService` |
| `joopjs/ui` | `JoopLoaderService`, `JoopAlertService`, `JoopPaginationService`, `JoopPrintService`, `JoopA11yService`, `JoopVirtualScroll` |
| `joopjs/native-bridge` | `JoopNativeBridgeService` |
| `joopjs/deeplink` | `JoopDeeplinkService` |
| `joopjs/cache` | `JoopCacheService` |
| `joopjs/network` | `JoopWebSocketService`, `JoopStreamingService` |
| `joopjs/analytics` | `JoopAnalyticsService`, `consoleAnalyticsAdapter` |
| `joopjs/validation` | `JoopFormValidator` |
| `joopjs/utilities` | Date, Hijri, QR, barcode, chart data, geo, media, clipboard, image, sanitizer, PDF, watermark, notification, geo utils |
| `joopjs/forms` | `JoopFormBuilder`, `JoopField`, `JoopForm` |
| `joopjs/pwa` | `JoopServiceWorkerService`, `JoopNotificationService` |
| `joopjs/router` | `JoopRouterService` |
| `joopjs/ai` | `JoopAiClient`, `JoopToolRegistry` |
| `joopjs/state` | `JoopStore`, `createStore`, `createSlice` |
| `joopjs/workers` | `JoopWorkerPool`, `JoopCompressionService` |
| `joopjs/workflow` | `JoopStateMachine`, `JoopWorkflowEngine` |
| `joopjs/sync` | `JoopSyncEngine`, `CRDTCounter`, `CRDTSet`, `CRDTRegister`, `CRDTText`, `mergeCRDT` |
| `joopjs/platform` | `JoopPlatform`, all adapter interfaces |
| `joopjs/react` | `createJoopReact()`, `JoopProvider`, all hooks (v1.8.0+) |
| `joopjs/angular` | `provideJoop()`, `JoopModule`, `fromJoop()`, `toJoop()`, `createZonedFromJoop()`, all injection tokens (v1.8.0+) |
| `joopjs/vue` | `createJoopVue()`, `provideJoop`, `useJoop`, all composables (v2.0.0+) |

---

## [2.0.5] — AI Rules Setup

**0 new services · 0 new tests · 2168 tests total · DX release**

### Added

- **`npx joopjs setup-ai`** — one-command AI assistant setup for consumers. Copies joopjs-aware rules into the user's project so that Claude Code, Cursor, Windsurf, GitHub Copilot, Gemini CLI, and Codex all understand the joopjs API out of the box.

- **`scripts/setup-ai.mjs`** — the setup script, published as the `joopjs` bin entry. Copies consumer-facing rules from `node_modules/joopjs/` to the correct locations in the user's project. Skips existing files by default; `--force` flag overwrites. Prints a post-install summary with the exact CLAUDE.md skill lines to add.

- **`ai-rules/`** — new directory included in the npm package. Contains consumer-only versions of `GEMINI.md` and `AGENTS.md` (usage guide only, no library-dev content).

- **Consumer AI rules** included in the npm package for the first time:
  - `.claude/skills/` — 7 consumer skills: `setup`, `banking`, `finance`, `security`, `auth`, `encryption`, `observables`
  - `.cursor/rules/joopjs.mdc` — Cursor rule (auto-applied to all `.ts` / `.tsx` / `.js` files)
  - `.windsurf/rules/joopjs.md` — Windsurf Cascade rule
  - `.github/copilot-instructions.md` — GitHub Copilot instructions
  - `ai-rules/GEMINI.md` — Gemini CLI consumer guide
  - `ai-rules/AGENTS.md` — Codex / OpenAI Agents consumer guide

- **Library-dev skills** (not published to npm) — 4 new Claude Code skills for library contributors:
  - `/dev-build` — dev environment, build commands, tsup config, dist structure
  - `/dev-testing` — Vitest commands, test conventions, writing tests for new services
  - `/dev-contributing` — full checklist for adding a service or sub-path, code conventions
  - `/dev-release` — local and CI release flow, preflight checks, CHANGELOG format

- **Cursor dev rules** (`.cursor/rules/dev-contributing.mdc`) and **Windsurf dev rules** (`.windsurf/rules/dev-contributing.md`) — development-focused rule files for library contributors using Cursor or Windsurf.

- **`GEMINI.md`** and **`AGENTS.md`** at project root — library-dev guides for Gemini CLI and Codex contributors.

### Changed

- `package.json` `"files"` — extended to include `ai-rules/`, `scripts/setup-ai.mjs`, the 7 consumer Claude skills, `.cursor/rules/joopjs.mdc`, `.windsurf/rules/joopjs.md`, and `.github/copilot-instructions.md`.
- `package.json` `"bin"` — added `"joopjs": "scripts/setup-ai.mjs"` enabling `npx joopjs setup-ai`.
- `README.md` — added **AI Assistant Setup** section immediately after Install; npm badge updated to v2.0.5.

### User workflow

```bash
npm install joopjs
npx joopjs setup-ai           # first time
npx joopjs setup-ai --force   # after upgrading joopjs
```

---

## [2.0.0] — Vue Composables

**0 new core services · 39 new tests · 1080 tests total · 90 test files · 1 new sub-path**

### Added

- **`joopjs/vue`** sub-path — Vue 3.x integration using the same factory pattern as React: `createJoopVue(Vue)` receives the Vue instance as a parameter so the package never imports `vue` directly. Marks `vue >=3.0.0` as an optional peer dependency. Ships 12 composables:
  - `provideJoop(instance)` — registers a `JoopInstance` into the component tree via Vue `provide()`
  - `useJoop()` — access the full `JoopInstance` from any child component
  - `useJoopNetworkMonitor()` — reactive `online`, `type`, `effectiveType`, `isMetered` refs; auto-destroys monitor on `onUnmounted`
  - `useJoopAppLifecycle()` — reactive `state` and `isActive` computed refs; auto-destroys listener on `onUnmounted`
  - `useJoopKeyVault(key, config?)` — AES-GCM vault `value`/`loading` refs + `set()`, `remove()`, `refresh()`
  - `useJoopAuth()` — `isLoggedIn` ref reactive to `isLoggedIn$()`, plus `login()` → `setLoggedIn(true)` and `logout()`
  - `useJoopTheme()` — `active` ref + `themes` computed array + `setTheme(name)`
  - `useJoopI18n()` — `language`/`direction` refs reactive to `language$()`, plus `t()` and `setLanguage()`
  - `useJoopLoader()` — `isLoading` ref + `show()` / `hide()`
  - `useJoopConfig<T>(key)` — computed ref returning typed config value
  - `useJoopFeatureFlag(key, ctx?)` — `enabled` and `variant` computed refs
  - `useJoopStore(selector)` — selector-based reactive ref; auto-unsubscribes on `onUnmounted`
- `package.json` `peerDependencies`: `vue >=3.0.0` (optional).
- `package.json` exports: `./vue` sub-path entry.
- `tsup.config.ts`: added `vue/index` entry point; added `vue` to externals.

### Tests added

| File | Tests |
|---|---|
| `tests/vue-composables.test.ts` | 39 (factory, provide/inject, auth, theme, i18n, loader, config, flags, store, network, lifecycle, key vault) |

---

## [1.9.0] — Banking Depth

**4 new services · 100 new tests · 1041 tests total · 89 test files**

### Added

- **`JoopFraudDetection`** (`joopjs/banking`) — 5-signal fraud scoring engine with velocity count, velocity amount, large-amount, geo-distance, and time-anomaly evaluators. Reactive `result$` observable and `onAlert()` handler registration. Score 0-100 → level (low/medium/high/critical) → recommendation (allow/review/challenge/block). Inline Haversine distance for geo scoring with no external deps.
- **`JoopOpenBankingClient`** (`joopjs/banking`) — PSD2/OBIE-spec Open Banking client. OAuth2 client-credentials with token caching. Endpoints: `getAccounts()`, `getTransactions()`, `createConsent()`, `getConsentStatus()`, `revokeConsent()`, `initiatePayment()`. FAPI headers: `x-fapi-interaction-id`, `x-fapi-financial-id`, `x-ob-consent-id`. Requires config via `createJoop`.
- **`JoopPaymentOrchestrator`** (`joopjs/banking`) — 8-state 3DS payment session lifecycle (`idle → initiating → awaiting_3ds → processing_3ds → authorised → settled → failed → cancelled`). Per-session `JoopBehaviorSubject` for reactive UI binding via `session$(sessionId)`. Card PAN masked to last-4 immediately after `initiate()`. AbortController per request with configurable `timeoutMs`. Requires config via `createJoop`.
- **`JoopStatementParser`** (`joopjs/banking`) — Auto-detect parser for 4 statement formats. MT940: regex-based field extraction (`:20:/:25:/:60F:/:61:/:86:`). OFX/QFX: SGML `<STMTTRN>` block extraction. QIF: line-by-line state machine (D/T/P/M/N/^). CSV: configurable column mapping, auto date-format detection, European number format support, RFC 4180-compliant quoted field parsing. All parsers produce stable deterministic transaction IDs.
- `JoopInstance.fraud` — `JoopFraudDetection` instance available on every `createJoop()` call.
- `JoopInstance.statementParser` — `JoopStatementParser` instance available on every `createJoop()` call.
- `JoopInstance.openBanking` — `JoopOpenBankingClient | null` (null until configured).
- `JoopInstance.payment` — `JoopPaymentOrchestrator | null` (null until configured).

### Tests added

| File | Tests |
|---|---|
| `tests/fraud-detection.test.ts` | 19 |
| `tests/open-banking.test.ts` | 15 |
| `tests/payment-orchestrator.test.ts` | 18 |
| `tests/statement-parser.test.ts` | 48 (MT940, OFX, QIF, CSV, edge cases) |

---

## [1.8.0] — Framework Bindings

**0 new core services · 25 new tests · 941 tests total · 85 test files · 2 new sub-paths**

### Added

- **`joopjs/react`** sub-path — Zero-dependency React integration using factory pattern `createJoopReact(React)`. Avoids any direct `import from 'react'` so the package stays framework-agnostic. Ships 12 hooks:
  - `JoopProvider` — React context provider
  - `useJoop` — access full `JoopInstance` inside the tree
  - `useJoopNetworkMonitor` — reactive online/offline + network-type state
  - `useJoopAppLifecycle` — reactive foreground/background state
  - `useJoopKeyVault` — AES-GCM vault get/set/remove with loading state
  - `useJoopAuth` — login state reactive subscription
  - `useJoopTheme` — active theme name + `setTheme()`
  - `useJoopI18n` — language + direction + `t()` + `setLanguage()`
  - `useJoopLoader` — loading state + `show()` / `hide()`
  - `useJoopConfig` — typed config value
  - `useJoopFeatureFlag` — enabled + variant
  - `useJoopStore` — selector-based store subscription
- **`joopjs/angular`** sub-path — Angular 17+ integration without importing `@angular/core`. String-based injection tokens (14 total: `JOOP`, `JOOP_AUTH`, `JOOP_THEME`, `JOOP_I18N`, `JOOP_HTTP`, `JOOP_LOADER`, `JOOP_ALERT`, `JOOP_ROUTER`, `JOOP_ANALYTICS`, `JOOP_FEATURE_FLAGS`, `JOOP_AUDIT_LOG`, `JOOP_STORE`, `JOOP_KEY_VAULT`, `JOOP_NETWORK`, `JOOP_LIFECYCLE`). Factory functions: `provideJoop(instance)` for standalone apps, `JoopModule.forRoot()` for NgModule, `fromJoop()` / `toJoop()` for RxJS bridging, `createZonedFromJoop()` for `NgZone` change detection.
- `package.json` `peerDependencies`: `react >=18`, `@angular/core >=17`, `rxjs >=7` (all optional).
- `package.json` exports: `./react` and `./angular` sub-path entries.
- `tsup.config.ts`: added `react/index` and `angular/index` entry points; added `react`, `react-dom`, `@angular/core`, `@angular/common`, `rxjs` to externals.

### Tests added

| File | Tests |
|---|---|
| `tests/react-hooks.test.ts` | 11 |
| `tests/angular.test.ts` | 14 |

---

## [1.7.0] — Mobile Core Services

**5 new services · 48 new tests · 916 tests total · 83 test files**

### Added

#### `joopjs/device` — Mobile Core

- **`JoopKeyVault`** — Secure key-value store with optional AES-GCM encryption (PBKDF2, 100k iterations, SHA-256). Without an `encryptionKey` stores raw plaintext; with one, every value is stored as base64-encoded `IV ‖ ciphertext`. Methods: `set / get / remove / clear / has / rotateKey`. `rotateKey(newKey, keys)` re-encrypts a set of keys atomically — useful for key rotation without data loss. Accepts a `JoopKeyVaultAdapter` injection for React Native Keychain or MMKV-encrypted storage.

- **`JoopBiometricAuth`** — WebAuthn/FIDO2 biometric authentication for web; adapter delegation for React Native. `enroll(userId, displayName)` creates a platform passkey via `navigator.credentials.create()` with ES256 + RS256 pubKeyCredParams and `residentKey: 'preferred'`. `authenticate(options?)` calls `navigator.credentials.get()` and returns `true/false` — never throws. `isAvailable()` wraps `PublicKeyCredential.isUserVerifyingPlatformAuthenticatorAvailable()`. `canAuthenticate()` provides a quick availability check without prompting. Accepts a `JoopBiometricAdapter` (e.g. `react-native-biometrics`).

- **`JoopPushService`** — Web Push + local notification service. `requestPermission()` wraps `Notification.requestPermission()`. `subscribe(options)` registers with a service worker's `PushManager` using a VAPID key and returns `{ endpoint, p256dh, auth }` ready to send to your server. `show(payload)` fires a local `Notification`. `setBadgeCount(n)` / `clearBadge()` wrap the Badging API. `onMessage(handler)` for foreground push payloads — returns unsubscribe function. `destroy()` removes all handlers and adapter subscriptions. Accepts a `JoopPushAdapter` (e.g. FCM/APNs wrappers for React Native).

- **`JoopAppLifecycle`** — App state machine (`'active' | 'background' | 'inactive' | 'unknown'`) driven by `document.visibilitychange` + `window.focus/blur` on web. `state$()` reactive observable; `getState()` synchronous snapshot; `isActive()` boolean predicate. `onForeground(fn)` fires each time the app returns from background; `onBackground(fn)` fires each time it leaves foreground. `destroy()` removes all DOM event listeners. Accepts a `JoopLifecycleAdapter` (e.g. React Native `AppState` wrapper).

- **`JoopNetworkMonitor`** — Connectivity and network-type monitor. `isOnline()` reflects `navigator.onLine`. `getState()` returns `{ online, type, effectiveType, downlink, rtt, saveData, metered }` — reads `NetworkInformation` API when available. `isMetered()` returns `true` for cellular connections (or when `metered` flag is set). `onOnline(fn)` / `onOffline(fn)` / `onTypeChange(fn)` reactive callbacks — each returns an unsubscribe function. `destroy()` removes all listeners. Accepts a `JoopNetworkAdapter` (e.g. React Native NetInfo).

#### Platform adapter interfaces (added to `joopjs/platform`)

| Interface | Methods |
|---|---|
| `JoopKeyVaultAdapter` | `get(key) / set(key, val) / remove(key) / clear()` — all async |
| `JoopBiometricAdapter` | `isAvailable() / authenticate(reason?)` |
| `JoopPushAdapter` | `requestPermission() / getToken(serverKey?) / onMessage(handler) / setBadgeCount?(count)` |
| `JoopLifecycleAdapter` | `getState() / onChange(handler)` — handler receives `JoopAppState` |
| `JoopNetworkAdapter` | `getState() / onChange(handler)` — handler receives `JoopNetworkState` |

### React Native quick start (v1.7.0 additions)

```ts
import { JoopPlatform } from 'joopjs/platform';
import ReactNativeBiometrics from 'react-native-biometrics';
import NetInfo from '@react-native-community/netinfo';
import { AppState } from 'react-native';

JoopPlatform.init({
  adapter: {
    biometric: {
      isAvailable: async () => {
        const { available } = await new ReactNativeBiometrics().isSensorAvailable();
        return available;
      },
      authenticate: async (reason = 'Confirm identity') => {
        const { success } = await new ReactNativeBiometrics()
          .simplePrompt({ promptMessage: reason });
        return success;
      },
    },
    lifecycle: {
      getState: () => {
        const s = AppState.currentState;
        return s === 'active' ? 'active' : s === 'background' ? 'background' : 'inactive';
      },
      onChange: (fn) => {
        const sub = AppState.addEventListener('change', s =>
          fn(s === 'active' ? 'active' : s === 'background' ? 'background' : 'inactive'));
        return () => sub.remove();
      },
    },
    network: {
      getState: () => ({ online: true, type: 'unknown' }),
      onChange: (fn) => NetInfo.addEventListener(state =>
        fn({ online: !!state.isConnected, type: state.type as 'wifi' | 'cellular' })),
    },
  },
});
```

---

## [1.6.0] — Platform Foundation

**1 new module · 15 new tests · 868 tests total**

### Added

#### `joopjs/platform` — Platform Abstraction Layer (new sub-path)

- **`JoopPlatform`** — Central platform registry and adapter injection point. `JoopPlatform.init({ adapter })` is called once at app startup to register platform-specific implementations. Auto-detects `type: 'web' | 'react-native' | 'node' | 'unknown'` via `navigator.product`, `process.versions.node`, and `window` presence. Predicates: `isWeb()`, `isMobile()`, `isNode()`. `capabilities()` returns a snapshot of available APIs: `compression`, `workers`, `crypto`, `dom`, `localStorage`, `sessionStorage`, `serviceWorker`. `getStorage()` / `getSessionStorage()` return the best available `JoopStorageAdapter` (custom → native → in-memory fallback). `_reset()` for test teardown.

- **`JoopStorageAdapter`** — `getItem / setItem / removeItem / clear?`. Implemented by `localStorage`, `sessionStorage`, MMKV, and the built-in memory fallback. Used as the standard storage contract throughout the library.

- **`JoopCompressionAdapter`** — `compress(data, format) / decompress(data, format)` returning `Promise<Uint8Array>`. Drop-in for `fflate` or `pako` on React Native.

- **`JoopWorkerAdapter`** — `run<T,R>(fn, arg, timeoutMs?) / terminate?()`. For platforms with custom threading (e.g. `react-native-threads`).

- **`JoopCryptoAdapter`** — `subtle: SubtleCrypto / getRandomValues<T>(array): T`. For environments with non-standard WebCrypto placement.

### Changed (non-breaking)

- **`JoopCompressionService`** — Checks `JoopPlatform.getAdapter().compression` first; falls through to native `CompressionStream`; throws a descriptive error with setup instructions if neither is available. `isSupported()` now also returns `true` when a compression adapter is registered.

- **`JoopWorkerPool`** — Detects `Worker` availability via `_workersAvailable()`. Falls back to main-thread `setTimeout(0)` execution (or a registered `JoopWorkerAdapter`) when Workers are unavailable. `run()` / `map()` / `terminate()` API is unchanged.

- **`JoopOIDCClient`** — `config.storage` widened from `Storage` to `JoopStorageAdapter`. Defaults through `sessionStorage → JoopPlatform.getSessionStorage()`. Added `buildLoginUrl(options)` for React Native (returns a URL string instead of redirecting). `logout()` guards `window.location` assignment to be web-only.

- **`JoopIdempotencyService`** — `config.storage` widened to `JoopStorageAdapter`. Replaced `Storage.length / key()` enumeration with an internal `_keys: Set<string>` — compatible with any adapter. Fixed: `generate()` returning an existing key now registers it in `_keys` so `clearAll()` covers it.

- **`JoopStreamingService`** — Skips native `EventSource` when `JoopPlatform.isMobile()` is true; always uses the fetch-based SSE path on React Native.

### React Native quick start (v1.6.0)

```ts
import { JoopPlatform } from 'joopjs/platform';
import { MMKV } from 'react-native-mmkv';
import { gzip, ungzip } from 'fflate';

JoopPlatform.init({
  adapter: {
    storage: new MMKV(),
    compression: {
      compress:   (d, _fmt) => new Promise((res, rej) => gzip(d,   (e, r) => e ? rej(e) : res(r))),
      decompress: (d, _fmt) => new Promise((res, rej) => ungzip(d, (e, r) => e ? rej(e) : res(r))),
    },
  },
});
```

---

## [1.5.0] — AI-Era Enterprise Pack

**21 new services · 198 new tests · 853 tests total**

### Added

#### `joopjs/ai` — AI / LLM (new sub-path)

- **`JoopAiClient`** — Multi-provider LLM gateway. Supports OpenAI, Anthropic, Gemini, Ollama, and custom providers via a `JoopAiProvider` config. Streaming via SSE fetch with delta accumulation, tool-call parsing, AbortSignal cancellation. Provider-specific request builders keep the API uniform: `chat(messages, options?)`, `stream(messages, options?)`, `complete(prompt, options?)`. Returns `JoopAiUsage` token counts.

- **`JoopToolRegistry`** — Function-calling registry for LLM tool use. `register(name, schema, handler)`, `dispatch(name, args)`. Format converters: `toOpenAiFormat()`, `toAnthropicFormat()`, `toGeminiFormat()` emit provider-ready tool definitions.

- **`JoopStreamingService`** *(moved to `joopjs/network`)* — SSE streaming for both native `EventSource` (GET) and fetch-based POST streams. Parses `id:/event:/data:` fields, dispatches typed `JoopSSEMessage` events, auto-reconnects up to `maxReconnects`.

#### `joopjs/security` additions — Behavioral & Risk

- **`JoopBehavioralBiometrics`** — Bot-detection via passive behavioral telemetry: keystroke intervals + dwell times, mouse speed + acceleration, scroll velocity, touch force. `record()` starts passive collection. `getScore()` uses coefficient-of-variation thresholds to return a `JoopBiometricScore`. `compare(profile)` computes similarity to a saved baseline. `anomaly$()` observable emits z-score deviations. `destroy()` removes all DOM listeners.

- **`JoopRiskEngine`** — Composite risk scorer with 7 weighted built-in factors: `failed_attempts`, `biometrics`, `mfa`, `known_device`, `vpn`, `time_anomaly`, `transaction_amount`. `evaluate(context)` returns `JoopRiskScore` with numeric score (0-100) and `'low' | 'medium' | 'high' | 'critical'` level. `addFactor()` for custom signals. `reset()` clears counters.

#### `joopjs/workers` — Concurrency (new sub-path)

- **`JoopWorkerPool`** — Web Worker thread pool. Serializes any function via `.toString()` into a Blob-URL worker. Round-robin dispatch across configurable `poolSize`. Per-task `timeoutMs` with abort. Automatic drain queue for backpressure. `run<T,R>(fn, arg)` / `map<T,R>(items, fn)` / `terminate()`. Falls back to main-thread execution when Workers are unavailable (see v1.6.0 PAL changes).

- **`JoopCompressionService`** — `CompressionStream`/`DecompressionStream` wrapper for `gzip`, `deflate`, and `deflate-raw`. `compress(data) / decompress(data)` operate on `Uint8Array`. `compressString(text) / decompressString(b64)` work with base64. `ratio(original, compressed)` reports efficiency. `isSupported()` checks platform.

#### `joopjs/state` — Observable Store (new sub-path)

- **`JoopStore<S>`** — Observable state container. `dispatch(type, payload?)` runs the matching reducer. `select(selector)` returns a subscription that only emits on value change. Unlimited undo/redo via `undo()` / `redo()` with internal `_history` / `_future` stacks. `createSlice(name, initialState, reducers)` for namespaced reducer registration. `createStore(config)` factory. `getState()` synchronous snapshot.

#### `joopjs/sync` — Local-First / Offline (new sub-path)

- **`JoopSyncEngine`** — Offline-first mutation queue. `mutate(operation, payload)` enqueues a `JoopMutation` and auto-syncs when online. Batched POST sync with per-result `ok/conflict` processing. Exponential retry up to `maxRetries`. `status$()` observable: `'idle' | 'syncing' | 'error'`. `resolve(id, strategy)` resolves conflicts with `'local' | 'remote' | 'merge'`.

- **`CRDTCounter / CRDTSet / CRDTRegister / CRDTText`** — Four CvRDT implementations. `CRDTCounter`: PN-Counter with per-node increment/decrement maps; `increment(nodeId)` / `decrement(nodeId)` / `value()`. `CRDTSet`: Add-Wins set with `add(item)` / `remove(item)` / `has(item)`. `CRDTRegister<T>`: Last-Write-Wins by `{ timestamp, nodeId }` comparison. `CRDTText`: simplified RGA for collaborative text editing. `mergeCRDT(a, b)` generic helper for all CRDT types.

#### `joopjs/workflow` — State Machine & Workflow (new sub-path)

- **`JoopStateMachine<S,E>`** — XState-lite FSM. `define(config)` with declarative `on` transition maps, optional guard functions, `onEnter`/`onExit` lifecycle hooks, and action callbacks. `send(event)` resolves the transition and fires hooks. `current()` / `state$()`. `toMermaid()` exports a `stateDiagram-v2` diagram string.

- **`JoopWorkflowEngine`** — DAG-based saga orchestrator. `define(name, steps[])` where each step has `run`, optional `after: string[]` dependencies, and `compensate` rollback. Topological parallel execution; timeout wrapping per step. `execute(workflowName, context)` returns `JoopWorkflowRun`. Automatic saga compensation on any step failure.

#### `joopjs/security` additions — Privacy / Zero-Knowledge

- **`JoopPIIScanner`** — Pattern-based PII detection. Built-in patterns: credit cards, SSN, email, US phone, IBAN, passport, IP address, JWT, API keys. `scan(text)` returns `JoopPIIFinding[]` with match indices. `redact(text)` replaces findings in reverse-index order for stability. `pseudonymize(text)` replaces with SHA-256 truncated hashes. Custom `JoopPIIPattern` support.

- **`JoopSecretSharing`** — Shamir's Secret Sharing over the secp256k1 256-bit prime field. `split(secret, n, k)` produces `n` shares; any `k` suffice for reconstruction. `combine(shares)` uses Lagrange interpolation over the prime field. `serializeShare(share)` / `deserializeShare(b64)` for base64 transport. All arithmetic uses BigInt to avoid float precision loss.

- **`JoopDifferentialPrivacy`** — Statistical privacy. `addLaplaceNoise(value, sensitivity, epsilon)` via inverse-CDF sampling. `addGaussianNoise(value, sensitivity, epsilon, delta)` via Box-Muller transform. `privatizeCount(count, sensitivity, epsilon)` clamps result to non-negative integer. `randomizedResponse(truth, epsilon)` probabilistic flip. `computePrivacyBudget(queries, epsilon)` cumulative budget accounting.

- **`JoopWatermarkService`** — Dual-mode watermarking. `apply(el, text, options?)` tiles a canvas-rendered text overlay as a CSS `backgroundImage` on the target element. `embedInImage(imageData, payload)` encodes an arbitrary string into the R-channel LSBs of an `ImageData` object. `extractFromImage(imageData)` recovers the payload. Works entirely in the browser with no external dependencies.

#### `joopjs/observability` additions

- **`JoopWebVitals`** — Core Web Vitals collector via `PerformanceObserver`. Tracks LCP (Largest Contentful Paint), CLS (Cumulative Layout Shift, session-window accumulation), INP (Interaction to Next Paint), FCP (First Contentful Paint), TTFB (Time to First Byte). `getRating(metric, value)` returns `'good' | 'needs-improvement' | 'poor'` against Google thresholds. `onVital(handler)` / `getSnapshot()` / `flush()`.

- **`JoopSessionRecorder`** — DOM session recorder via `MutationObserver` + click/input/scroll/resize/popstate listeners. Auto-redacts `type=password` and `[data-pii]` inputs. `start()` / `stop()` / `pause()` / `resume()` flow control. `maxEvents` cap. `addEvent(type, data)` for custom instrumentation. `export('json' | 'compressed')` returns a serialized `JoopSessionRecord`.

#### `joopjs/auth` additions

- **`JoopOIDCClient`** — Full OpenID Connect client with PKCE. `discover(issuerUrl)` fetches `.well-known/openid-configuration`. `login(options?)` generates a SHA-256 code challenge, persists state, and redirects to the authorization endpoint. `handleCallback()` verifies state, exchanges the code for tokens, and schedules silent renewal. `silentRenew()` refreshes tokens before expiry. `buildLoginUrl(options)` for React Native in-app browser flows (v1.6.0 extension). `logout()` calls the end-session endpoint on web.

#### `joopjs/api` additions

- **`JoopWebhookVerifier`** — HMAC-SHA256 webhook signature verification. Provider-aware: Stripe (`t=,v1=` header format), GitHub (`sha256=` prefix), generic (raw signature). Timing-safe XOR comparison. `sign(payload, secret)` for outgoing webhooks. `verify(request, secret, options?)` returns `true/false`.

- **`JoopIdempotencyService`** — Client-side idempotency key management. `generate(opKey)` returns a live key or creates a new UUID-based one with TTL. `wrap(opKey, fn)` executes `fn(key)` and auto-clears on success (retains on failure for retry). `get(opKey)` / `clear(opKey)` / `clearAll()` / `gc()`. Keys survive page reload via `JoopStorageAdapter`.

### Bug fixes

- **`JoopSecretSharing`** — Fixed `_modMul` called with two arguments instead of three (missing `PRIME`), causing `"Cannot mix BigInt and other types"` on share recovery.
- **`JoopSyncEngine`** tests — `sync()` was guarded by `!this._online`; tests now explicitly set `_online` state before calling `engine.sync()`.
- **`JoopBehavioralBiometrics`** — Event handler signatures (`MouseEvent`, `TouchEvent`) cast to `EventListener` for TypeScript 5.9 strict DTS compatibility.

---

## [1.4.0] — Platform Services Pack

**16 new services · 160 new tests · 655 tests total**

### Added

#### `joopjs/storage` addition

- **`JoopIndexedDbService`** — Promise-based IndexedDB wrapper. `open(config)` creates stores with optional indexes. CRUD: `get<T>(store, key)`, `set(store, key, value)`, `delete(store, key)`, `getAll<T>(store)`, `getAllKeys(store)`, `bulkSet(store, entries)`. `query(store, indexName, range)` for indexed range queries. `count(store)` / `clear(store)` / `deleteDatabase()`. Full TypeScript generics throughout.

#### `joopjs/encryption` addition

- **`JoopCryptoUtils`** — Standalone WebCrypto utilities (also re-exported from `joopjs/utilities`). SHA-256/512/1 digest. HMAC-SHA256/512 with timing-safe `verify()`. PBKDF2 key derivation. `hashPassword(password, iterations?)` / `verifyPassword(password, hash)`. `randomBytes(n)` / `uuid()`. `hexToBytes` / `bytesToHex` / `bytesToBase64` / `base64ToBytes`. AES-GCM `deriveKey(password)`.

#### `joopjs/forms` — Form Builder (new sub-path)

- **`JoopFormBuilder`** — Reactive form state management without framework coupling. **`JoopField<T>`** tracks: `value`, `dirty`, `touched`, `valid`, `errors[]` — integrated with any `JoopValidationRule[]`. **`JoopForm`** aggregates multiple fields and exposes: `valid`, `dirty`, `getValue()`, `errors()`, `reset()`, `markAllTouched()`, `state$()` observable. `build(config)` factory.

#### `joopjs/core` additions

- **`JoopRateLimiter`** — `throttle(fn, ms)`, `debounce(fn, ms, immediate?)`, `debounceAsync(fn, ms)`, `once(key, fn)` per-key guard. Named bucket registry via `getBucket(name)`.

- **`JoopTokenBucket`** — Standalone token bucket: configurable `capacity` + `refillRate`. `consume(tokens?)` / `tryConsume(tokens?)` (waits for availability via polling). `available()` / `reset()`.

- **`JoopConsentService`** — GDPR/PDPA cookie consent. Five categories: `necessary / analytics / marketing / functional / personalization`. `grant(category)` / `revoke(category)` / `acceptAll()` / `rejectAll()`. `localStorage` persistence. `state$()` observable. `onChange(category, handler)` per-category callbacks.

#### `joopjs/session` addition

- **`JoopIdleService`** — Two-tier user idle detection. Listens to configurable DOM events (`mousemove`, `keydown`, `scroll`, `touchstart`). `idleMs` triggers `'idle'`; `expiredMs` triggers `'expired'`. `state$()` observable. `onIdle()` / `onActive()` / `onExpired()` callbacks. `reset()` restarts the timer.

#### `joopjs/utilities` additions

- **`JoopBarcodeService`** — Pure-SVG barcode generators. **Code128B** (full ASCII, mod-103 checksum). **EAN-13** (L/G/R parity encoding, check-digit validation). **Code39** (alphanumeric). `generateBarcodeDataUrl(text, type, options?)` for inline use.

- **`JoopPdfService`** — Minimal PDF 1.4 writer, zero dependencies. Text placement `text(x, y, content, options?)`. Line drawing `line(x1, y1, x2, y2)`. Filled/stroked rectangles. Auto-zebra tables with colored headers. Convenience builders: `receipt(lines)` / `statement(data)`. Output: `toDataUrl()` / `download(filename)`.

- **`JoopNotificationService`** — Web Push API wrapper: `requestPermission()`, `show(options)` (browser notification or SW notification), VAPID `subscribe(vapidKey)` / `unsubscribe()`, `onMessage(handler)` from service worker, `permission$()` observable.

- **`JoopGeoService`** — Geolocation: `getPosition(options?)` / `watchPosition(handler, options?)`. Haversine `distanceBetween(a, b)`. `isInRadius(center, point, radiusKm)`. Compass `bearing(from, to)`. `toDms(coord)` / `toDecimal(dms)`. `getPermissionState()`.

- **`JoopMediaService`** — Camera/microphone/screen access. `requestCamera()` / `requestMicrophone()` / `requestScreenShare()`. `capturePhoto(stream)` → `Blob`. `createRecorder(stream, options)` → `JoopMediaRecorder` with `start()` / `stop()` / `pause()` / `getBlob()`. `getDevices()` / `checkPermission(type)` / `attachStream(el, stream)`.

- **Chart data utilities** — `bucketByTime(data, interval)` (hourly/daily/weekly/monthly/yearly). `rollingAverage(data, window)`. `normalize(data)`. `cumulative(data)`. `movingSum(data, window)`. `aggregateByCategory(data, key)`. `percentChange(data)`. `detectTrend(data)` via linear regression. `sparklinePoints(data)` / `sparklineSvg(data, options)`.

#### `joopjs/ui` additions

- **`JoopA11yService`** — Accessibility utilities. `announce(message, politeness?)` uses ARIA live regions. `trapFocus(el, options?)` constrains Tab navigation; `releaseFocus(el)` / `releaseAllTraps()`. `createSkipLink(target, label?)`. `setPageTitle(title)`. `getFocusable(el)` returns all focusable descendants. `visuallyHide(el)` / `visuallyShow(el)`. `onFocusChange(handler)`.

- **`JoopVirtualScroll<T>`** — Virtualized list. Only renders the visible window + configurable `overscan` items. Top/bottom `JoopBehaviorSubject`-driven spacer elements maintain scroll height. `updateItems(items)` / `scrollTo(index)` / `getVisibleRange()` / `onRangeChange(handler)`.

#### `joopjs/pwa` — PWA (new sub-path)

- **`JoopServiceWorkerService`** — Service worker lifecycle: `register(url, options?)` / `unregister()`. Update detection via `onUpdateAvailable(handler)` (fires when a waiting worker is found). `postMessage(data)` / `onMessage(handler)` for SW communication. `onlineState$()` reflects `navigator.onLine`. `state$()` tracks `'unregistered' | 'installing' | 'installed' | 'activated'`.

#### `joopjs/router` — SPA Router (new sub-path)

- **`JoopRouterService`** — History API SPA routing. `navigate(path, state?)` / `replace(path)` / `back()` / `forward()` / `go(delta)`. Async guards with abort support: `addGuard(guard)`. `match(pattern)` with `:param` extraction and wildcard support. `buildUrl(pattern, params, query?)`. `parseUrl(url)` → `{ path, params, query, hash }`. `route$()` observable.

### Changed

- `JoopInstance` extended with 13 new service fields: `indexedDb`, `crypto`, `formBuilder`, `rateLimiter`, `idle`, `consent`, `pdf`, `notification`, `a11y`, `router`, `sw`, `geo`, `media`.
- `package.json` — 3 new sub-path exports: `./forms`, `./pwa`, `./router`.

---

## [1.3.0] — Enterprise Feature Pack

**15 new services · 143 new tests · 495 tests total**

### Added

#### `joopjs/cache` — TTL Cache (new sub-path)

- **`JoopCacheService`** — TTL in-memory cache with LRU eviction. `set<T>(key, value, ttlMs?)` / `get<T>(key)` (null on expiry, auto-deletes on read). `getOrSet<T>(key, factory, ttlMs?)` — compute-and-cache, factory called once. `has(key)` checks expiry. `prune()` removes expired entries (returns count). `ttl(key)` remaining ms or -1. `extend(key, ttlMs)`. `stats()` → `{ size, maxSize, totalHits }`. LRU eviction by `createdAt` when `maxSize` is reached.

#### `joopjs/network` — WebSocket (new sub-path)

- **`JoopWebSocketService`** — Auto-reconnect WebSocket with message queue. `connect(url, protocols?)` / `disconnect()`. `send(data: string | object)` — JSON-serialises objects; queues while disconnected. `on(type)` — filtered observable per message type. `state$()` → `'connecting' | 'open' | 'closing' | 'closed'`. `message$()` — all incoming messages as `JoopWsMessage`. Auto-reconnect up to `maxReconnects` with `reconnectMs` delay. Optional `pingIntervalMs` keep-alive.

#### `joopjs/analytics` — Analytics (new sub-path)

- **`JoopAnalyticsService`** — Adapter-pattern event pipeline. `addAdapter(adapter)` / `removeAdapter(name)`. `track(name, props?)` — emits to all adapters, queues events before first adapter. `page(name, props?)` — calls adapter `page()` and auto-tracks `page:<name>`. `identify(userId, traits?)`. `flush()` drains pre-adapter queue. `setContext(ctx)` merges into all event properties. `events$()` observable. **`consoleAnalyticsAdapter`** — built-in dev adapter.

#### `joopjs/validation` — Validation (new sub-path)

- **`JoopFormValidator`** — Rule-based form validation, no dependencies. Individual validators: `required`, `email`, `phone` (E.164), `minLength`, `maxLength`, `min`, `max`, `amount`, `iban`, `luhn`, `pattern`, `dateRange`. Each returns `string | null` (error or valid). `validate(value, rules)` → `{ valid, errors[] }`. `validateForm(form, schema)` → `Record<field, errors[]>`. `passwordStrength(password)` → `{ score: 0-4, label, suggestions[] }`.

#### `joopjs/core` additions

- **`JoopFeatureFlagService`** — Client-side feature flags with targeting. `define(flags)` / `loadFromUrl(url)`. `isEnabled(key, context?)` evaluates: `environments[]`, `allowedUsers[]`, `rolloutPercent` (deterministic FNV-1a hash). `setOverride(key, value)` / `clearOverrides()`. `getVariant(key, context?)`. `flags$()` observable.

#### `joopjs/auth` additions

- **`JoopMfaService`** — Multi-factor authentication orchestration.
  - **TOTP**: `generateTotpSecret(length?)`, `getTotpUri(secret, account, issuer?)`, `setupTotp(account, issuer?)` → `{ secret, uri, backupCodes }`, `verifyTotp(token, secret, window?)`.
  - **OTP Challenges**: `createChallenge(dest, channel, ttlMs?, maxAttempts?)` → challenge with `_code` for delivery, `verifyChallenge(id, token)` → `{ success, reason, attemptsLeft }`.
  - **Backup Codes**: `generateBackupCodes(count?, length?)`, `hashBackupCodes(codes)` (SHA-256), `verifyBackupCode(code, hashedCodes)` → `{ valid, remaining[] }`.
  - **Biometric**: `isBiometricAvailable()`, `registerBiometric(userId, displayName?)`, `verifyBiometric(credentialId)`.

#### `joopjs/encryption` addition

- **`JoopE2EService`** — RSA + AES-GCM hybrid end-to-end encryption.
  - Key gen: `generateKeyPair(bits?)` (RSA-OAEP) / `generateSigningKeyPair(bits?)` (RSA-PSS).
  - PEM I/O: `export/importPublicKeyPem`, `export/importPrivateKeyPem`. JWK I/O: `export/importPublicKeyJwk`.
  - Hybrid: `encryptForRecipient(plaintext, publicKey)` → `JoopE2EPayload` (RSA-OAEP wraps ephemeral AES-GCM key). `decryptFromSender(payload, privateKey)`. Handles arbitrary payload sizes.
  - Signatures (RSA-PSS, saltLength=32): `sign(data, privateKey)` → base64, `verify(data, sig, publicKey)` → boolean.
  - Session keys: `generateSessionKey()` / `wrapSessionKey(key, recipientPk)` / `unwrapSessionKey(wrapped, sk)`.
  - Direct RSA (< 190 bytes): `encryptRaw` / `decryptRaw`. Fingerprint: `fingerprint(publicKey)` → SHA-256 hex.

#### `joopjs/ui` additions

- **`JoopPaginationService`** — Page state manager. `setTotal(n)` / `setPage(n)` / `setPageSize(n)` — all clamp and recalculate. `nextPage()` / `prevPage()` / `firstPage()` / `lastPage()`. `getState()` → `{ page, pageSize, total, totalPages, hasPrev, hasNext, from, to }`. `getPages(maxVisible?)` → number array with ellipsis as `-1`. `state$()` observable.

- **`JoopPrintService`** — Browser print utilities. `printElement(el, title?, styles?)` opens a print window with cloned HTML. `printHtml(html, title?)`. `printReceipt(data)` — monospace receipt with padded label-value columns. `buildReceiptText(lines)` for preview without printing.

#### `joopjs/api` additions

- **`JoopExportService`** — CSV/JSON/text file downloads. `toCsv(data, options?)` — headers, delimiter, BOM, null-value config. Quote-escaping doubles `"` inside values. `tableToCsv(rows, headers?)`. `downloadCsv / downloadExcel / downloadJson / downloadText`.

#### `joopjs/banking` additions

- **Luhn algorithm + card utilities** — `validateLuhn(cardNumber)`, `luhnCheckDigit(partial)`, `getCardNetwork(cardNumber)` → `visa | mastercard | amex | discover | unionpay | jcb | diners | maestro | unknown`, `getCardInfo(cardNumber)` → `{ network, valid, lengths[], cvvLength, formatted }`, `formatCardNumber(cardNumber)`, `isCardExpired(month, year)`, `validateCvv(cvv, network?)`.

#### `joopjs/utilities` additions

- **`JoopClipboardService`** + `clipboard` singleton — `copy(text)` (`navigator.clipboard` → `execCommand` fallback), `paste()`, `copyElement(el)`, `events$()` observable.

- **`JoopImageService`** — Canvas-based image utilities: `compress(file, options?)` (JPEG/WebP), `resize(file, w, h)`, `toBase64(file)`, `getInfo(file)` → `{ width, height, size, type }`, `validate(file, options)` → `{ valid, errors[] }`, `thumbnail(file, size?)`.

- **Date & Hijri utilities** — `isBusinessDay`, `addBusinessDays`, `getBusinessDaysBetween` (configurable weekends — Fri/Sat for Gulf). `toHijri(date)` / `fromHijri(y, m, d)` via Julian Day Number. `formatHijri`, `formatDate(date, format, locale?)`, `timeAgo(date, locale?)`, `parseDate`, `daysBetween`, `isDateInRange`.

- **QR code generator** (`generateQr`, `generateQrDataUrl`, `generatePaymentQr`) — ISO 18004 compliant, versions 1-10. EC levels `L | M | Q | H`. Full Reed-Solomon over GF(256). All 8 mask patterns evaluated by penalty score. Finder/alignment/timing patterns, dark module, format info. Pure SVG output, no dependencies. `generatePaymentQr(amount, currency, reference, options?)`.

### `JoopInstance` new fields (v1.3.0)

`cache`, `ws`, `analytics`, `featureFlags`, `mfa`, `e2e`, `formValidator`, `pagination`, `print`, `exporter`, `clipboard`, `image`

---

## [1.2.0] — Framework-Agnostic Services Pack

**9 new services · 94 new tests · 352 tests total**

Expanded the SDK with theme, i18n, UI, and session services rewritten as framework-agnostic TypeScript — no Angular or RxJS dependencies.

### Added

#### `joopjs/theme` — Theme Engine (new sub-path)

- **`JoopThemeService`** — CSS custom-property theme engine. `load(themesOrUrl)` registers theme definitions from an inline array or remote JSON. `apply(theme | name)` / `applyById(id)` / `applyDefault()` switch themes at runtime. Each palette generates CSS variables: `--joop-pr-500: #1976d2`, `--joop-pr-c-500: #ffffff` (contrast). `applyVars(vars)` injects arbitrary CSS custom properties. `getMapped()` → `JoopThemeInfo[]` for theme-picker UIs. `active$()` / `activeId$()` observables.

#### `joopjs/i18n` — Internationalisation (new sub-path)

- **`JoopI18nService`** — Language, culture, and RTL management. Auto-detects RTL for 14 languages: `ar`, `arc`, `ckb`, `dv`, `fa`, `ha`, `he`, `khw`, `ks`, `ps`, `sd`, `ur`, `uz_AF`, `yi`. `setLanguage(lang)` / `setLanguageCulture(culture)`. `loadFromUrl(lang, url)` / `loadFromObject(lang, obj)` — repeated calls merge translations. `t(key, params?)` — dot-notation key resolution (`'ERRORS.TITLE'`) with `{{ param }}` interpolation; falls back to fallback language then raw key. `isRtl()` / `getDirection()` / `applyDirection(el?)`. `language$()` / `culture$()` observables.

#### `joopjs/ui` — UI Services (new sub-path)

- **`JoopLoaderService`** — Request-counter loading state. `show()` / `hide()` increment/decrement a counter: loader shows when `0 → 1`, hides when it returns to `0`. No flicker on concurrent requests. `forceShow()` / `reset()`. Optional `delayMs` to suppress brief-request spinners. `isLoading()` / `isLoading$()` / `pendingCount`.

- **`JoopAlertService`** — Pure-DOM toast and dialog system, zero framework dependency. Toasts: `toast(message, options?)` — four types (`success / error / warning / info`), four positions, auto-dismiss, optional close button. Dialogs: `dialog(options)` → `Promise<string>` (button action). Inline alerts: `showInline(data)` / `hideInline()` / `inline$()`. `open$()` / `dismiss()`.

#### `joopjs/native-bridge` — Native Bridge (new sub-path)

- **`JoopNativeBridgeService`** — Bidirectional native-WebView message bridge. `sendToNative(target, method, data)` auto-detects: iOS `webkit.messageHandlers`, Android `window[target][method]`, legacy `document.location.href` scheme. `dispatch(message)` pushes to JS-side subscribers. `onMessage(type)` filtered subscription. `startListening(allowedOrigins?)` registers `window.message` with optional origin allowlist. `allMessages$()` / `destroy()`.

#### `joopjs/deeplink` — Deeplink (new sub-path)

- **`JoopDeeplinkService`** — Cross-tab deeplink coordination via `localStorage` storage events. `setModule(module, data?)` writes and immediately removes the payload (so the event fires reliably), emits `close$(true)`. `startListening(key?)` installs a storage event handler. `requestClose()` emits `close$(false)`. `readIncoming(key?)` reads a payload left by another tab on page load. `close$()` / `incoming$()` / `module$()` observables.

#### `joopjs/api` additions

- **`JoopFileSaverService`** — File download utility. `download(url, options?)` fetches, extracts filename from `Content-Disposition`, routes to iOS WebView path (`webkit.messageHandlers` base64 postMessage) or standard browser path (`<a>` + `URL.createObjectURL`). `saveBlob(blob, fileName)` / `blobToBase64(blob)` / `getMimeType(fileName)` / `isIosWebView(handlerName?)`.

- **`JoopResponseMapper`** — Normalizes legacy v6 `compDet[]` and current v7+ `components{}` API responses into `JoopMappedResponse`. `map(raw)` auto-detects format. Extracts `scope` (first char: `M/O/N/S`) and `dataKey`. `buildScopedMap(response)` groups by scope prefix. `getComponent(response, key)`. `hasError` flag.

#### `joopjs/utilities` additions

- `flattenObj(obj)` / `buildQueryString(params)` / `buildFormData(params, files?)` / `urlEncode(params)` / `getRandomNum(min, max)`

### `JoopInstance` new fields (v1.2.0)

`theme`, `i18n`, `loader`, `alert`, `nativeBridge`, `deeplink`, `fileSaver`, `responseMapper`

---

## [1.1.0] — Enterprise & Banking Pack

**22 new services · 134 new tests · 258 tests total**

### Added

#### `joopjs/security` — Secure Storage (new sub-path)

- **`JoopSecureStorage`** — AES-GCM encrypted wrapper for `localStorage`, `sessionStorage`, or in-memory storage. Transparently encrypts all values at rest. Returns `null` on decryption failure. `local / session / memory` storage type config. `getItem / setItem / removeItem / clear`.

- **Sanitizer utilities** — `escapeHtml`, `stripHtml`, `sanitizeInput`, `stripSqlKeywords`, `sanitizeFileName`, `isValidUrl`, `isValidEmail`, `isValidPhone`, `isAlphanumeric`, `truncate`. XSS and injection prevention at all user-input boundaries.

#### `joopjs/auth` additions

- **`JoopTokenService`** — JWT access/refresh token management. Decodes JWT payloads (without verification). `isExpired(token)`. Auto-schedules refresh 30s before expiry via `onRefreshNeeded` callback. `setTokens(access, refresh)` / `getAccessToken()` / `getRefreshToken()` / `clearTokens()`.

- **`JoopPKCEService`** — RFC 7636 PKCE OAuth2 helper. `generateCodeVerifier()` — 64-byte crypto-random, base64url-encoded. `generateCodeChallenge(verifier)` — SHA-256, base64url-encoded. `generateState()`. `buildAuthUrl(config)` — complete authorization URL with challenge.

- **`JoopOtpService`** — TOTP (RFC 6238 / HMAC-SHA-1) and one-time code generation. `generateTotp(secret, time?)` / `verifyTotp(token, secret, window?)`. `generateNumericOtp(length?)` / `generateAlphanumericOtp(length?)` for SMS/email flows.

- **`JoopBiometricService`** — WebAuthn/FIDO2 platform authenticator wrapper. `register(userId, displayName, rpId?)` / `authenticate(credentialId, rpId?)` / `isAvailable()` / `generateChallenge()` / `credentialIdToBase64(id)`.

#### `joopjs/api` additions

- **`JoopInterceptorPipeline`** — Request/response/error interceptor pipeline. `addRequestInterceptor(fn)` / `addResponseInterceptor(fn)` / `addErrorInterceptor(fn)` — each returns an unsubscribe function. `executeRequest(req)` / `executeResponse(res)` / `executeError(err)` runs the chain.

- **`withRetry(fn, config)`** — Exponential backoff retry. `maxAttempts`, `baseDelayMs`, `maxDelayMs`, `jitter` (multiplicative), `retryOn(err)` predicate to skip non-retriable errors.

- **`JoopCircuitBreaker`** — Three-state circuit breaker. `closed` (normal) → `open` (blocking after `failureThreshold`) → `half-open` (recovery probe after `timeoutMs`). `execute(fn)` / `reset()` / `getState()`.

- **`JoopOfflineQueue`** — Queues requests when offline, drains on `window.online`. `enqueue(fn)` / `drainNow()` / `clear()` / `size`.

- **`JoopPollingService`** — Interval-based polling. `start(name, fn, intervalMs, options?)` — `immediate` first-run, `stopWhen(result)` predicate, `onError(err)` handler. `stop(name)` / `stopAll()`. `result$(name)` observable.

#### `joopjs/session` additions

- **`JoopMultiTabSync`** — `BroadcastChannel`-based cross-tab event bus. `broadcast(type, payload?)` / `on(type, handler)`. Built-in types: `logout`, `login`, `session-warning`, `token-refresh`. Filters own messages automatically.

- **`JoopConcurrentSessionService`** — Heartbeat-based duplicate session detection. Broadcasts a session ID at `heartbeatMs` intervals. `onDuplicateDetected(handler)` / `start()` / `stop()`.

#### `joopjs/observability` — Observability (new sub-path)

- **`JoopCorrelationService`** — Generates and tracks `X-Correlation-ID` + `X-Request-ID`. `headers()` → `Record<string, string>`. `newCorrelationId()` / `currentCorrelationId()`.

- **`JoopAuditLog`** — Bounded, immutable audit trail. Records: login, logout, transfer, profile-change, biometric, custom. `addEntry(action, details?)` / `getEntries(filter?)` / `exportJson()` / `onEntry(handler)`. Configurable `maxEntries`.

- **`JoopPerformanceService`** — `mark(name)` / `measure(name, start, end?)` pair. `timeAsync(name, fn)` / `timeSync(name, fn)` wrappers. `getAverage(name)` for baselines. `getMark(name)` / `getAll()`.

- **`JoopErrorReporter`** — Pluggable error capture. `addHandler(fn)` / `report(error, context?)`. Enriches each `JoopErrorReport` with `userId`, `sessionId`, structured context. `clearHandlers()`.

#### `joopjs/device` — Device (new sub-path)

- **`JoopDeviceFingerprintService`** — SHA-256 fingerprint from user agent, language, timezone, screen resolution, hardware concurrency, and touch points. `fingerprint()` (async). `getStableId()` caches in `localStorage` for consistent cross-session identification.

#### `joopjs/dev` — Developer Tools (new sub-path)

- **`JoopMockService`** — Intercepts `fetch` globally in development. `register(pattern, handler, options?)` — string substring or RegExp URL matching with optional method filter and `delayMs`. `enable()` / `disable()` at runtime without reload. `unregister(pattern)` / `clear()`.

#### `joopjs/banking` — Banking Utilities (new sub-path)

**Masking** — `maskCard`, `formatCard`, `maskIban`, `maskAccountNumber`, `maskEmail`, `maskPhone`, `maskName`

**IBAN** — `validateIban` (mod-97, 80 countries), `formatIban` (4-char groups), `parseIban` → `{ country, checkDigits, bban }`, `getIbanCountry`, `getSupportedIbanCountries`

**Currency & Amounts** — `formatCurrency` (Intl.NumberFormat), `parseCurrencyString`, `convertCurrency`, `getCurrencySymbol`, `formatAmount`. Safe arithmetic (integer-based): `safeAdd`, `safeSubtract`, `safeMultiply`, `safeDivide`, `roundAmount`, `toMinorUnits`, `fromMinorUnits`, `sumAmounts`, `clampAmount`, `isValidAmount`.

**Transaction IDs** — `generateTransactionId(prefix?)` → `TXN-<timestamp>-<hex>`, `generateReferenceNumber`, `parseTransactionId`, `isValidTransactionId`, `generateBatchId` → `BATCH-<hex>`

**Receipts** — `JoopReceiptService` — structured receipt builder with plain-text, JSON, and HTML renderers.

### `JoopInstance` new fields (v1.1.0)

`secureStorage`, `token`, `pkce`, `otp`, `biometric`, `interceptors`, `circuitBreaker`, `offlineQueue`, `polling`, `multiTab`, `concurrentSession`, `correlation`, `auditLog`, `perf`, `errorReporter`, `deviceFingerprint`, `receipt`, `mock`

---

## [1.0.0] — Initial Release

**14 services · 124 tests · 10 test files**

### Added

#### Reactive primitives (framework-agnostic)

- **`JoopSubject`** — Pub/sub with `subscribe(fn)` / `next(value)` / `unsubscribe(fn)` / `complete()`. Returns an unsubscribe function.
- **`JoopBehaviorSubject<T>`** — Extends `JoopSubject`; holds the current value. New subscribers receive the last emitted value immediately. `getValue()` synchronous read.
- **`JoopObservable<T>`** — Read-only view over a `JoopSubject`. `subscribe(fn)` / `pipe(fn)` for synchronous transforms.

#### `joopjs/core` — Core Services (new sub-path)

- **`JoopConfigService`** / `configService` singleton — `load(urlOrObject)` fetches remote JSON or accepts an inline object. `get<T>(key, default?)` nested dot-notation access. `set(key, value)` / `has(key)` / `reset()`. `config$()` observable.
- **`JoopLogger`** / `logger` singleton — Configurable logging with pub/sub. Levels: `debug / info / warn / error`. Per-level filtering. `onLog(handler)` for structured log forwarding. `setLevel(level)` / `disable()` / `enable()`.

#### `joopjs/storage` — Storage (new sub-path)

- **`JoopScopeMapService`** — Four-scope in-memory store: `app`, `page`, `screen`, `session`. `set(scope, key, value)` / `get(scope, key)` / `delete(scope, key)` / `clear(scope)` / `getAll(scope)`.
- **`JoopDataStorage`** — Typed scope-based key-value store with `menu$()` observable. `setData(scope, key, value)` / `getData(scope, key)` / `deleteKey(scope, key)` / `clearScope(scope)`.

#### `joopjs/auth` — Auth (new sub-path)

- **`JoopAuthService`** — Login state observable. `login(user)` / `logout()` / `isLoggedIn()` / `user$()` / `getUser()`.

#### `joopjs/api` — HTTP Client (new sub-path)

- **`JoopHttpClient`** — Fetch-based HTTP client. `get/post/put/patch/delete(url, options?)`. Configurable `timeoutMs`. Upload progress via `onUploadProgress`. `setBaseUrl(url)` / `setDefaultHeaders(headers)`. Returns typed `JoopResponse<T>`.
- **`JoopRequestService`** — High-level dispatcher. `send(config)` with E2E encryption support (AES/RSA orchestration), loader tracking, and error normalization. `sendBackground(config)` for fire-and-forget.
- `isHttpError(err)` — type guard for `JoopHttpError`.

#### `joopjs/encryption` — Encryption (new sub-path)

- **`JoopGcmService`** — AES-256-GCM symmetric encryption. `encrypt(data, key)` / `decrypt(ciphertext, key)` via WebCrypto. `generateKey()` / `exportKey(key)` / `importKey(raw)`.
- **`JoopRsaService`** — RSA-OAEP-SHA256 asymmetric encryption. `generateKeyPair()` / `encrypt(data, publicKey)` / `decrypt(ciphertext, privateKey)`. SPKI/PKCS8 key export/import.
- **`JoopAesService`** — AES E2E orchestrator: session-key mode (RSA-wrapped AES session key) and request mode (per-request AES-GCM + RSA key transport). Used by `JoopRequestService`.
- **`JoopX25519Service`** — X25519 ECDH key exchange via `@noble/curves`. `generateKeyPair()`. `computeSharedSecret(privateKey, publicKey)`. HKDF-SHA512 key derivation into ChaCha20-Poly1305. `encrypt(data, recipientPublic)` / `decrypt(ciphertext, privateKey, senderPublic)`.

#### `joopjs/session` — Session (new sub-path)

- **`JoopSessionTimeoutService`** — Two-stage idle session timeout. Listens to DOM activity events. `warningMs` triggers a warning; `timeoutMs` triggers logout. `onWarning(handler)` / `onTimeout(handler)`. `reset()` / `start()` / `stop()`.

#### `joopjs/` — Core models and utilities

- **`JoopClientProfile`** / `clientProfile` singleton — Runtime session state holder. `set(profile)` / `get()` / `profile$()` / `clear()`.
- **`JoopPluginService`** — `install(plugin, context?)` / `uninstall(name)` / `getPlugin(name)`. Prevents duplicates. Async `install` hooks.
- **`JoopEnvironmentService`** — Named environment profiles with `apiBaseUrl` and feature flags. `detect()` via `NODE_ENV` / hostname. `isFeatureEnabled(flag)`.
- **`JoopHealthService`** — `register(name, checkFn)` / `registerPingUrl(name, url)` / `runAll()` → `JoopHealthCheckResult[]` with latency.

#### Utility functions

`removeTrailingSlash`, `objectToQueryString`, `deepClone`, `randomString`, `bufferToHex`, `getDeviceInfo`, `isBrowser`

### Build

- Dual ESM + CJS bundles via tsup
- Full TypeScript declarations (`.d.ts`)
- 6 sub-path exports: `joopjs/encryption`, `joopjs/storage`, `joopjs/auth`, `joopjs/api`, `joopjs/core`, `joopjs/session`
- External peer deps: `@noble/ciphers`, `@noble/curves`, `base64-arraybuffer`
- Zero Angular / RxJS / framework dependencies
