# Incept AP Coverage Dashboard

A static, public dashboard that answers one question:

> **Where in AP is the closed learning loop actually closed?**

The loop runs in five stages: **Diagnose → Decide → Act → Re-test → Score.** Four are system-side automation that vary by subject; the fifth (**Re-test**, meaning *assigning a new test to the student*) is currently open for every subject — nothing schedules, notifies, or assigns automatically. That's the loop's single universal product gap.

This dashboard maps each stage's status per AP subject, so the team and leadership can see at a glance which subjects work end-to-end and which have open seams.

## Live URL

The dashboard is deployed via GitHub Pages: <https://incepttrilogy.github.io/incept-coverage-dashboard/>

## What's inside

- **`index.html`** — single-page dashboard. No build step, no framework, no runtime dependencies. Reads `data.json` and renders three views (coverage matrix, per-subject loop status, contradictions).
- **`data.json`** — the matrix data. Hand-edited weekly. This is the file you change when status changes.
- **`styles.css`** — the look and feel.
- **`WEEKLY_REVIEW.md`** — checklist for the weekly refresh.

## How the data is sourced

The dashboard's status values come from three repos:

| Field | Source |
|---|---|
| Enrolled subjects + student counts | `timeback-loops/data/may_ap_exam_course_summary.csv` |
| Hole-filler subject registration | `incept-hole-filler/src/hole_filler/subjects.py` |
| FRQ grader status | `incept-test-builder/engine/grader_api/app.py` + `engine/regrade/subject_registry.py` |
| CED coverage | `timeback-loops/data/ced_ap_*.json` |
| Calibration status | `incept-test-builder/engine/grader_api/calibration/` |

The dashboard does **not** call any API at runtime. It's static. To refresh, edit `data.json` and bump `as_of`.

## Status legend

- **Closed (green ✓)** — the stage runs end-to-end with no human in the loop.
- **Partial (amber ◐)** — wired up but with a known caveat (e.g., grader uses a generic fallback prompt instead of subject-specific scoring).
- **Open (red ✗)** — not wired up or known broken; requires manual intervention or has no path.

## How often this is updated

**Weekly review** — the team owner runs through `WEEKLY_REVIEW.md` and updates `data.json`. The dashboard self-flags as stale if the `as_of` date is older than 7 days.

## Why a static dashboard

Three reasons:
1. The status changes weekly, not hourly. A live data pipeline would be over-engineered.
2. A single committed JSON file is auditable — every status change has a git history.
3. Andy (the audience) just needs a stable URL he can bookmark.

If/when status changes more often than weekly, this can graduate into a live dashboard backed by Turso + the existing dashboard's API.

## Contributing / corrections

Open an issue or PR. The data schema is documented inline in `data.json` and via the `data_sources` block.

## License

Internal Incept tooling. Public repository for transparency, not for external reuse.
