# PROMPT 12: Performance view (Stage 6/9)

## Requirements
1. Form: case count, mesh elements, seed, max workers (validated, units, defaults), Run/Cancel, progress bar showing current worker count and repetition.
2. Results: system info table, workload table, timing table (p, five times, median, speedup, efficiency, max diff), Plotly charts (median time vs p; speedup vs p with ideal line; efficiency vs p), pass/fail agreement badge, launch-to-ready shown in separate card, disclosure banner when CPU cap applies, "Overhead and limitations" text panel, and CSV/JSON/PNG exports.
3. UI remains interactive during run (verify by switching views mid-run, job preserved).

## Tests
- Positive: render with real result fixture captured from P11 run.
- Negative: backend error -> ErrorPanel.
- Edge: cancel then rerun; navigating away and back shows same job.

## Acceptance
Full run from UI; exported CSV equals backend result; no hard-coded numbers in UI.

## Stop If
The analysis text claims speedup that data does not show. Text must be conditional on measured values or neutral.

