> **SSOT – Canonical Source of Truth**
> Scope: Master SSOT for **dogtrainersdirectory.com.au** — documentation authority model, system snapshot, and change arbitration.
> Status: Active · Generated: 2025-12-19

# SSOT_MASTER — Dog Trainers Directory (DTD)

This document is the **top-level authority** for the DTD documentation set in this repository.  
If a document conflicts with this SSOT, **this SSOT wins** until explicitly amended via the change protocol.


> **Canonical path:** This SSOT must live at `DOCS/SSOT_MASTER.md` in the docs repo. Treat any other filename as a temporary export only.
---

## 0. Authority model (what wins when docs disagree)

### 0.1 Precedence order
1. **Tier 0:** `DOCS/SSOT_MASTER.md` (this document)
2. **Tier 1:** Canonical SSOT documents (SSOT-badged, plus explicitly canonical SSOTs listed below)
3. **Tier 2:** Supporting specs/runbooks (implementation notes, automation runbooks, handoff snapshots)
4. **Tier 3:** Evidence logs (launch runs, incident logs)
5. **Tier 4:** Drafts and legacy conflicts (`DOCS/_drafts/`, `DOCS/_legacy/`, `DOCS/_legacy_conflicts/`) — **non-authoritative**

### 0.2 “Explicitly canonical” docs (even if not SSOT-badged)
These documents are treated as **Tier 1** because they define invariants or platform constants:
- `DOCS/blueprint_ssot_v1.1.md` — domain model + taxonomy + UX invariants
- `DOCS/LISTING_LOGIC_SSOT.md` — listing/search logic constraints (owner-facing + admin)
- `DOCS/handoff/SCHEMA_NOTES.md` — schema, enums, and core RPC signatures
- `DOCS/handoff/ENV_MATRIX.md` — environment variables contract (local/preview/prod)
- `DOCS/STRIPE_TEST_MODE_ALIGNMENT_FINAL.md` — **test-mode** Stripe canonical product/price IDs

### 0.3 Change arbitration rule
If a contributor wants to change behaviour, they must:
- Update this SSOT **first**, or
- Add an entry to `DOCS/CHANGE_CONTROL_LOG.md` explaining why SSOT remains unchanged.

---

## 1. Product snapshot (what DTD is)

### 1.1 Purpose
DTD is a **mobile-first directory + triage platform** for dog owners to find:
- Dog trainers / behaviour consultants
- Emergency resources (vets, shelters, crisis trainers)
- Council-local resources (LGA routing)

### 1.2 Primary user roles
- **Dog owner (anonymous or user):** search by age/stage → issue → suburb/council; use emergency triage when needed.
- **Trainer / business:** onboarding + profile management; ABN verification is the trust signal.
- **Emergency resource:** discoverable via emergency lookup; verification workflows apply.
- **Admin / ops:** daily ops queue, verification, moderation, feature gating, telemetry review.

### 1.3 Non-negotiable product invariants
- **Age-first UX:** age/stage selection before issue before location (locked in UI + search logic).
- **ABN verification as canonical trust signal:** ABN status + name-match threshold governs “verified” signalling.
- **Geography constants:** Melbourne councils are **28**; suburbs are **138** (records) and must map to a council.
- **Docs-first governance:** changes must be represented in SSOT + change control, not inferred from code.

---

## 2. Architecture snapshot (what the system is built from)

### 2.1 Major components
- **Web app:** Next.js (App Router) codebase (in sibling repo `DTD`)
- **Database + auth:** Supabase Postgres (RLS, auth-linked profiles)
- **Search:** Supabase RPC search functions (see §3.3)
- **Payments:** Stripe Checkout + webhook handler (feature gated)
- **AI pipelines:** provider wrapper + per-pipeline kill-switch modes (triage, moderation, verification, ops digest)
- **Telemetry:** search + latency + audit tables; preflight verification harnesses

