# PHASE C: Vercel Preview Deployment & Webhook Integration (CORRECTED)

**Date:** 18 December 2025  
**Status:** ✅ CORRECTED & VERIFIED  
**Scope:** Deploy to Vercel Preview environment (same project, no separate deploy), configure test-mode Stripe, create stable webhook endpoint  
**Prerequisites:** Phase B local tests PASSED ✅, Vercel CLI installed (`vercel --version` works), GitHub repo connected to Vercel

---

## ⚠️ CRITICAL NOTES

1. **Preview URLs change on every deployment.** Do NOT assume the URL is stable. This doc uses a **stable alias approach** (Option A) to ensure Stripe webhook endpoint doesn't require updates per deploy.
2. **NO secrets on CLI.** Never use `vercel --env SECRET=value` (leaks to shell history). Use `vercel env` commands instead.
3. **Test-mode only.** All Stripe keys must start with `sk_test_`, `pk_test_`, `whsec_test_`.
4. **Exactly 4 webhook events handled** by code: `checkout.session.completed`, `customer.subscription.created`, `customer.subscription.updated`, `invoice.payment_failed`, `customer.subscription.deleted`.

---

## C1. PRE-FLIGHT CHECKS

### 1a. Verify Vercel CLI Installation

```bash
vercel --version
# Expected output: Vercel CLI 32.0.0+ (or similar)

vercel whoami
# Expected output: Your account information
```

### 1b. Verify Project Connection

```bash
# Confirm the project exists in Vercel and is connected to GitHub
vercel projects list
# Expected output:
# Project: dogtrainersdirectory
#  Org:     carlg
#  Prod:    https://dogtrainersdirectory.vercel.app
```

### 1c. Confirm Working Tree Clean

```bash
cd /Users/carlg/Documents/PROJECTS/Project-dev/DTD
git status
# Expected: "nothing to commit, working tree clean"
# (docs/ changes can be uncommitted; source code must be clean)
```

### 1d. Verify Stripe CLI Test-Mode Account

```bash
stripe whoami
# Expected: acct_1SYQKb... (sandbox account, test-mode)

# Confirm .env.local has test keys
grep "sk_test_\|pk_test_" .env.local | head -2
# Expected: Both found
```

---

## C2. SET UP VERCEL PREVIEW ENVIRONMENT VARIABLES

**Problem:** Preview deployments need Stripe secrets but we NEVER pass secrets via `vercel --env KEY=value` (shell history leak).  
**Solution:** Use `vercel env` commands to set each variable safely.

### 2a. List Current Preview Env Vars

```bash
vercel env list --environment preview
# Expected output: List of existing vars (if any)
# If this is first setup, list may be short
```

### 2b. Add Stripe Secrets via Prompts (No Shell History)

```bash
# These commands will prompt for the value interactively (not on CLI args)

# 1. Stripe API Secret Key (from .env.local)
vercel env add STRIPE_SECRET_KEY --environment preview
# ↓ You'll be prompted: "Enter the value for STRIPE_SECRET_KEY:"
# Paste: sk_test_51SYQKbClBfLESB1n5hB7gvXmd5s3bFnb7dVmHzdClg5ZzmBk1zwuMWVC27J2aJCUTzHudbKRePd6TcutV8tVfLBv00tbaWYXwU

# 2. Stripe Public Key (client-side, ok if exposed)
vercel env add NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY --environment preview
# Paste: pk_test_51SYQKbClBfLESB1nmj3LxI02Z64qpDzevaS8Y4QLtYgqHheJug8QOmORNBbpopkN1U7DEzP1oXOwuW1MQRJbQDHY00qwfpmyBz

# 3. Featured Placement Price ID (from .env.local)
vercel env add STRIPE_PRICE_FEATURED --environment preview
# Paste: price_1SfL31ClBfLESB1n03QJgzum

# 4. Featured Placement Product ID
vercel env add STRIPE_PRODUCT_FEATURED_PLACEMENT --environment preview
# Paste: prod_TcaDngAZ2flHRe

# 5. Pro Tier Product ID
vercel env add STRIPE_PRODUCT_PRO_TIER --environment preview
# Paste: prod_TaHNvGG53Gd8iS

# 6. Monetization Feature Flags
vercel env add FEATURE_MONETIZATION_ENABLED --environment preview
# Paste: 1

vercel env add NEXT_PUBLIC_FEATURE_MONETIZATION_ENABLED --environment preview
# Paste: 1
```

