# DTD Operator Cheat Sheet (Non-Technical)

**Goal:** Run the entire website day-to-day from the Admin Panel without needing a developer.

---

## The 60-Second Rule

If you’re unsure what to do, always go here first:

1. **Dashboard Check** → review **Status Strip** for red lights
2. **Queues** → clear **Emergency**, then **ABN**, then **Reviews**
3. **Cron Health** → confirm the system is running
4. **Errors** → confirm nothing is spiking

---

## Daily Routine

### Start of Day (5–10 minutes)

1. Open **Admin → Dashboard**

   * Check the **Status Strip** for any red items.
   * If red, investigate before moving on.

2. Open **Admin → Queues**
   Work top-to-bottom in this order:

   1. **Emergency Verification** (must be zero if possible)
   2. **ABN Manual Review**
   3. **Pending Reviews**
   4. **Flagged Profiles**
   5. **Scaffolded Listings**

3. Open **Admin → Cron Health**

   * Confirm no job is **Late** or **Failed**.

4. Open **Admin → Errors**

   * Confirm no big spike in the last 24 hours.

✅ Done when: Status Strip shows no red items, queues aren’t building up, Cron Health is OK.

---

### Midday Check (2–5 minutes)

1. **Status Strip**: check for new red items
2. **Queues**: check **oldest item age**
3. **Cron Health**: confirm jobs still OK

---

### End of Day (5 minutes)

1. **Status Strip**: no red items
2. **Cron Health**: OK (no Late/Failed)
3. **Errors**: stable (no rising trend)
4. **Queues**: Emergency + ABN not piling up

---

## How to Work the Status Strip (Alerts)

### What each alert means

Each alert is a “task”. It tells you:

* What’s wrong
* Why it matters
* The fastest safe action to take

### Buttons you’ll see (and what they do)

* **Open Queue** → takes you to the exact workbench to clear it
* **Run Now** → safely runs a job immediately (only use once)
* **Retry** → reattempts a failed automation task
* **View Details** → shows the evidence (timestamps, reasons)
* **Acknowledge** → marks “I saw this”
* **Snooze** → hides it temporarily (comes back if still broken)
* **Escalate** → creates a clear incident note for a developer

### Basic rule

* If it’s **Critical**: act now.
* If it’s **High**: act today.
* If it’s **Medium**: act when you can, but don’t ignore recurring ones.
* If it’s **Low**: monitor.

---

## Queue Workbenches (What you do and how)

### 1) Emergency Verification Queue (highest priority)

**What it is:** Emergency resources waiting for human confirmation.
**Your goal:** Keep it near zero.

Actions:

* **Approve** if the resource is real and correct
* **Reject** if incorrect/unverifiable
* **Needs follow-up** if missing info (if available)

Finish condition:

* Pending count is reduced and oldest item is not growing.

---

### 2) ABN Manual Review Queue

**What it is:** ABN checks that couldn’t be auto-verified.

Actions:

* **Approve ABN** if it matches and is active
* **Reject ABN** if invalid/mismatch
* **Request correction** if name/spelling mismatch (if supported)

Finish condition:

* ABN Manual queue stops growing; oldest item age decreases.

Escalate if:

* Many failures suddenly appear at once (possible service outage).

---

### 3) Pending Reviews Queue

**What it is:** Reviews waiting for approval.

Actions:

* **Approve** if valid
* **Reject** if spam/abusive/irrelevant
* **Escalate** if uncertain or legally sensitive

Finish condition:

* Pending reviews count decreases.

---

### 4) Flagged Profiles Queue

**What it is:** Listings flagged by rules or AI.

Actions:

* **Clear flag** if safe/accurate
* **Keep flagged** if needs correction
* **Disable listing** only if serious

Finish condition:

* No backlog; repeat offenders identified.

---

### 5) Scaffolded Listings Queue

**What it is:** Auto-generated/scraped listings waiting confirmation.

Actions:

* **Approve & publish**
* **Reject**
* **Edit basics** (if available)

Finish condition:

* Items are processed; low-quality items are rejected early.

---

## Cron Health (Automations)

### What “Late” means

A job hasn’t run when expected.

Your actions:

1. Click **View Details**
2. Click **Run Now** (once)
3. If it fails again → **Open Playbook** → follow steps
4. If still failing → **Escalate**

### What “Failed” means

The last run ended in error.

Your actions:

1. Open the job
2. Click **Retry** (if offered)
3. Check if related queues are growing
4. Escalate if it repeats

---

## Errors Dashboard (Incidents)

### What to look for

* A sharp rise in errors (spike)
* Same error repeating often
* Errors tied to core flows: triage, search, onboarding, promote

Your actions:

* If minor: Acknowledge and monitor
* If spike: Escalate with summary

### Escalation message template (copy/paste)

* What page is affected:
* What time it started:
* Error type/title:
* How many occurrences:
* What you tried (run now / retry):
* Current impact (users blocked?):

---

## DLQ / Retries

### When to use Retry

Use **Retry** if:

* A job failed due to a temporary issue
* The playbook recommends retry

Don’t spam Retry:

* Retry once, then verify.
* If it fails twice → escalate.

---

## Golden Rules (So you don’t break things)

1. Don’t click “Run Now” repeatedly.
2. Use **Acknowledge** when you’ve seen an issue.
3. Use **Snooze** only if you understand why it’s safe to wait.
4. Always clear **Emergency** and **ABN** queues first.
5. If unsure, escalate early with a good summary.

---

## Quick “What do I do if…” answers

### “Status Strip shows CRITICAL: Cron job failed”

* Open it → Run Now → Verify → If fails again, escalate.

### “ABN queue keeps growing”

* Work the ABN queue first.
* If many new failures at once, escalate (possible outage).

### “Emergency verification items appear”

* Clear them immediately (approve/reject).
* Don’t leave these overnight.

### “No search results / users complain”

* Check Errors first (spike?)
* Check Cron Health (is search/telemetry running?)
* Escalate with details (what page, when, what happened)

---

**End of cheat sheet**
