# Local Development Environment Snapshot

**Generated:** 2025-12-17  
**Repository Branch:** merge/phase5-core

## Version Information

```
Node.js:  v20.19.2
npm:      11.6.2
```

## Git Status

**Current Branch:** `merge/phase5-core` (tracking `origin/merge/phase5-core`)  
**Default Branch:** `main`

**Modified Files (uncommitted):**
```
.github/copilot-instructions.md
src/app/search/page.tsx
src/app/trainer/[id]/page.tsx
src/app/trainers/[id]/page.tsx
tests/e2e/search-and-trainer.spec.ts-snapshots/*
src/components/e2e/TrainerFallbackClient.tsx
```

**Recent Commits (Last 10):**
```
36f7fad (HEAD -> merge/phase5-core, origin/merge/phase5-core) test(e2e): add emergency e2e controls and optional skip for Playwright webServer
8a67154 test(e2e): render emergency e2e controls under NEXT_PUBLIC_E2E_TEST_MODE
3c648e8 chore: lint fixes
bd1f723 chore: remove temporary debug stdout writes from emergency routes
de60b0e feat(api/emergency): include classification/medical payload and resourceId in responses for smoke tests
d05aa24 fix(api/emergency): fallback to deterministic mode if resolveLlmMode throws (smoke test compatibility)
22f4c72 debug: print route errors to stdout for smoke test debugging
058ddbb fix(api/emergency): make verify/triage weekly endpoints tolerant for tests; return snake_case metrics
7d70f89 fix(api/emergency): accept legacy message field and tolerate missing supabase mock; test: render dashboards instead of pages for smoke tests
3ec5db6 test: restrict vitest include globs to repo tests and exclude e2e
```

## Running Locally

### 1. Prerequisites
```bash
# Install Node.js v24 (recommended; v20 also works)
nvm install 24 && nvm use 24

# Install dependencies
npm ci

# Create .env.local with Supabase credentials (see ENV_MATRIX.md)
cat > .env.local << 'EOF'
NEXT_PUBLIC_SUPABASE_URL=https://...supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=...
SUPABASE_SERVICE_ROLE_KEY=REDACTED
ABR_GUID=REDACTED
LLM_PROVIDER=zai
ZAI_API_KEY=REDACTED
EOF
```

### 2. Start Development Server
```bash
npm run dev
# Server listens on http://localhost:3000
```

### 3. Run Quality Checks
```bash
npm run lint          # ESLint (eslint.config.mjs)
npm run type-check    # TypeScript type checking (strict mode)
npm run smoke         # Vitest unit tests
npm run e2e           # Playwright E2E tests (requires running Next.js server)
npm run verify:launch # Full verification gate (type-check → smoke → lint → doc-divergence → env-ready)
```

### 4. Build & Start Production
```bash
npm run build         # Turbopack build (Next.js 16+)
npm start             # Start production server
```

## Service Dependencies

### Supabase (Remote, Default)
- URL: Auto-detected from `NEXT_PUBLIC_SUPABASE_URL`
- Auth: Email-based + Trainer account claims
- DB: PostgreSQL with 25+ tables, RPC functions, edge functions
- Storage: Profile images, business documents (optional)
- **No local Docker required** — uses cloud-hosted remote project

### Supabase Local (Optional, Advanced)
If you need local Supabase for isolated testing:
```bash
# Install Supabase CLI (macOS)
brew install supabase/tap/supabase

# Start local Supabase emulator (Docker required)
supabase start

# Configure .env.local to point to local project
NEXT_PUBLIC_SUPABASE_URL=http://localhost:54321
NEXT_PUBLIC_SUPABASE_ANON_KEY=... # from 'supabase status'
SUPABASE_SERVICE_ROLE_KEY=... # from 'supabase status'

# Stop when done
supabase stop
```

### Stripe (Feature-Flagged, Optional)
- **Default:** Disabled (feature flag off)
- **Local testing** (if enabled): Use Stripe CLI webhook forwarder
  ```bash
  stripe listen --forward-to http://localhost:4243/api/webhooks/stripe-dtd
  ```
- **Port:** Custom harness uses port 4243 (not 3000) to avoid conflicts

### LLM Provider (Z.AI or OpenAI, Optional)
- **Default:** Z.AI (`LLM_PROVIDER=zai`)
- **Fallback:** OpenAI if Z.AI unavailable
- **Graceful degradation:** If both unavailable, deterministic rules apply
- **Used for:** Emergency triage classifier, daily ops digest, review moderation

## Ports & URLs

