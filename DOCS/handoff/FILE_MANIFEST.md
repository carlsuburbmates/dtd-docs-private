# HANDOFF File Manifest

**Purpose:** Complete inventory of files/folders in DTD_CONTEXT_PACK.zip for external reviewer  
**Total Files:** ~250+ across docs, config, and source code  
**Total Size:** ~5MB (uncompressed)

---

## HANDOFF/DOCS/ — Documentation & Analysis

| File | Purpose | Size |
|------|---------|------|
| **PROJECT_STATE.md** | 1-page overview: what DTD is, user roles, status, key decisions, top 10 code pointers, constraints | ~8KB |
| **ENV_MATRIX.md** | Complete table of 30+ environment variables: names, purpose, where used, dev/prod, notes (NO VALUES) | ~12KB |
| **KNOWN_ISSUES.md** | 10 known issues/blockers with repro steps, severity, suggested fixes, related files | ~18KB |
| **LOCAL_SNAPSHOT.md** | Node/npm versions, git status/log, setup instructions, service dependencies, port matrix, troubleshooting | ~15KB |
| **SCHEMA_NOTES.md** | Database schema overview: enums, 20+ core tables, RPC functions, indexes, encryption, retention, migrations | ~20KB |
| **FILE_MANIFEST.md** | This file; inventory of HANDOFF contents with purposes and exclusions | ~5KB |

**Total Docs:** ~78KB

---

## HANDOFF/REPO_SNAPSHOT/ — Project Source Code & Configuration

### Root Configuration Files

| File | Purpose |
|------|---------|
| **package.json** | NPM dependencies, dev/test scripts (dev, lint, type-check, smoke, e2e, verify:launch, etc.) |
| **package-lock.json** | Lockfile pinning exact dependency versions (Node 20.19.2, npm 11.6.2) |
| **README.md** | Project overview, status, core features, stack, getting started, data sources, TODO |
| **next.config.js** | Next.js 14 App Router configuration |
| **tsconfig.json** | TypeScript strict mode configuration (noImplicitAny, strict: true) |
| **tailwind.config.js** | Tailwind CSS configuration for styling |
| **postcss.config.js** | PostCSS plugins for CSS processing |
| **playwright.config.ts** | Playwright E2E test configuration |
| **vitest.config.ts** | Vitest unit test configuration |

### Source Code (`src/`)

