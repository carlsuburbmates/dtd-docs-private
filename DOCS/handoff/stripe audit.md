Read [](file:///Users/carlg/Documents/PROJECTS/Project-dev/DTD/DOCS/MONETIZATION_ROLLOUT_PLAN.md#1-1), lines 1 to 150

Based on **MONETIZATION_ROLLOUT_PLAN.md** and cross-referenced with SSOT and STRIPE_COMPLETE_ASSESSMENT.md, here's what **SHOULD be created and expected** in the Stripe platform per documentation:

---

## STRIPE PLATFORM: EXPECTED STATE (Per Docs)

### 1. PRODUCTS (Should have: 1 production product)

| Product | Name | Purpose | Status |
|---------|------|---------|--------|
| ✅ **REQUIRED** | Featured Placement | One-time $20 AUD / 30-day listing boost | MUST EXIST |
| ⚠️ OPTIONAL (Future) | Pro Tier | Future subscription tier (Phase 5+, deferred) | MAY EXIST but unused |

**Current Reality:** 3 products exist (Featured ✅, Pro ✅, legacy test ❌)

---

### 2. PRICES (Should have: 1 production price)

| Price | Amount | Currency | Type | Interval | For | Status |
|-------|--------|----------|------|----------|-----|--------|
| ✅ **REQUIRED** | $20.00 | AUD | **ONE-TIME** | N/A | Featured Placement | MUST EXIST |
| ⚠️ OPTIONAL | $X.XX | AUD | recurring | month | Pro Tier (future) | MAY EXIST but unused |

**Spec (from MONETIZATION_ROLLOUT_PLAN.md):**
```
Featured Placement: $20 AUD / 30-day placement / one-time purchase
```

**Current Reality:** ❌ MISMATCH
- Actual: `$29.99 USD, recurring/month`
- Expected: `$20.00 AUD, one-time`

---

### 3. WEBHOOK ENDPOINTS (Should have: 1 production endpoint)

**Required Endpoint:**
```
URL: https://dogtrainersdirectory.com.au/api/webhooks/stripe
Signing Secret: whsec_live_... (to be generated)
Events: 6 (list below)
```

**Events to Register (Per MONETIZATION_ROLLOUT_PLAN.md):**

| Event | Handler Status | Required | Notes |
|-------|---|---|---|
| `checkout.session.completed` | ✅ Implemented | **CRITICAL** | Validate metadata, create payment_audit row, activate slot or enqueue |
| `payment_intent.succeeded` | ✅ Implemented | OPTIONAL | Guard case for redirect failures |
| `invoice.payment_failed` | ✅ Implemented | OPTIONAL | Future subscriptions + alerting |
| `customer.subscription.deleted` | ✅ Implemented | OPTIONAL | Future subscription tiers |
| `charge.refunded` | ❌ NOT implemented | **DEFERRED** | Refund flow (not in Phase 9A) |
| `charge.dispute.created` | ❌ NOT implemented | **DEFERRED** | Chargeback handling (not in Phase 9A) |

**Current Reality:** ❌ ZERO ENDPOINTS REGISTERED

---

### 4. CHECKOUT METADATA (Required Fields)

**Per MONETIZATION_ROLLOUT_PLAN.md:**
> Checkout metadata (required): `trainer_id`, `business_id`, `lga_id`, `tier`, and optional context (`desired_start`, `queue_position_hint`). Webhooks must reject any session lacking these fields.

**Metadata Contract:**
```json
{
  "business_id": "UUID",
  "trainer_id": "UUID",
  "lga_id": "UUID",
  "tier": "basic",
  "desired_start": "YYYY-MM-DD",
  "queue_position_hint": "N"
}
```

**Current Implementation (from src/lib/monetization.ts):**
```typescript
metadata: {
  business_id: businessId,
  plan_id: STRIPE_PRICE_FEATURED
}
```

⚠️ **GAP:** Code passes only `business_id` + `plan_id`; missing `trainer_id`, `lga_id`, `tier`

---

### 5. PAYMENT FLOW & STATE TRANSITIONS

**Per MONETIZATION_ROLLOUT_PLAN.md, Webhook section:**

```
1. checkout.session.completed fires
   ↓
2. Validate metadata (trainer_id, business_id, lga_id, tier present)
   ↓
3. Log to payment_audit: event_type = "checkout_session_completed"
   ↓
4. Check featured_slots_status for LGA:
   - If current_featured_count < 5: ACTIVATE immediately
   - Else: ENQUEUE in featured_queue
   ↓
5. Insert metrics row (start_timestamp, end_timestamp = start + 30 days)
   ↓
6. Send notification email ("Placement live" or "You're position N")
```

**DB Tables Required:**
- `payment_audit` (immutable event log) ✅ EXISTS
- `business_subscription_status` (current state) ✅ EXISTS
- **MISSING:** `featured_slots_status` (per-LGA slot counter)
- **MISSING:** `featured_queue` (waiting list)
- **MISSING:** `featured_placements_metrics` (start/end timestamps + activation state)

---

### 6. REFUND & DISPUTE HANDLING (Deferred to Phase 9C+)

**Per MONETIZATION_ROLLOUT_PLAN.md:**

| Event | Expected Behavior | Status |
|-------|-------------------|--------|
| **Refund requested** | Window: 3 days after purchase; refund via Stripe → webhook mirrors in `payment_audit` + deactivate slot | **NOT YET IMPLEMENTED** |
| **Dispute/chargeback** | On `charge.dispute.created`, create support ticket, lock placement, surface in admin UI | **NOT YET IMPLEMENTED** |

---

### 7. ADMIN SURFACE & TELEMETRY

**Per MONETIZATION_ROLLOUT_PLAN.md + Code:**

| Component | Expected | Status |
|-----------|----------|--------|
| `/api/admin/monetization/overview` | Query `payment_audit` + `business_subscription_status`, calculate failure rates, health status | ✅ IMPLEMENTED |
| "Subscription Health" admin card | Show active slots, queue lengths, failure/sync counts, latency | ✅ CODE PATH EXISTS |
| Alert triggers | Latency spike (`monetization_api`), payment failures, sync errors | ✅ ALERTS DEFINED |
| Admin resync endpoint | `/api/admin/monetization/resync` for manual webhook replay | ✅ IMPLEMENTED |

---

### 8. FEATURE FLAGS (Required: Default OFF)

**Per MONETIZATION_ROLLOUT_PLAN.md:**

| Flag | Server/Client | Default | Required |
|------|---|---|---|
| `FEATURE_MONETIZATION_ENABLED` | Server | `0` (OFF) | ✅ YES |
| `NEXT_PUBLIC_FEATURE_MONETIZATION_ENABLED` | Client | `0` (OFF) | ✅ YES |
| `E2E_TEST_MODE` | Both | `0` (OFF) | ✅ YES (for testing) |

**Current State:** ✅ All defaulting to OFF (safe by default)

---

### 9. SECURITY & COMPLIANCE

**Per MONETIZATION_ROLLOUT_PLAN.md:**

| Requirement | Expected | Status |
|-------------|----------|--------|
| Signature verification | ALWAYS verify `Stripe-Signature` using `STRIPE_WEBHOOK_SECRET` | ✅ IMPLEMENTED |
| Bypass for testing | Only allow E2E bypass via `E2E_TEST_MODE=1` or `x-e2e-stripe-test` header | ✅ IMPLEMENTED |
| Idempotency | Skip duplicate `event.id` values; store in `payment_audit` | ✅ IMPLEMENTED |
| Production never unsigned | Local dev can bypass; production/staging must NEVER accept unsigned payloads | ✅ ENFORCED |
| Refund policy | 3 days post-purchase only; thereafter non-refundable (delivered service) | **NOT ENFORCED IN CODE** |
| Tax compliance | Retain records 7 years (ATO); Stripe retains 2.9% + $0.30 fees | **OPERATIONAL** |

---

### 10. LAUNCH GATES (From MONETIZATION_ROLLOUT_PLAN.md)

**Must be satisfied before enabling feature flags:**

| Gate | Requirement | Evidence |
|------|-------------|----------|
| 1. Feature flag | Flip ON only after go-live rehearsal | Documented sign-off |
| 2. Webhook dry-run | Verified via staging `scripts/preprod_e2e.sh` + manual Stripe CLI replay | Evidence in launch_runs/ |
| 3. Payment audit trail | ≥1 `checkout_session_created` + `subscription_active` entry in staging | Query output captured |
| 4. Alerts + dashboards | Admin "Subscription Health" shows OK; no unsuppressed warnings | Dashboard screenshot |
| 5. Launch checklist | `DOCS/LAUNCH_READY_CHECKLIST.md` updated with E2E run + webhook + audit evidence | Link in section 10 |

**Current Status:** ❌ GATES NOT MET (webhook endpoints not registered, price misconfigured)

---

## SUMMARY: STRIPE PLATFORM CHECKLIST

### ✅ EXISTS & CORRECT
- [x] Test account created (acct_1SYQKbClBfLESB1n)
- [x] Featured Placement product exists
- [x] Test charges working (4 successful)
- [x] Stripe API accessible

### ❌ MISSING OR WRONG
- [ ] **Featured Placement price: WRONG** ($29.99 USD ≠ $20.00 AUD; recurring ≠ one-time)
- [ ] **Webhook endpoints: ZERO** (should register 1 to https://...)
- [ ] `featured_slots_status` table (to track per-LGA slot limits)
- [ ] `featured_queue` table (to manage queue when >5 per council)
- [ ] `featured_placements_metrics` table (to track activation start/end times)
- [ ] Charges enabled on account (currently disabled)
- [ ] Live account created (only test account exists)
- [ ] KYC completed (details_submitted = false)

### ⚠️ DEFERRED (Phase 9C+)
- Refund webhook handler (`charge.refunded`)
- Dispute webhook handler (`charge.dispute.created`)
- Pro Tier product + pricing
- Subscription lifecycle management

---

**Reference:** STRIPE_COMPLETE_ASSESSMENT.md for detailed platform audit + action plan.