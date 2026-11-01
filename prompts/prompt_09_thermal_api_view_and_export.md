# PROMPT 9: Thermal API, Thermal Analysis view and CSV export (Stage 5/9)

## Requirements
1. `POST /api/thermal/solve` with element list (default `[10,20,40,80]`; validate 1-4096 entries of positive ints within limits) and optional problem overrides. Response: per mesh: node x, nodal T, element midpoints x, FE midpoint T, exact midpoint T, per-midpoint error, max midpoint error, max nodal error, dense exact curve, observed orders.
2. `POST /api/export/thermal` -> CSV. Columns: `N,element,x_mid_m,T_fe_mid_C,T_exact_mid_C,abs_error_C` (and second nodal CSV: `N,node,x_m,T_fe_C,T_exact_C`). Save default-run CSVs to `results/csv/`.
3. UI: parameter form (defaults shown, ranges, units), Run button, status, results: Plotly line chart (T [°C] vs x [m]) with exact curve plus trace per N (toggleable), second chart of max midpoint error vs N (log-log, with slope), table (N, h, max midpoint error, nodal error, observed order), "Assumptions and boundary conditions" panel, PNG export of both plots, CSV download.
4. Save plot PNGs into `results/plots/`.

## Tests
- Positive: endpoint numbers match P8 core tests.
- Negative: N=0, negative k, empty list, 10,000 entries -> clear envelope errors displayed inline.
- Edge: only one mesh (no convergence order shown, with an explanation); exported CSV re-parsed by test and compared with response.

## Acceptance
UI plot and table agree with `results/csv`; plot axes labelled with units; debugging insight appears in UI "Notes" panel if it truly occurred.

## Stop If
Any displayed number is computed in TypeScript (except unit formatting).

