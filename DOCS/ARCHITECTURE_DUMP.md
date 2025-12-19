## Schema

### emergency_triage_logs (remote `\d+`)
```
                                                                                                   Table "public.emergency_triage_logs"
       Column        |           Type           | Collation | Nullable |                      Default                      | Storage  | Compression | Stats target |                             Description                              
---------------------+--------------------------+-----------+----------+---------------------------------------------------+----------+-------------+--------------+----------------------------------------------------------------------
 id                  | bigint                   |           | not null | nextval('emergency_triage_logs_id_seq'::regclass) | plain    |             |              | 
 description         | text                     |           | not null |                                                   | extended |             |              | 
 predicted_category  | emergency_classification |           | not null |                                                   | plain    |             |              | 
 recommended_flow    | emergency_classification |           | not null |                                                   | plain    |             |              | 
 confidence          | numeric(5,2)             |           |          |                                                   | main     |             |              | 
 classifier_version  | text                     |           |          | 'phase5-rule-v1'::text                            | extended |             |              | 
 source              | text                     |           |          | 'rule_based'::text                                | extended |             |              | 
 user_suburb_id      | integer                  |           |          |                                                   | plain    |             |              | 
 user_lat            | numeric                  |           |          |                                                   | main     |             |              | 
 user_lng            | numeric                  |           |          |                                                   | main     |             |              | 
 resolution_category | emergency_classification |           |          |                                                   | plain    |             |              | 
 was_correct         | boolean                  |           |          |                                                   | plain    |             |              | 
 feedback_notes      | text                     |           |          |                                                   | extended |             |              | 
 metadata            | jsonb                    |           |          | '{}'::jsonb                                       | extended |             |              | 
 created_at          | timestamp with time zone |           |          | now()                                             | plain    |             |              | 
 resolved_at         | timestamp with time zone |           |          |                                                   | plain    |             |              | 
 decision_source     | text                     |           |          |                                                   | extended |             |              | Source of the classification: llm, deterministic, or manual_override
 ai_mode             | text                     |           |          |                                                   | extended |             |              | The AI execution mode active at the time: live, shadow, or disabled
 ai_provider         | text                     |           |          |                                                   | extended |             |              | 
 ai_model            | text                     |           |          |                                                   | extended |             |              | 
 ai_prompt_version   | text                     |           |          |                                                   | extended |             |              | 
 situation           | text                     |           |          |                                                   | extended |             |              | 
 location            | text                     |           |          |                                                   | extended |             |              | 
 contact             | text                     |           |          |                                                   | extended |             |              | 
 priority            | text                     |           |          |                                                   | extended |             |              | 
 follow_up_actions   | text[]                   |           |          |                                                   | extended |             |              | 
 dog_age             | text                     |           |          |                                                   | extended |             |              | 
 issues              | text[]                   |           |          |                                                   | extended |             |              | 
 classification      | text                     |           |          |                                                   | extended |             |              | 
Indexes:
    "emergency_triage_logs_pkey" PRIMARY KEY, btree (id)
    "idx_emergency_triage_logs_created_at" btree (created_at DESC)
    "idx_emergency_triage_logs_prompt_version" btree (ai_prompt_version) WHERE ai_prompt_version IS NOT NULL
    "idx_emergency_triage_logs_resolution" btree (resolution_category)
Check constraints:
    "emergency_triage_logs_decision_source_check" CHECK (decision_source = ANY (ARRAY['llm'::text, 'deterministic'::text, 'manual_override'::text]))
Access method: heap
```

