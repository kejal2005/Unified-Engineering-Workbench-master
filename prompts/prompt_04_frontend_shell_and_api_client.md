# PROMPT 4: Frontend shell and typed API client (Stage 3/9 foundation)

## Objective
Application shell wired to the real backend, with no fake data.

## Inspect
`ARCHITECTURE.md` API contract; `src/frontend/` state.

## Requirements
1. Vite + React + TS (strict) + Tailwind. Lock dependencies (`package-lock.json`). Zero CDN or remote font use. Use a system font stack or local woff2.
2. Layout: fixed header (app name, backend status badge, version), left sidebar with five entries (Geometry, Thermal Analysis, Performance, Hardware, Backpropagation), main workspace, bottom status bar. Restrained neutral palette, one accent, defined spacing tokens.
3. `services/api.ts`: typed request helper, envelope parsing into `ApiError` (code, message, hint), `useJob` hook (poll every 250 ms, cancel, unmount cleanup), heartbeat every 5 s.
4. Shared components: `FileInput`, `NumberField` (label, units, min/max, inline error), `StatusBanner` (idle/running/success/error), `ErrorPanel`, `DataTable`, `EmptyState`.
5. Each view currently shows an honest empty state ("No data loaded") and real backend connectivity. No fake controls.
6. Vite proxy to the backend for dev. Production build uses relative URLs.

## Constraints
No numerical code in TS. Keyboard accessible, labelled inputs.

## Tests
- Positive: component tests for `NumberField` validation and `ApiError` parsing (Vitest, dev-only dependency).
- Negative: backend unreachable -> clear status badge, not a crash.
- Edge: a `useJob` unmount while polling leaves no leaked timers.

## Commands
`npm ci`, `npm run build`, `npm test`, and run the server against `dist/`.

## Acceptance
The built bundle served by the C++ server shows the shell; the status reflects the real `/health`; the production build makes no external requests.

## Stop If
Any asset loads from the internet.

