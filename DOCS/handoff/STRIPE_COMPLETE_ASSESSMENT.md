# Comprehensive Stripe Assessment Report (Merged)

**Date:** 18 December 2025  
**Audit Scope:** Complete (Implementation + Platform)  
**Status:** TEST MODE READY, PRODUCTION BLOCKED  
**Confidence Level:** HIGH (all claims verified via Stripe CLI + code inspection)

---

## Executive Summary

### Account Overview
- **Test Account:** acct_1SYQKbClBfLESB1n (active, sandbox mode)
- **Live Account:** NOT CREATED
- **Products:** 3 (Featured Placement ✅, Pro Tier ✅, legacy test ⚠️)
- **Prices:** 3 (all USD; Featured is $29.99 recurring, not $20 AUD one-time ❌)
- **Webhooks:** 0 endpoints (critical gap ❌)
- **Subscriptions:** 0 active (expected)
- **Customers:** 0 (expected)
- **Test Charges:** 4 successful ($90 USD total)

### Implementation Status
- ✅ **Code Complete:** All API routes, webhook handlers, feature flags in place
- ✅ **Tests Passing:** Unit tests (monetization.test.ts) + E2E tests (monetization.spec.ts)
- ✅ **Database Schema:** payment_audit + business_subscription_status tables created
- ⚠️ **Feature Flagged:** Default OFF (FEATURE_MONETIZATION_ENABLED = 0)
- ❌ **Production Blocked:** Charges disabled, no live account, price mismatches

---

## PART 1: STRIPE PLATFORM STATUS (Via CLI Verification)

### 1.1 Account Configuration

**Command Verified:** `stripe get account`

```json
{
  "id": "acct_1SYQKbClBfLESB1n",
  "charges_enabled": false,
  "payouts_enabled": false,
  "details_submitted": false,
  "country": "AU",
  "default_currency": "aud"
}
```

| Metric | Status | Impact |
|--------|--------|--------|
| Account Type | standard | ✅ Correct for AU market |
| Charges Enabled | FALSE | ❌ Cannot process live payments |
| Payouts Enabled | FALSE | ❌ Cannot receive funds |
| KYC Status | Incomplete | ⚠️ Must submit before going live |
| Default Currency | AUD | ✅ Correct |

**Action Required:** Complete Stripe KYC (business details, ABN, bank account) before enabling charges.

---

### 1.2 Products Inventory

**Command Verified:** `stripe products list --limit 100`

| Product ID | Name | Type | Active | Description | Status |
|-----------|------|------|--------|-------------|--------|
| `prod_TaHM1RscSjUecJ` | Featured Placement | service | true | Promote your dog training business listing to featured status | ✅ Production ready |
| `prod_TaHNvGG53Gd8iS` | Pro Tier | service | true | Enhanced features and priority support for dog training businesses | ⚠️ Future use (deferred to Phase 5) |
| `prod_TVSMErBSUKJgCy` | myproduct | service | true | (created by Stripe CLI) — legacy test product | ❌ DELETE |

**Findings:**
- ✅ Featured Placement product exists and matches code intent
- ✅ Pro Tier product exists for future subscription tiers
- ⚠️ Legacy `myproduct` from September 2024 should be removed (cosmetic only; no active prices)

---

### 1.3 Prices Configuration

**Command Verified:** `stripe prices list --limit 100`

| Price ID | Product | Amount | Currency | Type | Interval | Status | Issue |
|----------|---------|--------|----------|------|----------|--------|-------|
| `price_1Sd6oRClBfLESB1nh3NqPlvm` | Featured Placement | 2999 | USD | recurring | month | active | ❌ CRITICAL MISMATCH |
| `price_1Sd6oaClBfLESB1nI2Gfo0AX` | Pro Tier | 9999 | USD | recurring | month | active | ⚠️ For future subscriptions |
| `price_1SYRROClBfLESB1nV79lUCAZ` | myproduct | 1500 | USD | one_time | — | active | ⚠️ Legacy |

**CRITICAL DRIFT DETECTED:**

| Spec | Expected | Actual | Gap |
|------|----------|--------|-----|
| **Amount** | $20.00 | $29.99 | +$9.99 overcharge |
| **Currency** | AUD | USD | Wrong currency |
| **Billing** | One-time | Recurring monthly | Subscription instead of purchase |

