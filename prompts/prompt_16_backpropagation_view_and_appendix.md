# PROMPT 16: Backpropagation view and handwritten appendix support (Stage 8/9)

## Requirements
1. UI: input form (defaults shown, model equations rendered as plain text/unicode), Run, table of forward values, gradient table (analytic, numerical, abs error, pass/fail against 1e-5), update table (old -> new), Plotly loss chart (loss vs step, axis labels, 100 steps), explanation panel about gradient direction and observed loss trend, and CSV/PNG export.
2. Create `docs/handwritten/README.md` stating: derivation must be handwritten and scanned by human (<=2 pages), what to include (forward pass, chain-rule steps for w, b, v, c, numeric values, simultaneous update, gradient-direction note), and numeric self-check table populated from program output after doing it by hand.

## Tests
- Positive: UI shows values equal to API.
- Negative: invalid learning rate -> inline error.
- Edge: lr large enough to oscillate (e.g. 5) shows loss chart correctly without crashing.

## Acceptance
Results CSV and plot saved in `results/`; handwritten scan placeholder instructions exist.

## Stop If
Assistant is asked to write the handwritten derivation itself.

