# PROMPT 17: UI consistency, accessibility and responsiveness pass (Stage 9)

## Requirements
1. Audit all five views against one design system: spacing scale, type scale, colour tokens, border/radius conventions (minimal rounding), table styles, form layouts. Remove any decoration without function.
2. Consistent status model (idle/running/success/error/cancelled) with distinct visual treatment for inputs, status, results, errors.
3. Responsive down to ~1024 px width and usable at 768 px (collapsible sidebar). Keyboard navigation, visible focus, labelled controls, `aria-live` for status/errors, adequate contrast, no colour-only meaning in plots and legends.
4. Loading and empty states everywhere; every export button works; every control has real behaviour.
5. Check there is no runtime AI-generated content, and no network use.

## Tests
- Positive: manual checklist per view (recorded).
- Negative: browser zoom 200%, long error message, overflowing table.
- Edge: window resize during 3D view and plots.

## Acceptance
`npm run build` succeeds with no TS errors/warnings; checklist is completed with screenshots at 1440, 1024, 768 px.

## Stop If
The pass changes any backend contract.

