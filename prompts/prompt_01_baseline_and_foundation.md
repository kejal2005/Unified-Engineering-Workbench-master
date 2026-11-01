# PROMPT 1: Requirements baseline and repository foundation (Stage 0–1)

## Objective
Create the authoritative documentation set, git repo and folder skeleton. No application code.

## Inspect
The assignment PDF, the master instruction, whether the repo exists.

## Requirements
1. `git init`. Add `.gitignore` (build dirs, `node_modules`, `dist`, IDE caches, `*.obj`, release staging), `.gitattributes` (LF for source, binary for samples), `.editorconfig`.
2. Create the full tree from Part 4 with `.gitkeep` only where a folder must exist empty.
3. Write `docs/PROJECT_SPEC.md`: every assignment requirement with ID (A1…F5, ZIP1…), numeric parameters verbatim, rubric table, shortlist gates, acceptance criteria. Include the expected midpoint-error values `q·h²/(8k)` for N=10/20/40/80 (1.25, 0.3125, 0.078125, 0.01953125 °C) as a derived check, marked "derived, to be verified by code".
4. Write `ARCHITECTURE.md` (layers, API contract, error envelope, lifecycle, thread rules, packaging), `IMPLEMENTATION_PLAN.md` (these 20 prompts), `DECISIONS.md` (D1–D6 and stack choices with reasons), `PROJECT_STATE.md` (template, Phase = P1 done), `TESTING.md`, `BUILD_AND_PACKAGING.md`, `LIMITATIONS.md`, `AI_USE_LOG.md` (header and honesty rules), README stub, and an effort log table (`docs/EFFORT_LOG.md`) for actual hours.

## Files
All the above.

## Constraints
No code, no invented results. The spec must quote numbers exactly as in the PDF.

## Tests
- Positive: every file present, links resolve.
- Negative: grep spec for TODO or empty sections.
- Edge: spec parameter values diff-checked against the PDF by hand.

## Commands
`git status`, `git log --oneline`, a directory listing.

## Acceptance
All 10 docs exist and are consistent; the traceability IDs in the spec match `IMPLEMENTATION_PLAN.md`; the first commit exists.

## Stop If
The spec omits any mandatory item or any number differs from the PDF.

