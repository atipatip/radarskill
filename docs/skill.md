---
name: radar-mosaic-nowcast-skill
version: 1.1.0
summary: >
  Agent skill for building, validating, and operating a practical Python pipeline for
  radar mosaic and radar nowcast from two real radars (NCK and NGK) using Universal
  Format (.uf) input.
owners:
  - radar-rnd-team
status: ready-for-team-use
compatibility:
  canonical: skill.md
  claude_code: .claude/agents/radar-mosaic-nowcast-agent.md
  codex: AGENTS.md
keywords:
  - radar mosaic
  - radar nowcast
  - universal format
  - pyart
  - xradar
  - wradlib
  - pysteps
  - verification
  - operations
---

# Purpose

This skill tells an AI coding agent how to work on a **real weather-radar engineering project** for:

- ingesting **UF** files from **NCK** and **NGK**,
- performing **QC**, **polar-to-Cartesian gridding**, and **radar mosaic**,
- generating **deterministic** and optionally **ensemble nowcasts**,
- evaluating forecast skill,
- and preparing the pipeline for **prototype** and then **operational** use.

The skill is optimized for agents that can read/edit files, run shell commands, inspect logs, and execute Python.

# Primary outcomes

The agent should be able to produce or improve:

1. UF ingestion scripts and metadata inspection tools.
2. QC and masking modules.
3. Gridding and mosaic modules for NCK + NGK.
4. Rain-rate conversion and nowcast modules.
5. Verification notebooks/scripts and benchmark reports.
6. Operational artifacts: runbooks, configs, CI checks, API/output packaging.

# When to use this skill

Use this skill when the task involves one or more of the following:

- reading or validating `.uf` radar files,
- investigating radar fields such as reflectivity, velocity, azimuth, elevation, range,
- building a common grid over Bangkok and nearby areas,
- combining NCK and NGK into one mosaic,
- running optical-flow / advection-based nowcast,
- evaluating lead-time skill,
- hardening a prototype for reliable repeated execution.

# When NOT to use this skill

Do not use this skill for:

- generic web app work unrelated to radar,
- non-radar geospatial work,
- long-range NWP forecasting,
- satellite-only nowcasting,
- policy or public communication tasks that do not require radar processing.

# Project assumptions

Unless the repository says otherwise, assume:

- input radars are **NCK** and **NGK**,
- raw input format is **Universal Format (.uf)**,
- the main deliverable is a **Python-first** workflow,
- the first operational target is **Bangkok and surrounding metropolitan area**,
- the initial baseline should be **2D mosaic + deterministic extrapolation nowcast**,
- the first success criterion is a **reproducible end-to-end prototype** rather than a perfect model.

If any of the following are missing, the agent must mark them as `unspecified` and continue with reasonable defaults:

- scan interval,
- exact radar coordinates and altitude,
- latency SLA,
- archive retention policy,
- preferred output CRS,
- production deployment environment.

# Recommended stack

## Core radar I/O and processing
- **Py-ART**: UF read, radar object inspection, radar utilities.
- **xradar**: UF to xarray/DataTree, modern xarray-friendly workflow.
- **wradlib**: georeferencing, beam geometry, interpolation, radar geospatial utilities.

## Numerical / data stack
- **numpy**
- **xarray**
- **scipy**
- **pandas**
- **dask** (only when data volume or repeated batch processing justifies it)

## Nowcast stack
- **pysteps** for motion estimation and extrapolation baselines.
- **OpenCV** or **scikit-image** only when custom motion estimation or image filtering is required.

## Visualization / export
- **matplotlib**
- **cartopy**
- **netCDF4** / **h5py** if repository uses NetCDF/HDF exports.

# Agent operating rules

## 1) Prefer the simplest working baseline first
Build the pipeline in this order unless the user explicitly asks otherwise:

1. read UF,
2. inspect metadata/fields,
3. QC baseline,
4. single-sweep or low-level gridding,
5. 2D reflectivity mosaic,
6. rain-rate transform,
7. deterministic nowcast,
8. verification,
9. only then add ensemble or advanced refinements.

## 2) Keep coordinate systems explicit
The agent must never leave coordinate assumptions implicit. Every grid-producing step must clearly define:

- projection / CRS,
- origin,
- x/y spacing,
- grid extent,
- units,
- height reference if 3D products are involved.

## 3) Treat QC as a first-class component
The agent must assume that poor QC will damage every downstream product.

Minimum QC checklist:
- obvious missing/corrupt files,
- invalid timestamps,
- impossible field ranges,
- non-meteorological clutter masking,
- overlap seam inspection,
- missing-data propagation rules.

## 4) Prefer mosaic-before-nowcast as the baseline architecture
Unless repository evidence shows otherwise, default recommendation is:

- **mosaic first**, then
- **nowcast from the mosaic field**.

Only choose nowcast-before-merge when there is a clearly justified reason such as:
- radically different radar quality by sector,
- station-specific outage handling experiments,
- research comparison of per-radar motion fields.

## 5) Make every result inspectable
For every algorithmic step, produce a debug-friendly artifact where practical:

- field inventory table,
- QC masks,
- gridded snapshot PNG,
- overlap comparison map,
- nowcast frames,
- metric summary by lead time.

# Expected inputs

The agent should be ready to accept these inputs:

```yaml
radars:
  - NCK
  - NGK
input_format: UF
sample_files:
  - NCK20240930164500.uf
  - NGK20240930164500.uf
area_of_interest: Bangkok Metropolitan Region
products:
  - reflectivity_mosaic_2d
  - rain_rate_mosaic_2d
  - deterministic_nowcast
optional_products:
  - CAPPI
  - CMAX
  - ensemble_nowcast
scan_interval_minutes: unspecified
latency_target_minutes_p95: unspecified
```

