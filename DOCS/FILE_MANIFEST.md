> **SSOT – Canonical Source of Truth**
> Scope: Documentation manifest for dogtrainersdirectory.com.au
> Status: Active · Last reviewed: 2025-12-20

# DOCS File Manifest

Purpose: authoritative inventory of SSOT docs and operational references. Use this as the index for conflict analysis and post-launch verification.

## Canonical SSOT docs (owner · purpose)
- `DOCS/ARCHITECTURE_DUMP.md` — Engineering · Ground-truth schema + file tree snapshot.
- `DOCS/Admin Panel Cheat Sheet.md` — Ops · Daily operator flow and queue priorities.
- `DOCS/operator-runbook.md` — Ops · Operational procedures and escalation playbooks.
- `DOCS/ai-kill-switch.md` — Engineering/Ops · AI modes, kill switch, decision source logging.
- `DOCS/LLM_Z_AI_IMPLEMENTATION.md` — Engineering · LLM wrapper behavior and fallback policy.
- `DOCS/OPS_TELEMETRY_ENHANCEMENTS.md` — Engineering/Ops · Telemetry coverage, alerting, and gaps.
- `DOCS/PRIORITY_3_ERROR_LOGGING_SPEC.md` — Engineering · Error logging schema and APIs.
- `DOCS/SUPABASE-QUICKSTART.md` — Engineering · Remote-first Supabase workflow and migration policy.
- `DOCS/MONETIZATION_ROLLOUT_PLAN.md` — Product/Finance · Featured placement rollout + gates.
- `DOCS/HANDOFF_COMPLETE_STRIPE_ALIGNMENT.md` — Engineering · Stripe test-mode alignment evidence.
- `DOCS/LISTING_LOGIC_SSOT.md` — Engineering/Product · Listing data model + verified mismatches.
- `DOCS/FRONTEND_VERIFICATION_FINDINGS.md` — Frontend · UI verification results and fixes.
- `DOCS/IMPLEMENTATION_REALITY_MAP.md` — Engineering · Verified implementation status map.
- `DOCS/user-workflow-design.md` — Product/Design · Intended user journeys and UX constraints.
- `DOCS/VERCEL_ENV.md` — Ops · Environment variable matrix.
- `DOCS/PRODUCTION_ENV_MIGRATION.md` — Ops · Env migration steps and checks.
- `DOCS/LAUNCH_READY_CHECKLIST.md` — Ops/Product · Go/no-go launch checklist.
- `DOCS/CHANGE_CONTROL_LOG.md` — Ops · Change log and governance trail.

## Product SSOT assets (owner · purpose)
- `DOCS/blueprint_ssot_v1.1.md` — Product · Core product SSOT and taxonomies.
- `DOCS/suburbs_councils_mapping.csv` — Data · Authoritative suburb-to-council mapping.

## Supporting references (owner · purpose)
- `DOCS/STRIPE_TEST_MODE_ALIGNMENT_FINAL.md` — Engineering · Stripe alignment deep reference.
- `DOCS/DATA_VALIDATION_IMPLEMENTATION.md` — Engineering/Data · Validation checks and reports.
- `DOCS/automation/` — Ops/Engineering · Automation runbooks and scripts.
- `DOCS/README.md` — Engineering · Docs repo entry point.

## Drafts and legacy references
- Drafts live under `DOCS/_drafts/` and are non-canonical until promoted.
- Historical manifests:
  - `DOCS/_drafts/FILE_MANIFEST.md` (draft)
  - `DOCS/_legacy_conflicts/FILE_MANIFEST.md` (legacy snapshot)

## Promotion rules
- Update this manifest first, then update the canonical doc.
- Record any SSOT change in `DOCS/CHANGE_CONTROL_LOG.md`.
