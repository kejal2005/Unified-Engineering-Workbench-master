# PROMPT 15: Backpropagation core in C++ (Stage 8)

## Requirements
1. `Model{w,b,v,c}`; `forward(x)`: `z=wx+b`, `h=tanh(z)`, `ŷ=vh+c`, `J=0.5(ŷ-y)^2`. Defaults: x=1, y=2, w=0.5, b=0, v=1, c=0, lr=0.1.
2. Analytic gradients (no autodiff, no generic graph): `e=ŷ-y`, `dJ/dc=e`, `dJ/dv=e*h`, `dJ/dh=e*v`, `dz=dJ/dh*(1-h^2)`, `dJ/dw=dz*x`, `dJ/db=dz`.
3. Central differences with eps=1e-6 on each parameter perturbing the loss function; absolute error per parameter <= 1e-5, else result is flagged failing.
4. One simultaneous update (all four use gradients from original parameters, applied together). Then 100 subsequent training steps with same lr, loss recorded at each step (step 0 = initial, then 100 after).
5. Arithmetic checks: `z=0.5`, `h=tanh(0.5)~0.4621172`, `ŷ~0.4621172`, `e~-1.5378828`, `J~1.18254`, `dJ/dv~-0.7107`, `dJ/dc~-1.5379`, `dJ/dw=dJ/db~-1.2095`, updated `w~0.6209`, `b~0.1209`, `v~1.0711`, `c~0.1538`.
6. `POST /api/backprop/run` returning forward values, analytic gradients, numeric gradients, errors, pass flag, updated parameters, loss series, and CSV export (`results/csv/backprop_loss.csv`).

## Tests
- Positive: gradients match hand values to 1e-4 and numerical ones to 1e-5; loss decreases monotonically for 100 steps at lr=0.1.
- Negative: NaN inputs, lr <= 0, steps = 0 or > limit -> errors.
- Edge: x=0 (dw=0), very large |w| (tanh saturation, gradient vanishing, no NaN), eps too large vs too small shows degraded error.

## Acceptance
All four gradient errors <= 1e-5 measured; update is demonstrably simultaneous.

## Stop If
Any gradient fails the tolerance.

