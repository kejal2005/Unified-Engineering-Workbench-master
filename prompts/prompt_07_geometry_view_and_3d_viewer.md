# PROMPT 7: Geometry view, 3D viewer (Stage 4/9)

## Objective
Fully interactive viewer driven by backend-parsed data.

## Requirements
1. File pickers for STL and CSV; show file name, size, parse status. Errors displayed with code, message and hint.
2. Three.js scene in a React component with proper disposal (geometries, materials, renderer, listeners). STL as a `BufferGeometry` mesh with simple shading plus optional edges; rotate (left), pan (right/shift), zoom (wheel), **Fit to view** button (bounding sphere), reset.
3. CSV as `Points`, coloured by temperature via a perceptually reasonable colour map defined in `utils` (pure function with unit tests). Legend: gradient bar, tMin/tMax, unit "°C", and handling for tMin==tMax. Point-size control.
4. Node inspection by raycasting (`Points` threshold scaled to bounds): show node_id, x/y/z in m, temperature in °C, with highlighted marker. Toggle between STL and cloud visibility. Axes/scale indicator in model units. Label the STL units as "as provided in file (not specified by STL format)".
5. A banner when the loaded CSV is the sample synthetic file (based on the header/file name, backed by a backend flag).

## Tests
- Positive: colour map endpoints and monotonicity, bounds fit calculation.
- Negative: bad files show errors without crashing the viewer.
- Edge: a single point, a degenerate (flat) bounding box, 100k points interaction (record measured frame behavior).

## Acceptance
Manual script in TESTING.md executed and recorded with screenshots; no console errors; memory does not grow on repeated loads.

## Stop If
Fit-to-view fails for the sample or a flat box.

