# Anu Replicator — Self-Contained Replication Package

## Purpose

A versioned, self-contained replication package with a four-phase architecture: Loading, Processing, Validation, Manual Adjustment.

**No agent is needed to run the replicator.** A researcher clones the package, sets API keys, runs `python replicate.py`, and gets complete validated output with a full SHA-256 hash audit trail.

This is the core "agent constructs, human verifies" pattern of the Anu Suite.

## Design Philosophy

1. **Self-contained**: All inputs, code, and configuration in one package
2. **Four-phase**: Loading (L##), Processing (P##), Validation (V##), Manual Adjustment (M##) cleanly separated
3. **Numbered scripts**: Execution order is explicit via L##/P##/V##/M## numbering
4. **Registry-driven**: `series_registry.json` is the single source of truth
5. **Dynamic paths**: All paths resolved via `pathlib`, no hardcoded paths
6. **Detailed reporting**: Every series prints construction tree, validation status, coverage
7. **Graceful degradation**: Missing API keys skip affected series, never crash the whole run

## Four-Phase Architecture

### Phase 1: Loading (L## scripts)
**Purpose**: Acquire all data from files and APIs.

**Input**: `data/user-inputs/` (read-only, human-provided)
**Output**: `data/raw-data/` (machine-fetched, never hand-edited)

| Script | Description |
|--------|-------------|
| `L00_load_all_data.py` | Orchestrator: discovers and runs all L## scripts in order |
| `L01_load_[name].py` | Loads a specific series |
| ... | One L## per series |

`L00` writes `data/raw-data/LOAD_LOG.json` with status per series, output file paths, vintage dates, observation counts.

### Phase 2: Processing (P## scripts)
**Purpose**: Construct, extend, and format all data series.

**Input**: `data/user-inputs/` + `data/raw-data/`
**Output**: `data/final-data/`

Each P## script:
1. Reads `series_registry.json` for construction steps and transforms
2. Loads parsed data from `data/raw-data/`
3. Loads research notes from `data/user-inputs/research/`
4. Executes construction steps in order, verifying each transform
5. Extends with API data using configured splice method
6. Validates against reference values
7. Writes to `data/final-data/series/`, `chopped/`, `extenbooks/`

### Phase 3: Validation (V## scripts)
**Purpose**: Verify all processed data against reference values, range checks, continuity, hash integrity.

| Script | Description |
|--------|-------------|
| `V00_validate_all.py` | Orchestrator |
| `V01_reference_values.py` | Compare outputs against published reference values |
| `V02_range_checks.py` | Verify all values fall within expected bounds |
| `V03_continuity.py` | Year-over-year continuity checks |
| `V04_completeness.py` | No missing years or unexpected NaN gaps |
| `V05_cross_series.py` | Cross-series consistency constraints |
| `V06_splice_quality.py` | Splice point transition quality |
| `V07_extension_overlap.py` | Extension overlap period correlation |
| `V08_hash_integrity.py` | SHA-256 verification of all files |

### Phase 4: Manual Adjustment (M## scripts)
**Purpose**: Apply documented, justified manual adjustments when automated processing cannot fully replicate the original.

Every manual adjustment must be documented in `config/ADJUSTMENT_MANIFEST.json` with id, series_affected, description, justification, and decision_ref.

## Master Orchestrator

```bash
python replicate.py                      # Full run: L00 -> P00 -> V00 -> M00
python replicate.py --load-only          # Just loading phase
python replicate.py --process-only       # Just processing
python replicate.py --validate-only      # Just validation
python replicate.py --skip-validation    # L00 -> P00 -> M00, skip V00
python replicate.py --series S001 S002   # Specific series only
python replicate.py --chapter 2          # All series in chapter
python replicate.py --dry-run            # Verify inputs, don't fetch APIs
```

## Folder Structure

```
ANU_REPLICATOR/
├── README.md
├── VERSION
├── requirements.txt
├── replicate.py                        # Master orchestrator
├── config/
│   ├── series_registry.json            # Single source of truth
│   ├── api_config.json
│   ├── api_keys.env.example
│   └── validation_config.json
├── data/
│   ├── user-inputs/                    # Read-only during replication
│   │   ├── chopped/
│   │   ├── provenance/
│   │   └── research/
│   ├── raw-data/                       # L## output
│   │   ├── api/
│   │   ├── parsed/
│   │   └── LOAD_LOG.json
│   └── final-data/                     # P## output (deliverables)
│       ├── series/
│       ├── chopped/
│       ├── extenbooks/
│       ├── shiny/
│       ├── reports/
│       └── PROCESS_LOG.json
├── scripts/
│   ├── loading/                        # L00, L01, L02, ...
│   ├── processing/                     # P00, P01, P02, ...
│   ├── validation/                     # V00, V01, ..., V08
│   └── manual/                         # M00, M01, ...
├── lib/                                # Shared library
│   ├── paths.py                        # pathlib-based resolution
│   ├── config_loader.py
│   ├── fetchers/                       # API fetcher modules
│   ├── transforms/                     # reindex, splice, aggregate
│   └── formats/                        # chopped_writer, extenbook_writer
└── docs/
    ├── DECISION_LOG.md
    └── ASSUMPTIONS.md
```

## Why This Architecture

### Reproducibility Without Agents
The agent writes the L##/P##/V##/M## scripts. After that, an agent is never needed. A future researcher runs `python replicate.py` and gets identical output — verified by hash matching.

### Numbered Scripts Are Self-Documenting
Anyone reading the package can immediately see: L01 runs first, then L02, then L03... then P01, P02... The execution order is visible in the file system. No hidden dependency graphs.

### Validation Is a First-Class Phase
Instead of validation being scattered through processing scripts (where it's easy to skip), it's a dedicated phase with 8 standardized checks. The validation report is itself a deliverable.

### Manual Adjustments Are Auditable
Sometimes data construction requires judgment calls that automated processing can't replicate (e.g., a one-time adjustment from the source author's appendix note). Rather than baking these into P## scripts, they're explicit M## scripts with full justification in `ADJUSTMENT_MANIFEST.json`.

## Console Output

The orchestrator prints a detailed banner, per-series construction trees, and a summary table:

```
================================================================
ANU REPLICATOR v3.0 — Starting Run
================================================================

API Keys: [OK] FRED, [OK] BEA, [MISSING] OECD
Input Files: 105 series found, 3 missing chopped CSVs

[L00] Loading Phase
  L01 S001 Industrial Production .................. SUCCESS  (1272 obs)
  L02 S002 Real Investment ......................... SUCCESS  (193 obs)
  ...

[P00] Processing Phase
  P01 S001 Industrial Production
    Subsource: S001-A (BEA LTEG 1860-1932) ......... OK
    Subsource: S001-B (FRED INDPRO 1933-2025) ...... OK
    Splice at 1933 ................................. CONTINUITY OK (1.2% diff)
    Validation: 5/5 reference checks ............... PASS
    Output: 2 files written ........................ OK
  ...

[V00] Validation Phase
  V01 Reference values ............................. 105/105 PASS
  V02 Range checks ................................. 105/105 PASS
  V03 Continuity ................................... 102/105 PASS (3 warnings)
  ...

================================================================
RUN COMPLETE: 105 SUCCESS, 0 PARTIAL, 0 FAILED
Duration: 4m 23s | Output: data/final-data/
================================================================
```

## Naming Convention

| Pattern | Phase |
|---------|-------|
| `L00_load_all_data.py` | Loading orchestrator (always L00) |
| `L{NN}_load_{series_name}.py` | Loading script |
| `P00_process_all_data.py` | Processing orchestrator (always P00) |
| `P{NN}_process_{series_name}.py` | Processing script |
| `V00_validate_all.py` | Validation orchestrator |
| `V{NN}_{check_name}.py` | Validation script |
| `M00_apply_all_adjustments.py` | Manual orchestrator |
| `M{NN}_adjust_{name}.py` | Manual adjustment script |

Script names include the series name (not just the number) for human readability. The number determines execution order.

## Integration with Other Anu Suite Skills

| Skill | Relationship |
|-------|-------------|
| Anu Research | research.json files bundled in `data/user-inputs/research/` |
| Anu Ingestion | series_registry.json bundled in `data/user-inputs/` |
| Anu Extension | Extension methodology defined by Extension skill, implemented in P## scripts |
| Anu Chopped | P## scripts write Chopped CSVs |
| Anu Extenbook | P## scripts write Extenbook Excels |
| Anu Shiny | Final-data feeds the Shiny dashboard |
| Anu Review | Review skill audits replicator package quality |
