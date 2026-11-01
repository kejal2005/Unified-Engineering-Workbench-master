# PROMPT 5: ASCII STL parser (Stage 4)

## Objective
Robust C++ STL parsing and validation plus the endpoint.

## Requirements
1. Parse ASCII STL (`solid ... facet normal ... outer loop ... vertex ... endloop endfacet ... endsolid`). Case-insensitive keywords, arbitrary whitespace, scientific notation via `std::from_chars` (or an equivalent locale-independent method).
2. Validate: non-empty; at least one facet; each facet has exactly 3 vertices; all numbers finite; facet count <= Config limit; size <= limit. Binary STL detection (e.g. starts with `solid` but contains non-text bytes, or size = 84 + 50n pattern) returns an actionable error: "Binary STL is not supported; export ASCII STL." Do not silently misparse.
3. Output: flat position array (float32-compatible), computed face normals when the file normal is zero or non-unit, bounds, triangle count, degenerate-triangle count (reported, not rejected).
4. `POST /api/geometry/stl` (raw text body). Errors carry line numbers.

## Files
`include/geometry/StlParser.*`, `src/geometry/StlParser.*`, transport route, `tests/geometry/*`, `tests/fixtures/stl/*`.

## Tests
- Positive: single triangle; cube (12 facets); scientific notation; CRLF.
- Negative: empty file, whitespace-only, missing `endloop`, 2 vertices, text instead of number, `nan`/`inf`, binary STL, truncated file.
- Edge: multiple solids; zero-area triangle; very long line; a file at exactly the limit and limit+1; Unicode BOM.

## Acceptance
All fixtures behave as specified; every error has code, line and hint; the largest passing synthetic STL is recorded with parse time and memory in TESTING.md as a measured size limit.

## Stop If
Any malformed fixture is accepted.

