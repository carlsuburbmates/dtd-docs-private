# DTD Project State Overview

## What is DTD?

dogtrainersdirectory.com.au is a Melbourne-specific, docs-first, AI-assisted directory connecting dog owners to trainers, behavior consultants, and emergency resources across 28 local government areas (councils). Built on Next.js 14, React 19, TypeScript, Supabase (Postgres + Auth + Functions), with Stripe payment infrastructure staged but not yet enabled. The service specializes in "age-first" triage (puppies, adolescent, adult, senior, rescue) and 13 locked behavior issue categories, returning verified trainers (ABN-checked) for suburb-based queries.

---

## User Roles & Primary Workflows

### 1. **Dog Owner** (Anonymous/User)
- Visits homepage, selects dog age/stage (mandatory)
- Selects 1+ behavior issues (optional, multi-select)
- Enters suburb or postcode to find trainers in their area
- Views results ranked by verification status and featured slots
- Can click emergency button for crisis routing (vets, shelters, crisis trainers)

### 2. **Trainer / Behaviour Consultant** (Registered Business)
- Receives email invite → claims their business listing
- Fills multi-step form: specialty ages, behavior issues, service types, contact info
- ABN is verified via ABR API (if ≥85% name match + Active status → verified badge)
- Appears in search results; can eventually subscribe to featured placement (Stripe, deferred)
- Can update their profile over time

### 3. **Emergency Resource** (Vet, Shelter, Crisis Trainer)
- Seeded from manual data sources + may accept scraper input (deferred)
- Appears in `/emergency` flow when dog owner triggers crisis routing
- Contact info re-verified quarterly via automated jobs
- Shows in admin emergency verification queue

### 4. **Admin / Ops User** (Operator, Product, MCP)
- Accesses `/admin` dashboard for business verification, emergency queues, telemetry
- Reviews AI-flagged profiles/reviews; approves/rejects moderation decisions
- Manages ABN allowlists and controlled batch verification runs
- Checks emergency resource status and re-verification metrics
- Can toggle telemetry overrides for services under maintenance

---

## Current Status

**Phases Completed: 1–5 (Database, Auth, Triage/Search, Directory/Profiles, Emergency Ops + Admin)**
- ✅ Database schema (Supabase Postgres) with businesses, reviews, emergency resources, AI logs, payment audit tables
- ✅ Auth system (Supabase + email-based Trainer/Owner flows)
- ✅ Home/Search UI with age-first triage → issue multi-select → suburb search → results
- ✅ Trainer profile pages with verification metadata and verified badge display
- ✅ Emergency help entry point (`/emergency`) with AI-assisted medical/stray/crisis classification and resource routing
- ✅ Admin dashboard with emergency verification queue, daily ops digest, AI review moderation queue
- ✅ ABN verification automation (ABR API lookup, ≥85% name match rule, fallback event logging)
- ✅ Emergency resource verification cron jobs (quarterly contact re-check)
- ✅ Stripe webhook infrastructure (idempotent handlers, payment audit table) — **disabled by feature flag until launch**
- ✅ Web scraper automation infrastructure — **deferred, behind feature flag**

**What's Complete:**
- Search/triage logic locked (phase 2, cannot revert to radius-only)
- All locked enums enforced (5 age categories, 13 behavior issues, 5 service types)
- ABN verification with canonical contract (JSON + SOAP fallback, ≥85% match threshold)
- Emergency triage classifier with optional LLM mode (fallback to deterministic)
- Admin dashboards with real-time telemetry, health status, emergency queues
- Playwright E2E test coverage for critical user paths
- Type-check (tsc strict mode) and lint (ESLint flat config) gates

**What's In Progress / Deferred:**
- Monetization (featured placement subscriptions via Stripe) — feature-flagged off, waiting for launch metrics (≥50 claimed trainers, stable ABN rate, review volume)
- Web scraper automation (bulk ingestion from council websites) — feature-flagged off, awaiting ≥95% accuracy sign-off
- Cron jobs: `npm run verify:launch` confirms Vercel cron configuration pending launch

**Known Broken / At-Risk:**
- None documented; however, see KNOWN_ISSUES.md for edge cases and manual verification steps.

---

## Key Design & Engineering Decisions