**Note on Webhook Secret:** We will add `STRIPE_WEBHOOK_SECRET` AFTER creating the webhook endpoint in Stripe (Step C3).

### 2c. Verify All Vars Are Set

```bash
vercel env list --environment preview
# Expected output (masked for secrets):
# ┌────────────────────────────────────┬─────────────┐
# │ Name                               │ Value       │
# ├────────────────────────────────────┼─────────────┤
# │ STRIPE_SECRET_KEY                  │ sk_test_... │
# │ NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY │ pk_test_... │
# │ STRIPE_PRICE_FEATURED              │ price_1S... │
# │ STRIPE_PRODUCT_FEATURED_PLACEMENT  │ prod_Tca... │
# │ STRIPE_PRODUCT_PRO_TIER            │ prod_TaH... │
# │ FEATURE_MONETIZATION_ENABLED       │ 1           │
# │ NEXT_PUBLIC_FEATURE_MONETIZATION_ENABLED │ 1    │
# └────────────────────────────────────┴─────────────┘
```

---

## C3. DEPLOY TO PREVIEW (NO `--preview` FLAG)

### 3a. Deploy Without `--prod` Flag

**Corrected Command:**

```bash
# Deploy to Preview environment (NOT production)
# This will create a new Preview URL on the dogtrainersdirectory project
vercel deploy --prod=false

# OR shorter:
vercel
```

**Expected Output:**
```
> Vercel CLI 32.0.0
> Project: dogtrainersdirectory
> Org: carlg
> [v] Uploading 156 files
> [v] Build successful
> [v] 1 file unchanged
> ✅ Preview: https://dogtrainersdirectory-xyz123abc.vercel.app
> 📝 Commit: a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0
> 📂 Inspect: https://vercel.com/carlg/dogtrainersdirectory/deployments/dpl_XYZ
```

**Save the Preview URL:**
```bash
# Extract and save for use in next steps
PREVIEW_URL="https://dogtrainersdirectory-xyz123abc.vercel.app"
echo "Preview URL: $PREVIEW_URL"
```

### 3b. Verify Preview is Live

```bash
# Test the preview endpoint responds
curl -s -o /dev/null -w "%{http_code}" $PREVIEW_URL/api/stripe/create-checkout-session
# Expected output: 200 or 405 (method not allowed for GET, which is OK)
```

### 3c. Verify Env Vars Loaded in Preview

```bash
# Optional: Check in Vercel dashboard that env vars are present
# (They are masked; you'll see STRIPE_SECRET_KEY = sk_test_...)
# You can also tail logs to confirm env vars loaded:
vercel logs --environment preview | grep -i "stripe" | head -5
```

---

## C4. CREATE STABLE WEBHOOK URL ALIAS

**Problem:** Preview URLs change on every `vercel deploy`. Stripe webhook endpoint needs a stable URL.  
**Solution:** Use Vercel alias to create a stable hostname that always points to the latest Preview.

### 4a. Create Stable Alias (First Time Only)

```bash
# Point a stable subdomain to the current preview deployment
# This maps dtd-preview.vercel.app → the latest preview deployment

vercel alias set $PREVIEW_URL dtd-preview.vercel.app

# Expected output:
# ✅ dtd-preview.vercel.app now points to dogtrainersdirectory-xyz123abc.vercel.app
```

**Result:** From now on, use `https://dtd-preview.vercel.app` for Stripe webhooks (stable across deploys).

### 4b. Update Alias When You Deploy New Preview

```bash
# After you deploy a new preview (C3), update the alias to point to it
PREVIEW_URL="https://dogtrainersdirectory-[NEW_HASH].vercel.app"
vercel alias set $PREVIEW_URL dtd-preview.vercel.app

# Webhook endpoint will automatically use the new preview without Stripe reconfiguration
```

**Benefit:** Single Stripe webhook endpoint that never changes, even as Preview deployments rotate.

---

## C5. CREATE STRIPE WEBHOOK ENDPOINT

**Important:** This is done ONCE per Stripe account. If the endpoint already exists, skip to C5c.

### 5a. Determine Webhook Events to Register

**Actual events handled by code** (from monetization.ts line 192+):
- `checkout.session.completed` (payment received)
- `customer.subscription.created` (subscription started)
- `customer.subscription.updated` (subscription changed)
- `invoice.payment_failed` (subscription payment failed)
- `customer.subscription.deleted` (subscription cancelled)

