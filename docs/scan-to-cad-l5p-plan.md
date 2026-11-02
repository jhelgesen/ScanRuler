# Scan-to-CAD extension: L5P water-pump benchmark

Status: Engineering specification (implementation not yet validated)  
Base: ScanRuler 0.9.4  
Branch: `feature/scan-to-cad`

## Objective

Turn imported STL scans (initial fixture: `l5pwater.stl`, units mm) into an inspectable engineering feature map and usable CAD reconstruction. Preserve upstream ScanRuler tools and behaviors.

## Initial scope

1. **Input validation**: import STL; report units, triangle count, bounds, disconnected components, mesh defects and coordinate basis; do not silently convert units.
2. **Automated geometry proposals**: find dominant planes and candidate cylinders using mesh neighborhoods and robust multi-candidate fitting. Record fit inlier count, sigma, geometric extent, axis, center, radius, confidence and residuals.
3. **Bolt-hole identification**: distinguish potential bores from exterior cylindrical bosses using topology and inward-facing normals where supported. Provide center point, diameter, axis direction, axial range, and group into patterns relative to a datum plane. Preserve ambiguity for partial/occluded holes instead of inventing them.
4. **Datum alignment**: suggest primary mating plane and orthogonal alignment; user confirms orientation before using XY coordinates. Persist transform.
5. **Review interface**: overlay detected features as visible opaque/high-contrast geometry, toggles and an editable list; selecting a row highlights that fit without changing orbit/pan mode.
6. **Export**: CSV/JSON hole table and datum coordinate system; analytic STEP surfaces/axes if supported; reconstruction STEP is a separate milestone requiring verified watertight B-rep and CAD-vs-STL comparison.
7. **Validation**: regression fixtures, tolerance sensitivity, visual snapshots, benchmark performance, and measurements against manually verified ground truth.

## Proposed pipeline

STL mesh -> normalize mm and validate -> connected components -> local normal/curvature cues -> planar/cylindrical seed candidates -> region growing with robust refits -> merge duplicates -> candidate bolt-hole classification -> choose datum -> project centers into datum XY -> user review -> exports.

## Acceptance tests for the L5P fixture

- Load `l5pwater.stl` without viewer regression or data loss.
- Detect the major mounting plane and visually overlay it, with fit residuals in mm.
- Report visible bolt-hole candidates with diameter, center XYZ, direction XYZ and confidence; permit user correction.
- Compute bolt-pattern XY distances relative to the confirmed datum; perform a second pass for missed or partial holes.
- Preserve normal camera orbit/pan/zoom during selection.
- Produce readable PNG screenshot and feature CSV, with repeatable output for the same scan.
- Verify STEP output by reopening it and checking topology and dimensional deviation against the original STL. No acceptance claim before testing.

## Implementation notes

- Reuse the existing 3D Measure fitting and selection logic before adding a parallel fitting engine.
- Keep new behavior opt-in to avoid modifying upstream measurement outputs.
- Avoid committing customer proprietary scan data to this public repository. Use synthetic/test fixtures or locally supplied scans only.
- Track license obligations: preserve ScanRuler's AGPL-3.0-only notices and review obligations before distribution.

## Milestones

M0: source audit / build and test upstream baseline.  
M1: autonomous planar/cylindrical feature candidate generation.  
M2: hole-axis identification and datum-relative hole table.  
M3: UX overlays, correction workflow, JSON/CSV and PNG.  
M4: reconstruction STEP and quantitative deviation verification.

## Notes

The earlier standalone OpenCascade prototype is a reference for requirements, not a drop-in source dependency. No claim is made here that L5P scans have been processed or geometry extracted yet.