### featured_placements (remote `\d+`)
```
                                                                          Table "public.featured_placements"
           Column           |           Type           | Collation | Nullable |                     Default                     | Storage  | Compression | Stats target | Description 
----------------------------+--------------------------+-----------+----------+-------------------------------------------------+----------+-------------+--------------+-------------
 id                         | integer                  |           | not null | nextval('featured_placements_id_seq'::regclass) | plain    |             |              | 
 business_id                | integer                  |           |          |                                                 | plain    |             |              | 
 lga_id                     | integer                  |           |          |                                                 | plain    |             |              | 
 stripe_checkout_session_id | text                     |           |          |                                                 | extended |             |              | 
 stripe_payment_intent_id   | text                     |           |          |                                                 | extended |             |              | 
 start_date                 | timestamp with time zone |           | not null |                                                 | plain    |             |              | 
 end_date                   | timestamp with time zone |           | not null |                                                 | plain    |             |              | 
 status                     | text                     |           |          | 'active'::text                                  | extended |             |              | 
 created_at                 | timestamp with time zone |           |          | now()                                           | plain    |             |              | 
 priority                   | integer                  |           |          | 0                                               | plain    |             |              | 
 slot_type                  | text                     |           |          | 'standard'::text                                | extended |             |              | 
 active                     | boolean                  |           |          | true                                            | plain    |             |              | 
 expiry_date                | timestamp with time zone |           |          |                                                 | plain    |             |              | 
Indexes:
    "featured_placements_pkey" PRIMARY KEY, btree (id)
    "featured_placements_stripe_checkout_session_id_key" UNIQUE CONSTRAINT, btree (stripe_checkout_session_id)
    "featured_placements_stripe_payment_intent_id_key" UNIQUE CONSTRAINT, btree (stripe_payment_intent_id)
    "idx_featured_placements_dates" btree (start_date, end_date)
    "idx_featured_placements_status" btree (status)
Check constraints:
    "featured_placements_status_check" CHECK (status = ANY (ARRAY['active'::text, 'expired'::text, 'cancelled'::text]))
Foreign-key constraints:
    "featured_placements_business_id_fkey" FOREIGN KEY (business_id) REFERENCES businesses(id) ON DELETE CASCADE
    "featured_placements_lga_id_fkey" FOREIGN KEY (lga_id) REFERENCES councils(id)
Access method: heap
```

### payment_audit (remote `\d+`)
```
                                                                 Table "public.payment_audit"
         Column         |           Type           | Collation | Nullable |           Default            | Storage  | Compression | Stats target | Description 
------------------------+--------------------------+-----------+----------+------------------------------+----------+-------------+--------------+-------------
 id                     | uuid                     |           | not null | gen_random_uuid()            | plain    |             |              | 
 business_id            | bigint                   |           |          |                              | plain    |             |              | 
 plan_id                | text                     |           | not null |                              | extended |             |              | 
 event_type             | text                     |           | not null |                              | extended |             |              | 
 status                 | text                     |           | not null |                              | extended |             |              | 
 stripe_customer_id     | text                     |           |          |                              | extended |             |              | 
 stripe_subscription_id | text                     |           |          |                              | extended |             |              | 
 metadata               | jsonb                    |           |          | '{}'::jsonb                  | extended |             |              | 
 originating_route      | text                     |           |          |                              | extended |             |              | 
 created_at             | timestamp with time zone |           | not null | timezone('utc'::text, now()) | plain    |             |              | 
 sync_error             | text                     |           |          |                              | extended |             |              | 
Indexes:
    "payment_audit_pkey" PRIMARY KEY, btree (id)
    "payment_audit_business_idx" btree (business_id)
    "payment_audit_event_idx" btree (event_type, created_at)
Access method: heap
```

### search_telemetry (remote `\d+`)
```
                                                     Table "public.search_telemetry"
    Column    |           Type           | Collation | Nullable |      Default      | Storage  | Compression | Stats target | Description 
--------------+--------------------------+-----------+----------+-------------------+----------+-------------+--------------+-------------
 id           | uuid                     |           | not null | gen_random_uuid() | plain    |             |              | 
 operation    | text                     |           | not null |                   | extended |             |              | 
 suburb_id    | integer                  |           |          |                   | plain    |             |              | 
 suburb_name  | text                     |           |          |                   | extended |             |              | 
 result_count | integer                  |           | not null | 0                 | plain    |             |              | 
 latency_ms   | integer                  |           | not null |                   | plain    |             |              | 
 success      | boolean                  |           | not null | true              | plain    |             |              | 
 error        | text                     |           |          |                   | extended |             |              | 
 timestamp    | timestamp with time zone |           | not null | now()             | plain    |             |              | 
Indexes:
    "search_telemetry_pkey" PRIMARY KEY, btree (id)
    "idx_search_telemetry_operation" btree (operation)
    "idx_search_telemetry_suburb_id" btree (suburb_id)
    "idx_search_telemetry_timestamp" btree ("timestamp" DESC)
Foreign-key constraints:
    "search_telemetry_suburb_id_fkey" FOREIGN KEY (suburb_id) REFERENCES suburbs(id)
Access method: heap
```

