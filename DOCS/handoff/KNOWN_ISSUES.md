# Known Issues & Blockers

**Last Updated:** 2025-12-17  
**Source:** DOCS, commit history, test results, manual investigation

---

## Category 1: Confirmed Bugs

### Issue #1: ABR API Fallback When GUID Invalid or Network Unavailable
**Status:** Known, expected behavior  
**Severity:** Medium (graceful degradation)

**Description:**
- When `ABR_GUID` is invalid, missing, or ABR API is unreachable, ABN verification falls back to recording a `abn_fallback_event`
- Business is marked `abn_verified=false` and verification status becomes `unverified`
- User can still register but without verified badge

**Expected Behavior:**
- Warning logged to admin dashboard via `/api/admin/abn/fallback-stats`
- Fallback rate tracked; alert fired if > threshold (configurable)

**Actual Behavior:**
- Fallback events silently logged; ops must check admin dashboard to discover failures
- No automatic email/Slack alert (ops would need to configure `ALERTS_*` env vars + cron job)

**Repro Steps:**
1. Set `ABR_GUID=invalid-guid` in `.env.local`
2. Attempt ABN verification via onboarding form or `/api/abn/verify` endpoint
3. Check `abn_fallback_events` table or admin dashboard

**Suggested Next Steps:**
- Implement automatic alert if fallback rate exceeds threshold (documented in OPS_TELEMETRY_ENHANCEMENTS.md)
- Add Slack/email integration for critical ABN failures
- Test with real ABR_GUID before production

**Related Files:**
- `src/lib/abr.ts` (parser + fallback logic)
- `src/lib/abnFallback.ts` (event logging)
- `/api/admin/abn/fallback-stats` (dashboard)

---

### Issue #2: LLM Provider Unavailable Doesn't Block Critical Flows
**Status:** Known, intended (graceful degradation)  
**Severity:** Low (non-critical feature)

**Description:**
- Emergency triage, daily ops digest, and review moderation all support LLM providers (Z.AI, OpenAI)
- If LLM provider is unavailable or credentials missing, system falls back to deterministic rules
- This is intentional but can mask configuration errors

**Expected Behavior:**
- All flows degrade gracefully; ops alerted if LLM unavailable

**Actual Behavior:**
- Deterministic fallback is silent; admin dashboard may show "LLM unavailable" but no escalation
- Ops must manually check telemetry dashboards to discover LLM issues

**Repro Steps:**
1. Set `LLM_PROVIDER=zai` but leave `ZAI_API_KEY` empty
2. Trigger emergency triage (should classify without LLM)
3. Check `/api/admin/ai-health` dashboard for LLM status

**Suggested Next Steps:**
- Add telemetry logging when LLM fallback occurs
- Implement alert threshold: if fallback rate exceeds N% for > M minutes, escalate
- Document in runbooks that LLM unavailability is expected during testing

**Related Files:**
- `src/lib/llm.ts` (provider wrapper + fallback)
- `src/lib/medicalDetector.ts` (deterministic fallback rules)
- `/api/admin/ai-health` (health dashboard)

---

## Category 2: Edge Cases & Limitations

### Issue #3: Age-First Flow Cannot Be Skipped
**Status:** By design (product invariant)  
**Severity:** Informational

**Description:**
- All search flows enforce age selection BEFORE suburb/issue selection
- UI does not allow proceeding without age; API (`/api/triage`) requires age parameter

**Why:** Training methodologies differ significantly by age/stage (puppies vs seniors); product rule is non-negotiable.

**Limitations:**
- Users cannot search for "any age" trainers (must pick specific age)
- Trainers cannot indicate they work with "all ages" (must select specific ages)

**Workaround:**
- If trainer works with all ages, they select all 5 age categories manually
- If user unsure about age, help text suggests "Adult dogs (18m-7y)" as default for unknown dogs

**Related Files:**
- `src/app/page.tsx` (enforced in UI)
- `src/app/search/` (enforced in search flow)
- `src/types/database.ts` (AgeSpecialty enum validation)

---

### Issue #4: Suburb-Council Mapping Not Dynamic
**Status:** By design (immutable geographic data)  
**Severity:** Informational

