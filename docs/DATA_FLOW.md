# Anu Suite Data Flow

This document traces a single data series from raw source to final visualization, showing which artifacts each Anu skill produces.

## End-to-End Trace: S001 (Industrial Production)

### Stage 0: Adequacy
**Skill**: anu-adequacy
**Input**: Knowledge Base directory
**Output**: `Technical/docs/chapters/CH02_ADEQUACY_REPORT.json`

The agent verifies that:
- KB files exist for chapter 2 with methodology text
- BEA LTEG 1966 source is identified
- FRED INDPRO is accessible
- Reference values are extractable from book figures

**Result**: `overall_adequacy: ADEQUATE` → pipeline proceeds.

### Stage 1: Research
**Skill**: anu-research
**Input**: KB files
**Output**: `Technical/research/S001_research.json`

The agent extracts every quote, citation, footnote, and methodology description for S001 from the knowledge base. Records 47 entries across `methodology_description`, `source_citation`, `figure_context`, `splicing_note` types.

### Stage 2: Ingestion
**Skill**: anu-ingestion
**Input**: research file, KB files, raw data sources
**Outputs**:
- `Technical/series_registry.json` (S001 entry added)
- `Technical/decompositions/S001_DECOMPOSITION.md`
- `Technical/dprs/S001_DPR.md`

The registry now contains:
```json
"S001": {
  "subseries": {
    "S001-A": {"period": "1860-1932", "source_ref": "SRC-BEA-LTEG", ...},
    "S001-B": {"period": "1933-2025", "source_ref": "SRC-FRED-INDPRO", ...}
  },
  "extension": {"method": "reindex_at_splice", "splice_year": 1933},
  "reference_values": [{"year": 1929, "value": 25.4}, ...]
}
```

### Stage 3: Extension
**Skill**: anu-extension
**Input**: registry, research file
**Output**: `Technical/eprs/S001_EPR.md` (Extension Provenance Report)

EPR documents:
- Splice year: 1933
- Reindexing formula
- API source (FRED INDPRO)
- Validation against Shaikh's published values

### Stage 4: Replicator
**Skill**: anu-replicator
**Inputs**: registry, research, raw data
**Outputs**:
- `scripts/loading/L01_load_industrial_production.py`
- `scripts/processing/P01_process_industrial_production.py`
- `data/raw-data/api/FRED_INDPRO_20260401.json`
- `data/raw-data/parsed/S001_parsed.csv`
- `data/final-data/series/S001_final.csv`

Running `python replicate.py --series S001` executes:
1. L01: Fetch from FRED + parse Chopped CSV
2. P01: Reindex, splice, validate
3. V01: Reference value check (1929 = 25.4 ± 0.5)

### Stage 5a: Chopped
**Skill**: anu-chopped
**Input**: data_dict from P01
**Output**: `data/final-data/chopped/S001.csv`

```csv
Year|S001-A: BEA LTEG 1966 [1860-1932] Index 1899=100|S001-B: FRED INDPRO [1933-2025] Index 2017=100|S001-EXT: FRED Extension [2011-2025]|S001-F: Reindexed Extension
year,S001-A,S001-B,S001-EXT,S001-F
1860,2.1,,,
...
2025,,108.2,108.2,
```

Plus `SUBSOURCE_METADATA.json` with viz_label, role, period, units per column.

### Stage 5a: Extenbook
**Skill**: anu-extenbook
**Input**: data_dict, research, registry
**Output**: `data/final-data/extenbooks/S001_industrial_production.xlsx`

4 sheets:
- **Data**: All subsource columns
- **Provenance**: Source citations, access dates
- **Research**: Methodology quotes from Shaikh
- **Construction**: Step-by-step recipe

### Stage 5b: Visualization Export
**Skill**: anu-shiny (writer)
**Input**: chopped CSVs, registry
**Outputs**:
- `data/final-data/shiny/chapter_02.csv`
- `data/final-data/shiny/series_catalog.json`
- `data/final-data/shiny/SUBSOURCE_METADATA.json`

The Shiny app reads these files. No further processing needed.

### Stage 6: Variant (if applicable)
**Skill**: anu-variant
**Output**: Variant entries in registry

S001 has only one variant (canonical). Profit rate (S037) has three variants tracked here.

### Stage 7: Ledger
**Skill**: anu-ledger
**Output**: `Technical/LEDGER.json` updated

S001 row marked complete across all artifacts.

### Stage 8: Review
**Skill**: anu-review
**Output**: `AUDIT_REPORT.md`

12-dimension audit. S001 scores 88/90.

### Stage 9: Pipeline Validation
**Skill**: anu-pipeline
**Output**: Pipeline integrity check passed

End-to-end hash audit confirms reproducibility invariant holds.

## The Single Source of Truth

Throughout this entire flow, **`series_registry.json` is the only source of truth**. Every skill reads from it; only `anu-ingestion` writes to it. This is what makes the architecture coherent — there is no parallel configuration, no second source that can drift out of sync.

If you change the registry, every downstream artifact derives from the new registry on next regeneration. The hash audit trail makes any divergence detectable.

## How a Reviewer Verifies the Flow

A reviewer can trace S001 from output back to source:

1. Open Extenbook → see Data, Provenance, Research, Construction sheets
2. Construction sheet says: "Step 1: Load S001-A from BEA LTEG Table A-15"
3. Open `data/raw-data/parsed/S001_parsed.csv` → see the parsed data
4. Open `data/user-inputs/chopped/S001.csv` → see the original input
5. Hash matches across files (recorded in LOAD_LOG.json)
6. Original source citation in Provenance sheet → can be downloaded from BEA

The chain is unbroken from raw source to final visualization.