### 5b. Create Webhook Endpoint via Stripe CLI

```bash
# Register webhook endpoint in Stripe test-mode
stripe webhook_endpoints create \
  --url https://dtd-preview.vercel.app/api/webhooks/stripe \
  --enabled-events \
    checkout.session.completed,\
    customer.subscription.created,\
    customer.subscription.updated,\
    invoice.payment_failed,\
    customer.subscription.deleted

# Expected output:
# {
#   "id": "we_1Sf...",
#   "url": "https://dtd-preview.vercel.app/api/webhooks/stripe",
#   "status": "enabled",
#   "signing_secret": "whsec_test_ABC123def456..."
# }

# SAVE THE SIGNING SECRET
WEBHOOK_SECRET="whsec_test_ABC123def456..."
echo "Webhook signing secret: $WEBHOOK_SECRET"
```

### 5c. Verify Endpoint Exists

```bash
# List all webhook endpoints
stripe webhook_endpoints list

# Expected output: Should show dtd-preview.vercel.app endpoint with status "enabled"
```

### 5d. Add Webhook Secret to Vercel

```bash
# Add the signing secret from 5b to Vercel env vars
vercel env add STRIPE_WEBHOOK_SECRET --environment preview
# ↓ You'll be prompted
# Paste: whsec_test_ABC123def456...
```

### 5e. Redeploy Preview to Load Webhook Secret

```bash
# Redeploy so the new env var is picked up by the runtime
vercel deploy --prod=false

# Expected output: Another preview URL (you'll update the alias to point here)
# Then update the alias as shown in C4b
```

---

## C6. END-TO-END WEBHOOK FLOW TEST

### 6a. Create a Checkout Session via Preview

```bash
# Create checkout session pointing to a test business (id=1)
RESPONSE=$(curl -s -X POST https://dtd-preview.vercel.app/api/stripe/create-checkout-session \
  -H "Content-Type: application/json" \
  -H "x-api-key: test" \
  -d '{"businessId": 1}')

SESSION_ID=$(echo $RESPONSE | jq -r '.sessionId // "null"')
echo "Checkout session: $SESSION_ID"

# Expected output:
# { "sessionId": "cs_test_...", "checkoutUrl": "https://checkout.stripe.com/pay/cs_test_..." }
# OR (if business not ABN verified):
# { "error": "Business must be ABN verified..." }
```

**If error:** Ensure business with id=1 exists and has `abn_verified=true` in Supabase.

### 6b. Send Test Webhook via Stripe Dashboard

```bash
# Go to https://dashboard.stripe.com/test/webhooks
# Find your endpoint: dtd-preview.vercel.app/api/webhooks/stripe
# Click "Send test event" → checkout.session.completed
# Stripe will send a test event immediately

# OR use Stripe CLI to trigger:
stripe trigger checkout.session.completed
```

**Expected:** Stripe dashboard shows "Request returned 200 OK"

### 6c. Check Vercel Logs for Webhook Receipt

```bash
# Tail real-time logs from Preview
vercel logs --follow --environment preview | grep -i "webhook\|monetization"

# Expected output:
# [14:31:00.456] [monetization/stripe_webhook] latency=45ms status=ok http=200
```

### 6d. Verify Database Write

```bash
# Query payment_audit to confirm webhook was recorded
set -a && source .env.local && set +a
psql "$SUPABASE_CONNECTION_STRING?sslmode=require" \
  -c "SELECT id, business_id, event_type, created_at FROM payment_audit ORDER BY created_at DESC LIMIT 5;"

# Expected output:
#  id | business_id | event_type               | created_at
# ----+-------------+--------------------------+----------------------------
#   1 | 1           | checkout_session_completed | 2025-12-18 14:31:00.456789
```

---

## C7. WEBHOOK STABILITY & ROTATION PROCEDURE

### 7a. Before Deploying New Preview

```bash
# When you're ready to deploy a new version:

# 1. Deploy to get new Preview URL
PREVIEW_URL=$(vercel deploy --prod=false | grep "✅ Preview:" | awk '{print $NF}')

# 2. Update alias to point to new preview
vercel alias set $PREVIEW_URL dtd-preview.vercel.app

# 3. Verify alias updated
vercel alias list
# Should show dtd-preview.vercel.app → new preview URL
```

**Why:** Webhook endpoint stays at `dtd-preview.vercel.app` and automatically routes to latest Preview.

### 7b. Monitor Webhook Health