### cron_job_runs (remote `\d+`)
```
                                                                                           Table "public.cron_job_runs"
    Column     |           Type           | Collation | Nullable |                  Default                  | Storage  | Compression | Stats target |                        Description                         
---------------+--------------------------+-----------+----------+-------------------------------------------+----------+-------------+--------------+------------------------------------------------------------
 id            | bigint                   |           | not null | nextval('cron_job_runs_id_seq'::regclass) | plain    |             |              | 
 job_name      | text                     |           | not null |                                           | extended |             |              | 
 started_at    | timestamp with time zone |           | not null |                                           | plain    |             |              | 
 completed_at  | timestamp with time zone |           |          |                                           | plain    |             |              | 
 status        | text                     |           |          |                                           | extended |             |              | 
 error_message | text                     |           |          |                                           | extended |             |              | 
 duration_ms   | integer                  |           |          |                                           | plain    |             |              | Execution time in milliseconds (completed_at - started_at)
 metadata      | jsonb                    |           |          | '{}'::jsonb                               | extended |             |              | 
 created_at    | timestamp with time zone |           |          | now()                                     | plain    |             |              | 
Indexes:
    "cron_job_runs_pkey" PRIMARY KEY, btree (id)
    "idx_cron_job_runs_name" btree (job_name)
    "idx_cron_job_runs_started_at" btree (started_at DESC)
    "idx_cron_job_runs_status" btree (status)
Check constraints:
    "cron_job_runs_status_check" CHECK (status = ANY (ARRAY['running'::text, 'success'::text, 'failed'::text]))
Access method: heap
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
src/app/directory/page.tsx
src/app/trainers
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

### EmergencyTriageLog (local types)
From `src/types/database.ts`:
```ts
export interface EmergencyTriageLog {
  id: string
  created_at: string
  dog_age: string | null
  issues: string[] | null
  decision_source: 'llm' | 'deterministic' | 'manual_override'
  classification: string
  situation?: string
  location?: string
  contact?: string
  priority?: string
  follow_up_actions?: string[]
  ai_mode?: string
  ai_provider?: string
  ai_model?: string
  description?: string
  predicted_category?: string
  recommended_flow?: string
  confidence?: number
  classifier_version?: string
  source?: string
  user_suburb_id?: number
  user_lat?: number
  user_lng?: number
  resolution_category?: string
  was_correct?: boolean
  resolved_at?: string
  feedback_notes?: string
  metadata?: Record<string, unknown>
  ai_prompt_version?: string
}
```

### FeaturedPlacement (local types)
From `src/types/database.ts`:
```ts
export interface FeaturedPlacement {
  id: number
  business_id: number
  lga_id: number
  stripe_checkout_session_id?: string
  stripe_payment_intent_id?: string
  start_date: string
  end_date: string
  expiry_date?: string | null
  status: 'active' | 'expired' | 'cancelled'
  priority: number
  slot_type: 'hero' | 'premium' | 'standard'
  active: boolean
  created_at: string
}
```

### PaymentAudit (local types)
From `src/types/database.ts`:
```ts
export interface PaymentAudit {
  id: string
  business_id?: number | null
  plan_id: string
  event_type: string
  status: string
  stripe_customer_id?: string | null
  stripe_subscription_id?: string | null
  metadata?: Record<string, unknown>
  originating_route?: string | null
  sync_error?: string | null
  created_at: string
}
```

## Remote Schema Verification Reference

- Verified: 2025-12-20 02:26 AEDT
- Method: read-only `psql` checks against remote Supabase (`\d+`)
- Checks: `emergency_triage_logs` columns (including `dog_age`, `issues`, `classification`), `featured_placements`, `payment_audit`, `search_telemetry`, `cron_job_runs`
- Reference report path: `/Users/carlg/Documents/supabase-report.md`
