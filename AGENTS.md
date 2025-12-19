# Agent rules (DTD docs repo)

This repository is the canonical documentation/runbooks/SSOT store.

## Where things go
- Put longform docs and runbooks under `DOCS/` (use existing subfolders where possible).
- App code changes belong in the sibling repo: `../DTD/` (GitHub: `dogtrainersdirectory`).

## Doc governance
- Canonical SSOT docs must include the SSOT badge string: `SSOT – Canonical Source of Truth`.
- Archived/conflicting docs must include: `ARCHIVED / CONFLICTING DOCUMENT`.
- For neutral indices/changelogs, use `<!-- DOCS_DIVERGENCE_IGNORE: supporting index or changelog -->`.
- When operator-facing UI task signals change (Status Strip, Task Summary, Queues), update `DOCS/Admin Panel Cheat Sheet.md` and `DOCS/operator-runbook.md`.
