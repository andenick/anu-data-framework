# Anu Suite — Agent-Driven Data Construction Framework

**A 12-skill framework for AI agents to construct empirical data projects that produce outputs reproducible without agents.**

---

## What the Anu Suite Solves

When AI agents construct empirical research data, three problems emerge:

1. **Reproducibility**: How do you guarantee a future researcher can reproduce results without re-running the agent?
2. **Provenance**: How do you track which data came from where, and what transformations were applied?
3. **Verification**: How do humans audit agent-constructed data without trusting the agent's word?

The Anu Suite solves these through a registry-driven architecture where every agent action is recorded, every data point traces back to a primary source, and the final pipeline runs as plain Python (or R, Stata, etc.) without any agent in the loop.

Named after Anwar Shaikh's meticulous approach to empirical data construction — the gold standard for scholarly economic research.

---

## The 12 Skills

| # | Skill | Purpose |
|---|-------|---------|
| 1 | **anu-research** | Mine knowledge base for quotes, references, methodology, footnotes per series |
| 2 | **anu-ingestion** | KB construction, data import, absorption, decomposition, provenance documentation |
| 3 | **anu-extension** | Faithful API-based data extension with fidelity framework |
| 4 | **anu-chopped** | Machine-readable CSV format (Row 1 metadata, Row 2 IDs, Row 3+ data) |
| 5 | **anu-extenbook** | Human-readable 4-sheet Excel (Data, Provenance, Research, Construction) |
| 6 | **anu-replicator** | Self-contained packages with L##/P##/V##/M## phases |
| 7 | **anu-shiny** | Plotly Dash interactive visualization with multi-source traces |
| 8 | **anu-review** | 12-dimension quality audit framework |
| 9 | **anu-pipeline** | 10-stage multi-agent orchestrator |
| 10 | **anu-variant** | Methodology variant tracking |
| 11 | **anu-ledger** | Data checkout/checkin coverage tracking |
| 12 | **anu-adequacy** | Statistical adequacy testing framework |

Each skill is a markdown specification (in `specs/`) that an AI agent loads and executes. Skills are version-controlled and composable.

---

## Architecture

### Data Flow

```
Knowledge Base / Source Documents
        │
        ▼
  [Anu Research]  ─────────────────────►  S###_research.json
        │                                  (per-series dossier)
        ▼
  [Anu Ingestion] ─────────────────────►  series_registry.json
        │                                  (single source of truth)
        ▼
  [Anu Extension] ─────────────────────►  Extension methodology
        │
        ▼
  [Anu Replicator]
    Loading (L##)  ──────────────────────►  data/raw-data/
    Processing (P##) ────────────────────►  data/final-data/
    Validation (V##) ────────────────────►  VALIDATION_REPORT.json
    Manual Adjust (M##) ─────────────────►  ADJUSTMENT_MANIFEST.json
        │
        ├─────────► [Anu Chopped]    ────►  Validated CSVs
        ├─────────► [Anu Extenbook]  ────►  4-sheet Excel workbooks
        └─────────► [Anu Shiny]      ────►  Interactive Dash app
        │
        ▼
  [Anu Review]   ──────────────────────►  Quality audit (12 dimensions)
```

### The Single Source of Truth

Every output reads from `series_registry.json`:

```json
{
  "registry_version": "2.0",
  "series": {
    "S001": {
      "name": "US Industrial Production Index",
      "chapter": 2,
      "viz_column": "IndProd",
      "subseries": {
        "S001-A": {
          "period": "1860-1932",
          "source": "BEA LTEG 1966, Table A-15",
          "units": "Index 1899=100"
        },
        "S001-B": {
          "period": "1933-2025",
          "source": "FRED INDPRO",
          "units": "Index 2017=100"
        }
      },
      "extension": {
        "method": "reindex_at_splice",
        "splice_year": 1933
      }
    }
  }
}
```

All downstream artifacts derive from this:
- Chopped CSVs auto-generate Row 1 metadata
- Extenbooks pull provenance from registry
- Shiny app reads viz_column aliases
- Validation scripts check expected reference values

### Hash-Based Audit Trail

Every input and output file gets a SHA-256 hash recorded in `LOAD_LOG.json` and `PROCESS_LOG.json` with timing metadata:

