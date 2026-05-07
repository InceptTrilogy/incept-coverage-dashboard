# Weekly Review Checklist

Run this every Monday (or whenever scope changes). The whole review takes ~20 minutes if nothing has changed, longer if you have to chase down a status flip.

## 1. Refresh the source-of-truth files

Pull latest on the three source repos:

```bash
cd /Users/ilmych/conductor/repos/incept-hole-filler && git pull
cd /Users/ilmych/conductor/repos/incept-test-builder && git pull
cd /Users/ilmych/conductor/repos/timeback-loops && git pull
```

## 2. Re-check each row of `data.json`

For every subject in `data.json`, confirm the four columns:

### Hole-filler registered

```bash
grep -o "ap-[a-z-]*" /Users/ilmych/conductor/repos/incept-hole-filler/src/hole_filler/subjects.py | sort -u
```

If a subject's slug appears in `subjects.py`, set `hole_filler.registered: true`. Otherwise `false` and add a note.

### FRQ grader status

```bash
# Full graders (have a dedicated subject directory):
ls /Users/ilmych/conductor/repos/incept-test-builder/engine/grader_api/graders/

# Subject registry (full + fallback):
grep -E "register|GRADER" /Users/ilmych/conductor/repos/incept-test-builder/engine/regrade/subject_registry.py
```

Map each subject to one of:
- `full` — has a dedicated grader directory under `engine/grader_api/graders/`
- `fallback` — registered in `subject_registry.py` but no dedicated grader (uses BaseGrader)
- `missing` — not in `subject_registry.py` at all (KeyError on call)

### Calibration

Check `engine/grader_api/calibration/cb_data/` for fixtures and any per-subject calibration runs. Note: a calibration run can exist outside this directory if the team validated against a different gold-standard set — when in doubt, ask whoever last ran calibration.

### Loop stages per subject

Per stage, the rule of thumb:

| Stage | Closed when | Partial when | Open when |
|---|---|---|---|
| Diagnose | CED file exists AND tagger coverage ≥ 80% | CED exists, tagger coverage 50–80% | No CED file, or tagger coverage < 50% |
| Decide | `nextTest.js` returns a recommendation given current signal | Recommendation works but tests are missing for some buckets | Subject has no practice tests in TimeBack inventory |
| Act | Subject is in `subjects.py` AND a recent course was generated | Subject is in `subjects.py` but no recent generation observed | Subject is not in `subjects.py` |
| Assign | An automated assignment / notification / schedule pushes the student to take the next recommended test | Recommendation surfaces in a place the student/guide reliably sees but no auto-action | Recommendation is shown only on the dashboard (today's default — `open` for every subject) |
| Re-test | Dedicated FRQ grader AND calibrated AND result writeback into TimeBack confirmed | Grader exists but calibration weak or writeback unverified | No grader path (KeyError) |

Set `overall` to the worst of the **four system stages** (Diagnose, Decide, Act, Re-test). The Assign stage is intentionally excluded from `overall` because it's a system-wide gap (uniformly open today); rolling it into overall would collapse the variation between subjects. Surface Assign separately in the contradictions block until the gap is closed.

## 3. Review the contradictions list

For each entry in `contradictions`:
- Has the underlying gap been fixed? If yes, remove it.
- Has student impact changed (enrollment moved, exam date passed)? Update `students_affected`.
- Is severity still right? (`high` blocks an exam this week, `medium` is a real gap, `low` is FYI.)

Look for **new** contradictions:
- Any subject where hole-filler says ✓ but FRQ grader says ✗ (or vice versa).
- Any subject in the roster but missing from CED.
- Any subject in `subjects.py` but not in the May roster (orphan registration — usually fine, just note it).

## 4. Bump `as_of`

Edit `data.json`:

```json
"as_of": "YYYY-MM-DD"
```

Set this to today. The dashboard surfaces a "stale" banner if the date is older than `stale_after_days` (default 7).

## 5. Spot-check the rendered dashboard

```bash
cd /Users/ilmych/conductor/repos/incept-coverage-dashboard
python3 -m http.server 8000
# Open http://localhost:8000
```

Confirm:
- All 17 subjects render in alphabetical order.
- Coverage matrix counts match `data.json`.
- Per-stage table is color-correct.
- Contradictions section reflects current state.

## 6. Commit + push

```bash
git add data.json
git commit -m "weekly review: $(date +%Y-%m-%d)"
git push
```

GitHub Pages picks up the change automatically. The deploy usually completes within 60 seconds.

## 7. Notify the team (optional)

If the headline number changed (e.g., "11 closed → 13 closed" or a new contradiction landed), drop a one-line note in #ap-loop with the dashboard URL.

## Edge cases

**A subject moved from `partial` → `closed`.** Double-check that the underlying calibration / fix is real, not just a stale audit. If unsure, ask the engineer who ships the change to confirm.

**A new AP subject was added to the May roster.** Add a row in `data.json` and audit it from scratch using the rules in step 2.

**A contradiction has been outstanding for >2 weeks.** Surface to leadership — either prioritize the fix, or change severity to reflect the team's actual stance on it.

**A subject's enrollment hits 0.** Remove it from `data.json` (don't track subjects with no students). The dashboard is for operational decisions, not historical record.