| Service | Port | URL | Notes |
|---------|------|-----|-------|
| Next.js Dev | 3000 | http://localhost:3000 | App dev server |
| Next.js Prod | 3000 | http://localhost:3000 | After `npm start` |
| Supabase Local (API) | 54321 | http://localhost:54321 | If using local emulator |
| Supabase Local (Studio) | 54323 | http://localhost:54323 | Admin UI if using local |
| Stripe Webhook Harness | 4243 | http://localhost:4243 | Custom webhook forwarder |

## File Structure Overview

```
DTD (repo root)
├── .github/
│   ├── workflows/              # CI/CD workflows (lint, type-check, verify-launch, etc.)
│   └── copilot-instructions.md # AI agent onboarding (this file)
├── src/
│   ├── app/                    # Next.js App Router pages + API routes
│   │   ├── page.tsx            # Home / search triage page
│   │   ├── search/             # Search results page
│   │   ├── trainers/[id]/      # Trainer profile (ID-based)
│   │   ├── emergency/          # Emergency help flow
│   │   ├── admin/              # Admin dashboard
│   │   ├── api/                # API routes (triage, ABN, emergency, etc.)
│   │   └── middleware.ts       # Auth + redirect middleware
│   ├── components/             # Reusable React components
│   ├── lib/                    # Utilities (API client, ABR parser, LLM wrapper, etc.)
│   ├── types/                  # TypeScript types + validators (database.ts)
│   └── styles/                 # Tailwind CSS config
├── supabase/
│   ├── migrations/             # Schema migrations (applied in order)
│   ├── schema.sql              # Snapshot of current schema
│   └── functions/              # Edge functions (optional)
├── tests/
│   ├── e2e/                    # Playwright E2E tests
│   ├── unit/                   # Vitest unit tests
│   └── fixtures/               # Test data
├── DOCS/                       # Project documentation (SSOT, runbooks, etc.)
├── scripts/                    # Utility scripts (ABN batch, allowlist gen, etc.)
├── package.json                # Dependencies + npm scripts
├── tsconfig.json               # TypeScript config (strict mode)
├── next.config.js              # Next.js config
├── playwright.config.ts        # Playwright E2E config
├── vitest.config.ts            # Vitest unit test config
└── .env.local                  # Local secrets (git-ignored, create manually)
```

## Common Development Tasks

### Starting Fresh
```bash
# Clean node_modules and reinstall
rm -rf node_modules && npm ci

# Reset Next.js cache
rm -rf .next && npm run dev
```

### Running Specific Tests
```bash
# Single test file
npm run smoke -- tests/unit/search.test.ts

# Tests matching pattern
npm run test -- --grep "emergency"

# Watch mode for development
npm run test:watch
```

### Checking Database Schema
```bash
# If using remote Supabase, pull latest schema
supabase db pull

# Compare local migrations with schema snapshot
git diff supabase/schema.sql
```

### Building for Production
```bash
# Build with Turbopack (Next.js 16+)
NEXT_TELEMETRY_DISABLED=1 npm run build

# Check bundle size
ls -lh .next/server/app/emergency/page.js
```

### Debugging
```bash
# Enable verbose logging
DEBUG=* npm run dev

# Check environment readiness
./scripts/check_env_ready.sh

# Validate docs vs code
python3 scripts/check_docs_divergence.py --base-ref origin/main
```

## Troubleshooting

### "Cannot find module '@/types/database'"
- Check that `tsconfig.json` includes `"baseUrl": "."` and `"@/*": ["./src/*"]`
- Restart TypeScript language server (Cmd+Shift+P → "TypeScript: Restart TS Server")

### "Supabase connection refused"
- Verify `NEXT_PUBLIC_SUPABASE_URL` is correct and project is active
- Check `.env.local` values match Supabase project settings
- If using local Supabase, ensure `supabase start` is running

### "Type error: Cannot find type AgeSpecialty"
- Run `npm run type-check` to confirm
- If error persists, check `src/types/database.ts` exports enums
- Verify imports use `import type` for types, not `import` for values

### "E2E tests fail with 'Playwright timeout'"
- Ensure `npm run dev` is running before starting E2E tests
- Check `NEXT_PUBLIC_E2E_TEST_MODE` is NOT set (disables server-side emulation)
- Increase timeout: `npm run e2e -- --timeout 30000`

### "ABN verification returns 'fallback event'"
- Check `ABR_GUID` is set correctly
- Verify network connectivity (ABR API call may be failing)
- Check `abn_fallback_events` table for recent failures
- See `/api/admin/abn/fallback-stats` dashboard for historical rate

