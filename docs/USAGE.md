# JoopJS Usage Guide

> Author: **Kundan Singh**
> Package: `joopjs` — Enterprise TypeScript SDK

A comprehensive guide for developers integrating JoopJS into their applications.

---

## Table of Contents

- [Installation](#installation)
- [Core Concepts](#core-concepts)
- [Bootstrap](#bootstrap)
- [Banking Services](#banking-services)
  - [Digital Wallet](#digital-wallet)
  - [Loan Servicing](#loan-servicing)
  - [FX Forwards](#fx-forwards)
  - [Ledger & Reconciliation](#ledger--reconciliation)
  - [Limit Management](#limit-management)
  - [Standing Orders & Mandates](#standing-orders--mandates)
  - [Payments (Bill, Split, URI)](#payments-bill-split-uri)
  - [Cards & Disputes](#cards--disputes)
  - [Insurance & Remittance](#insurance--remittance)
- [Finance Services](#finance-services)
  - [Mutual Funds & SIP](#mutual-funds--sip)
  - [Budget Tracker](#budget-tracker)
  - [Portfolio Tracker](#portfolio-tracker)
  - [Tax Calculator](#tax-calculator)
  - [Cash Flow Forecast](#cash-flow-forecast)
- [Security Services](#security-services)
  - [Sanctions Screening](#sanctions-screening)
  - [AML Engine](#aml-engine)
  - [Risk Engine](#risk-engine)
  - [KYC State Machine](#kyc-state-machine)
  - [Fraud Detection](#fraud-detection)
- [Auth & Sessions](#auth--sessions)
  - [Auth Service](#auth-service)
  - [JWT](#jwt)
  - [OTP & TOTP](#otp--totp)
  - [MFA](#mfa)
  - [PKCE / OAuth 2.0](#pkce--oauth-20)
  - [OIDC](#oidc)
  - [SSO](#sso)
  - [Session Lifecycle](#session-lifecycle)
- [Encryption](#encryption)
  - [AES-GCM Symmetric](#aes-gcm-symmetric)
  - [X25519 Key Exchange](#x25519-key-exchange)
  - [E2E Encryption Flow](#e2e-encryption-flow)
  - [Hashing & Password Storage](#hashing--password-storage)
- [Reactive Observables](#reactive-observables)
- [Framework Integration](#framework-integration)
  - [React](#react)
  - [Angular](#angular)
  - [Vue](#vue)
- [Error Handling](#error-handling)
- [TypeScript Conventions](#typescript-conventions)
- [Sub-path Imports & Tree-shaking](#sub-path-imports--tree-shaking)
- [Testing with JoopJS](#testing-with-joopjs)

---

## Installation

```bash
npm install joopjs
# yarn add joopjs
# pnpm add joopjs
```

No mandatory peer dependencies. React / Angular / Vue are optional.

---

## Core Concepts

### Services are plain classes

Every feature in JoopJS is a plain TypeScript class. No dependency injection container, no decorators, no Angular `@Injectable` required.

```typescript
import { JoopDigitalWalletService } from 'joopjs';

const wallet = new JoopDigitalWalletService();   // ✅ just a class
```

Create one instance per feature module and share it as a singleton.

### Amounts are `number`

```typescript
wallet.topUp(w.id, 1000);      // ✅ plain number
wallet.topUp(w.id, '1000');    // ❌ wrong — TypeScript error
```

### Timestamps are epoch milliseconds

```typescript
loan.createLoan({ ..., maturityDate: Date.now() + 90 * 86_400_000 });  // ✅
loan.createLoan({ ..., maturityDate: new Date() });                     // ❌
```

### Currency is an ISO-4217 string

```typescript
wallet.createWallet('u-001', { currency: 'USD' });   // ✅ 'USD', 'EUR', 'AED'
wallet.createWallet('u-001', { currency: 840 });      // ❌
```

### Errors are standard `Error`

All services throw standard JavaScript `Error` objects with descriptive messages. Always wrap in `try/catch`.

### Observables are not RxJS

JoopJS uses `JoopSubject` / `JoopBehaviorSubject` — lightweight reactive primitives:
- `.subscribe(callback)` → returns an unsubscribe function
- `.next(value)` → emit a new value (**never `.emit(value)`**)
- `.getValue()` → read current value (BehaviorSubject only)

---

## Bootstrap

```typescript
import { createJoop } from 'joopjs';

const joop = createJoop({
  env: 'production',      // 'development' | 'staging' | 'production'
  appId: 'my-banking-app',
  currency: 'USD',
  locale: 'en',
  logLevel: 'warn',
  apiBaseUrl: 'https://api.mybank.com',
  timeout: 30_000,
});
```

`createJoop` is optional — you can instantiate any service directly without it. Use it when you want a central config registry or when using the Angular/Vue plugins.

---

## Banking Services

### Digital Wallet

```typescript
import { JoopDigitalWalletService } from 'joopjs';
import type { JoopWallet, JoopWalletTxnType } from 'joopjs';

const wallet = new JoopDigitalWalletService();

// Create a wallet
const w = wallet.createWallet('user-001', {
  currency: 'USD',
  label: 'Primary Wallet',
  initialBalance: 0,
});

// Fund and transact
wallet.topUp(w.id, 1000);
wallet.pay(w.id, 49.99, 'merchant-001', 'Netflix subscription');
wallet.transfer(w.id, anotherWalletId, 200);

// Freeze / unfreeze / close
wallet.freeze(w.id);
wallet.unfreeze(w.id);
wallet.close(w.id);       // throws if balance > 0

// Query
const balance  = wallet.getBalance(w.id);             // number
const txns     = wallet.getTransactions(w.id, 20);    // last 20, newest first
const allWalls = wallet.getWalletsByUser('user-001'); // JoopWallet[]

// Reactive balance updates
const unsub = wallet.balance$().subscribe(({ walletId, balance }) => {
  updateBalanceDisplay(walletId, balance);
});
// cleanup: unsub()
```

---

### Loan Servicing

```typescript
import { JoopLoanServicingService } from 'joopjs';

const loans = new JoopLoanServicingService();

// Create a loan — EMI is auto-calculated
const loan = loans.createLoan({
  borrowerName: 'Alice Smith',
  borrowerId:   'u-001',
  principalAmount:            120_000,
  annualInterestRatePercent:  9.5,
  tenureMonths:               36,
  currency:                   'USD',
  disbursedAt:                Date.now(),
});
// loan.ref        → 'LN-2026-000001'
// loan.emiAmount  → 3843.57 (calculated via annuity formula)

// Record monthly payment (interest-first allocation)
loans.recordPayment(loan.id, loan.emiAmount);

// Partial payment
loans.recordPayment(loan.id, 2000);   // marks installment as 'partial'

// Waive an installment
loans.waveInstallment(loan.id, 3);    // waive installment #3

// Lifecycle
loans.markDefaulted(loan.id);

// Queries
const schedule = loans.getSchedule(loan.id);          // JoopInstallment[]
const balance  = loans.getOutstandingBalance(loan.id); // current principal
const accrued  = loans.getAccruedInterest(loan.id);   // interest since last payment
const stmt     = loans.getLoanStatement(loan.id);      // full statement
```

---

### FX Forwards

```typescript
import { JoopFxForwardService } from 'joopjs';

const fx = new JoopFxForwardService();

// Set spot rates (inverse auto-created)
fx.setSpotRate('USD', 'EUR', 0.92);   // also sets EUR→USD = 1/0.92

// Create a forward contract
const fwd = fx.createForward({
  type:            'sell',               // or 'buy'
  baseCurrency:    'USD',
  quoteCurrency:   'EUR',
  notionalAmount:  100_000,
  contractRate:    0.92,
  maturityDate:    Date.now() + 90 * 86_400_000,
  counterparty:    'Deutsche Bank',
});
// fwd.ref → 'FWD-2026-000001'

// Settle at maturity
const settlement = fx.settleForward(fwd.id, 0.89);
// PnL for 'sell': (contractRate - spotRate) * notional = (0.92 - 0.89) * 100_000 = 3_000

// Cancel before maturity
fx.cancelForward(fwd.id);

// Mark to market (unrealized P&L)
const mtm = fx.getMarkToMarket(fwd.id);    // { contractRate, currentSpotRate, pnl }

// Portfolio queries
const exposure  = fx.getExposure('USD');                 // JoopFxExposure[]
const expiring  = fx.getExpiring(7 * 86_400_000);        // maturing in 7 days
const open      = fx.getOpenContracts();
```

---

### Ledger & Reconciliation

```typescript
import { JoopLedgerService, JoopReconciliationService } from 'joopjs';

// ── Double-Entry Ledger ──
const ledger = new JoopLedgerService();

ledger.addAccount('1001', 'Cash',          'asset');
ledger.addAccount('2001', 'Accounts Payable', 'liability');
ledger.addAccount('4001', 'Sales Revenue', 'revenue');
ledger.addAccount('5001', 'Rent Expense',  'expense');

// Post a balanced journal entry
const entry = ledger.postEntry('Monthly rent payment', [
  { accountCode: '5001', debit: 1500, credit: 0 },
  { accountCode: '1001', debit: 0,    credit: 1500 },
]);
// entry.ref → 'JE-2026-000001'

const cashBalance = ledger.getBalance('1001');   // 1000 - 1500 = -500
const trial       = ledger.getTrialBalance();
// { rows: [...], totalDebits: 1500, totalCredits: 1500, isBalanced: true }

// ── Reconciliation ──
const recon = new JoopReconciliationService();

const session = recon.createSession('ACC-001', bankStatementItems, internalLedgerItems);

// Auto-match by amount + date (with tolerance)
recon.autoMatch(session.id, 0.01, 3);   // $0.01 tolerance, 3-day window

// Manually match remaining
recon.manualMatch(session.id, 'internal-txn-id', 'bank-txn-id');

const summary = recon.getSummary(session.id);
// { matchedPairs, unmatchedInternal, unmatchedBank, totalDifference, isReconciled }
```

---

### Limit Management

```typescript
import { JoopLimitManagementService } from 'joopjs';

const limits = new JoopLimitManagementService();

// Set a daily transaction limit for an account
limits.setLimit({
  scope:     'account',
  scopeId:   'ACC-001',
  type:      'daily',
  currency:  'USD',
  maxAmount: 10_000,
  enabled:   true,
});

// Check before processing a transaction
const check = limits.checkLimit('account', 'ACC-001', 'daily', 'USD', 7500);
// null  → no limit configured for this scope/type/currency
// { allowed: true, remaining: 2500, used: 7500 } → within limit
// { allowed: false, remaining: 0, used: 10000 }  → limit exceeded

if (check && !check.allowed) throw new Error('Daily limit exceeded');

// Record actual usage after processing
limits.recordUsage('account', 'ACC-001', 'daily', 'USD', 7500);
```

**Limit types:** `'per-transaction'` | `'daily'` | `'weekly'` | `'monthly'` | `'yearly'`

---

### Standing Orders & Mandates

```typescript
import { JoopStandingOrderService, JoopMandateService } from 'joopjs';

// ── Standing Orders (push payments) ──
const so = new JoopStandingOrderService();

const order = so.create({
  fromAccount:       'ACC-001',
  toAccount:         'ACC-002',
  toBeneficiaryName: 'John Rent',
  amount:            1200,
  currency:          'USD',
  frequency:         'monthly',
  startDate:         Date.now(),
  endDate:           Date.now() + 365 * 86_400_000,
});
// order.ref → 'SO-2026-000001'

so.execute(order.id, 'success');      // executes and advances nextExecutionAt
const due = so.getDue();              // orders due now (or overdue)
so.pause(order.id);
so.resume(order.id);
so.cancel(order.id);

// ── Mandates (pull payments / direct debit) ──
const mandates = new JoopMandateService();

const m = mandates.create({
  userId:       'u-001',
  creditorName: 'Netflix',
  amount:       15.99,
  currency:     'USD',
  frequency:    'monthly',
  startDate:    Date.now(),
});

mandates.execute(m.id, 15.99);        // records execution, advances next date
mandates.pause(m.id);
mandates.resume(m.id);
mandates.cancel(m.id);
const dueMandates = mandates.getDue();
```

---

### Payments (Bill, Split, URI)

```typescript
import { JoopBillPaymentService, JoopSplitPaymentService, JoopPaymentUriService } from 'joopjs';

// ── Bill Payments ──
const bills = new JoopBillPaymentService();
bills.addBiller({ id: 'ELEC-001', name: 'City Electric', category: 'utilities' });
const bill    = bills.createBill({ billerId: 'ELEC-001', accountNumber: 'ACCT-1234', amount: 150, dueDate: Date.now() + 7 * 86_400_000 });
const payment = bills.pay(bill.id, 150, 'wallet');
const upcoming = bills.getUpcoming(7 * 86_400_000);    // due in next 7 days

// ── Split Payments ──
const split = new JoopSplitPaymentService();
const exp = split.createExpense({ title: 'Team Dinner', totalAmount: 240, paidById: 'u-001', method: 'equal' });
split.addParticipant(exp.id, { userId: 'u-002', name: 'Bob' });
split.addParticipant(exp.id, { userId: 'u-003', name: 'Carol' });
// Each owes: 240 / 3 = $80
split.settle(exp.id, 'u-002');                          // Bob pays back
const balances = split.getBalances(exp.id);             // [{ userId, owes, settled }]

// ── Payment URIs ──
const uriSvc = new JoopPaymentUriService();
const upiUri = uriSvc.generate({ scheme: 'upi', payeeVpa: 'merchant@bank', amount: 100, currency: 'INR', description: 'Order #123' });
// → 'upi://pay?pa=merchant@bank&pn=merchant&am=100&cu=INR&tn=Order%20%23123'
const parsed = uriSvc.parse(upiUri);                    // { scheme, payeeVpa, amount, ... }
```

---

### Cards & Disputes

```typescript
import { JoopCardManagementService, JoopDisputeService } from 'joopjs';

// ── Cards ──
const cards = new JoopCardManagementService();
const card  = cards.issueCard({ userId: 'u-001', cardType: 'virtual', network: 'visa', currency: 'USD' });

cards.setSpendingLimits(card.id, { daily: 500, monthly: 5000, perTransaction: 200 });
cards.setControls(card.id, { onlineAllowed: true, internationalAllowed: false, contactlessAllowed: true });

const check = cards.checkTransaction(card.id, { amount: 100, merchant: 'Amazon', channel: 'online' });
// { allowed: true, reason: null }

cards.freeze(card.id);
cards.unfreeze(card.id);

// ── Disputes ──
const disputes = new JoopDisputeService();
const d = disputes.file({
  userId:        'u-001',
  transactionId: 'TXN-001',
  reason:        'unauthorized',
  amount:        99.99,
  description:   'Did not make this purchase',
});
disputes.uploadEvidence(d.id, { type: 'screenshot', description: 'Bank statement showing unknown charge' });
disputes.resolve(d.id, 'resolved-in-favor', 'Full refund issued');
const stats = disputes.getStats();   // { total, open, resolved, rejectedCount }
```

---

### Insurance & Remittance

```typescript
import { JoopInsuranceService, JoopRemittanceService } from 'joopjs';

// ── Insurance ──
const ins = new JoopInsuranceService();
const policy = ins.addPolicy({
  type:           'life',
  holderName:     'Alice Smith',
  holderId:       'u-001',
  coverageAmount: 500_000,
  annualPremium:  2400,
  frequency:      'monthly',
  startDate:      Date.now(),
  endDate:        Date.now() + 365 * 86_400_000,
  currency:       'USD',
});
ins.recordPremiumPayment(policy.id, 200);    // monthly premium
ins.fileClaim(policy.id, 1000, 'Hospital admission');
ins.renewPolicy(policy.id, Date.now() + 730 * 86_400_000);
const expiring = ins.getExpiring(30 * 86_400_000);     // expiring in 30 days

// ── Remittance ──
const rem = new JoopRemittanceService();
rem.setExchangeRate('USD', 'INR', 83.5);
const quote = rem.getQuote('USD', 'INR', 1000);
// { sendAmount: 1000, fee: 25, receiveAmount: 81543.75, exchangeRate: 83.5 }

const tx = rem.initiate({
  senderId:       'u-001',
  senderName:     'Alice Smith',
  receiverId:     'u-002',
  receiverName:   'Bob Patel',
  sourceCurrency: 'USD',
  targetCurrency: 'INR',
  sendAmount:     1000,
  channel:        'swift',
});
// tx.ref → 'RMT-2026-000001'
rem.updateStatus(tx.id, 'completed');
```

---

## Finance Services

### Mutual Funds & SIP

```typescript
import { JoopMutualFundService } from 'joopjs';

const mf = new JoopMutualFundService();

// Register a fund
mf.registerFund({
  id:         'EQ001',
  name:       'Equity Growth Fund',
  category:   'equity',
  currentNav: 45.50,
  currency:   'USD',
  minInvestment: 100,
});

// Invest (creates/updates holding, units = amount / NAV)
const holding = mf.invest('EQ001', 'inv-001', 5000);
// holding.units = 109.89, holding.averageCostNav = 45.50

// Update NAV (refreshes all holdings: currentValue, unrealizedGain, returnPercent)
mf.updateNav('EQ001', 48.00);
const h = mf.getHolding('EQ001', 'inv-001');
// h.currentValue = 5274.72, h.returnPercent = +5.49%

// Redeem units
const redemption = mf.redeem('EQ001', 'inv-001', 50);
// redemption.proceeds = 50 * 48.00 = 2400, redemption.realizedGain

// Systematic Investment Plan (SIP)
const sip = mf.createSip('EQ001', 'inv-001', 500, 'monthly', Date.now(), {
  endDate: Date.now() + 365 * 86_400_000,
});
// sip.status = 'active'

mf.executeSip(sip.id);        // invests 500, advances nextExecutionDate by 1 month
mf.pauseSip(sip.id);
mf.resumeSip(sip.id);
mf.cancelSip(sip.id);

// Queries
const allSips      = mf.getSipsByInvestor('inv-001');
const allHoldings  = mf.getHoldingsByInvestor('inv-001');
const fundDetails  = mf.getFund('EQ001');
```

**SIP frequencies:** `'weekly'` | `'monthly'` | `'quarterly'`

---

### Budget Tracker

```typescript
import { JoopBudgetService } from 'joopjs';

const budget = new JoopBudgetService();

const b = budget.createBudget({ name: 'April Budget', period: 'monthly', currency: 'USD', startDate: Date.now() });
const food  = budget.addCategory(b.id, { name: 'Food',      limit: 500 });
const rent  = budget.addCategory(b.id, { name: 'Rent',      limit: 1500 });
const utils = budget.addCategory(b.id, { name: 'Utilities', limit: 200 });

budget.recordSpend(b.id, food.id,  120, 'Groceries');
budget.recordSpend(b.id, food.id,   45, 'Restaurant');
budget.recordSpend(b.id, rent.id, 1500, 'Monthly rent');

const report = budget.getReport(b.id);
// { categories: [{ name, limit, spent, remaining, overBudget }], totalLimit: 2200, totalSpent: 1665 }

const alerts = budget.getAlerts(b.id, 80);
// categories at >80% utilisation → [{ categoryId, name, pctUsed: 100 }] (rent is 100%)
```

---

### Portfolio Tracker

```typescript
import { JoopPortfolioService } from 'joopjs';

const portfolio = new JoopPortfolioService();

const p = portfolio.createPortfolio({ name: 'Retirement Fund', currency: 'USD', userId: 'u-001' });

portfolio.addHolding(p.id, { symbol: 'AAPL', name: 'Apple Inc', quantity: 10, avgCostPrice: 150, currentPrice: 175, assetClass: 'equity' });
portfolio.addHolding(p.id, { symbol: 'BND', name: 'Bond ETF', quantity: 50, avgCostPrice: 78, currentPrice: 80, assetClass: 'bond' });

portfolio.updatePrice(p.id, 'AAPL', 185);   // refresh price

const summary = portfolio.getSummary(p.id);
// { totalInvested: 5400, currentValue: 5850, totalGain: 450, gainPercent: 8.33%, holdings: [...] }

portfolio.rebalance(p.id, { equity: 70, bond: 30 });  // target allocation %
```

---

### Tax Calculator

```typescript
import { JoopTaxCalculatorService } from 'joopjs';

const tax = new JoopTaxCalculatorService();

tax.setTaxProfile({ jurisdiction: 'US', filingStatus: 'single', year: 2025 });
tax.addSlab(11_000, 10);     // 10% up to $11k
tax.addSlab(44_725, 12);     // 12% up to $44.7k
tax.addSlab(95_375, 22);     // 22% up to $95.4k

const result = tax.calculate(75_000);
// { taxableIncome: 75000, taxLiability: 13234, effectiveRate: 17.6%, slabs: [...] }

const capitalGain = tax.calcCapitalGainsTax(50_000, 18, 'US');   // long-term rate
```

---

### Cash Flow Forecast

```typescript
import { JoopCashFlowService } from 'joopjs';

const cf = new JoopCashFlowService();

cf.addCashFlow({ name: 'Salary',     amount: 8000,  frequency: 'monthly', startDate: Date.now(), type: 'inflow' });
cf.addCashFlow({ name: 'Rent',       amount: 2000,  frequency: 'monthly', startDate: Date.now(), type: 'outflow' });
cf.addCashFlow({ name: 'Insurance',  amount: 200,   frequency: 'monthly', startDate: Date.now(), type: 'outflow' });
cf.addCashFlow({ name: 'Bonus',      amount: 5000,  frequency: 'yearly',  startDate: Date.now(), type: 'inflow' });

const endDate = Date.now() + 90 * 86_400_000;
const forecast = cf.forecast(Date.now(), endDate);
// { periods: [{ date, inflows, outflows, netCashFlow, runningBalance }], summary }

const summary = cf.getSummary(Date.now(), endDate);
// { totalInflows, totalOutflows, netCashFlow, avgMonthlyNet }
```

---

## Security Services

### Sanctions Screening

```typescript
import { JoopSanctionsScreeningService } from 'joopjs';
import type { JoopSanctionsEntity, JoopScreeningResult } from 'joopjs';

const sanctions = new JoopSanctionsScreeningService();

// Load sanctions lists
sanctions.loadList('ofac', [
  { id: 'e-001', name: 'Kim Jong-un', aliases: ['Dear Leader', 'Supreme Leader'],
    type: 'individual', country: 'KP', lists: ['ofac', 'un'] },
  { id: 'e-002', name: 'Acme Shell Corp', aliases: [], type: 'entity', country: 'RU', lists: ['eu'] },
]);
sanctions.loadList('un', unEntities);

// React to hits before they are returned
sanctions.hit$().subscribe(hit => {
  auditLog.record({ event: 'sanctions_hit', entity: hit.entity.name, matchType: hit.matchType });
});

// Screen a customer
const result: JoopScreeningResult = sanctions.screen({
  name:    customer.fullName,
  country: customer.country,
  dob:     customer.dateOfBirth,
});

if (result.status === 'hit') {
  throw new Error(`Sanctions match: ${result.matches[0].entity.name} (score: ${result.matches[0].score})`);
}
if (result.status === 'review') {
  await escalateToComplianceTeam(result);
}

// Disable a list temporarily (e.g. during list maintenance)
sanctions.disableList('eu');
sanctions.enableList('eu');

// Batch screening
const batchResults = sanctions.screenBatch(customerNames);
```

**Match types:**
- `'exact'` — normalised name matches exactly
- `'alias'` — matches a known alias
- `'fuzzy'` — Levenshtein similarity above threshold (default 0.85)

---

### AML Engine

```typescript
import { JoopAmlService } from 'joopjs';

const aml = new JoopAmlService();

// Configure rules
aml.addRule({
  id:        'R001',
  name:      'Large Cash Deposit',
  type:      'threshold',
  threshold: 10_000,
  currency:  'USD',
  action:    'flag',
  enabled:   true,
});

aml.addRule({
  id:        'R002',
  name:      'Structuring Pattern',
  type:      'pattern',
  pattern:   'structuring',
  action:    'block',
  enabled:   true,
});

// Check a transaction
const alert = aml.checkTransaction({
  id:        'TXN-001',
  amount:    12_000,
  currency:  'USD',
  type:      'cash-deposit',
  userId:    'u-001',
  timestamp: Date.now(),
});

if (alert) {
  // { alertId, ruleId, severity: 'high', recommendation: 'File SAR' }
  await fileReport(alert);
}

// Subscribe to all alerts
aml.alert$().subscribe(alert => notifyComplianceOfficer(alert));
```

---

### Risk Engine

```typescript
import { JoopRiskEngineService } from 'joopjs';

const risk = new JoopRiskEngineService();

// Configure weighted risk factors
risk.addFactor({ key: 'velocityScore', weight: 0.35, transform: (v: number) => Math.min(v / 10, 1) });
risk.addFactor({ key: 'locationAnomaly', weight: 0.25 });
risk.addFactor({ key: 'deviceScore',     weight: 0.25 });
risk.addFactor({ key: 'timeAnomaly',     weight: 0.15 });

// Set custom thresholds
risk.setThreshold('medium',   0.35);
risk.setThreshold('high',     0.65);
risk.setThreshold('critical', 0.85);

// Evaluate a login event
const score = risk.evaluate({
  velocityScore:    7,     // normalised to 0.7 by transform
  locationAnomaly:  0.4,
  deviceScore:      0.2,
  timeAnomaly:      0.1,
});
// score.total = 0.35*0.7 + 0.25*0.4 + 0.25*0.2 + 0.15*0.1 = 0.4175
// score.level = 'medium'

if (score.level === 'high' || score.level === 'critical') {
  requireMfa();
}

risk.score$().subscribe(({ score, level }) => dashboardUpdate(score, level));
```

---

### KYC State Machine

```typescript
import { JoopKycStateMachineService } from 'joopjs';

const kyc = new JoopKycStateMachineService();

// Submit documents
const submission = kyc.submit({
  userId:         'u-001',
  documentType:   'passport',
  documentNumber: 'A1234567',
  expiryDate:     Date.now() + 2 * 365 * 86_400_000,
  nationality:    'US',
});
// submission.status = 'pending_review'

// Verify (compliance officer action)
kyc.verify(submission.id, 'approved', { notes: 'All documents valid' });
// submission.status = 'verified', kyc.getStatus('u-001').kycLevel = 2

// Reject
kyc.verify(submission.id, 'rejected', { notes: 'Expired document', requestResubmission: true });

kyc.status$().subscribe(update => updateCustomerKycBadge(update.userId, update.verificationStatus));
```

---

### Fraud Detection

```typescript
import { JoopFraudDetection } from 'joopjs';

const fraud = new JoopFraudDetection();

fraud.addRule({
  id:        'velocity',
  name:      'High TX Velocity',
  condition: (ctx) => ctx.txnCount > 5,
  risk:      40,
});

fraud.addRule({
  id:        'geo',
  name:      'Multi-Country Access',
  condition: (ctx) => ctx.countries.size > 2,
  risk:      60,
});

const result = fraud.assess({
  txnCount:  8,
  countries: new Set(['US', 'NG', 'RU']),
  amount:    3000,
  hour:      2,   // 2 AM
});
// result.risk = 100, result.level = 'critical', result.triggeredRules = ['velocity', 'geo']

fraud.alert$().subscribe(alert => blockTransaction(alert));
```

---

## Auth & Sessions

### Auth Service

```typescript
import { JoopAuthService } from 'joopjs';

const auth = new JoopAuthService(storageService);   // storageService is optional

const session = await auth.login('alice@bank.com', 'secure-password');
// session: { userId, sessionId, accessToken, refreshToken, expiresAt }

const refreshed = await auth.refresh(session.refreshToken);

await auth.logout(session.sessionId);

const me = auth.getCurrentUser();    // JoopAuthUser | null

auth.session$().subscribe(session => {
  if (!session) redirectToLogin();
});

auth.onEvent('login',          e => console.log('Login:',  e.userId));
auth.onEvent('logout',         e => console.log('Logout:', e.userId));
auth.onEvent('session-expired',e => showSessionExpiredModal());
```

---

### JWT

```typescript
import { JoopJwtService } from 'joopjs';

const jwt = new JoopJwtService({ secret: process.env.JWT_SECRET!, expiryMs: 3_600_000 });

// Sign a payload
const token = jwt.sign({ userId: 'u-001', role: 'admin', permissions: ['read', 'write'] });

// Verify (returns null if invalid or expired)
const claims = jwt.verify(token);
if (!claims) throw new Error('Invalid or expired token');
// claims: { userId, role, permissions, iat, exp }

// Decode without verification (for inspection only)
const decoded = jwt.decode(token);

// Revoke (blacklist — add to deny list in storage)
jwt.revoke(token);
const blocked = jwt.isRevoked(token);   // true

// Renew — extend expiry without re-authentication
const newToken = jwt.renew(token);
```

---

### OTP & TOTP

```typescript
import { JoopOtpService } from 'joopjs';

const otp = new JoopOtpService({ expiryMs: 5 * 60_000, length: 6 });

// Numeric OTP
const { otp: code, expiresAt } = otp.generate('u-001', 'login');
// Send code via SMS/email
const valid = otp.verify('u-001', 'login', code);    // true (auto-invalidated after use)

// TOTP (Google Authenticator compatible)
const secret = otp.generateTotpSecret();              // base32 secret key
const qrUri  = otp.getTotpUri('alice@bank.com', secret, 'MyBank App');
// Render qrUri as QR code for user to scan

const totpValid = otp.verifyTotp(secret, userEnteredCode);
```

---

### MFA

```typescript
import { JoopMfaService } from 'joopjs';

const mfa = new JoopMfaService();

// Enroll a method
mfa.enroll('u-001', 'totp');     // or 'sms', 'email', 'hardware-key'
const enrolled = mfa.getEnrolledMethods('u-001');   // ['totp']

// Challenge / respond flow
const challenge = mfa.challenge('u-001');
// challenge: { challengeId, method: 'totp', hint: 'Enter code from authenticator' }

const result = mfa.respond(challenge.challengeId, '483920');
// { passed: true, sessionUpgraded: true }

mfa.unenroll('u-001', 'totp');
mfa.status$().subscribe(e => console.log('MFA event:', e.type, e.userId));
```

---

### PKCE / OAuth 2.0

```typescript
import { JoopPKCEService } from 'joopjs';

const pkce = new JoopPKCEService({
  clientId:               'my-banking-app',
  redirectUri:            'https://app.example.com/callback',
  authorizationEndpoint:  'https://auth.mybank.com/authorize',
  tokenEndpoint:          'https://auth.mybank.com/token',
  scopes:                 ['openid', 'profile', 'accounts', 'transactions'],
});

// Step 1: Build the authorization URL
const { url, codeVerifier, state } = pkce.buildAuthorizationUrl();
sessionStorage.setItem('codeVerifier', codeVerifier);
sessionStorage.setItem('oauthState',   state);
window.location.href = url;   // redirect user to bank's login page

// Step 2: Handle the callback (after user authenticates)
const storedVerifier = sessionStorage.getItem('codeVerifier')!;
const tokens = await pkce.exchangeCode(authorizationCode, storedVerifier);
// { accessToken, refreshToken, idToken, expiresIn }

// Step 3: Refresh when expired
const newTokens = await pkce.refreshTokens(tokens.refreshToken);

// Parse the ID token
const userInfo = pkce.parseIdToken(tokens.idToken);
```

---

### OIDC

```typescript
import { JoopOIDCClient } from 'joopjs';

const oidc = new JoopOIDCClient({
  issuer:   'https://auth.mybank.com',
  clientId: 'my-banking-app',
});

await oidc.loadDiscovery();   // fetches /.well-known/openid-configuration

const loginUrl = oidc.buildLoginUrl({
  redirectUri: 'https://app.example.com/callback',
  scopes:      ['openid', 'email', 'profile'],
  prompt:      'login',
});

const user = await oidc.processCallback(window.location.href);
const info  = await oidc.getUserInfo(accessToken);

// Single log-out
await oidc.endSession(idToken, 'https://app.example.com/logout');
```

---

### SSO

```typescript
import { JoopSsoService } from 'joopjs';

const sso = new JoopSsoService();

sso.registerProvider({
  id:           'azure-ad',
  name:         'Azure AD',
  type:         'oidc',
  clientId:     process.env.AZURE_CLIENT_ID!,
  clientSecret: process.env.AZURE_CLIENT_SECRET!,
  discoveryUrl: 'https://login.microsoftonline.com/tenant-id/v2.0',
});

sso.setRoleMapping('azure-ad', {
  'BankAdmins':   'admin',
  'BankTellers':  'teller',
  'BankViewers':  'viewer',
});

const loginUrl = sso.getLoginUrl('azure-ad', { redirectUri: '/auth/callback' });

const session = await sso.handleCallback('azure-ad', callbackParams);
// session.user: { id, email, name, groups, provider: 'azure-ad' }

const appRole = sso.mapRole('azure-ad', 'BankAdmins');   // 'admin'
```

---

### Session Lifecycle

```typescript
import { JoopIdleService, JoopMultiTabSyncService, JoopConcurrentSessionService } from 'joopjs';

// Idle timeout — warn after 14 minutes, auto-logout at 15
const idle = new JoopIdleService({ idleTimeoutMs: 14 * 60_000, warningBeforeMs: 60_000 });
idle.start();
idle.idle$().subscribe(({ type }) => {
  if (type === 'warning') showInactivityWarning();
  if (type === 'timeout') logoutUser();
});

// Multi-tab sync — logout in all tabs when one logs out
const sync = new JoopMultiTabSyncService({ channel: 'joopjs-session' });
sync.broadcast({ type: 'logout' });
sync.sync$().subscribe(msg => {
  if (msg.type === 'logout') auth.logout();
});

// Concurrent session management
const concurrent = new JoopConcurrentSessionService();
concurrent.register('u-001', 'session-abc');
concurrent.invalidateOthers('u-001', 'session-abc');   // kill other devices
const active = concurrent.getActiveSessions('u-001');
```

---

## Encryption

### AES-GCM Symmetric

```typescript
import { JoopGcmService } from 'joopjs';

const gcm = new JoopGcmService();

// Generate a 256-bit AES key
const key = await gcm.generateKey();

// Encrypt a string
const { ciphertext, iv } = await gcm.encrypt('Sensitive PAN: 4111111111111111', key);

// Decrypt
const plaintext = await gcm.decrypt(ciphertext, key, iv);

// Encrypt an object
const encObj = await gcm.encryptObject({ userId: 'u-001', balance: 5000 }, key);
const obj     = await gcm.decryptObject(encObj, key);

// Persist the key
const exported = await gcm.exportKey(key);        // base64 string — store securely
const restored = await gcm.importKey(exported);
```

---

### X25519 Key Exchange

```typescript
import { JoopX25519Service, JoopGcmService } from 'joopjs';

const dh  = new JoopX25519Service();
const gcm = new JoopGcmService();

// Alice and Bob each generate key pairs (do this once per session)
const aliceKeys = dh.generateKeyPair();
// { privateKey: Uint8Array(32), publicKey: Uint8Array(32) }

const bobKeys   = dh.generateKeyPair();

// Exchange public keys over the network
// Alice sends aliceKeys.publicKey → Bob
// Bob sends bobKeys.publicKey → Alice

// Each side derives the SAME shared secret (ECDH)
const aliceShared = dh.deriveSharedSecret(aliceKeys.privateKey, bobKeys.publicKey);
const bobShared   = dh.deriveSharedSecret(bobKeys.privateKey,   aliceKeys.publicKey);
// aliceShared === bobShared (both Uint8Array(32))

// Derive an AES-GCM key from the shared secret
const sessionKey = await dh.deriveAesKey(aliceShared);

// Now encrypt/decrypt messages with the session key
const enc = await gcm.encrypt('Hello from Alice', sessionKey);
const msg = await gcm.decrypt(enc.ciphertext, sessionKey, enc.iv);
```

---

### E2E Encryption Flow

```typescript
import { JoopE2EService } from 'joopjs';

const e2e = new JoopE2EService();

// Server: initiate a session, return public key to client
const serverSession = await e2e.initSession('session-001');

// Client: initialise with server's public key
await e2e.initSession('session-001', serverSession.publicKey);

// Both sides can now exchange encrypted messages
const enc = await e2e.encrypt('session-001', 'Transfer confirmation: $500 to Bob');
const dec = await e2e.decrypt('session-001', enc.ciphertext, enc.iv);
```

---

### Hashing & Password Storage

```typescript
import { JoopHashService } from 'joopjs';

const hash = new JoopHashService();

// Cryptographic hashes
const h256 = await hash.sha256('data');     // hex string
const h512 = await hash.sha512('data');
const hmac = await hash.hmac('message', 'secret-key');    // HMAC-SHA256

// Password storage (PBKDF2 — safe for storing passwords)
const stored  = await hash.hashPassword('my-password');
// { hash: '<hex>', salt: '<hex>' }

const matches = await hash.verifyPassword('my-password', stored.hash, stored.salt);   // true
const wrong   = await hash.verifyPassword('wrong',       stored.hash, stored.salt);   // false

// File integrity
const checksum = await hash.checksum(fileBuffer);          // SHA-256 hex
const valid    = await hash.verifyChecksum(fileBuffer, checksum);
```

---

## Reactive Observables

```typescript
import { JoopSubject, JoopBehaviorSubject } from 'joopjs';

// ── JoopSubject — event stream with no initial value ──
const paymentEvents$ = new JoopSubject<{ txnId: string; status: string }>();

const unsub = paymentEvents$.subscribe(event => {
  console.log('Payment:', event.txnId, event.status);
});

paymentEvents$.next({ txnId: 'TXN-001', status: 'completed' });

unsub();   // stop listening

// ── JoopBehaviorSubject — holds current value, replays to new subscribers ──
const connectionState$ = new JoopBehaviorSubject<'connected' | 'disconnected'>('connected');

// New subscribers immediately receive the current value
const unsub2 = connectionState$.subscribe(state => updateStatusBar(state));
// → immediately calls updateStatusBar('connected')

connectionState$.next('disconnected');   // all subscribers notified

const current = connectionState$.getValue();   // 'disconnected'

unsub2();

// ── Building reactive services ──
class MyPaymentService {
  private _status$ = new JoopBehaviorSubject<'idle'|'processing'|'done'>('idle');

  status$() { return this._status$; }   // expose as read-only

  async process(amount: number) {
    this._status$.next('processing');
    await submitPayment(amount);
    this._status$.next('done');
  }
}
```

**DO / DON'T table:**

| Action | Correct | Wrong |
|---|---|---|
| Emit a value | `.next(value)` | `.emit(value)` — does not exist |
| Get current value | `.getValue()` | `.value` — does not exist |
| Stop subscribing | Call the returned function `unsub()` | `.unsubscribe()` — does not exist |
| Use RxJS operators | Compose in your own code | `.pipe(map(...))` — not RxJS |

---

## Framework Integration

### React

```tsx
// singleton at module level — NOT inside a component
import { JoopDigitalWalletService, JoopAuthService } from 'joopjs';
const walletSvc = new JoopDigitalWalletService();
const authSvc   = new JoopAuthService();

// Hook pattern — subscribe in useEffect, return unsub for cleanup
import { useEffect, useState } from 'react';

function useWalletBalance(walletId: string) {
  const [balance, setBalance] = useState(0);
  useEffect(() => {
    return walletSvc.balance$().subscribe(({ walletId: id, balance }) => {
      if (id === walletId) setBalance(balance);
    });
  }, [walletId]);
  return balance;
}

function BalanceCard({ walletId }: { walletId: string }) {
  const balance = useWalletBalance(walletId);
  return <div>Balance: ${balance.toFixed(2)}</div>;
}

// Auth context
function useAuth() {
  const [user, setUser] = useState(authSvc.getCurrentUser());
  useEffect(() => {
    return authSvc.session$().subscribe(session => {
      setUser(session ? authSvc.getCurrentUser() : null);
    });
  }, []);
  return user;
}
```

---

### Angular

```typescript
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideJoop } from 'joopjs/angular';
import { createJoop } from 'joopjs';

export const appConfig: ApplicationConfig = {
  providers: [
    provideJoop(createJoop({ env: 'production', appId: 'banking-app' })),
  ],
};

// component.ts — inject pre-configured services
import { inject, Component, OnInit, OnDestroy } from '@angular/core';
import { JOOP_AUTH, JOOP_WALLET } from 'joopjs/angular';

@Component({ selector: 'app-dashboard', template: `<div>{{ balance }}</div>` })
export class DashboardComponent implements OnInit, OnDestroy {
  private auth   = inject(JOOP_AUTH);
  private wallet = inject(JOOP_WALLET);
  balance = 0;
  private unsub?: () => void;

  ngOnInit() {
    this.unsub = this.wallet.balance$().subscribe(({ balance }) => {
      this.balance = balance;
    });
  }

  ngOnDestroy() { this.unsub?.(); }
}
```

---

### Vue

```typescript
// main.ts
import { createApp } from 'vue';
import { createJoopVue } from 'joopjs/vue';
import { createJoop } from 'joopjs';
import App from './App.vue';

createApp(App)
  .use(createJoopVue(createJoop({ env: 'production', appId: 'banking-app' })))
  .mount('#app');
```

```vue
<!-- BalanceWidget.vue -->
<script setup lang="ts">
import { useJoopWallet } from 'joopjs/vue';
const { balance, topUp, pay } = useJoopWallet('wallet-001');
</script>

<template>
  <div class="balance-widget">
    <span>Balance: ${{ balance }}</span>
    <button @click="topUp(100)">+ $100</button>
  </div>
</template>
```

---

## Error Handling

All services throw standard `Error` objects. Use `try/catch` at every service call boundary:

```typescript
async function processPayment(walletId: string, amount: number) {
  try {
    const result = wallet.pay(walletId, amount, 'merchant-001', 'Purchase');
    return result;
  } catch (err) {
    if (err instanceof Error) {
      // Common messages:
      // 'Wallet not found: wallet-123'
      // 'Insufficient balance: have $50, need $100'
      // 'Wallet is frozen'
      // 'Cannot close wallet with non-zero balance'
      console.error('Payment failed:', err.message);
      throw err;
    }
  }
}
```

Services **never** swallow errors silently. A thrown error always means the operation did not complete.

---

## TypeScript Conventions

```typescript
// ── Data types ──
const amount:   number = 5000;        // never string or BigDecimal
const currency: string = 'USD';       // ISO-4217
const ts:       number = Date.now();  // epoch ms
const id:       string = 'u-001';

// ── All public types export as Joop* ──
import type {
  JoopWallet, JoopWalletStatus, JoopWalletTxnType,
  JoopLoanAccount, JoopInstallment, JoopRepaymentStatus,
  JoopFxForward, JoopForwardType, JoopForwardStatus,
  JoopMutualFund, JoopFundHolding, JoopSip, JoopSipStatus,
  JoopSanctionsEntity, JoopScreeningResult, JoopSanctionMatchType,
  JoopAuthUser, JoopAuthSession,
} from 'joopjs';
```

---

## Sub-path Imports & Tree-shaking

Import from sub-paths to include only what you need in your bundle:

```typescript
// Only pulls in encryption code
import { JoopGcmService } from 'joopjs/encryption';

// Only pulls in auth code
import { JoopAuthService, JoopJwtService } from 'joopjs/auth';

// Full India utilities
import { validatePan, validateUpi } from 'joopjs/india';
```

All 35 sub-paths have `"sideEffects": false` in the package manifest — bundlers (webpack, Vite, Rollup, esbuild) will tree-shake unused exports automatically.

---

## Testing with JoopJS

JoopJS services are plain classes — easy to test with any framework (Vitest, Jest, Mocha).

```typescript
import { describe, it, expect, beforeEach } from 'vitest';
import { JoopDigitalWalletService } from 'joopjs';

describe('Digital Wallet', () => {
  let svc: JoopDigitalWalletService;

  beforeEach(() => {
    svc = new JoopDigitalWalletService();   // fresh instance per test
  });

  it('creates a wallet with zero balance', () => {
    const w = svc.createWallet('u-001', { currency: 'USD' });
    expect(svc.getBalance(w.id)).toBe(0);
  });

  it('tops up balance', () => {
    const w = svc.createWallet('u-001', { currency: 'USD' });
    svc.topUp(w.id, 500);
    expect(svc.getBalance(w.id)).toBe(500);
  });

  it('throws when paying more than balance', () => {
    const w = svc.createWallet('u-001', { currency: 'USD' });
    svc.topUp(w.id, 100);
    expect(() => svc.pay(w.id, 200, 'merchant', 'Purchase')).toThrow();
  });

  it('emits balance change via observable', () => {
    const w = svc.createWallet('u-001', { currency: 'USD' });
    const received: number[] = [];
    const unsub = svc.balance$().subscribe(({ balance }) => received.push(balance));
    svc.topUp(w.id, 100);
    svc.topUp(w.id, 200);
    unsub();
    expect(received).toContain(100);
    expect(received).toContain(300);
  });
});
```

---

*Last updated: 2026 · Author: Kundan Singh*
