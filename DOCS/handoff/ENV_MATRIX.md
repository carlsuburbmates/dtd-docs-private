# Environment Variables Matrix

| Variable | Purpose | Where Used | Dev/Prod | Notes |
|----------|---------|-----------|----------|-------|
| **NEXT_PUBLIC_SUPABASE_URL** | Supabase project URL (client-side) | `src/lib/supabase.ts`, `src/lib/api.ts` | Dev/Staging/Prod | Public; embedded in client JS; project-specific |
| **NEXT_PUBLIC_SUPABASE_ANON_KEY** | Supabase anon key (client-side) | `src/lib/supabase.ts` | Dev/Staging/Prod | Public; embedded in client JS; enables auth + queries |
| **SUPABASE_SERVICE_ROLE_KEY** | Supabase service role (server-side admin) | `/api/*` routes, server-side utilities | Dev/Staging/Prod | Secret; never expose to client; bypasses RLS |
| **SUPABASE_URL** | Supabase project URL (server-side duplicate) | Backend config | Dev/Staging/Prod | Mirrors NEXT_PUBLIC_SUPABASE_URL; legacy |
| **SUPABASE_CONNECTION_STRING** | Direct Postgres connection (admin scripts only) | `scripts/abn_*.py`, `scripts/generate_allowlist.py` | Prod-Ops | Secret; connection string format; optional for local dev |
| **SUPABASE_PGCRYPTO_KEY** | PostgreSQL pgcrypto key for encryption | Trainer profile encryption | Dev/Staging/Prod | Integer; determines encryption context |
| **ABR_GUID** | Australian Business Register API credential | `src/lib/abr.ts`, `/api/abn/verify` | Dev/Staging/Prod | Secret; GUID format; required for ABN lookup calls |
| **STRIPE_SECRET_KEY** | Stripe secret key (server-side) | `/api/webhooks/stripe`, `/api/stripe/*` | Dev/Staging/Prod | Secret; starts with `sk_live_` or `sk_test_`; feature-flagged |
| **STRIPE_WEBHOOK_SECRET** | Stripe webhook signing secret | `/api/webhooks/stripe/route.ts` | Dev/Staging/Prod | Secret; starts with `whsec_`; used to verify webhook signatures |
| **STRIPE_PRICE_FEATURED** | Stripe price ID for featured placements | `/api/stripe/create-checkout-session` | Dev/Staging/Prod | Stripe API ID; feature-flagged |
| **NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY** | Stripe publishable key (client-side) | Client checkout form (if enabled) | Dev/Staging/Prod | Public; embedded in client JS; feature-flagged |
| **RESEND_API_KEY** | Resend email API key (optional) | Email delivery system | Dev/Staging/Prod | Secret; optional (can use Supabase SMTP instead) |
| **LLM_PROVIDER** | LLM provider choice | `src/lib/llm.ts` | Dev/Staging/Prod | Options: `zai` (default) or `openai`; fallback to deterministic if unavailable |
| **ZAI_API_KEY** | Z.AI API key | `src/lib/llm.ts` | Dev/Staging/Prod | Secret; used for emergency classifier, digest, moderation |
| **ZAI_BASE_URL** | Z.AI API base URL | `src/lib/llm.ts` | Dev/Staging/Prod | Usually `https://api.z.ai/api/paas/v4` |
| **OPENAI_API_KEY** | OpenAI API key (fallback) | `src/lib/llm.ts` | Dev/Staging/Prod | Secret; optional fallback if ZAI unavailable |
| **OPENAI_BASE_URL** | OpenAI API base URL (fallback) | `src/lib/llm.ts` | Dev/Staging/Prod | Usually `https://api.openai.com/v1` |
| **LLM_DEFAULT_MODEL** | Default LLM model ID | `src/lib/llm.ts` | Dev/Staging/Prod | Model name (e.g., `gpt-4-mini` or `glm-4.6`); provider-specific |
| **FEATURE_MONETIZATION_ENABLED** | Enable Stripe monetization features (server) | `/api/stripe/*`, `/app/promote/` | Dev/Staging/Prod | Off by default; set to `1` to enable; requires Stripe keys |
| **NEXT_PUBLIC_FEATURE_MONETIZATION_ENABLED** | Enable Stripe monetization UX (client) | `/app/promote/page.tsx`, checkout forms | Dev/Staging/Prod | Off by default; set to `1` to enable; mirrors server flag |
| **E2E_TEST_MODE** | Enable Playwright E2E test mode | `/api/stripe/*`, emergency endpoints | Dev/Staging | Stubs real Stripe calls + LLM calls for testing |
| **NEXT_PUBLIC_E2E_TEST_MODE** | E2E mode flag (client-side) | Test controls in emergency page | Dev/Staging | Exposes debug controls in Playwright tests |
| **FEATURE_SCRAPER_ENABLED** | Enable web scraper automation | Bulk import workflows | Dev/Staging | Off by default; deferred until ≥95% accuracy sign-off |
| **AUTO_APPLY** | Enable automated apply mode in scripts | `scripts/abn_*.py` | Prod-Ops | Set to `true` for `abn_batch:*:apply` npm scripts; ops-only |
| **CRON_SECRET** | Vercel cron job authentication token | Vercel cron trigger validation | Prod | Secret; used to verify legitimate cron job requests |
| **SENTRY_DSN** | Sentry error tracking endpoint (optional) | Error boundary + `src/lib/errorLog.ts` | Dev/Staging/Prod | Optional; Sentry project DSN |
| **LOGFLARE_API_KEY** | Logflare logging API key (optional) | Structured logging | Dev/Staging/Prod | Optional; Logflare project API key |
| **LOGFLARE_SOURCE_ID** | Logflare source ID (optional) | Structured logging | Dev/Staging/Prod | Optional; Logflare source ID |
| **ALERTS_EMAIL_TO** | Email address for ops alerts | Alert delivery | Prod | Ops email; alerts sent to this address |
| **ALERTS_SLACK_WEBHOOK_URL** | Slack webhook for ops alerts | Alert delivery | Prod | Secret; Slack channel webhook URL |
| **NODE_ENV** | Node.js environment | Build/runtime behavior | Dev/Staging/Prod | `development`, `staging`, `production`; set by Next.js/Vercel |
| **RUNTIME_ENV** | Custom runtime environment tag | `src/lib/config.ts` | Dev/Staging/Prod | `local`, `staging`, `production`; distinguishes from NODE_ENV |
| **VERCEL_ENV** | Vercel deployment environment | Deployment context | Prod | `development`, `preview`, `production`; set by Vercel |
| **VERCEL_URL** | Vercel preview/production URL | Deployment URLs | Prod | Auto-set by Vercel; used for dynamic URL generation |
| **APP_ORIGIN** | App origin URL for redirects | Auth redirects, email links | Dev/Staging/Prod | Base URL (e.g., `http://localhost:3000` or `https://dogtrainersdirectory.com.au`) |
| **NEXT_PUBLIC_SITE_URL** | Public site URL (client-side) | Social meta tags, redirects | Dev/Staging/Prod | Full URL visible to clients |
| **NEXT_PUBLIC_APP_URL** | Public app URL (client-side) | Navigation, links | Dev/Staging/Prod | Mirrors APP_ORIGIN; used in client JS |
| **PGCRYPTO_KEY** | PgCrypto encryption context (legacy) | Profile encryption | Dev/Staging/Prod | Integer; legacy reference; use SUPABASE_PGCRYPTO_KEY instead |
| **AI_GLOBAL_MODE** | Global AI mode override | Emergency/LLM decision | Dev/Staging/Prod | Options: `enabled`, `disabled`, `deterministic`; overrides provider availability |
| **TRIAGE_AI_MODE** | Triage-specific AI mode | Emergency triage classifier | Dev/Staging/Prod | Options: same as AI_GLOBAL_MODE |
| **VERIFICATION_AI_MODE** | Verification job AI mode | Emergency verification sweep | Dev/Staging/Prod | Options: same as AI_GLOBAL_MODE |
| **DIGEST_AI_MODE** | Daily ops digest AI mode | LLM-powered digest | Dev/Staging/Prod | Options: same as AI_GLOBAL_MODE |
| **MODERATION_AI_MODE** | Review moderation AI mode | AI review flagging | Dev/Staging/Prod | Options: same as AI_GLOBAL_MODE |

