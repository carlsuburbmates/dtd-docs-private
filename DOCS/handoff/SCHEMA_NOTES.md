# Database Schema Overview

**Source:** `supabase/schema.sql` + `supabase/migrations/`  
**Last Updated:** 2025-12-09  
**Scope:** PostgreSQL schema for dogtrainersdirectory.com.au

---

## Enums (Locked Types)

All enums are immutable and enforced at the database level:

### `region`
Inner City, Northern, Eastern, South Eastern, Western

### `user_role`
trainer, admin

### `verification_status`
pending, verified, rejected, manual_review

### `age_specialty`
puppies_0_6m, adolescent_6_18m, adult_18m_7y, senior_7y_plus, rescue_dogs

### `behavior_issue`
pulling_on_lead, separation_anxiety, excessive_barking, dog_aggression, leash_reactivity, jumping_up, destructive_behaviour, recall_issues, anxiety_general, resource_guarding, mouthing_nipping_biting, rescue_dog_support, socialisation

### `service_type`
puppy_training, obedience_training, behaviour_consultations, group_classes, private_training

### `resource_type`
trainer, behaviour_consultant, emergency_vet, urgent_care, emergency_shelter

---

## Core Tables

### `councils` (28 records)
Local government areas in Melbourne. Immutable lookup table.

**Columns:** id, name, region, postcode  
**Key:** Seeded from `suburbs_councils_mapping.csv`  
**Note:** Never modify; any change requires RFC + migration

### `suburbs` (138 records)
Melbourne suburbs with geographic data. Immutable lookup table.

**Columns:** id, name, council_id (FK), region, postcode, latitude, longitude  
**Key:** Auto-derived from councils; supports geocoding queries  
**Indexes:** (council_id), (name), (region)

### `profiles` (Auth-linked)
Supabase Auth integration. Trainer/owner profiles.

**Columns:** id (PK, UUID), email, created_at, updated_at  
**Link:** Supabase Auth users (one-to-one)

### `businesses` (Main Content)
Trainers, consultants, vets, shelters. Core resource type.

**Key Columns:**
- id (UUID)
- profile_id (FK → profiles)
- name, email, phone, website
- description, address, suburb_id (FK → suburbs)
- resource_type (trainer | behaviour_consultant | emergency_vet | urgent_care | emergency_shelter)
- abn (Australian Business Number, unique, nullable)
- abn_verified (boolean; computed from ABN verification)
- verification_status (pending | verified | rejected | manual_review)
- is_featured (boolean; for paid promotion slots)
- is_deleted (boolean; soft-delete flag)
- claimed (boolean; trainer confirmed email)
- created_at, updated_at, last_verified_at

**Indexes:** (abn), (verification_status), (is_featured), (is_deleted), (suburb_id)

### `trainer_specializations` (Age-focused)
Trainers' age specialties (one or more).

**Columns:** id, business_id (FK), age_specialty (enum)  
**Key:** Allows trainers to select multiple ages (e.g., puppies + adolescent)

### `trainer_behavior_issues` (Issue-focused)
Trainers' behavior issue expertise (one or more).

**Columns:** id, business_id (FK), behavior_issue (enum)  
**Key:** Allows trainers to select multiple issues (e.g., separation anxiety + aggression)

### `trainer_services` (Service-focused)
Trainers' service types (one or more).

**Columns:** id, business_id (FK), service_type (enum)  
**Key:** Allows trainers to offer multiple service types (e.g., group + private)

### `reviews`
User reviews of trainers/services.

**Key Columns:**
- id (UUID)
- business_id (FK)
- reviewer_id (FK → profiles, nullable for anonymous)
- rating (1–5 stars)
- text (review content, max 500 chars)
- ai_moderation_status (pending | approved | rejected)
- ai_moderation_reason (nullable, reason for AI decision)
- is_deleted (soft-delete)
- created_at, updated_at

### `abn_verifications`
ABN verification audit log.

**Key Columns:**
- id (UUID)
- business_id (FK)
- abn (business ABN)
- name_match_score (float 0–1)
- abr_status (ABNStatus from ABR API: Active | Cancelled | etc.)
- verified (boolean; TRUE if match >= ~0.85 + Active)
- error_message (nullable; if lookup failed)
- verification_date, next_verification_date
- created_at

**Purpose:** Historical record of all ABN verifications for audit/dispute resolution

### `abn_fallback_events`
Records when ABN verification falls back (API unavailable, invalid GUID, etc.).

**Key Columns:**
- id (UUID)
- business_id (FK, nullable)
- abn (string)
- reason (api_unavailable | invalid_guid | network_error | etc.)
- response_code (HTTP status, if applicable)
- created_at

**Purpose:** Alert ops if fallback rate exceeds threshold

### `emergency_resources`
Emergency vets, shelters, crisis trainers (cached from initial seed + manual data).

**Key Columns:**
- id (UUID)
- resource_type (emergency_vet | urgent_care | emergency_shelter | trainer)
- name, phone, emergency_phone, email
- address, suburb_id (FK → suburbs)
- emergency_hours (string; e.g., "24/7", "7pm–6am")
- services_offered (JSONB; array of service tags)
- is_verified (boolean; from last verification run)
- last_verified_at (timestamp)
- verification_status (active | inactive | unverified | unknown)
- created_at, updated_at

**Note:** Quarterly re-verification via `/api/emergency/verify` cron job

### `emergency_triage_logs`
Audit log of emergency triage events.

