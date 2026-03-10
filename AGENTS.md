# AGENTS.md

This repository contains a radar-engineering workflow for **Radar Mosaic + Radar Nowcast** using real radar data from:

- **NCK** = Nong Khaem
- **NGK** = Nong Chok

The main input format is **Universal Format (`.uf`)**.

## What Codex should optimize for

When working in this repository, prioritize the following in order:

1. **Correct UF ingestion and field inspection**
2. **Reliable QC and masking**
3. **A reproducible common grid**
4. **A stable NCK+NGK mosaic**
5. **A deterministic nowcast baseline**
6. **Verification by lead time**
7. **Operational repeatability**

Do not jump straight to advanced ensemble nowcasts before the baseline works end-to-end.

## Repository conventions

- Keep configuration in `configs/`.
- Keep reusable code in `src/`.
- Keep runnable entrypoints in `scripts/`.
- Keep human-readable docs in `docs/`.
- Prefer small composable modules over notebook-only logic.

## Default architecture

Unless the task explicitly asks for research comparison, use this architecture:

- ingest UF
- QC
- grid each radar to a common domain
- build **mosaic first**
- prepare reflectivity or rain-rate field
- run deterministic nowcast from the mosaic
- verify by lead time

## Important engineering rules

### 1) Never leave the grid implicit
Every gridded output must document:
- CRS / projection
- grid origin
- x/y spacing
- extent
- units
- height reference when relevant

### 2) QC is mandatory
At minimum, check:
- file integrity
- timestamp validity
- field presence
- implausible value ranges
- clutter / noise masking
- overlap seam behavior

### 3) Prefer inspectable outputs
Whenever practical, generate:
- field inventory CSV/JSON
- QC masks
- mosaic preview PNG
- seam comparison plots
- nowcast frames
- verification summaries

### 4) Make degraded mode explicit
If one radar is unavailable or poor quality:
- fall back to single-radar processing if possible,
- mark outputs as degraded,
- do not silently pretend it is a full dual-radar mosaic.

## Recommended tools/libraries

Use these by default unless the repo already standardizes differently:

- **Py-ART** for UF reading and radar-object inspection
- **xradar** for xarray/DataTree-centric UF workflows
- **wradlib** for radar georeferencing and geospatial utilities
- **numpy / xarray / scipy / pandas** for data processing
- **pysteps** for motion estimation and extrapolation baseline
- **matplotlib / cartopy** for diagnostics and map products

## Typical tasks Codex may receive

- add a UF field-inspection script
- map field names from NCK/NGK into a normalized internal schema
- build or fix a gridding pipeline
- fix overlap seams in the mosaic
- add dBZ→rain-rate conversion
- run deterministic nowcast for 15/30/60/90/120 min leads
- add evaluation metrics and regression checks
- harden scripts for repeated scheduled runs

## Acceptance criteria for changes

A change is not complete unless most of these apply:

- code runs from a documented command
- assumptions are explicit
- file paths and outputs are predictable
- logs/errors include radar/time/file context
- at least one debug artifact exists for new processing steps
- verification is updated if forecast behavior changes

## Suggested commands

If the repository contains these scripts, use them instead of inventing new ad-hoc commands:

```bash
python scripts/inspect_uf.py --input data/raw/NCK20240930164500.uf
python scripts/build_mosaic.py --config configs/mosaic.yaml
python scripts/run_nowcast.py --config configs/nowcast.yaml
python scripts/evaluate_nowcast.py --config configs/verify.yaml
```

If these commands do not exist yet and the user asks for them, create them.

## Things that commonly go wrong

- UF readers expose different field names or metadata layout
- sweeps/times are not aligned across radars
- CRS or axis order is wrong
- overlap weighting creates visible seams
- thresholding removes weak but real precipitation
- motion estimation becomes unstable with noisy fields
- verification compares fields on mismatched grids or units

## Instruction priority

If this file conflicts with a more specific task from the user, follow the user.
If this file conflicts with repository code or configs already agreed by the team, preserve compatibility and explain the tradeoff.

## Related docs

See also:
- `docs/skill.md`
- `docs/workflow.mmd`
- `docs/HOW_TO_USE.md`