# Expected outputs

The agent should structure outputs into clear artifacts such as:

- `ingest_report.json`
- `field_inventory.csv`
- `qc_report.md`
- `mosaic_config.yaml`
- `mosaic_YYYYMMDDHHMM.nc`
- `nowcast_YYYYMMDDHHMM_leadXX.nc`
- `verification_summary.csv`
- `verification_report.md`
- `runbook.md`

# Standard workflow the agent should follow

## Step 1 — Ingest
- Read UF file(s).
- Identify radar/site metadata.
- Enumerate available fields and sweeps.
- Validate timestamps and detect missing or suspicious metadata.

## Step 2 — Preprocess / QC
- Choose the sweep(s) needed for the target product.
- Apply baseline clutter / noise masking.
- Handle missing or implausible bins.
- If velocity is used, document dealiasing status explicitly.

## Step 3 — Georeference and grid
- Convert radar polar coordinates to a common grid.
- Use a clearly declared projection appropriate for Bangkok operations.
- Keep the first prototype resolution practical: typically 1 km unless requirements justify 0.5 km.

## Step 4 — Build mosaic
- Merge NCK and NGK onto one common domain.
- Apply a documented overlap strategy.
- Export mosaic and produce a seam diagnostic.

## Step 5 — Prepare nowcast field
- Decide whether the nowcast input is reflectivity or rain rate.
- Apply thresholds and denoise carefully.
- Keep the transform consistent between training/baseline and evaluation.

## Step 6 — Motion estimation
- Use recent frames with explicit time spacing.
- Start with a robust baseline supported by the codebase, usually Lucas–Kanade or a pysteps-supported method.
- Save the motion field for inspection.

## Step 7 — Extrapolation
- Generate lead times explicitly, e.g. 15, 30, 60, 90, 120 minutes.
- Preserve grid and metadata consistency.
- Mark degraded-confidence leads if the methodology is simple extrapolation only.

## Step 8 — Verification
- Compare forecast vs observation on the same grid and same units.
- Break metrics down by lead time.
- When possible, stratify by rainfall threshold and event type.

## Step 9 — Operational packaging
- Add logging, error classification, monitoring, and fallback logic.
- Make outputs discoverable and machine-readable.

# Quality gates

The agent should not claim success unless these are satisfied:

## Ingestion gate
- UF files open successfully.
- Fields and times are enumerated.
- No silent field-name mismatches.

## Mosaic gate
- Grid definition is documented.
- Overlap seams are visually checked.
- Missing-data behavior is explicit.

## Nowcast gate
- Input field units are documented.
- Motion field has been inspected.
- At least one lead-time evaluation exists.

## Operational gate
- Commands are repeatable.
- Failure modes are logged.
- Outputs can be regenerated from raw input and config.

# Error handling policy

## Missing metadata
If metadata is incomplete, infer cautiously, mark assumptions clearly, and continue only if the inference is low-risk.

## One radar unavailable
Switch to degraded single-radar mode and mark outputs clearly as degraded.

## Motion estimation unstable
Fallback to a simpler motion method or reduce lead-time claims.

## QC uncertainty
Prefer conservative masking over optimistic false signal retention when creating operational rainfall products.

# Coding style for this project

- Write small composable Python functions.
- Separate I/O, transforms, configuration, and plotting.
- Keep notebook-only logic out of production modules.
- Put constants in config files, not inline magic numbers.
- Preserve units in variable names where it helps readability.
- Use typed function signatures where practical.
- Raise informative exceptions with radar/time/file context.

# Suggested repository layout

```text
project/
  AGENTS.md
  docs/
    skill.md
    runbook.md
    verification.md
  configs/
    mosaic.yaml
    nowcast.yaml
  data/
    raw/
    interim/
    processed/
  src/
    io/
    qc/
    grid/
    mosaic/
    nowcast/
    verify/
    export/
  scripts/
    inspect_uf.py
    build_mosaic.py
    run_nowcast.py
    evaluate_nowcast.py
  tests/
```

# Deliverable templates

## Minimum practical prototype
The agent should aim to produce:
- UF reader/inspector,
- one common grid,
- 2D reflectivity mosaic,
- rain-rate transform,
- deterministic nowcast,
- simple verification report.

## Operational candidate
The agent should additionally produce:
- config-driven runs,
- logging and health checks,
- degraded-mode fallback,
- benchmark cases,
- release checklist.

# Example tasks

## Example 1
**Task:** Build a first prototype from two UF samples.

**Expected behavior:**
- inspect both files,
- list fields and times,
- create one common grid,
- generate one mosaic snapshot,
- create a 15–60 min deterministic nowcast baseline,
- save plots and a short report.

## Example 2
**Task:** Investigate a seam artifact in the overlap region.

**Expected behavior:**
- compare per-radar gridded fields,
- verify timing mismatch,
- inspect weighting strategy,
- propose or implement a corrected overlap rule,
- regenerate diagnostics.

## Example 3
**Task:** Harden the pipeline for repeatable operations.

**Expected behavior:**
- add config files,
- create run scripts,
- implement structured logging,
- define degraded mode,
- add a verification regression check.

# Evaluation checklist

- [ ] UF input is readable and reproducible.
- [ ] Radar fields are explicitly mapped.
- [ ] Grid/projection are documented.
- [ ] Overlap handling is documented.
- [ ] Nowcast method and assumptions are documented.
- [ ] Verification metrics are generated by lead time.
- [ ] Failure and degraded modes are handled explicitly.
- [ ] Outputs are operationally inspectable.

# Versioning notes

- Update `version` whenever workflow assumptions, required tools, or acceptance criteria change.
- Keep `skill.md` as the canonical long-form reference.
- Derive `AGENTS.md` and Claude subagent files from this source to avoid divergence.