**Key Columns:**
- id (UUID)
- classification (medical | stray | crisis)
- medical_condition (nullable; if classification = medical)
- resource_id (FK → emergency_resources, if routed to specific resource)
- routed_to_type (trainer | vet | shelter | etc.)
- user_suburb_id (FK → suburbs, nullable)
- ai_used (boolean; TRUE if LLM classifier ran)
- created_at

**Purpose:** Track triage event audit trail + weekly metrics aggregation

### `emergency_triage_weekly_metrics`
Aggregated weekly metrics from triage logs.

**Key Columns:**
- id (UUID)
- week_start_date (date)
- total_triages (int)
- medical_count, stray_count, crisis_count (int)
- routed_to_vet, routed_to_shelter, routed_to_trainer (int)
- ai_classification_rate (float; % where AI classifier ran)
- created_at

**Purpose:** Weekly summary for ops digest + analytics

### `emergency_resource_verification_runs`
Metadata for verification sweep runs.

**Key Columns:**
- id (UUID)
- started_at (timestamp)
- completed_at (nullable; null if in-progress)
- total_resources_checked (int)
- passed_count, failed_count, unknown_count (int)
- created_at

### `emergency_resource_verification_events`
Detailed log of per-resource verification checks.

**Key Columns:**
- id (UUID)
- run_id (FK → verification_runs)
- resource_id (FK → emergency_resources)
- check_type (http_head | phone_call | manual_review | etc.)
- result (passed | failed | unknown)
- error_message (nullable)
- created_at

**Purpose:** Audit trail for ops to review failed verifications

### `ai_review_decisions`
AI moderation decisions for reviews + profiles.

**Key Columns:**
- id (UUID)
- target_type (review | profile)
- target_id (UUID; FK to reviews or businesses)
- ai_decision (approved | rejected | flag_for_review)
- ai_reason (reason for decision)
- human_decision (nullable; approved | rejected after ops review)
- human_reason (nullable)
- created_at, updated_at

**Purpose:** Track AI vs human decisions for transparency + model improvement

### `payment_audit`
Stripe payment events audit log.

**Key Columns:**
- id (UUID)
- business_id (FK → businesses)
- event_type (checkout_session_created | payment_intent_succeeded | subscription_updated | etc.)
- stripe_event_id (string; unique, prevents deduping)
- stripe_event_data (JSONB; full event payload, sanitized of secret keys)
- status (pending | success | failed | sync_error)
- error_message (nullable)
- created_at

**Purpose:** Non-repudiation audit trail for payments; detect duplicate events

### `daily_ops_digests`
Daily summary digests for ops team.

**Key Columns:**
- id (UUID)
- digest_date (date)
- summary_text (text; LLM-generated or fallback)
- pending_reviews_count (int)
- pending_verifications_count (int)
- abn_fallback_rate_24h (float)
- created_at

**Purpose:** Daily briefing for ops

### `ops_overrides`
Telemetry overrides set by ops (e.g., "service temporarily down").

**Key Columns:**
- id (UUID)
- service_name (string; e.g., "abr_api", "emergency_verify")
- override_status (healthy | degraded | down | investigating)
- notes (string; reason for override)
- expires_at (timestamp; auto-expire after N hours)
- created_at, updated_at

**Purpose:** Allow ops to mark services as temporarily down without code changes

### `latency_metrics`
Performance metrics for critical APIs.

**Key Columns:**
- id (UUID)
- endpoint (string; e.g., "/api/triage", "/api/emergency/triage")
- latency_ms (int; response time in milliseconds)
- status_code (int; HTTP response code)
- created_at

**Purpose:** Performance monitoring; feed into admin dashboard + alerts

---

## Key RPC Functions

### `search_trainers(lat, lon, radius, age, issue, suburb, is_featured)`
Main search entry point. Filters trainers by age/issue/suburb, ranked by verification + featured status.

**Returns:** Array of { business: {...}, is_featured, abn_verified, distance_km }

### `search_emergency_resources(lat, lon, resource_type, suburb_id)`
Emergency resource lookup. Returns verified vets/shelters/crisis trainers.

**Returns:** Array of emergency_resource records

---

## Encryption & Secrets

### PgCrypto Extension
PostgreSQL pgcrypto extension enables AES-256 encryption for sensitive fields.

**Function:** `pgp_sym_decrypt(text, key)` decrypts trainer contact fields (optional).

**Note:** Key passed via `pgcrypto.key` connection setting (set in Node.js before query).

---

## Indexes

Performance-critical indexes:
- `(abn)` – for ABN verification lookups
- `(suburb_id)` – for suburb-based search
- `(verification_status)` – for admin queues
- `(is_featured)` – for featured placement ranking
- `(resource_type, is_verified)` – for emergency resource filtering
- `(created_at DESC)` – for audit logs, reverse chronological queries

---

## Migrations

All schema changes are applied via timestamped migration files in `supabase/migrations/`:
- `20250101*` – Phase 1 base schema (councils, suburbs, businesses)
- `20250201*` – Phase 2 search RPC refinements
- `20250203*` – Phase 3 ABN verification tables
- `20250205*` – Phase 4 Stripe payment audit
- `20250207*` – Phase 5 emergency + AI tables

Migrations are applied in order; schema snapshot (`supabase/schema.sql`) is kept in sync for CI drift detection.

---

## Data Retention

- **Soft-deletes:** Businesses, reviews marked `is_deleted = true` (never purged).
- **Audit logs:** ABN verifications, emergency verification events, payment audits kept indefinitely.
- **Triage logs:** Kept for 1 year (archiving via scheduled job, if configured).
- **Temporary tables:** E.g., latency_metrics rows older than 30 days purged (optional retention policy).

