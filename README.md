# Anu Data Framework

**A 14-skill protocol for AI agents to construct empirical data projects that produce outputs reproducible without agents.**

---

## The Problem

When AI agents construct empirical research data, three problems emerge:

1. **Reproducibility**: How do you guarantee a future researcher can reproduce results without re-running the agent?
2. **Provenance**: How do you track which data came from where, and what transformations were applied?
3. **Verification**: How do humans audit agent-constructed data without trusting the agent's word?

The Anu Data Framework solves these through a registry-driven architecture where every agent action is recorded, every data point traces back to a primary source, and the final pipeline runs as plain Python without any agent in the loop.

---

## The 14 Skills

| # | Skill | Version | Purpose |
|---|-------|---------|---------|
| 0 | **[Rules](specs/00_rules.md)** | — | Mandatory invariants (no synthetic data, no proxies, unit safety) |
| 1 | **[Research](specs/01_research.md)** | v2.0 | Mine knowledge base for methodology, quotes, and data sources per series |
| 2 | **[Ingestion](specs/02_ingestion.md)** | v4.0 | KB construction, data import, absorption, series decomposition, provenance |
| 3 | **[Extension](specs/03_extension.md)** | v3.3 | Faithful API-based data extension with 10-principle fidelity framework |
| 4 | **[Chopped](specs/04_chopped.md)** | v2.0 | Machine-readable CSV format (metadata headers, structured column IDs) |
| 5 | **[Extenbook](specs/05_extenbook.md)** | v3.2 | Human-readable 4-sheet Excel (Data, Provenance, Research, Construction) |
| 6 | **[Replicator](specs/06_replicator.md)** | v3.0 | Self-contained packages with L##/P##/V##/M## phases + hash audit |
| 7 | **[Visualize](specs/07_visualize.md)** | v5.0 | Interactive visualization (R Shiny + Plotly) with multi-source traces |
| 8 | **[Review](specs/08_review.md)** | v4.0 | 13-dimension quality audit with weighted scoring and certification |
| 9 | **[Pipeline](specs/09_pipeline.md)** | v3.0 | Multi-agent orchestrator across all 14 skills |
| 10 | **[Variant](specs/10_variant.md)** | v1.4 | Methodology variant tracking and documentation |
| 11 | **[Ledger](specs/11_ledger.md)** | v2.2 | Artifact coverage tracking (which series have which outputs) |
| 12 | **[Adequacy](specs/12_adequacy.md)** | v1.2 | Post-research readiness gate (are data sources sufficient?) |
| 13 | **[Data](specs/13_data.md)** | v2.0 | AnuData Architecture — the underlying format standard for all packages |
| 14 | **[Publish](specs/14_publish.md)** | v1.0 | Publication pipeline (scrub, package, validate for GitHub release) |

Each skill is a Markdown protocol specification that an AI agent loads and executes. Skills are version-controlled and composable.

---

## Architecture

```
Source Documents / APIs
        |
        v
  [Research]  ──────────────────────>  S###_research.json
        |                              (per-series methodology dossier)
        v
  [Adequacy]  ──────────────────────>  Data source readiness gate
        |
        v
  [Ingestion] ──────────────────────>  series_registry.json
        |                              (single source of truth)
        v
  [Extension] ──────────────────────>  Extension provenance records
        |
        v
  [Replicator]
    Loading (L##)  ─────────────────>  data/raw-data/
    Processing (P##) ───────────────>  data/final-data/
    Validation (V##) ───────────────>  VALIDATION_REPORT.json
    Manual Adjust (M##) ────────────>  ADJUSTMENT_MANIFEST.json
        |
        +──────> [Chopped]     ─────>  Structured CSVs
        +──────> [Extenbook]   ─────>  4-sheet Excel workbooks
        +──────> [Visualize]   ─────>  Interactive dashboard
        |
        v
  [Review]    ──────────────────────>  13-dimension quality audit
        |
        v
  [Publish]   ──────────────────────>  GitHub release package
```

---

## Quick Start

The Anu Data Framework is a set of protocol specifications, not a software library. To use it:

### With Claude Code
```bash
# Have an agent read a spec and apply it
claude "Read specs/02_ingestion.md and apply this protocol to construct
        a series_registry.json for the Maddison world GDP database"
```

### As a Reference
Read the specs in `specs/` to understand the data construction patterns, then implement them in your own project. The specs work with any AI agent platform or manual workflow.

### See It in Action
The [Measuring the Wealth of Nations](https://github.com/andenick/measuring-the-wealth-of-nations) replication package was built entirely using this framework — 59 series, 85 scripts, 15 validators, 0 failures.

---

## The Single Source of Truth

Every output reads from `series_registry.json`:

```json
{
  "series": {
    "T506": {
      "name": "Rate of Exploitation (e = S*/V*)",
      "chapter": 5,
      "year_range": [1948, 2024],
      "units": "ratio",
      "content_type": "time_series",
      "subseries": {
        "T506-A": { "source": "Shaikh & Tonak 1994", "period": [1948, 1989] },
        "T506-EXT": { "source": "BLS CES + BEA NIPA", "period": [1990, 2024] }
      },
      "construction": [
        { "step": 1, "op": "load", "subseries": ["T506-A"] },
        { "step": 2, "op": "derive", "formula": "e = S*/V*" },
        { "step": 3, "op": "splice", "at_year": 1989 }
      ],
      "validation": {
        "reference_values": { "1948": 1.70, "1989": 2.44 }
      }
    }
  }
}
```

See `schemas/series_registry_schema.json` for the full schema.

---

## Mandatory Rules

The framework enforces [8 mandatory rules](specs/00_rules.md):

1. **Single Source of Truth** — `series_registry.json` governs all outputs
2. **No Synthetic Data** — every value traces to a real source
3. **No Proxies** — use the exact series, not a substitute
4. **No Lazy Splices** — extend formulas by extending components
5. **Unit Documentation** — every series declares its units
6. **Content Type Classification** — only time series get extensions
7. **Source Material Reading** — reviews must check the original text
8. **Pipeline Ordering** — skills execute in dependency order

These rules exist because their absence caused specific, documented failures in real projects.

---

## Repository Structure

```
anu-data-framework/
  specs/           14 protocol specifications (Markdown)
  schemas/         JSON schemas for the series registry
  examples/        Sample registry and data files
  docs/            Architecture and data flow documentation
  README.md        This file
  CLAUDE.md        Quick orientation for AI code reviewers
  LICENSE          MIT
```

---

## Named After

Anwar Shaikh's meticulous approach to empirical data construction — the gold standard for scholarly economic research. The framework was developed while replicating Shaikh & Tonak's *Measuring the Wealth of Nations* (1994) and Shaikh's *Capitalism: Competition, Conflict, Crises* (2016).
