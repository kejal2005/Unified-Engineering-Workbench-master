# PROMPT 8: Thermal FEM core (Stage 5)

## Objective
Correct 1D FEM solver in C++ with verification, no UI.

## Requirements
1. `ThermalProblem{L,A,k,q,T0,TL}` with validation (all positive/finite; N >= 1). Defaults: L=1, A=1e-4, k=10, q=10000, T0=100, TL=20.
2. Two-node element: `Ke = (kA/h)[[1,-1],[-1,1]]`, `fe = (qAh/2)[1,1]`. Global assembly into tridiagonal storage. Dirichlet via row/column elimination so system remains symmetric. Impose both BCs correctly: modify RHS by `-K[i,fixed] * Tfixed`.
3. Thomas solver (guard against zero pivots -> error). A dense Gaussian reference solver exists only in the test target.
4. Interpolation: `T_fe(x)` within an element by linear shape functions. Midpoint values per element.
5. Exact: `T(x) = 100 - 80*x/L + q/(2*k)*x*(L-x)`, evaluated by one function. Errors: nodal max and midpoint max using FE interpolation. Convergence: observed order `log2(e_N / e_2N)`.
6. A `solveBatchCase` hook (pure function: problem+N -> temperatures) to be reused by P10, with no allocation shared between calls.

## Tests
- Positive: N=1 and N=2 hand-computed K, f, T; midpoint max errors for N=10/20/40/80 within 1e-9 of 1.25, 0.3125, 0.078125, 0.01953125; BCs satisfied to 1e-12; Thomas vs dense for N=10/40 agree 1e-10. Observed order ~ 2.0.
- Negative: k=0, k<0, A<0, N=0, NaN, T0 > 1e308.
- Edge: T0==TL; q=0 (linear profile, error~0); N=1; N=1,000,000 (record time, no stack overflow).

## Challenge Test (for AI Log)
Deliberately ask the assistant, in a separate chat, to state the expected nodal error and the midpoint error, then compare with your code. Record whether its claim was correct. Do not fabricate a mistake.

## Debugging Insight
Record the one you actually encounter (e.g. nodal error ~ 1e-13 despite O(h^2) midpoint error and why) in `docs/DEBUG_INSIGHT.md`.

## Acceptance
Every test passes; the midpoint expectations are confirmed by the program output, not assumed.

## Stop If
Midpoint errors don't reproduce, or nodal-only error is used in any acceptance.

