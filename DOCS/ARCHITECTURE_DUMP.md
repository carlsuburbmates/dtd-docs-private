<!-- DOCS_DIVERGENCE_IGNORE: supporting index or changelog -->

## Schema

### emergency_triage_logs
Create table definition not found in this repo (no matches in `supabase/schema.sql` or `supabase/migrations/`). Only alterations and references exist.

From `supabase/migrations/20251206010000_ai_automation_complete.sql`:
```sql
ALTER TABLE emergency_triage_logs 
ADD COLUMN IF NOT EXISTS ai_prompt_version TEXT;

CREATE INDEX IF NOT EXISTS idx_emergency_triage_logs_prompt_version 
ON emergency_triage_logs(ai_prompt_version) WHERE ai_prompt_version IS NOT NULL;
```

`decision_source` is referenced (not defined) in the AI health summary view in the same migration:
```sql
COUNT(*) FILTER (WHERE COALESCE(decision_source, 'deterministic') = 'llm') as ai_decisions,
COUNT(*) FILTER (WHERE COALESCE(decision_source, 'deterministic') = 'deterministic') as deterministic_decisions,
COUNT(*) FILTER (WHERE COALESCE(decision_source, 'deterministic') = 'manual_override') as manual_overrides
```

### search_telemetry
From `supabase/migrations/20241208020000_search_telemetry.sql`:
```sql
create table if not exists public.search_telemetry (
    id uuid primary key default gen_random_uuid() not null,
    operation text not null, -- 'search_suburbs', 'triage_search', etc.
    suburb_id int references public.suburbs(id),
    suburb_name text, -- Store suburb name for easier analysis
    result_count int not null default 0,
    latency_ms int not null, -- Response time in milliseconds
    success boolean not null default true,
    error text, -- Error message if operation failed
    timestamp timestamptz not null default now()
);
```
Note: no `p95` column exists in the table definition. A `p95_latency` value is computed in the `get_search_latency_stats` function in the same migration.

### payment_audit
From `supabase/migrations/20251209101000_create_payment_tables.sql`:
```sql
create table if not exists public.payment_audit (
  id uuid primary key default gen_random_uuid(),
  business_id bigint references public.businesses(id) on delete set null,
  plan_id text not null,
  event_type text not null,
  status text not null,
  stripe_customer_id text,
  stripe_subscription_id text,
  metadata jsonb default '{}'::jsonb,
  originating_route text,
  created_at timestamptz not null default timezone('utc', now())
);
```
Note: no `sync_error` column exists in this definition.

### featured_placements
From `supabase/schema.sql`:
```sql
CREATE TABLE featured_placements (
    id SERIAL PRIMARY KEY,
    business_id INTEGER REFERENCES businesses(id) ON DELETE CASCADE,
    lga_id INTEGER REFERENCES councils(id),
    stripe_checkout_session_id TEXT UNIQUE,
    stripe_payment_intent_id TEXT UNIQUE,
    start_date TIMESTAMP WITH TIME ZONE NOT NULL,
    end_date TIMESTAMP WITH TIME ZONE NOT NULL,
    status TEXT DEFAULT 'active' CHECK (status IN ('active', 'expired', 'cancelled')),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```
Note: no `expiry_date` or `active` column exists in this definition.