### 2.2 Feature flags / rollout controls (canonical)
- `FEATURE_MONETIZATION_ENABLED` + `NEXT_PUBLIC_FEATURE_MONETIZATION_ENABLED` — gates all Stripe UX and server monetization paths
- `FEATURE_SCRAPER_ENABLED` (and/or `SCRAPER_ENABLED`) — gates scraper automation; **off by default**
- `E2E_TEST_MODE` + `NEXT_PUBLIC_E2E_TEST_MODE` — disables real Stripe/LLM calls in tests

### 2.3 AI kill-switch controls (canonical)
- `AI_GLOBAL_MODE` — default AI mode for all pipelines
- Per-pipeline overrides:
  - `TRIAGE_AI_MODE`
  - `MODERATION_AI_MODE`
  - `VERIFICATION_AI_MODE`
  - `DIGEST_AI_MODE`
See: `DOCS/ai-kill-switch.md` for mode definitions and rollout procedure.

---

## 3. Data model and contracts (what must remain consistent)

### 3.1 Locked enums (Tier 1)
Authoritative list: `DOCS/handoff/SCHEMA_NOTES.md`  
Locked enums include (not exhaustive): `region`, `user_role`, `verification_status`, `age_specialty`, `behavior_issue`, `service_type`, `resource_type`.

### 3.2 Core tables (Tier 1)
Authoritative list: `DOCS/handoff/SCHEMA_NOTES.md`

**Core content & identity**
- `councils` (**28** records)
- `suburbs` (**138** records)
- `profiles` (auth-linked)
- `businesses` (main directory entity)
- `reviews` (user-generated)

**Taxonomy join tables**
- `trainer_specializations`
- `trainer_behavior_issues`
- `trainer_services`

**Trust + verification**
- `abn_verifications`
- `abn_fallback_events`

**Emergency + ops**
- `emergency_resources`
- `emergency_triage_logs`
- `emergency_triage_weekly_metrics`
- `emergency_resource_verification_runs`
- `emergency_resource_verification_events`

**AI + moderation**
- `ai_review_decisions`
- (plus any moderation queue tables defined in Phase reports)

**Monetization + audit**
- `payment_audit`
- `business_subscription_status` (when enabled)
- `featured_placements` (when enabled)

**Telemetry**
- `daily_ops_digests`
- `ops_overrides`
- `latency_metrics`
- `search_telemetry`
- `cron_job_runs`

### 3.3 Key RPC functions (Tier 1)
Authoritative signatures: `DOCS/handoff/SCHEMA_NOTES.md`
- `search_trainers(lat, lon, radius, age, issue, suburb, is_featured)`
- `search_emergency_resources(lat, lon, resource_type, suburb_id)`

---

## 4. Monetization SSOT (test mode canonical)

### 4.1 Policy
- Monetization is **feature-flag gated** (see §2.2).
- Test mode alignment is defined in: `DOCS/STRIPE_TEST_MODE_ALIGNMENT_FINAL.md`.

### 4.2 Canonical test-mode product/price IDs
**Featured Placement (one-time payment)**
- Product: `prod_TcaDngAZ2flHRe`
- Price: `price_1SfL31ClBfLESB1n03QJgzum`
- Amount: **$20.00 AUD** (2000 cents)
- Placement rule: **max 5 per council** (queue system)

**Pro Tier**
- Product: `prod_TaHNvGG53Gd8iS`
- Price: **deferred / reserved** unless explicitly introduced and documented in SSOT change control

**Known non-canonical Stripe artifacts (do not reference in code)**
- `prod_TVSMErBSUKJgCy` — documented as legacy junk test product
- Other legacy IDs may appear in handoff notes; they are superseded by the alignment report above.

---

## 5. Operational readiness (launch gates)

### 5.1 Launch readiness definition (Tier 1)
See: `DOCS/LAUNCH_READY_CHECKLIST.md`

Launch readiness is governed by:
- Verification harness outcomes (`npm run verify:launch` as referenced in ops docs)
- Ops runbooks (`DOCS/operator-runbook.md`, admin cheat sheets)
- Telemetry coverage (`DOCS/OPS_TELEMETRY_ENHANCEMENTS.md`)