```json
{
  "series_id": "S001",
  "status": "SUCCESS",
  "input_hash": "a7f3...",
  "output_hash": "b9e2...",
  "duration_ms": 1247,
  "timestamp": "2026-04-26T14:32:18Z"
}
```

A future researcher can verify they have the same data by recomputing hashes. Any modification — accidental or intentional — is detectable.

---

## Why This Matters for AI Safety

The Anu Suite demonstrates three patterns directly relevant to safe AI deployment:

### 1. Agent Constructs, Human Verifies
Agents construct the pipeline (writing L## loaders, P## processors, V## validators). The final pipeline runs without any agent in the loop. A researcher clones the package, sets API keys, runs `python replicate.py`, and gets identical output.

### 2. Protocol Over Code
Skills are markdown specifications, not compiled libraries. Agent behavior is governed by readable text that humans can audit, version-control, and modify. This is a more transparent approach than black-box agent orchestration.

### 3. Provenance Is First-Class
Every data point traces back to a primary source through the registry. Every transformation is logged with timing and hashes. There is no way for the agent to silently introduce changes — the audit trail makes it impossible.

---

## Quick Start

### As a Specification (read the protocols)
```bash
# Browse the 12 skills
ls specs/
# Read the master overview
cat docs/ARCHITECTURE.md
# See the registry schema
cat schemas/series_registry_schema.json
```

### As an AI Agent (load the skills)
With Claude Code or similar agent harness:
```bash
# Skills can be loaded as instructions:
claude "Using the anu-research skill in specs/01_research.md, mine the KB for series S001"
```

### As a Researcher (use the patterns)
The repo includes:
- Schema definitions for `series_registry.json`
- Templates for L##/P##/V##/M## scripts
- Example registry from a real project (sanitized subset)
- Reference implementation: see [capitalism-data](https://github.com/andenick/capitalism-data) for the suite applied at scale

---

## Reference Implementation

The Anu Suite has been applied to a complete academic replication project: extending Anwar Shaikh's *Capitalism: Competition, Conflict, Crises* (2016) with 105 data series across 235 years of US economic history (1790-2025).

Results:
- 209 figures with full metadata
- 105 series identified and numbered
- 13 series extended to 2025 using BEA, FRED, BLS APIs
- 182 Extenbook Excel workbooks with full provenance
- Interactive R Shiny dashboard
- 100% validation pass rate

See [github.com/andenick/capitalism-data](https://github.com/andenick/capitalism-data).

---

## Repository Structure

```
anu-suite/
├── README.md              # This file
├── CLAUDE.md              # Reviewer quick-start
├── LICENSE                # MIT
├── specs/
│   ├── 01_research.md     # Per-series research dossier protocol
│   ├── 02_ingestion.md    # KB construction, decomposition
│   ├── 03_extension.md    # Faithful API extension framework
│   ├── 04_chopped.md      # Machine-readable CSV format
│   ├── 05_extenbook.md    # Human-readable Excel format
│   ├── 06_replicator.md   # Self-contained replication packages
│   ├── 07_shiny.md        # Interactive visualization standard
│   ├── 08_review.md       # 12-dimension quality audit
│   ├── 09_pipeline.md     # Multi-agent orchestrator
│   ├── 10_variant.md      # Methodology variant tracking
│   ├── 11_ledger.md       # Coverage tracking
│   └── 12_adequacy.md     # Statistical adequacy framework
├── schemas/
│   ├── series_registry_schema.json
│   └── series_id_spec.md
├── examples/
│   ├── sample_series_registry.json
│   ├── sample_chopped.csv
│   └── sample_extenbook_structure.md
└── docs/
    ├── ARCHITECTURE.md
    └── DATA_FLOW.md
```

---

## Portfolio Context

This is one of four repositories demonstrating an integrated system for AI-driven economic research:

1. **[hdarp](https://github.com/andenick/hdarp)** — Input layer: get data into the pipeline (multi-engine OCR consensus)
2. **[anu-suite](https://github.com/andenick/anu-suite)** — Protocol layer: structure agent tasks (this repo)
3. **[nickydata](https://github.com/andenick/nickydata)** — Reproducibility layer: language-agnostic pipeline architecture
4. **[capitalism-data](https://github.com/andenick/capitalism-data)** — Demonstration layer: 235 years of economic data extended to 2025

---

## License

MIT — All specifications and schemas may be freely used, modified, and distributed.