### cron_job_runs
From `supabase/migrations/20251206010000_ai_automation_complete.sql`:
```sql
CREATE TABLE IF NOT EXISTS cron_job_runs (
  id BIGSERIAL PRIMARY KEY,
  job_name TEXT NOT NULL,
  started_at TIMESTAMPTZ NOT NULL,
  completed_at TIMESTAMPTZ,
  status TEXT CHECK (status IN ('running', 'success', 'failed')),
  error_message TEXT,
  duration_ms INT,
  metadata JSONB DEFAULT '{}'::jsonb,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

## File Tree
Output of `find src -maxdepth 3 -not -path '*/.*'`:
```
src
src/middleware
src/middleware/validation.ts
src/types
src/types/database.ts
src/app
src/app/promote
src/app/promote/promote-panel.tsx
src/app/promote/page.tsx
src/app/wizard
src/app/wizard/layout.tsx
src/app/admin
src/app/admin/queue-card.tsx
src/app/admin/enhanced-dashboard.tsx
src/app/admin/triage
src/app/admin/ai-health
src/app/admin/cron-health
src/app/admin/layout.tsx
src/app/admin/errors
src/app/admin/page.tsx
src/app/admin/reviews
src/app/emergency
src/app/emergency/layout.tsx
src/app/emergency/page.tsx
src/app/triage
src/app/triage/page.tsx
src/app/directory
src/app/directory/fetchDirectoryRegions.test.ts
src/app/directory/page.tsx
src/app/trainers
src/app/trainers/get_trainer_profile.test.ts
src/app/trainers/[id]
src/app/trainers/page.tsx
src/app/search
src/app/search/metadata.ts
src/app/search/page.tsx
src/app/search/SearchClient.tsx
src/app/layout.tsx
src/app/api
src/app/api/test
src/app/api/health
src/app/api/business
src/app/api/admin
src/app/api/emergency
src/app/api/abn
src/app/api/public
src/app/api/telemetry
src/app/api/webhooks
src/app/api/trainer
src/app/api/onboarding
src/app/api/validation
src/app/api/stripe
src/app/page.tsx
src/app/globals.css
src/app/trainer
src/app/trainer/[id]
src/app/onboarding
src/app/onboarding/page.tsx
src/tests
src/tests/unit
src/tests/unit/evaluate.test.ts
src/tests/unit/admin_reviews.test.ts
src/tests/unit/admin_featured.test.ts
src/tests/fixtures
src/tests/fixtures/ai_golden_triage_sample.json
src/utils
src/utils/apiWrapper.ts
src/utils/normalize.ts
src/styles
src/styles/theme.css
src/components
src/components/ui
src/components/ui/SuburbAutocomplete.tsx
src/components/ui/Loading.tsx
src/components/ui/Button.tsx
src/components/ui/Callout.tsx
src/components/ReviewList.tsx
src/components/layout
src/components/layout/AppHeader.tsx
src/components/layout/AppFooter.tsx
src/components/admin
src/components/admin/AdminStatusStrip.tsx
src/components/admin/AiHealthDashboard.tsx
src/components/admin/TelemetryOverrideToggle.tsx
src/components/admin/ErrorMetricsChart.tsx
src/components/admin/TriageMetricsChart.tsx
src/components/admin/CronHealthDashboard.tsx
src/components/emergency
src/components/emergency/EmergencyControls.tsx
src/components/triage
src/components/triage/EmergencyGate.tsx
src/components/DecisionSourceBadge.tsx
src/components/e2e
src/components/e2e/EmergencyE2EControls.tsx
src/components/SearchAutocomplete.tsx
src/lib
src/lib/abr.test.ts
src/lib/errorAlerts.ts
src/lib/eval.ts
src/lib/contracts.ts
src/lib/constants
src/lib/constants/taxonomies.ts
src/lib/e2eTestUtils.ts
src/lib/emergencyTriage.ts
src/lib/emergency.ts
src/lib/triageLog.ts
src/lib/utils
src/lib/utils/cn.ts
src/lib/api.ts
src/lib/monetization.ts
src/lib/errorLog.ts
src/lib/llm.ts
src/lib/prompt-versions.ts
src/lib/telemetryLatency.ts
src/lib/featured-constants.ts
src/lib/ai-types.ts
src/lib/moderation.ts
src/lib/abr.ts
src/lib/supabase.ts
src/lib/medicalDetector.ts
src/lib/encryption.ts
src/lib/abnFallback.ts
src/lib/services
src/lib/services/moderation-service.ts
src/lib/digest.ts
src/lib/alerts.ts
```

Verified paths exist:
- `src/lib/llm.ts`
- `src/lib/digest.ts`
- `src/app/api/admin/latency/route.ts`
- `src/components/admin/AdminStatusStrip.tsx`

## Types

### Trainer or Listing
No `Trainer` or `Listing` interface/type found in the repo. Closest listing-related interface is `Business` in `src/types/database.ts`:
```ts
export interface Business {
  id: number
  profile_id: string
  name: string
  phone?: string
  email?: string
  website?: string
  address?: string
  suburb_id: number
  bio?: string
  pricing?: string
  abn?: string
  abn_verified: boolean
  verification_status: VerificationStatus
  featured_until?: string
  is_active: boolean
  created_at: string
  updated_at: string
}
```

### InboxItem or AdminTask
No `InboxItem` or `AdminTask` interface/type found in the repo.

### Review
From `src/types/database.ts`:
```ts
export interface Review {
  id: number
  business_id: number
  reviewer_name: string
  reviewer_email: string
  rating: number
  title?: string
  content?: string
  is_approved: boolean
  created_at: string
  updated_at: string
}
```

## Phase 2 Findings
Note: Supabase is remote in this environment; no local Docker is used, so schema inspection is limited to repo artifacts and generated types (none found).

### Generated Supabase Types
No generated Supabase type files found. Searches for `database.types.ts`, `types/supabase.ts`, and similar returned no matches. The only Supabase client file is `src/lib/supabase.ts`, which does not contain schema typings.

No TypeScript table definitions were found for:
- `emergency_triage_logs` (or `triage_logs`)
- `decision_source`
- `daily_ops_digests`

### Inbox Reverse-Engineering (Admin Queues)
No `Inbox` component found. The admin queue UI is implemented via `src/app/admin/page.tsx` and `src/app/admin/queue-card.tsx`.

`QueuePayload` (data shape loaded from `/api/admin/queues`) in `src/app/admin/page.tsx`:
```ts
type QueuePayload = {
  reviews: Array<{
    id: number
    business_id: number
    reviewer_name: string
    rating: number
    title: string
    content?: string
    created_at: string
  }>
  abn_verifications: Array<{
    id: number
    business_id: number
    abn: string
    similarity_score: number
    status: string
    created_at: string
  }>
  flagged_businesses: Array<{
    id: number
    name: string
    verification_status: string
    is_active: boolean
    featured_until: string | null
  }>
}
```

`QueueCard` props in `src/app/admin/queue-card.tsx` (data prop passed to the component):
```ts
type QueueCardProps = {
  title: string
  items: { id: number; title: string; meta: string; body: string; action?: 'review' }[]
  onReview?: (id: number, action: 'approve' | 'reject') => Promise<void>
}
```

Fields in the queue UI items do not include `severity`, `source`, or `action_type`.

### Kill Switch Constants (Decision Source / Modes)
No `decision_source` constants found in `src/lib/llm.ts`.

`DecisionSource` and `DecisionMode` are defined in `src/lib/ai-types.ts`:
```ts
export type DecisionSource = 'llm' | 'deterministic' | 'manual_override'

export type DecisionMode = 'live' | 'shadow' | 'disabled'
```