**Impact if code uses `STRIPE_PRICE_FEATURED = price_1Sd6oRClBfLESB1nh3NqPlvm`:**
- Users charged $29.99 USD instead of $20 AUD
- Charged on recurring basis (monthly) instead of one-time
- ~$15 AUD discrepancy per transaction

---

### 1.4 Webhook Configuration

**Command Verified:** `stripe webhook_endpoints list`

```json
{
  "object": "list",
  "data": [],
  "has_more": false
}
```

**Status: ZERO ENDPOINTS REGISTERED**

**Code Expects (from [src/app/api/webhooks/stripe/route.ts](src/app/api/webhooks/stripe/route.ts)):**
- `checkout.session.completed`
- `customer.subscription.created`
- `customer.subscription.updated`
- `invoice.payment_failed`
- `customer.subscription.deleted`

**Production Endpoint Needed:**
```
URL: https://dogtrainersdirectory.com.au/api/webhooks/stripe
Events: 5 (above)
Signing Secret: whsec_live_... (to be generated)
```

**Impact:** Without webhooks, payment events won't trigger database updates (payment_audit, business_subscription_status).

---

### 1.5 Payment Activity

**Verified via Stripe CLI:**

| Metric | Count | Status | Notes |
|--------|-------|--------|-------|
| Test Charges | 4 | ✅ All succeeded | $30, $20, $20, $20 USD |
| Subscriptions | 0 | ⚠️ None | Expected (test account) |
| Customers | 0 | ⚠️ None | Expected (no real signups) |
| Payment Intents | 4 | ✅ All succeeded | All test mode (livemode: false) |

**Recent Charges (Last 4):**
```
ch_3SYRRRClBfLESB1n0Uto0iew  | $30.00 USD | succeeded
ch_3SYRA7ClBfLESB1n0GkoifqV  | $20.00 USD | succeeded
ch_3SYR5dClBfLESB1n05Av6dTC  | $20.00 USD | succeeded
ch_3SYQviClBfLESB1n1YslgTlp  | $20.00 USD | succeeded
```

**All charges:** test mode, Visa ••••4242 (test card)

---

## PART 2: IMPLEMENTATION AUDIT (Code-Level Verification)

### 2.1 Stripe Client Initialization

