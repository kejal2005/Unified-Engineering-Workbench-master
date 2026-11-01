# PROMPT 6: Nodal temperature CSV parser and sample data (Stage 4)

## Objective
CSV parsing with an exact schema, plus the sample files with provenance.

## Requirements
1. Header must match `node_id,x_m,y_m,z_m,temperature_C` exactly (trim whitespace, tolerate a UTF-8 BOM and CRLF, reject any other header with an `E_SCHEMA_MISMATCH` message showing expected vs found).
2. Each row: 5 fields; `node_id` a non-negative integer (reject duplicates); coordinates and temperature finite doubles; reject empty cells. Line-numbered errors. Empty file -> `E_EMPTY_FILE` with an explanation; header-only -> "no data rows".
3. Output: arrays plus `tMin`, `tMax`, node count, units (`m`, `°C`).
4. `POST /api/geometry/nodal-csv`. Limits per Config.
5. Create `data/samples/sample_part.stl` (small, self-authored, a simple solid with a few dozen facets) and `data/samples/sample_nodal_temperature_SYNTHETIC.csv` (analytically generated field, e.g. a smooth gradient on a grid; the filename and header comment in provenance must say SYNTHETIC). Write `data/provenance/*.md` (author, creation method, units, schema notes, licence, "not an ANSYS export"). The generator, if any, lives in `scripts/` and is only a data-creation tool.

## Tests
- Positive: valid file, BOM, CRLF, columns with extra spaces, negative temperatures.
- Negative: wrong header, missing column, extra column, `NaN`, `Inf`, `1e999`, text in numeric field, duplicate id, negative id, empty/header-only.
- Edge: all temperatures equal (tMin==tMax, legend must still work, so the API returns the values and the UI handles it in P7); a single node; a size-limit boundary.

## Acceptance
The samples load through the real endpoint; provenance exists for both; measured CSV limits are recorded.

## Stop If
The synthetic CSV is not labelled synthetic in both file name and provenance.