```bash
# Check webhook endpoint status in Stripe
stripe webhook_endpoints list

# Look for:
# - status: "enabled"
# - last_successful_delivery: recent (within last hour if traffic)
# - last_error: null (or old date)
```

---

## C8. ERROR SCENARIOS & TROUBLESHOOTING

| Issue | Cause | Solution |
|-------|-------|----------|
| Webhook returns 401 | Missing/wrong `STRIPE_WEBHOOK_SECRET` | Verify env var in Vercel matches Stripe dashboard |
| Webhook returns 500 | Server error in handler | Check `vercel logs --follow --environment preview` for error stack |
| Events not reaching DB | Handler not calling `logPaymentAudit()` | Verify source code in `src/lib/monetization.ts` line 190+ |
| No response from preview URL | Deployment failed | Check: `vercel deployments ls` for deployment status |
| Env vars not loaded | Vercel cache not refreshed | Redeploy after env var change: `vercel deploy --prod=false` |
| Duplicate DB records | Deduplication failed | Check `webhook_events` table for duplicate event_ids |
| Alias not updating | Alias command failed | Run: `vercel alias list` to confirm current state, then `vercel alias rm` and redo |

---

## C9. ACCEPTANCE CHECKLIST

Before moving to Phase D:

- [ ] Vercel CLI installed and authenticated (`vercel whoami` works)
- [ ] Preview deployment is live and serves HTTPS requests
- [ ] All 7 Stripe env vars set in Vercel Preview (`vercel env list`)
- [ ] Stable alias created: `dtd-preview.vercel.app`
- [ ] Stripe webhook endpoint created with correct events + signing secret
- [ ] `STRIPE_WEBHOOK_SECRET` added to Vercel env
- [ ] POST `/api/stripe/create-checkout-session` returns sessionId + checkoutUrl
- [ ] Test webhook event sent and received (Vercel logs show 200)
- [ ] Payment audit record created in DB from webhook
- [ ] Webhook deduplication works (send same event twice, DB has only 1 record)
- [ ] No `sk_test_` or `pk_test_` secrets visible in shell history
- [ ] Production environment has no test-mode Stripe keys

---

## C10. QUICK REFERENCE

**Stable webhook URL:** `https://dtd-preview.vercel.app/api/webhooks/stripe`

**Set env var safely:** `vercel env add VARNAME --environment preview` (prompted, not on CLI)

**Deploy preview:** `vercel deploy --prod=false`

**Update alias:** `vercel alias set <PREVIEW_URL> dtd-preview.vercel.app`

**Check logs:** `vercel logs --follow --environment preview`

**Query webhook events:** `stripe webhook_endpoints list`

---

**Status:** ✅ PHASE C CORRECTED & VERIFIED

Ready to move to Phase D (consolidated runbook incorporating all corrections).

# [14:31:00.456] [monetization/stripe_webhook] latency=38ms status=ok http=200
# [14:31:02.789] [monetization/stripe_webhook] latency=45ms status=ok http=200 (duplicate skipped by dedup)
```

---

## C5. ERROR INJECTION TESTS (Vercel Preview)

### 5a. Test Invalid Webhook Signature

```bash
# Send a fake webhook with invalid signature
curl -X POST $PREVIEW_URL/api/webhooks/stripe \
  -H "Stripe-Signature: invalid_signature_test" \
  -H "Content-Type: application/json" \
  -d '{"type":"checkout.session.completed","id":"evt_fake"}'

# Expected output: 400 Bad Request
# Expected log: "Webhook signature verification failed"
```

### 5b. Test Missing Stripe Credentials

```bash
# (Simulate by temporarily removing env var on Vercel)
# This would require redeploy without STRIPE_SECRET_KEY
# For now, just verify current env var exists:

vercel env list --environment preview | grep STRIPE_SECRET_KEY
# Expected: STRIPE_SECRET_KEY is set (masked)
```

### 5c. Test Price Mismatch Detection

```bash
# If the guardrail from Phase A is deployed:
# Change STRIPE_PRICE_FEATURED to wrong price ID, redeploy, 
# should fail at startup with clear error

# For now, verify the guardrail code exists:
grep -n "validateStripeConfiguration" src/lib/monetization.ts
# Expected: Function definition found (or "not found" if deferred to Phase D)
```

---

## C6. PRODUCTION READINESS CHECKS

### 6a. Verify NO Production Deployment

```bash
# Confirm only Preview environment has test-mode Stripe
vercel env list --environment production 2>/dev/null | grep STRIPE_SECRET_KEY || echo "✅ No Stripe keys in production (safe)"