**Description:**
- Suburb-to-council mapping is locked in `DOCS/suburbs_councils_mapping.csv` (138 suburb rows, 28 councils)
- Changes require RFC + SSOT update + CI drift check + manual merge
- No dynamic discovery; mapping is seeded into `Suburbs` table at launch

**Why:** Geographic boundaries change infrequently; manual curation ensures accuracy and prevents data drift.

**Limitations:**
- Cannot add new suburbs without RFC
- Cannot remove suburbs (only soft-delete/archive)
- Postcode/lat/lon must be manually verified

**Workaround:**
- File RFC in DOCS/CHANGE_CONTROL_LOG.md
- Update CSV + FILE_MANIFEST.md
- Re-generate allowlists if ABN batches affected
- Merge + deploy

**Related Files:**
- `DOCS/suburbs_councils_mapping.csv` (source of truth)
- `supabase/migrations/` (seeding migration)
- `DOCS/FILE_MANIFEST.md` (change tracking)

---

### Issue #5: Monetization Stripe Checkout Not Testable Without Real Keys
**Status:** By design (feature-flagged)  
**Severity:** Informational

**Description:**
- Stripe checkout (`/api/stripe/create-checkout-session`) requires real `STRIPE_SECRET_KEY` + `STRIPE_PUBLISHABLE_KEY`
- E2E tests use `E2E_TEST_MODE=true` to stub Stripe calls (fake session ID returned)
- Cannot test real Stripe flow locally without live test keys

**Why:** Stripe requires signature verification; mock responses would be detected as invalid.

**Workaround:**
- Use Stripe test keys in staging environment
- Use `E2E_TEST_MODE=true` in local tests to skip real Stripe calls
- Test real flow only in staging with test Stripe account

**Related Files:**
- `/api/stripe/create-checkout-session` (checkout handler)
- `tests/e2e/monetization.spec.ts` (E2E coverage with stubs)
- `DOCS/MONETIZATION_ROLLOUT_PLAN.md` (Stripe contract)

---

## Category 3: Manual Verification Steps

### Issue #6: Emergency Resource Verification Requires Manual Review
**Status:** Known (quarterly task)  
**Severity:** Medium (data quality)

**Description:**
- Emergency vets, shelters, and crisis trainers must be re-verified quarterly
- Current `/api/emergency/verify` cron job attempts HTTP HEAD requests to contact numbers (limited validation)
- Many resources require manual phone calls or web checks to confirm current status

**Expected Behavior:**
- Automated verification sufficient; resources marked verified/expired automatically

**Actual Behavior:**
- Automated checks catch ~70% of issues (broken websites, dead phone numbers)
- Remaining 30% require ops team to manually call and verify (no SLA)

**Suggested Next Steps:**
- Implement twilio-based verification calls (optional enhancement)
- Document quarterly manual review checklist in ops runbook
- Track verification recency in admin dashboard (highlight >90 days old)

**Related Files:**
- `/api/emergency/verify` (auto-verification endpoint)
- `src/app/admin/` (verification queue UI)
- `DOCS/operator-runbook.md` (manual procedures)

---

### Issue #7: ABN Name Match Score Threshold (~85%) May Be Soft
**Status:** Known (heuristic)  
**Severity:** Low (acceptable trade-off)

**Description:**
- ABN name matching uses ~85% threshold (calculated via name similarity heuristic)
- Threshold is not strict; dependent on string comparison implementation
- Some edge cases (accents, abbreviations, legal vs trading names) may not meet threshold

**Expected Behavior:**
- All legitimate ABN registrations pass ≥85% match

**Actual Behavior:**
- Some legitimate businesses fail match (e.g., "ABC Dog Training Inc." vs "ABC Dog Training Pty Ltd")
- Flagged for manual review; ops approves/rejects

**Suggested Next Steps:**
- Audit recent rejections to calibrate threshold
- Consider fuzzy matching library (e.g., Levenshtein distance)
- Document specific patterns that fail (accents, suffixes)

**Related Files:**
- `src/lib/abr.ts` (name matching logic)
- `/api/abn/verify` (API endpoint)
- DOCS/automation/ABN-ABR-GUID_automation/ABR-ABN-Lookup.md (contract)

---

## Category 4: Data Gaps