### 1. **Feature Flags for Controlled Rollout**
- `FEATURE_MONETIZATION_ENABLED` (server) + `NEXT_PUBLIC_FEATURE_MONETIZATION_ENABLED` (client) gates all Stripe-related UX
- `FEATURE_SCRAPER_ENABLED` gates web scraper automation; disabled by default
- `E2E_TEST_MODE` disables real Stripe/LLM calls in Playwright tests
- All flags default to OFF; enabled only when criteria are met (documented in master plan)

### 2. **ABN Verification as Canonical Trust Signal**
- ABN matching rule: `ABNStatus='Active' && name_match_score >= ~0.85`
- Canonical implementation in `src/lib/abr.ts` (JSON parser + SOAP fallback)
- Fallback events logged to `abn_fallback_events` table when API is unavailable
- Yearly re-verification jobs scheduled to catch status changes
- Ops can manually manage allowlists via CSV + Python scripts for controlled batch runs

### 3. **Age-First UX as Non-Negotiable Product Invariant**
- All search flows enforce: age/stage selection BEFORE issue multi-select BEFORE suburb search
- This drives different training methodologies (puppies vs seniors) and is locked in UI + RPC logic
- Cannot be reverted per SSOT mandate (see PHASE_2_FINAL_COMPLETION_REPORT.md for QA evidence)

### 4. **Docs-First Architecture**
- Single source of truth (SSOT): `DOCS/blueprint_ssot_v1.1.md` defines all taxonomies, UX rules, geography
- Implementation reality map: `DOCS/IMPLEMENTATION_REALITY_MAP.md` tracks CONFIRMED-WORKING vs UNKNOWN flows
- Launch checklist: `DOCS/LAUNCH_READY_CHECKLIST.md` defines AI-ready vs Operator-only gates
- CI check "doc divergence" prevents code/docs drift
- All major decisions require RFC + SSOT update before code merge

### 5. **Soft-Delete + Audit Logging**
- No hard-deletes for Businesses; use `is_deleted=true` flag
- All verification events (ABN, emergency, reviews) logged to audit tables
- Enables compliance, dispute resolution, and transparency

### 6. **Async AI Moderation (Human Approved)**
- AI flags reviews/profiles as spam/harmful but humans approve/reject
- LLM provider configurable: Z.AI (default) or OpenAI; falls back to deterministic rules if unavailable
- False positives tracked for transparency and model refinement

### 7. **Remote-First Development Environment**
- Default dev setup: remote Supabase project (no local Docker required)
- Optional: local Supabase CLI emulator for isolated migration testing
- CI applies migrations to scratch DB, diffs schema snapshot to detect drift
- All secrets stored in CI secrets manager or local `.env.local` (git-ignored)

### 8. **Idempotent Stripe Webhook Handler**
- Webhook signature verification mandatory (never skip)
- Dedupes events, returns 2xx as soon as DB succeeds
- Async failures logged as `sync_error` for ops visibility
- E2E mode (`E2E_TEST_MODE=true`) stubs Stripe calls for testing without credentials

---

## Top 10 "Where to Look in Code" Pointers

1. **Taxonomy Enums & Validators** → `src/types/database.ts`
   - All locked enum definitions and runtime validators (age, issue, service type, region, etc.)

2. **Search & Triage Entry Point** → `src/app/page.tsx` (home) + `src/app/search/` (search flow)
   - Age-first UX, issue multi-select, suburb autocomplete, RPC call to `search_trainers()`

3. **Trainer Profile Page** → `src/app/trainers/[id]/page.tsx`
   - Server-side rendering of business details, reviews, verification metadata

4. **Emergency Help Workflow** → `src/app/emergency/` + `/api/emergency/*`
   - Triage form, medical classifier, crisis routing, weekly metrics aggregation

5. **ABN Verification** → `src/lib/abr.ts` + `/api/abn/verify`
   - ABR API parser, ≥85% name match logic, JSON + SOAP fallback

6. **Admin Dashboard** → `src/app/admin/` + `/api/admin/*`
   - Business verification queue, emergency resources, telemetry health, alert overrides

7. **Feature Flags & Config** → `src/lib/flags.ts` (if present) or inline `process.env.FEATURE_*` checks
   - Monetization, scraper, E2E test mode guards