| Folder | Purpose | Files |
|--------|---------|-------|
| **src/app/** | Next.js App Router pages + API routes | 30+ files |
| → **page.tsx** | Home page with age-first search triage UI | - |
| → **search/** | Search results page, RPC integration | - |
| → **trainers/[id]/** | Trainer profile detail page (ID-based) | - |
| → **trainer/[id]/** | Legacy trainer profile (fallback, may redirect) | - |
| → **directory/** | Browse all trainers directory | - |
| → **emergency/** | Emergency help flow (medical/stray/crisis triage) | - |
| → **admin/** | Admin dashboard (business queue, emergency verification, telemetry) | - |
| → **api/** | API endpoints: `/triage`, `/abn/verify`, `/emergency/*`, `/admin/*`, `/webhooks/stripe/*` | 15+ files |
| → **middleware.ts** | Auth middleware, request routing | - |
| → **layout.tsx** | Root layout, global providers, Supabase auth | - |
| **src/components/** | Reusable React components | 25+ files |
| → **triage/** | Emergency triage form, gate component, issue selector | - |
| → **admin/** | Admin dashboard panels, status strip | - |
| → **search/** | Search UI components, suburb autocomplete | - |
| → **common/** | Shared components (buttons, cards, loaders) | - |
| **src/lib/** | Utility libraries & helpers | 20+ files |
| → **api.ts** | Supabase API client, search helper | - |
| → **supabase.ts** | Supabase client initialization (client + admin) | - |
| → **abr.ts** | ABR API parser (JSON + SOAP), name matching ≥85% | - |
| → **abnFallback.ts** | ABN fallback event logging | - |
| → **llm.ts** | LLM provider wrapper (Z.AI/OpenAI, deterministic fallback) | - |
| → **medicalDetector.ts** | Emergency medical classifier | - |
| → **moderation.ts** | AI review moderation heuristics | - |
| → **errorLog.ts** | Structured error logging with context payloads | - |
| → **triage.ts** | Triage logic helpers | - |
| **src/types/** | TypeScript types & runtime validators | - |
| → **database.ts** | All enum definitions + validator functions (AgeSpecialty, BehaviorIssue, etc.) | - |
| **src/styles/** | Tailwind CSS and global styles | - |

### Database (`supabase/`)

| File | Purpose | Lines |
|------|---------|-------|
| **migrations/** | 10+ timestamped SQL migration files | 2500+ |
| → `20250101*` | Phase 1 base schema (councils, suburbs, businesses, auth) | - |
| → `20250201*` | Phase 2 search RPC refinements | - |
| → `20250203*` | Phase 3 ABN verification tables | - |
| → `20250205*` | Phase 4 Stripe payment audit schema | - |
| → `20250207*` | Phase 5 emergency + AI moderation tables | - |
| **schema.sql** | Complete database schema snapshot (applied by CI to verify migrations) | 485 lines |
| **README.md** | Supabase local setup instructions | - |

### Tests (`tests/`)

| Folder | Purpose | Files |
|--------|---------|-------|
| **e2e/** | Playwright E2E tests | 5+ |
| → `search-and-trainer.spec.ts` | Search triage → trainer profile flow | - |
| → `emergency.spec.ts` | Emergency triage form, routing | - |
| → `monetization.spec.ts` | Stripe checkout stub (feature-flagged) | - |
| → `admin.spec.ts` | Admin dashboard UI | - |
| → `*-snapshots/` | Visual regression baselines (PNG screenshots) | 10+ |
| **unit/** | Vitest unit tests | 5+ |
| → `search.test.ts` | RPC mocking, search logic | - |
| → `abr.test.ts` | ABN parser unit tests | - |
| → `monetization.test.ts` | Stripe webhook handler, idempotency | - |
| → `emergency.test.ts` | Classifier, medical detector | - |
| **fixtures/** | Test data, mocks, factories | 3+ |

### Scripts (`scripts/`)

| File | Purpose | Language |
|------|---------|----------|
| **generate_allowlist.py** | Generate ABN allowlist JSON from CSV template | Python 3 |
| **abn_controlled_batch.py** | Run controlled ABN verification batch (dry-run / apply mode) | Python 3 |
| **abn_recheck.py** | Yearly ABN re-verification script | Python 3 |
| **abn_recheck.sh** | Shell wrapper for cron jobs | Bash |
| **check_docs_divergence.py** | CI check: ensure SSOT docs match code | Python 3 |
| **check_env_ready.sh** | Pre-flight env var check before deploy | Bash |
| **local_db_start_apply.sh** | Start Supabase emulator + apply migrations | Bash |
| **preprod_verify.sh** | Full pre-prod gate: type-check → smoke → lint → divergence → env-ready | Bash |
| **preprod_e2e.sh** | Start Next.js + run E2E tests | Bash |
| **examples/controlled_abn_list.example.json** | Example ABN allowlist format (archived) | JSON |

### CI/CD Workflows (`.github/`)

| File | Purpose |
|------|---------|
| **workflows/verify-launch.yml** | Main CI: type-check, smoke, lint, doc-divergence, env-ready; required for all PRs |
| **workflows/lint.yml** | ESLint check on every PR |
| **workflows/abn-recheck.yml** | Scheduled ABN re-verification (Vercel cron) |
| **copilot-instructions.md** | AI agent onboarding (this project's instructions) |

### Project Documentation (`DOCS/`)

All SSOT docs, phase reports, automation runbooks, and implementation guides (~96 files):

| Category | Key Files | Purpose |
|----------|-----------|---------|
| **SSOT & Specs** | blueprint_ssot_v1.1.md, implementation/master_plan.md | Product spec, domain model, taxonomies, UX rules, governance |
| **Implementation** | IMPLEMENTATION_REALITY_MAP.md, FRONTEND_VERIFICATION_FINDINGS.md | Current status: CONFIRMED-WORKING vs UNKNOWN flows |
| **Launch** | LAUNCH_READY_CHECKLIST.md, PHASE_5_FINAL_COMPLETION_REPORT.md | Go/no-go gates, phase completion evidence |
| **Monetization** | MONETIZATION_ROLLOUT_PLAN.md | Stripe contract, payment audit schema, webhook requirements |
| **Automation** | operator-runbook.md, automation/ABN-ABR-GUID_automation/* | Ops procedures, ABN allowlists, verification, alerts |
| **Data** | suburbs_councils_mapping.csv | 138 suburbs, 28 councils, postcode/lat/lon (immutable) |
| **History** | PHASE_*_FINAL_COMPLETION_REPORT.md (1–5), WEEK_*_COMPLETION_REPORT.md | Historical implementation notes per phase |
| **Emergency** | automation/OPS_RUNBOOK_EMERGENCY_VERIFICATION.md | Emergency resource verification procedures |
| **Observability** | OPS_TELEMETRY_ENHANCEMENTS.md | Health dashboards, alerts, latency metrics |

---

## Excluded / Intentionally NOT Included

### Why Excluded

| Item | Reason |
|------|--------|
| **.env*, .env.local, .env.production** | Contain secrets (REDACTED; contact maintainer) |
| **node_modules/** | Large dependency tree (~300MB); use `npm ci` to regenerate |
| **.next/, dist/, build/, coverage/** | Build artifacts; regenerate with `npm run build` |
| **.git/, .github/CODEOWNERS** | Git history; not needed for context pack |
| **Supabase .branches/** | Supabase deployment branches; local-only |
| **logs/, tmp/, caches/** | Temporary runtime files |
| **.vercel/, .turbo/** | Build caches |
| **Large binaries** | Test snapshots only PNG; excluded video/media files |

### If You Need These

**Secrets & Credentials:**
- Contact repo maintainer for `.env.local` template
- Request: ABR_GUID, Stripe keys (test), LLM keys (if using), Supabase project info

**Build Artifacts:**
- Run `npm run build` locally to generate `.next/`
- Run `npm ci` to reinstall node_modules

**Git History:**
- Clone from GitHub: `git clone https://github.com/carlsuburbmates/dogtrainersdirectory.git`

---

## Security & Redaction Checklist

**✓ Scan Results:** All high-risk patterns (`sk_live_`, `whsec_`, `service_role` values) verified as NOT present in HANDOFF/

**Redactions Applied:**
- Env var values masked as `REDACTED` in documentation
- No actual credentials in any file
- No private keys, JWTs, or API tokens
- All references to secrets use placeholder names only

**Safe to Share:** Yes — HANDOFF/ is fully sanitized and suitable for external review.

---

## How to Use This Pack

1. **Extract:** Unzip `DTD_CONTEXT_PACK.zip` to a working directory
2. **Read First:** Start with `HANDOFF/DOCS/PROJECT_STATE.md` (2-minute overview)
3. **Deep Dive:** Then read KNOWN_ISSUES.md + SCHEMA_NOTES.md
4. **Setup Context:** Refer to LOCAL_SNAPSHOT.md for environment details
5. **Code Review:** Explore `HANDOFF/REPO_SNAPSHOT/src/` + `DOCS/` for implementation
6. **Questions:** Use FILE_MANIFEST.md to locate relevant files

---

## Contact & Support

**Questions about this pack?**
- Review `PROJECT_STATE.md` first (top 10 "where to look" pointers)
- Check `KNOWN_ISSUES.md` for common blockers
- Read specific runbook docs in `REPO_SNAPSHOT/DOCS/automation/`

**Need credentials to run locally?**
- Contact repo maintainer for ABR_GUID, Stripe keys, LLM keys

**Found an issue in the codebase?**
- File GitHub issue with reproduction steps
- Reference `KNOWN_ISSUES.md` to check if already documented