---

## Configuration Sources

### `.env.local` (Local Development, Git-Ignored)
Create manually with remote Supabase credentials:
```bash
NEXT_PUBLIC_SUPABASE_URL=https://...supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=...
SUPABASE_SERVICE_ROLE_KEY=...
# ... other secrets
```

### Vercel Dashboard (Staging/Production Secrets)
Secrets stored in Vercel environment:
- `SUPABASE_SERVICE_ROLE_KEY`
- `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`
- `ABR_GUID`, `ZAI_API_KEY`, `OPENAI_API_KEY`
- `CRON_SECRET`, `ALERTS_*`

### `config/env_required.json` (CI Gate)
Lists mandatory env vars per environment:
- Checked by `scripts/check_env_ready.sh`
- Blocks `npm run verify:launch` if missing

### `.env.example` (If Present)
Template file showing which vars are needed (NO values).

---

## Secret Scanning Pattern

High-risk patterns to never commit:
- `sk_live_` or `sk_test_` (Stripe keys)
- `whsec_` (Stripe webhook secrets)
- `-----BEGIN` (private keys)
- `service_role` / `SUPABASE_SERVICE_ROLE_KEY` with actual values
- `REDACTED` values in documentation (intentional masking)

CI scanners should fail on these patterns; use git hooks or pre-commit tools to prevent accidental commits.