### 5.2 Evidence logs
Evidence logs (Tier 3) are located in:
- `DOCS/launch_runs/` (preflight outputs)
- Incident log files (e.g., `DOCS/CI_INCIDENT_20251214.md`); naming convention: CI_INCIDENT_YYYYMMDD.md

---

## 6. Change protocol (how SSOT stays authoritative)

### 6.1 Required steps for any change
1. **Classify the change:** behaviour vs wording vs evidence vs legacy cleanup.
2. **Update SSOT (Tier 0) if behaviour/invariants change.**
3. Update affected Tier 1 docs (schema notes, blueprint, listing logic, env matrix, etc.).
4. Record change in `DOCS/CHANGE_CONTROL_LOG.md` (include date, summary, file list).
5. If files move:
   - Update `DOCS/README.md` (index)
   - Update any internal links
6. If a conflict remains unresolved:
   - Add an entry to a conflict register section in SSOT and mark as **OPEN**.

### 6.2 “Stop conditions” for agents (Copilot, etc.)
Agents must stop and report (no edits) if:
- SSOT does not define a required constant (e.g., council count, product IDs, RPC signature),
- Two Tier 1 docs conflict, or
- A requested change expands scope beyond documented non-goals.

---

## Appendix A — Repository inventory (covers the docs repo)

This inventory lists **all markdown files** in the repo, with an authority tier per §0.  
Use it to decide where a change belongs and whether it requires SSOT arbitration.