8. **Error Logging & Context** → `src/lib/errorLog.ts` + error context payloads
   - Structured error logging with `extra` payload, used by admin dashboards

9. **Stripe Webhook Handler** → `/api/webhooks/stripe/route.ts`
   - Signature verification, idempotent deduping, payment audit logging

10. **Database Schema & Migrations** → `supabase/migrations/` + `supabase/schema.sql`
    - All tables, columns, indexes, RPC functions; migrations applied in order

---

## Known Constraints

### Local Development
- Requires Node.js v24 (Active LTS); nvm recommended
- Default remote Supabase setup requires internet connectivity
- Local Supabase CLI optional; Docker recommended if using (see `supabase/README.md`)
- `.env.local` must be created manually with remote project credentials (not in repo)

### Environment Variables
- ~25 env vars required across dev/staging/prod (see ENV_MATRIX.md)
- Secrets (ABR_GUID, Stripe keys, LLM keys, JWT secrets) NOT in repo; sourced from CI secrets manager
- `SUPABASE_CONNECTION_STRING` only needed for certain admin scripts; not required for normal dev

### Geography
- Locked to Melbourne metropolitan area (28 councils, 138 suburbs as of 2025-12-09)
- Suburb-to-council mapping immutable; changes require RFC + FILE_MANIFEST update + CI drift check
- No interstate or rural areas in scope

### Taxonomies
- Age/stage, behavior issues, service types are locked enums (never add free-text)
- Changes require SSOT blueprint RFC + QA sign-off before merge

### ABN Verification
- Requires valid Australian Business Register GUID (stored in `ABR_GUID` env var)
- Fallback tolerance: ≥~85% name match + ABNStatus='Active' only
- ABR API calls can fail; fallback events logged; ops alerted if rate exceeds threshold

### Monetization
- Feature-flagged and disabled by default
- Stripe webhook testing uses custom local harness (port 4243, not 3000)
- Go-live blocked until: ≥50 claimed trainers claimed + stable ABN rate + review volume

### Email & Observability
- Email system configurable: Supabase SMTP + Resend (optional), or bring-your-own
- Sentry/Logflare optional for observability; some admin dashboards check for these before rendering

---

## Getting Started Locally

```bash
# 1. Use Node.js v24
nvm install 24 && nvm use 24

# 2. Install dependencies
npm ci

# 3. Create .env.local with remote Supabase credentials
# (contact maintainer for: NEXT_PUBLIC_SUPABASE_URL, keys, ABR_GUID, etc.)
cat > .env.local << 'EOF'
NEXT_PUBLIC_SUPABASE_URL=https://...
NEXT_PUBLIC_SUPABASE_ANON_KEY=...
SUPABASE_SERVICE_ROLE_KEY=...
ABR_GUID=REDACTED
LLM_PROVIDER=zai
ZAI_API_KEY=REDACTED
FEATURE_MONETIZATION_ENABLED=
NEXT_PUBLIC_FEATURE_MONETIZATION_ENABLED=
EOF

# 4. Start dev server
npm run dev
# → Visit http://localhost:3000

# 5. Run verification gates
npm run type-check    # TypeScript
npm run lint          # ESLint
npm run smoke         # Unit tests
npm run e2e           # Playwright E2E (optional)
npm run verify:launch # Full AI launch gate
```

---

## Documentation Artifacts (in `/DOCS`)

- **blueprint_ssot_v1.1.md** – Product spec, domain model, taxonomies, UX rules, geography
- **IMPLEMENTATION_REALITY_MAP.md** – CONFIRMED-WORKING vs UNKNOWN flows (updated per phase)
- **LAUNCH_READY_CHECKLIST.md** – AI-ready vs Operator-only go/no-go gates
- **MONETIZATION_ROLLOUT_PLAN.md** – Stripe contract, metadata, payment audit schema
- **operator-runbook.md** – Ops procedures, ABN allowlists, batch runs, alert escalation
- **automation/ABN-ABR-GUID_automation/** – ABR API contract, parsing, allowlist gen, batch runner docs
- **OPS_TELEMETRY_ENHANCEMENTS.md** – Health dashboards, latency metrics, alert configuration