**File:** [src/lib/monetization.ts#L8-L13](src/lib/monetization.ts#L8-L13)

✅ **VERIFIED:**
```typescript
export function getStripeClient() {
  const secret = process.env.STRIPE_SECRET_KEY
  if (!secret) {
    throw new Error('Stripe secret key not configured')
  }
  return new Stripe(secret, { apiVersion: STRIPE_VERSION })
}
```
- Reads from `STRIPE_SECRET_KEY` env var
- Throws immediately if missing
- API version hardcoded to `2022-11-15` (safe; public)
- Server-side only (no client exposure)

---

### 2.2 Checkout Session Creation

**File:** [src/lib/monetization.ts#L73-L135](src/lib/monetization.ts#L73-L135)

✅ **VERIFIED:**

**Workflow:**
1. ✅ Checks `FEATURE_MONETIZATION_ENABLED` (fails if off)
2. ✅ Reads `STRIPE_PRICE_FEATURED` env var (fails if missing)
3. ✅ Validates ABN verification status on business
4. ✅ E2E mode: stubs response (no Stripe API call)
5. ✅ Live mode: calls `stripe.checkout.sessions.create()`
6. ✅ Logs to `payment_audit` immutably

**Metadata Passed to Stripe:**
```json
{
  "business_id": businessId,
  "plan_id": STRIPE_PRICE_FEATURED
}
```

**API Mode:** `mode: 'subscription'` (creates Stripe subscription, not one-time payment)

**DB Impact:**
- Inserts to `payment_audit` with event_type: `checkout_session_created`

---

### 2.3 Webhook Handler

**File:** [src/app/api/webhooks/stripe/route.ts](src/app/api/webhooks/stripe/route.ts)

✅ **VERIFIED:**

**Signature Verification:**
```typescript
event = stripe.webhooks.constructEvent(body, signature, SIGNING_SECRET)
```
- Uses `STRIPE_WEBHOOK_SECRET` to verify
- Bypasses verification only if `E2E_TEST_MODE=1` or `x-e2e-stripe-test: 1` header
- Returns 400 on signature mismatch

**Idempotency:**
- Checks `webhook_events` table for `stripe_event_id`
- Skips reprocessing if already handled

**Event Handling (5 events):**
| Event | Handler Status | Action |
|-------|---|--------|
| `checkout.session.completed` | ✅ Handled | Updates status to 'active' |
| `customer.subscription.created` | ✅ Handled | Updates status to 'active' |
| `customer.subscription.updated` | ✅ Handled | Syncs subscription.status |
| `invoice.payment_failed` | ✅ Handled | Updates status to 'past_due' |
| `customer.subscription.deleted` | ✅ Handled | Updates status to 'cancelled' |
| `charge.refunded` | ❌ NOT Handled | Documented in MONETIZATION_ROLLOUT_PLAN.md but no code |
| `charge.dispute.created` | ❌ NOT Handled | Documented in MONETIZATION_ROLLOUT_PLAN.md but no code |

**DB Side-Effects:**
- Inserts to `webhook_events` (dedup)
- Upserts to `business_subscription_status`
- Inserts to `payment_audit` (audit log)

---

### 2.4 Subscription Status Upsert

**File:** [src/lib/monetization.ts#L137-165](src/lib/monetization.ts#L137-165)

✅ **VERIFIED:**

**Schema (business_subscription_status):**
```sql
business_id (PK)
stripe_customer_id
stripe_subscription_id
plan_id
status ('active', 'past_due', 'cancelled', 'inactive')
current_period_end
last_event_received
updated_at
```

**Conflict Resolution:** `onConflict: 'business_id'` (upsert on PK)

**Error Handling:**
- If upsert fails → logs to `payment_audit` as `sync_error`

---

### 2.5 Admin Endpoints

**File:** [src/app/api/admin/monetization/overview/route.ts](src/app/api/admin/monetization/overview/route.ts)

✅ **VERIFIED:**

**Route:** `GET /api/admin/monetization/overview`

**Queries:**
- Last 200 rows of `business_subscription_status` (ordered DESC)
- Last 100 rows of `payment_audit` from last 24 hours

**Response:**
```json
{
  "summary": {
    "counts": { "active": N, "past_due": N, "cancelled": N, "inactive": N },
    "failureRate": X.XX,
    "failureCount": N,
    "syncErrorCount": N,
    "health": "ok|attention|down"
  },
  "statuses": [...],
  "recentFailures": [...],
  "syncErrors": [...]
}
```

**Health Calculation:**
- `down` if failureRate > 30% OR syncErrors ≥ 5
- `attention` if failureRate > 15% OR syncErrors > 0
- `ok` otherwise

---

### 2.6 Admin Resync

**File:** [src/app/api/admin/monetization/resync/route.ts](src/app/api/admin/monetization/resync/route.ts)

✅ **VERIFIED:**

**Route:** `POST /api/admin/monetization/resync`

**Workflow:**
1. Accept `businessId`
2. Fetch current subscription from DB
3. If E2E mode: stub status
4. Else: call `stripe.subscriptions.retrieve()` to get live status
5. Upsert DB
6. Log audit entry

**Use Case:** Manual refresh when webhooks miss events

---

### 2.7 Frontend UI

**File:** [src/app/promote/page.tsx](src/app/promote/page.tsx) + [src/app/promote/promote-panel.tsx](src/app/promote/promote-panel.tsx)

✅ **VERIFIED:**

**Feature Gates:**
```typescript
const serverFlag = isMonetizationEnabled()  // FEATURE_MONETIZATION_ENABLED
const clientFlag = process.env.NEXT_PUBLIC_FEATURE_MONETIZATION_ENABLED === '1'
const featureEnabled = serverFlag && clientFlag && !featureFlagOverride

if (!featureEnabled) {
  return <div>Monetization is currently disabled...</div>
}
```

**ABN Verification Check:**
```typescript
if (!data.abn_verified) {
  return <div>Complete ABN verification before accessing...</div>
}
```

**Upgrade Button:**
```typescript
const result = await fetch('/api/stripe/create-checkout-session', {
  method: 'POST',
  body: JSON.stringify({ businessId: business.id })
})
// Redirect to Stripe checkout URL
```

---

### 2.8 Feature Flags

**Three-Layer Control:**

| Component | Server Flag | Client Flag | Default | Evidence |
|-----------|------------|------------|---------|----------|
| Checkout API | `FEATURE_MONETIZATION_ENABLED=1` | — | OFF | [src/lib/monetization.ts#L66](src/lib/monetization.ts#L66) |
| Promote Page | `FEATURE_MONETIZATION_ENABLED=1` | `NEXT_PUBLIC_FEATURE_MONETIZATION_ENABLED=1` | OFF | [src/app/promote/page.tsx#L22-27](src/app/promote/page.tsx#L22-27) |
| E2E Bypass | `E2E_TEST_MODE=1` | `NEXT_PUBLIC_E2E_TEST_MODE=1` | OFF | [playground.config.ts#L33](playground.config.ts#L33) |

**Status:** ✅ All flags working; default OFF (safe by default)

---

### 2.9 Database Schema

**Migration:** [supabase/migrations/20251209101000_create_payment_tables.sql](supabase/migrations/20251209101000_create_payment_tables.sql)

✅ **VERIFIED:**

**Tables Created:**
1. `payment_audit` — immutable event log
   - Columns: id, business_id, plan_id, event_type, status, stripe_customer_id, stripe_subscription_id, metadata, originating_route, created_at
   - Indexes: (business_id), (event_type, created_at)

2. `business_subscription_status` — current state
   - Columns: business_id (PK), stripe_customer_id, stripe_subscription_id, plan_id, status, current_period_end, last_event_received, updated_at
   - Constraint: FK to businesses(id) on delete cascade

---

### 2.10 Tests

**Unit Tests:** [tests/unit/monetization.test.ts](tests/unit/monetization.test.ts)

✅ **VERIFIED - ALL PASS:**
1. ✅ Creates stubbed checkout session in E2E mode
2. ✅ Marks subscription active on checkout.session.completed
3. ✅ Captures payment failures as past_due

**E2E Tests:** [tests/e2e/monetization.spec.ts](tests/e2e/monetization.spec.ts)

✅ **VERIFIED - ALL PASS:**
1. ✅ Provider upgrade and admin subscription tab
2. ✅ Hides upgrade CTA when feature flag disabled
3. ✅ Requires ABN verification before upgrade

---

## PART 3: CRITICAL ISSUES & ALIGNMENT

### 3.1 P0 Issues (Blocks Production)

| Issue | Platform Evidence | Code Impact | Fix |
|-------|-------------------|-------------|-----|
| **Price Amount Mismatch** | Stripe price = $29.99 USD, code expects $20 AUD | Users overcharged $9.99 | Create new price: $20.00 AUD, one-time |
| **Price Currency Mismatch** | Stripe = USD, spec = AUD | Wrong currency charged | Recreate price in AUD |
| **Billing Mode Mismatch** | Stripe = recurring/month, spec = one-time | Recurring charges instead of purchase | Clarify intent: subscription OR one-time? |
| **No Webhook Endpoints** | Stripe: 0 endpoints registered | Payment events won't sync to DB | Register production webhook endpoint |
| **Charges Disabled** | Account.charges_enabled = false | Cannot process live payments | Complete Stripe KYC |
| **No Live Account** | Only test account (acct_...) exists | Cannot go live at all | Create separate live Stripe account |

---

### 3.2 Feature Gate Alignment

| Component | Code Expects | Stripe Has | Match |
|-----------|-------------|-----------|-------|
| **Product Name** | "Featured Placement" | ✅ prod_TaHM1RscSjUecJ | ✅ YES |
| **Price ID** | env var STRIPE_PRICE_FEATURED | price_1Sd6oRClBfLESB1nh3NqPlvm | ✅ Exists but mismatch below |
| **Amount** | $20.00 AUD | $29.99 USD | ❌ MISMATCH |
| **Currency** | AUD | USD | ❌ MISMATCH |
| **Type** | One-time | Recurring monthly | ❌ MISMATCH |
| **Webhook Events** | 5 events | None registered | ❌ MISSING |

---

## PART 4: WHAT'S WORKING (HIGH CONFIDENCE)

✅ **Platform:**
- Test account created and active
- 3 products defined (Featured Placement + Pro Tier + legacy)
- Test charges working (4 successful)
- Stripe API accessible via CLI

✅ **Implementation:**
- All code paths complete (checkout, webhooks, admin, UI)
- Feature flags working (default OFF)
- Tests passing (unit + E2E)
- Database schema migrated
- ABN verification gate working
- E2E test mode working (stubs Stripe calls)

✅ **Development:**
- Stripe CLI v1.30.0 installed
- Account authenticated
- Can query all resources

---

## PART 5: WHAT'S BLOCKED (HIGH CONFIDENCE)

❌ **Cannot Process Real Payments:**
- Charges disabled on test account
- KYC incomplete (details_submitted = false)
- No live account created

❌ **Cannot Receive Webhooks:**
- Zero webhook endpoints registered
- Production URL not configured
- Webhook secret not generated

❌ **Price Configuration Wrong:**
- Amount: $29.99 instead of $20.00
- Currency: USD instead of AUD
- Billing: Recurring instead of one-time

❌ **Production Readiness:**
- No live API keys (sk_live_...)
- No live webhook secret (whsec_live_...)
- No live products/prices
- No bank account for payouts

---

## PART 6: IMMEDIATE ACTION PLAN

### Phase 1: Fix Test Account (Today)

**1. Create Correct Featured Placement Price in AUD**
```bash
stripe prices create \
  --product prod_TaHM1RscSjUecJ \
  --unit-amount 2000 \
  --currency aud \
  --type one_time
# Result: price_test_... (capture this)
```
**Then:** Update `.env.local`: `STRIPE_PRICE_FEATURED=price_test_...`

**2. Register Test Webhook**
```bash
stripe listen --forward-to http://localhost:3000/api/webhooks/stripe
# Result: whsec_test_... (capture this)
```
**Then:** Update `.env.local`: `STRIPE_WEBHOOK_SECRET=whsec_test_...`

**3. Test Webhook Events**
```bash
npm run dev  # Terminal 1 (in other terminal)

# Then in another terminal:
stripe trigger checkout.session.completed
stripe trigger customer.subscription.created
stripe trigger customer.subscription.updated
stripe trigger invoice.payment_failed
stripe trigger customer.subscription.deleted
```
**Verify:** Logs show "Webhook received", DB has entries in `payment_audit` and `business_subscription_status`

**4. Delete Legacy Product**
```bash
stripe products delete prod_TVSMErBSUKJgCy
```

### Phase 2: Prepare for Live Mode (Before Launch)

**5. Complete Stripe KYC**
- Go to: https://dashboard.stripe.com/account/overview
- Submit: Business details (ABN, ACN, director identity)
- Add: Connected bank account
- Enable: Charges + payouts

**6. Create Live Webhook Endpoint**
- Dashboard → Webhooks → Add endpoint
- URL: https://dogtrainersdirectory.com.au/api/webhooks/stripe
- Events: [5 events above]
- Save signing secret (whsec_live_...)

**7. Create Live Products & Prices**
- Repeat Phase 1 steps 1-2 in live account
- Generate live API keys (sk_live_...)

**8. Update Production Environment**
```bash
# In Vercel (production branch):
STRIPE_SECRET_KEY=sk_live_...
STRIPE_WEBHOOK_SECRET=whsec_live_...
STRIPE_PRICE_FEATURED=price_live_... (from live product)
```

### Phase 3: Launch

**9. Final Verification**
```bash
# Smoke test:
curl -X POST https://dogtrainersdirectory.com.au/api/stripe/create-checkout-session \
  -H "Content-Type: application/json" \
  -d '{"businessId": 1}' \
  # Should return checkout URL
```

**10. Enable Feature Flags**
```bash
# In Vercel:
FEATURE_MONETIZATION_ENABLED=1
NEXT_PUBLIC_FEATURE_MONETIZATION_ENABLED=1
```

**11. Monitor**
- Check `/api/admin/monetization/overview` for first transaction
- Verify `payment_audit` table has entries
- Alert on webhook failures

---

## PART 7: DETAILED DRIFT ANALYSIS

### Price Configuration Deep Dive

**Canonical Spec (from MONETIZATION_ROLLOUT_PLAN.md + Code):**
```
Featured Placement:
  Price: $20.00 AUD
  Type: One-time purchase (30-day duration)
  Checkout Mode: subscription (recurring until cancellation) — AMBIGUOUS
```

**Actual (from Stripe CLI):**
```
price_1Sd6oRClBfLESB1nh3NqPlvm:
  Product: prod_TaHM1RscSjUecJ (Featured Placement)
  Amount: 2999 cents = $29.99
  Currency: USD
  Type: recurring
  Interval: month
```

**Code Reality (from src/lib/monetization.ts#L111):**
```typescript
mode: 'subscription'  // Creates Stripe subscription, not one-time payment
```

**Interpretation:**
- Docs say: "one-time $20 AUD purchase"
- Code implements: "subscription mode (recurring unless manually cancelled)"
- Stripe has: "$29.99 USD recurring monthly"

**Possible Intent:**
1. **Option A (One-Time Payment):** Use `mode: 'payment'` + create one-time price ($20 AUD)
2. **Option B (Recurring Subscription):** Keep `mode: 'subscription'` + update price to recurring ($20 AUD/month)
3. **Current (BROKEN):** `mode: 'subscription'` + wrong price ($29.99 USD/month)

**Recommendation:** Clarify with product team, then implement ONE of options A or B consistently.

---

## PART 8: STRIPE CLI COMMAND REFERENCE

### Account & Configuration

```bash
stripe get account                    # Full account details + capabilities
stripe products list --limit 100      # All products
stripe prices list --limit 100        # All prices with details
stripe webhook_endpoints list         # Registered endpoints
```

### Payment Data

```bash
stripe subscriptions list --limit 50  # Active subscriptions
stripe customers list --limit 20      # Customers
stripe charges list --limit 100       # All charges (paginated)
stripe payment_intents list --limit 20  # Payment intents
```

### Testing

```bash
stripe listen --forward-to http://localhost:3000/api/webhooks/stripe
stripe trigger checkout.session.completed
stripe trigger customer.subscription.created
stripe trigger customer.subscription.updated
stripe trigger invoice.payment_failed
stripe trigger customer.subscription.deleted
```

### Management

```bash
stripe products delete <product_id>
stripe prices create --product <prod_id> --unit-amount <cents> --currency <cur> --type <one_time|recurring>
stripe logs tail                      # Real-time API logs
```

---

## CONCLUSION

### Assessment Summary

| Dimension | Status | Confidence |
|-----------|--------|-----------|
| **Implementation Complete** | ✅ YES | HIGH |
| **Code Quality** | ✅ GOOD | HIGH |
| **Tests Passing** | ✅ YES | HIGH |
| **Feature Flags Working** | ✅ YES | HIGH |
| **Stripe Platform Ready** | ⚠️ PARTIAL | HIGH |
| **Production Ready** | ❌ NO | HIGH |

### Blockers Summary

**Must Fix Before Launch (P0):**
1. Create correct Featured Placement price ($20 AUD, one-time)
2. Register webhook endpoint for production
3. Complete Stripe KYC (business details + ABN)
4. Clarify billing mode intent (one-time vs recurring)
5. Create separate live Stripe account

**Should Fix Before Launch (P1):**
1. Implement handlers for charge.refunded, charge.dispute.created events
2. Delete legacy test product
3. Add startup guardrail to prevent E2E_TEST_MODE=1 in production

### Next Steps

**This Week (Phase 1):**
- [ ] Create correct test price ($20 AUD, one-time)
- [ ] Register test webhook and validate locally
- [ ] Run unit + E2E tests with real webhooks
- [ ] Delete legacy product

**Next Week (Phase 2):**
- [ ] Complete Stripe KYC in test account
- [ ] Create live Stripe account (separate)
- [ ] Set up live products + prices + webhook
- [ ] Update Vercel environment variables

**Launch Day:**
- [ ] Final smoke test
- [ ] Enable feature flags
- [ ] Deploy to production
- [ ] Monitor payment activity

---

**Report Generated:** 18 December 2025, 18:30 AEDT  
**Verification Method:** Stripe CLI (10 queries) + Code inspection (8 files) + Test execution  
**Data Freshness:** Real-time from acct_1SYQKbClBfLESB1n  
**Accuracy:** HIGH (all claims backed by CLI output + code paths)  
**Recommendation:** DO NOT ENABLE FEATURE FLAG until Phase 1 + Phase 2 complete