### Issue #8: Web Scraper Automation Deferred
**Status:** Not yet implemented (behind feature flag)  
**Severity:** Medium (scope limitation)

**Description:**
- Initial trainer dataset is manually sourced or invited directly
- Web scraper for council/local business websites exists as a framework but is disabled
- Scraper coverage: council business registries, local dog training websites (estimated 50-100 trainers)

**Why Deferred:**
- Requires ≥95% accuracy sign-off before enabling
- Current accuracy: ~75% (manual review rate too high)
- Legal review needed (website terms of service)

**Expected When Ready:**
- `FEATURE_SCRAPER_ENABLED=1` env var
- `npm run scraper:run:staging` command available
- Dry-run output reviewed by ops before applying

**Related Files:**
- `scripts/scraper_*.py` (if present; may be in separate branch)
- `DOCS/implementation/master_plan.md` (scraper roadmap)
- `FEATURE_SCRAPER_ENABLED` env var guard

---

### Issue #9: Incomplete Emergency Resource Dataset
**Status:** Partial (50+ resources seeded, completeness TBD)  
**Severity:** Low (acceptable for MVP)

**Description:**
- Emergency resources (vets, shelters, crisis trainers) seeded with initial 50+ records
- Coverage: Inner City + major suburbs only
- Remaining ~88 suburbs have limited or no emergency resources

**Expected When Complete:**
- All 138 suburbs should have ≥1 emergency resource
- Quarterly verification complete for all resources

**Current Gap:**
- ~40% of suburbs underrepresented
- Manual curation required; scraper not ready

**Suggested Next Steps:**
- Publish manual review checklist for ops to complete data
- Add feedback mechanism for dog owners to report missing resources
- Track resource count per suburb in admin dashboard

**Related Files:**
- `supabase/migrations/20250207150000_phase5_emergency_seed.sql` (seeding)
- `/api/admin/` (resource dashboard)
- `DOCS/emergency-resources-spreadsheet.csv` (if present)

---

## Category 5: CI/CD & Testing

### Issue #10: Playwright E2E Tests May Timeout Locally
**Status:** Intermittent (environment-dependent)  
**Severity:** Low (CI usually passes)

**Description:**
- `npm run e2e` sometimes times out on slower machines or when app server slow to start
- Default timeout: 30 seconds per test
- Symptoms: "Timeout waiting for action 'click'"

**Workaround:**
```bash
# Increase timeout
npm run e2e -- --timeout 60000

# Or start dev server separately before E2E
npm run dev &
sleep 3  # wait for server to start
npm run e2e
```

**Related Files:**
- `playwright.config.ts` (timeout configuration)
- `tests/e2e/*.spec.ts` (test files)

---

## Resolved Issues (Historical Reference)

### ✅ Issue: Phase 2 "Radius Only" Regression (RESOLVED)
- **Was:** Search could revert to radius-only filtering (losing age/issue/suburb context)
- **Now:** Locked in SSOT, enforced in UI + RPC, verified in PHASE_2_FINAL_COMPLETION_REPORT.md
- **Evidence:** `npm run smoke` includes regression test

### ✅ Issue: TypeScript Compilation Errors (RESOLVED)
- **Was:** Emergency APIs, admin dashboards, error logging had untyped contexts
- **Now:** All ErrorContext + payload types aligned with SSOT, `npm run type-check` passes
- **Date Fixed:** 2025-12-09

### ✅ Issue: Missing ABN Fallback Metrics (RESOLVED)
- **Was:** No visibility into ABN verification failures
- **Now:** `abn_fallback_events` table + `/api/admin/abn/fallback-stats` dashboard
- **Date Fixed:** Phase 5 completion

---

## Escalation Procedure

**If you encounter an issue not listed above:**

1. Check `DOCS/IMPLEMENTATION_REALITY_MAP.md` for flow status (CONFIRMED-WORKING vs UNKNOWN)
2. Verify against `DOCS/blueprint_ssot_v1.1.md` (spec vs implementation)
3. Run `npm run verify:launch` to check health gates
4. Check `/api/admin/health?extended=1` dashboard for telemetry status
5. If blocking: file issue in GitHub with reproduction steps + evidence
6. Ops escalation: email `ALERTS_EMAIL_TO` or post to `ALERTS_SLACK_WEBHOOK_URL`