| Authority | Path | Title | Notes |
|---|---|---|---|
| Tier 1 (Canonical) | `DOCS/ARCHITECTURE_DUMP.md` | Schema | Canonical SSOT |
| Tier 1 (Canonical) | `DOCS/IMPLEMENTATION_VERIFICATION.md` | Implementation Verification Checklist |  |
| Tier 1 (Canonical) | `DOCS/LISTING_LOGIC_SSOT.md` | 1. System summary | Canonical SSOT |
| Tier 1 (Canonical) | `DOCS/PHASE_1_FINAL_COMPLETION_REPORT.md` | Phase 1 Final Completion Report |  |
| Tier 1 (Canonical) | `DOCS/PHASE_2_FINAL_COMPLETION_REPORT.md` | Phase 2 – Triage + Filtering Engine Final Completion Report |  |
| Tier 1 (Canonical) | `DOCS/PHASE_3_FINAL_COMPLETION_REPORT.md` | Phase 3 – Directory + Trainer Profiles Completion Report |  |
| Tier 1 (Canonical) | `DOCS/PHASE_4_FINAL_COMPLETION_REPORT.md` | Phase 4 – Manual Trainer Onboarding Completion Report |  |
| Tier 1 (Canonical) | `DOCS/PHASE_5_FINAL_COMPLETION_REPORT.md` | Phase 5 – Emergency Ops & Admin Dashboard Completion Report |  |
| Tier 1 (Canonical) | `DOCS/STRIPE_TEST_MODE_ALIGNMENT_FINAL.md` | Stripe Test-Mode Alignment Report: Featured Placement 30-Day One-Time |  |
| Tier 1 (SSOT-badged) | `AGENTS.md` | Agent rules (DTD docs repo) | SSOT badge |
| Tier 1 (SSOT-badged) | `DOCS/DNS_ENV_READY_CHECKS.md` | DNS & Environment Readiness Checks | SSOT badge |
| Tier 1 (SSOT-badged) | `DOCS/FILE_MANIFEST.md` | DOCS File Manifest | SSOT badge |
| Tier 1 (SSOT-badged) | `DOCS/FRONTEND_VERIFICATION_FINDINGS.md` | Frontend Verification Findings - CORRECTED | SSOT badge |
| Tier 1 (SSOT-badged) | `DOCS/IMPLEMENTATION_REALITY_MAP.md` | Implementation Reality Map | SSOT badge |
| Tier 1 (SSOT-badged) | `DOCS/LAUNCH_READY_CHECKLIST.md` | Launch-Ready Checklist | SSOT badge |
| Tier 1 (SSOT-badged) | `DOCS/LINTING_RESTORE_PLAN.md` | Linting Restoration Plan | SSOT badge |
| Tier 1 (SSOT-badged) | `DOCS/MONETIZATION_ROLLOUT_PLAN.md` | Monetization Rollout Plan | SSOT badge |
| Tier 1 (SSOT-badged) | `DOCS/OPS_TELEMETRY_ENHANCEMENTS.md` | Operations Telemetry Enhancements | SSOT badge |
| Tier 1 (SSOT-badged) | `DOCS/PLAN_REVIEW.md` | Plan Review & Approval Status | SSOT badge |
| Tier 1 (SSOT-badged) | `DOCS/PRODUCTION_DNS_PLAN.md` | Production DNS Plan | SSOT badge |
| Tier 1 (SSOT-badged) | `DOCS/PRODUCTION_ENV_MIGRATION.md` | Production Environment Migration Plan | SSOT badge |
| Tier 1 (SSOT-badged) | `DOCS/VERCEL_ENV.md` | Vercel environment variables — Project: dogtrainersdirectory (sanitized) | SSOT badge |
| Tier 1 (SSOT-badged) | `DOCS/automation/ABN-ABR-GUID_automation/ABN-Release-Notes.md` | ABN Verification — Release Notes | SSOT badge |
| Tier 1 (SSOT-badged) | `DOCS/automation/ABN-ABR-GUID_automation/ABN-Rollout-Checklist.md` | ABN Verification Rollout Checklist | SSOT badge |
| Tier 1 (SSOT-badged) | `DOCS/automation/ABN-ABR-GUID_automation/ABR-ABN-Lookup.md` | ABN Lookup Integration Spec — ABR / Web Services | SSOT badge |
| Tier 1 (SSOT-badged) | `DOCS/automation/PHASE_9B_OPERATOR_CHECKLIST.md` | Phase 9B Operator Checklist – Staging Monetization Drill | SSOT badge |
| Tier 1 (SSOT-badged) | `DOCS/automation/PHASE_9B_QUICK_REFERENCE.md` | Phase 9B – Quick Reference Checklist | SSOT badge |
| Tier 1 (SSOT-badged) | `DOCS/automation/automation-checklist.md` | Automation Checklist (aligned to phases) | SSOT badge |
| Tier 1 (SSOT-badged) | `DOCS/blueprint_ssot_v1.1.md` | dogtrainersdirectory.com.au – Complete Conceptual Blueprint (SSOT) | SSOT badge |
| Tier 1 (SSOT-badged) | `DOCS/db/MIGRATIONS_INDEX.md` | Migrations index (supabase/migrations) | SSOT badge |
| Tier 2 (Snapshot) | `DOCS/handoff/ENV_MATRIX.md` | Environment Variables Matrix | Handoff / Snapshot |
| Tier 2 (Snapshot) | `DOCS/handoff/FILE_MANIFEST.md` | HANDOFF File Manifest | Handoff / Snapshot |
| Tier 2 (Snapshot) | `DOCS/handoff/KNOWN_ISSUES.md` | Known Issues & Blockers | Handoff / Snapshot |
| Tier 2 (Snapshot) | `DOCS/handoff/LOCAL_SNAPSHOT.md` | Local Development Environment Snapshot | Handoff / Snapshot |
| Tier 2 (Snapshot) | `DOCS/handoff/PROJECT_STATE.md` | DTD Project State Overview | Handoff / Snapshot |
| Tier 2 (Snapshot) | `DOCS/handoff/SCHEMA_NOTES.md` | Database Schema Overview | Handoff / Snapshot |
| Tier 2 (Snapshot) | `DOCS/handoff/STRIPE_COMPLETE_ASSESSMENT.md` | Comprehensive Stripe Assessment Report (Merged) | Handoff / Snapshot |
| Tier 2 (Snapshot) | `DOCS/handoff/stripe audit.md` | STRIPE PLATFORM: EXPECTED STATE (Per Docs) | Handoff / Snapshot |
| Tier 2 (Supporting spec/runbook) | `DOCS/ai/agent_style_rules.md` | Agent Style Rules — Visual & Tone Constraints | AI / Policy |
| Tier 2 (Supporting spec/runbook) | `DOCS/ai/ai_agent_execution_v2_corrected.md` | dogtrainersdirectory.com.au – AI agent Implementation Strategy v2.1 (CORRECTED) | AI / Policy |
| Tier 2 (Supporting spec/runbook) | `DOCS/ai/ai_automation_assessment.md` | AI Automation Assessment | AI / Policy |
| Tier 2 (Supporting spec/runbook) | `DOCS/automation/ABN-ABR-GUID_automation/ABR-migration-matched_json.md` | Runbook — Apply `matched_json` migration to remote DB (Supabase Dashboard) | Automation / Runbook |
| Tier 2 (Supporting spec/runbook) | `DOCS/automation/ENV_SYNC_GUIDE.md` | Environment Sync & Artifact Packaging | Automation / Runbook |
| Tier 2 (Supporting spec/runbook) | `DOCS/automation/LAUNCH_WORKFLOW_N1.md` | Launch Workflow — n1 (canonical) | Automation / Runbook |
| Tier 2 (Supporting spec/runbook) | `DOCS/automation/OPS_RUNBOOK_ABN_FALLBACK_REVERIFICATION.md` | Operations Runbook: ABN Fallback & Re-Verification | Automation / Runbook |
| Tier 2 (Supporting spec/runbook) | `DOCS/automation/OPS_RUNBOOK_AI_REVIEW_MODERATION.md` | Operations Runbook: AI Review Moderation | Automation / Runbook |
| Tier 2 (Supporting spec/runbook) | `DOCS/automation/OPS_RUNBOOK_EMERGENCY_VERIFICATION.md` | Operations Runbook: Emergency Resource Verification | Automation / Runbook |
| Tier 2 (Supporting spec/runbook) | `DOCS/automation/PHASE_2_COMPLETE_SCRAPER_QA.md` | Phase 2 Complete – Triage + Filtering Engine Validated | Automation / Runbook |
| Tier 2 (Supporting spec/runbook) | `DOCS/automation/PHASE_9B_STAGING_HARDENING_RUNBOOK.md` | Phase 9B – Staging Hardening Runbook (Monetization Drill) | Automation / Runbook |
| Tier 2 (Supporting spec/runbook) | `DOCS/automation/REMOTE_DB_MIGRATIONS.md` | Remote Supabase migrations — safe apply | Automation / Runbook |
| Tier 2 (Supporting spec/runbook) | `DOCS/automation/STRIPE/DELIVERY_SUMMARY.md` | DELIVERY SUMMARY: Phase C Review & Corrections | Automation / Runbook |
| Tier 2 (Supporting spec/runbook) | `DOCS/automation/STRIPE/PHASE_A_STRIPE_AUDIT.md` | PHASE A: Stripe Implementation Audit & Guardrails | Automation / Runbook |
| Tier 2 (Supporting spec/runbook) | `DOCS/automation/STRIPE/PHASE_B_LOCAL_E2E_RUNBOOK.md` | PHASE B: Local E2E Acceptance Runbook | Automation / Runbook |
| Tier 2 (Supporting spec/runbook) | `DOCS/automation/STRIPE/PHASE_C_CORRECTIONS_SUMMARY.md` | PHASE C: Corrections & Safety Fixes Summary | Automation / Runbook |
| Tier 2 (Supporting spec/runbook) | `DOCS/automation/STRIPE/PHASE_C_REVIEW_EXECUTIVE_SUMMARY.md` | Phase C Review & Corrections: Executive Summary | Automation / Runbook |
| Tier 2 (Supporting spec/runbook) | `DOCS/automation/STRIPE/PHASE_C_VERCEL_PREVIEW_DEPLOYMENT.md` | PHASE C: Vercel Preview Deployment & Webhook Integration (CORRECTED) | Automation / Runbook |
| Tier 2 (Supporting spec/runbook) | `DOCS/automation/STRIPE/PHASE_D_CONSOLIDATED_RUNBOOK.md` | PHASE D: Consolidated Stripe Test-Mode Operationalization Runbook (UPDATED) | Automation / Runbook |
| Tier 2 (Supporting spec/runbook) | `DOCS/automation/STRIPE/README_STRIPE_CONSOLIDATION.md` | Stripe Test-Mode Consolidation — Complete Documentation | Automation / Runbook |
| Tier 2 (Supporting spec/runbook) | `DOCS/automation/STRIPE/STRIPE_PHASES_SUMMARY.md` | Stripe Consolidation: Phases A-D Summary | Automation / Runbook |
| Tier 2 (Supporting spec/runbook) | `DOCS/automation/STRIPE/webhook/README_DTD.md` | DTD Webhook Harness (Local) | Automation / Runbook |
| Tier 2 (Supporting spec/runbook) | `DOCS/automation/automation_secrets.md` | Automation secrets and safe usage | Automation / Runbook |
| Tier 2 (Supporting spec/runbook) | `DOCS/implementation/abn_validation_implementation.md` | ABN Validation Implementation Guide | Implementation spec |
| Tier 2 (Supporting spec/runbook) | `DOCS/implementation/ai_automation_assessment.md` | ai_automation_assessment.md | Implementation spec |
| Tier 2 (Supporting spec/runbook) | `DOCS/implementation/master_plan.md` | Master Implementation Plan | Implementation spec |
| Tier 2 (Supporting) | `DOCS/Admin Panel Cheat Sheet.md` | DTD Operator Cheat Sheet (Non-Technical) |  |
| Tier 2 (Supporting) | `DOCS/CHANGE_CONTROL_LOG.md` | Change Control Log |  |
| Tier 2 (Supporting) | `DOCS/DATA_VALIDATION_IMPLEMENTATION.md` | Week 2: Data Quality & Validation Implementation |  |
| Tier 2 (Supporting) | `DOCS/DEPRECATION_STAGING.md` | Deprecation Staging |  |
| Tier 2 (Supporting) | `DOCS/HANDOFF_COMPLETE_STRIPE_ALIGNMENT.md` | STRIPE TEST-MODE ALIGNMENT: COMPLETE HANDOFF PACKAGE |  |
| Tier 2 (Supporting) | `DOCS/IMPLEMENTATION_PLAN_UPDATED.md` | DTD Implementation Plan - UPDATED |  |
| Tier 2 (Supporting) | `DOCS/LLM_Z_AI_IMPLEMENTATION.md` | LLM Implementation with Z.ai | AI / Policy |
| Tier 2 (Supporting) | `DOCS/PHASE_2_STATUS_AND_WEEK_3_PREP.md` | Completion Report: Phases 1-2 (Weeks 1-2) |  |
| Tier 2 (Supporting) | `DOCS/PRIORITY_3_ERROR_LOGGING_SPEC.md` | Priority 3: Error Logging & Monitoring — Technical Spec |  |
| Tier 2 (Supporting) | `DOCS/QUICK_REFERENCE_6_WEEK_PLAN.md` | Quick Reference: 6-Week Implementation Plan |  |
| Tier 2 (Supporting) | `DOCS/README.md` | Documentation index | Index |
| Tier 2 (Supporting) | `DOCS/SUPABASE-QUICKSTART.md` | Recommended (default) developer workflow — remote Supabase dev/staging (no Docker required) | Database / Schema |
| Tier 2 (Supporting) | `DOCS/WEEK_3_COMPLETION_REPORT.md` | Week 3: Error Logging & Monitoring - Final Completion Report |  |
| Tier 2 (Supporting) | `DOCS/WEEK_3_ONWARDS_ROADMAP.md` | Implementation Roadmap: Week 3 & Beyond |  |
| Tier 2 (Supporting) | `DOCS/ai-kill-switch.md` | AI Kill-Switch Guide | AI / Policy |
| Tier 2 (Supporting) | `DOCS/hand_over.md` | Hand‑Over Document – Maximum AI Automation (Dog Trainers Directory) |  |
| Tier 2 (Supporting) | `DOCS/operator-runbook.md` | Dog Trainers Directory - Operator Runbook |  |
| Tier 2 (Supporting) | `DOCS/user-workflow-design.md` | User Workflow Design Guide – Dog Trainers Directory |  |
| Tier 2 (Supporting) | `IMPLEMENTATION_SUMMARY.md` | Week 1 Implementation Summary |  |
| Tier 2 (Supporting) | `PHASE5_MERGE_PLAN.md` | Phase 5 Integration - Incremental Merge Plan |  |
| Tier 2 (Supporting) | `README.md` | DTD Documentation Repository (Private) | Index |
| Tier 2 (Supporting) | `WEEK4_ERROR_FIXES.md` | Week 4 Implementation - Error Checking & Fixes Report |  |
| Tier 2 (Supporting) | `WEEK4_IMPLEMENTATION_SUMMARY.md` | Week 4 AI Automation Implementation - Summary & Verification |  |
| Tier 2 (Supporting) | `assessment/baseline-report.md` | Baseline Assessment |  |
| Tier 2 (Supporting) | `assessment/updated-assessment-2025-12-15.md` | Updated Assessment & Prioritised Remediation (2025-12-15) |  |
| Tier 2 (Supporting) | `supabase/LOCAL_SETUP.md` | Supabase setup — remote-first developer Quickstart (no Docker required) |  |
| Tier 3 (Evidence log) | `DOCS/CI_INCIDENT_20251214.md` | CI Release Blocker – Incident Report | Run log / Evidence |
| Tier 3 (Evidence log) | `DOCS/launch_runs/README.md` | Launch Run Artifact Policy | Run log / Evidence |
| Tier 3 (Evidence log) | `DOCS/launch_runs/launch-prod-20251212-ai-preflight.md` | Launch Run – production – AI preflight | Run log / Evidence |
| Tier 3 (Evidence log) | `DOCS/launch_runs/launch-prod-20251213-ai-preflight.md` | Launch Run – production – 20251213 | Run log / Evidence |
| Tier 3 (Evidence log) | `DOCS/launch_runs/launch-prod-20251214-ai-preflight.md` | Launch Run – production – 20251214 | Run log / Evidence |
| Tier 3 (Evidence log) | `DOCS/launch_runs/launch-production-20251211-preflight.md` | Launch Run – production – 2025-12-11 Preflight | Run log / Evidence |
| Tier 3 (Evidence log) | `DOCS/launch_runs/launch-staging-20251209-ops-dry-run.md` | Launch Run – staging – 2025-12-09 18:00 AEDT | Run log / Evidence |
| Tier 3 (Evidence log) | `DOCS/launch_runs/launch-staging-20251211-monetization-preflight.md` | Launch Run – staging – 2025-12-11 Monetization Preflight | Run log / Evidence |
| Tier 4 (Draft) | `DOCS/_drafts/FILE_MANIFEST.md` | DOCS File Manifest (Draft) | Draft |
| Tier 4 (Draft) | `DOCS/_drafts/PHASES_SUMMARY.md` | Phases Summary (Draft) | Draft |
| Tier 4 (Draft) | `DOCS/_drafts/REVIEW_CHECKLIST.md` | Local Review Checklist (Draft) | Draft |
| Tier 4 (Legacy) | `DOCS/_legacy/README.md` | DOCS/_legacy — archived/superseded documentation | Legacy |
| Tier 4 (Legacy) | `DOCS/_legacy_conflicts/FILE_MANIFEST.md` | File System Manifest - dogtrainersdirectory.com.au Project | Legacy |
| Tier 4 (Legacy) | `DOCS/_legacy_conflicts/FRONTEND_IMPLEMENTATION_GAP_ANALYSIS.md` | Frontend Implementation Gap Analysis | Legacy |
| Tier 4 (Legacy) | `DOCS/_legacy_conflicts/README_STRIPE_WEBHOOK.md` | Stripe webhook test harness | Legacy |
| Tier 4 (Legacy) | `DOCS/_legacy_conflicts/webhook_README_DTD.md` | DTD Webhook Harness (Local) | Legacy |