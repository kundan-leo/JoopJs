# Getting Started

## Installation

```bash
npm install joopjs
```

JoopJS requires **Node.js 18+** at build time. It runs in any modern browser (Chrome, Firefox, Safari, Edge) and Node.js 18+.

---

## Minimal Bootstrap

```typescript
import { createJoop } from 'joopjs';

const joop = await createJoop({
  app: {
    id: 'my-app',
    baseUrl: 'https://api.example.com',
  },
});
```

`createJoop()` is async because it may fetch remote config. It returns a fully constructed `JoopInstance` with every service ready.

---

## Full Config (all options)

```typescript
import { createJoop } from 'joopjs';
import type { JoopConfig } from 'joopjs';

const config: JoopConfig = {
  app: {
    id: 'my-app',
    baseUrl: 'https://api.example.com',
    logoutService: 'LOGOUT',
    externalUrl: 'https://external.example.com',
    useExternalAPI: false,
    requestTimeHeader: true,
    initScreen: '/home',
    initService: 'INIT',
  },
  timeout: {
    requestTimeout: 30000,
    session: {
      enable: true,
      timeout: 300000,        // 5 minutes
      waitTimeout: 60000,     // 1 minute warning
      ignoreServices: ['HEARTBEAT'],
    },
  },
  logger: {
    enable: true,
    enableDate: true,
    typeConfig: { all: true },
  },
  loader: {
    enable: true,
  },
  isDevMode: false,
};

const joop = await createJoop(config);
```

→ See [Configuration Reference](./configuration.md) for every property.

---

## Load Config from URL

Pass a URL string instead of an inline object — JoopJS will fetch and parse it at startup:

```typescript
const joop = await createJoop('/assets/config/app.config.json');
```

The JSON file must be shaped as `JoopConfig`.

---

## Using Services

All services are properties of the `JoopInstance`:

```typescript
// Encryption
const cipher = await joop.gcm.encrypt('secret text', 'TESTKEY123456789');
const plain  = await joop.gcm.decrypt(cipher, 'TESTKEY123456789');

// Auth
joop.auth.setLoggedIn(true, { menuObj: [{ label: 'Home' }] });
console.log(joop.auth.isLoggedIn()); // true

// Reactive auth state
const unsub = joop.auth.isLoggedIn$().subscribe(v => console.log('logged in:', v));

// HTTP
const data = await joop.http.get('https://api.example.com/users');

// Logger
joop.logger.info('App ready');
joop.logger.warn('Rate limit approaching', { remaining: 3 });

// Feature flags
joop.featureFlags.define('dark-mode', { enabled: true });
console.log(joop.featureFlags.isEnabled('dark-mode')); // true

// I18n
joop.i18n.setLanguage('ar');
console.log(joop.i18n.getDirection()); // 'rtl'

// Fraud detection
const result = await joop.fraud.evaluate({
  amount: 12000,
  currency: 'USD',
  merchantId: 'merch-1',
  userId: 'user-123',
  timestamp: Date.now(),
  ip: '185.60.112.40',
});
console.log(result.level);        // 'medium' | 'high' | 'critical' | 'low'
console.log(result.recommendation); // 'allow' | 'review' | 'challenge' | 'block'
```

---

## TypeScript

JoopJS is written in TypeScript and ships full `.d.ts` type declarations for all 35 sub-paths. No `@types` package needed.

```typescript
import type { JoopInstance, JoopConfig } from 'joopjs';
import type { JoopAuthComposable } from 'joopjs/vue';
import type { JoopAuthHook } from 'joopjs/react';
```

---

## Framework Integration

JoopJS provides first-class integrations for all three major frameworks. In each case, call `createJoop()` once at app startup, then distribute the instance via the framework's native DI/context mechanism.

### React 18+

```typescript
import React from 'react';
import { createJoopReact } from 'joopjs/react';

export const {
  JoopProvider,
  useJoop,
  useJoopAuth,
  useJoopTheme,
  useJoopI18n,
  useJoopLoader,
} = createJoopReact(React);
```

```tsx
// Root component
const joop = await createJoop(config);

function App() {
  return (
    <JoopProvider joop={joop}>
      <YourApp />
    </JoopProvider>
  );
}

// Any child
function Header() {
  const { isLoggedIn, logout } = useJoopAuth();
  return <button onClick={logout}>{isLoggedIn ? 'Log out' : 'Log in'}</button>;
}
```

→ [React Guide](./framework-guides/react.md)

---

### Angular 17+

```typescript
// main.ts / app.config.ts
import { provideJoop } from 'joopjs/angular';

const joop = await createJoop(config);

bootstrapApplication(AppComponent, {
  providers: [provideJoop(joop)],
});
```

```typescript
// Any component or service
import { inject } from '@angular/core';
import { JOOP_AUTH } from 'joopjs/angular';

@Component({ ... })
export class HeaderComponent {
  private auth = inject(JOOP_AUTH);
  isLoggedIn = this.auth.isLoggedIn();
}
```

→ [Angular Guide](./framework-guides/angular.md)

---

### Vue 3

```typescript
import * as Vue from 'vue';
import { createJoopVue } from 'joopjs/vue';

export const {
  provideJoop,
  useJoop,
  useJoopAuth,
  useJoopTheme,
  useJoopI18n,
} = createJoopVue(Vue);
```

```vue
<!-- Root component -->
<script setup>
const joop = await createJoop(config);
provideJoop(joop);
</script>

<!-- Any child component -->
<script setup>
const { isLoggedIn, logout } = useJoopAuth();
</script>
<template>
  <button @click="logout">{{ isLoggedIn ? 'Log out' : 'Log in' }}</button>
</template>
```

→ [Vue Guide](./framework-guides/vue.md)

---

## Sub-path Imports (Tree Shaking)

Import only the services you need:

```typescript
// Instead of importing everything
import { JoopGcmService } from 'joopjs';                    // full bundle

// Import from the specific sub-path
import { JoopGcmService } from 'joopjs/encryption';         // only encryption
import { JoopFeatureFlagService } from 'joopjs/core';       // only core
import { JoopFraudDetection } from 'joopjs/banking';        // only banking
```

Sub-path imports are fully tree-shaken — bundlers like Vite, webpack, and Rollup will only include what you use.

---

## Standalone Service Usage

Services can be used without `createJoop()` — instantiate them directly:

```typescript
import { JoopGcmService } from 'joopjs/encryption';
import { JoopFormValidator } from 'joopjs/validation';

// No createJoop() needed
const gcm = new JoopGcmService();
const cipher = await gcm.encrypt('data', 'key1234567890123');

const validator = new JoopFormValidator();
const result = validator.validate('not-an-email', ['required', 'email']);
```

This is useful for utility-style usage where you don't want the full app bootstrap.

---

## Interactive Playground

Explore all modules live in the browser:

```bash
npm run playground   # http://localhost:4500
```