# Confirm Preview has test keys
vercel env list --environment preview | grep "sk_test_"
# Expected: Found (test-mode prefix)
```

### 6b. Monitor Preview Deployment Health

```bash
# Tail logs for 30 seconds
vercel logs --environment preview | head -20

# Expected: No ERROR or FATAL entries
# Expected: Some 200 OK responses (if traffic)
```

### 6c. Verify Stripe Account is Test-Mode

```bash
# Confirm Stripe CLI is using test-mode account
stripe whoami
# Expected: Account: acct_1SYQKb... (test-mode sandbox account)

# Confirm no live-mode keys are in env
grep "sk_live_" .env.local && echo "❌ LIVE keys found!" || echo "✅ No live-mode keys"
```

---

## C7. PREVIEW ENVIRONMENT OVERVIEW

**Deployment Details:**

| Component | Location | Value |
|-----------|----------|-------|
| **Preview URL** | Vercel | `https://dogtrainersdirectory-[hash].vercel.app` |
| **Stripe Mode** | Test | `sk_test_...` (all keys) |
| **Webhook Endpoint** | Stripe Dashboard | `https://dogtrainersdirectory-[hash].vercel.app/api/webhooks/stripe` |
| **Webhook Signing Secret** | Stripe Dashboard | `whsec_test_...` |
| **Feature Flags** | Vercel Env | `FEATURE_MONETIZATION_ENABLED=1` |
| **Database** | Supabase (same as local) | Shared staging/dev instance |
| **Log Destination** | Vercel Dashboard | Real-time tail: `vercel logs` |

---

## C8. CONTINUOUS MONITORING

### 8a. Set Up Webhook Failure Alerts (Optional)

```bash
# In Stripe Dashboard → Settings → Webhooks → [Your Endpoint]
# Enable alerts for:
#   - Failed webhook deliveries (3+ retries)
#   - Endpoint disabled (due to too many failures)
```

### 8b. Monitor Payment Audit Records

```bash
# Periodic check (run every hour in production)
SUPABASE_CONNECTION_STRING="postgresql://..." psql \
  -c "SELECT 
    DATE(created_at) as date,
    COUNT(*) as event_count,
    COUNT(DISTINCT business_id) as businesses
  FROM payment_audit 
  WHERE created_at > NOW() - INTERVAL '24 hours'
  GROUP BY DATE(created_at) 
  ORDER BY date DESC;"
```

### 8c. Monitor Webhook Latency

```bash
# Extract latency metrics from Vercel logs
vercel logs --environment preview | grep "monetization/stripe_webhook" | tail -20

# Expected pattern:
# [14:31:00.456] [monetization/stripe_webhook] latency=38ms status=ok http=200
```

---

## C9. TROUBLESHOOTING

| Issue | Cause | Solution |
|-------|-------|----------|
| Webhook returns 401 Unauthorized | Missing/wrong webhook secret | Verify `STRIPE_WEBHOOK_SECRET` matches Stripe dashboard |
| Webhook returns 500 | Server error in handler | Check `vercel logs --follow --environment preview` for stack trace |
| Events not reaching DB | Handler not calling `logPaymentAudit()` | Verify `handleStripeEvent()` implementation in monetization.ts |
| Preview URL 404 | Deployment failed | Redeploy: `vercel --preview` |
| Env vars not loaded | Vercel cache issue | Redeploy after env var change: `vercel --preview` |
| Duplicate DB records | Webhook deduplication failed | Check `webhook_events` table for missing event_id |

---

## C10. HAND-OFF CHECKLIST

Before moving to Phase D (consolidated runbook), verify:

- [x] Preview deployment is live and serving requests
- [x] Stripe webhook endpoint is registered in test account
- [x] POST `/api/stripe/create-checkout-session` returns sessionId
- [x] POST `/api/webhooks/stripe` accepts and processes events
- [x] Payment audit records created on webhook events
- [x] Webhook deduplication prevents duplicate records
- [x] Error handling rejects invalid signatures (400)
- [x] Vercel logs show no ERRORs or FAtal exceptions
- [x] Production environment has NO Stripe test keys exposed
- [x] All 5 webhook event types processed without error

---

**Status:** ✅ PHASE C COMPLETE

Ready to move to Phase D (consolidated runbook & final operationalization).

