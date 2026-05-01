# Anu Suite Architecture

## The Core Insight

Empirical economic research data is built by humans, used by humans, but increasingly *constructed* by AI agents. The Anu Suite is a framework for that construction process where:

1. The agent writes the data construction code
2. The code can be run by anyone (no agent required)
3. Every artifact is auditable back to a primary source
4. The protocols governing agent behavior are themselves human-readable specifications

## The 12 Skills as a Pipeline

The 12 skills compose into a 10-stage pipeline:

| Stage | Skill(s) | Output |
|-------|----------|--------|
| 0 | anu-adequacy | Statistical adequacy report (gates the pipeline) |
| 1 | anu-research | Per-series research dossiers |
| 2 | anu-ingestion | series_registry.json, decompositions, DPRs |
| 3 | anu-extension | Extension methodology, EPRs |
| 4 | anu-replicator | L##/P##/V##/M## scripts, replication package |
| 5 | anu-chopped | Validated machine-readable CSVs |
| 5 | anu-extenbook | 4-sheet Excel workbooks |
| 5b | anu-shiny | Visualization export (chapter CSVs, catalog) |
| 6 | anu-variant | Methodology variant tracking |
| 7 | anu-ledger | Coverage tracking |
| 8 | anu-review | 12-dimension quality audit |
| 9 | anu-pipeline | Master orchestrator |

## Why a Single Source of Truth Matters

The `series_registry.json` is not just configuration — it's the contract between agent and human. Every output reads from it:

- **Chopped CSVs**: Row 1 metadata auto-generated from registry source descriptions
- **Extenbooks**: Provenance sheet auto-populated from registry
- **Shiny app**: viz_column aliases drive chart labels
- **Validation scripts**: Reference values pulled from registry
- **Replicator orchestrator**: Determines processing order and dependencies

If the registry is correct, all outputs are consistent. If the registry has a typo, the typo propagates uniformly (and is therefore detectable). There is no "did the Shiny app and the Extenbook diverge?" problem because they read from the same source.

## The Replicator Pattern

The Anu Replicator (skill 6) is the core "agent constructs, human verifies" pattern.

### What the agent does:
1. Read source materials (knowledge base, appendices)
2. Run Anu Research to build research dossiers
3. Run Anu Ingestion to populate series_registry.json
4. Run Anu Extension to plan API-based extensions
5. Generate L## (loading) scripts that fetch raw data
6. Generate P## (processing) scripts that construct series
7. Generate V## (validation) scripts that verify outputs
8. Generate M## (manual adjustment) scripts where needed

### What the human does:
1. Review the registry for correctness
2. Set API keys in `api_keys.env`
3. Run `python replicate.py`
4. Inspect VALIDATION_REPORT.json
5. (Optional) Modify and re-run

The agent is not in the loop during step 3-5. The package runs as plain Python.

## Hash-Based Reproducibility

Every input and output gets a SHA-256 hash:

```python
# In LOAD_LOG.json
{
  "series_id": "S001",
  "source_file": "BEA_LTEG_1966_TableA15.csv",
  "source_hash": "a7f3...",
  "output_file": "data/raw-data/parsed/S001_parsed.csv",
  "output_hash": "b9e2..."
}
```

Reproducibility is not an aspiration — it's a measurable invariant. Two runs with the same inputs produce identical hashes. Any divergence is detectable and traceable.

## The "Why Not Just Code" Question

A reasonable question: why protocols-as-markdown instead of protocols-as-Python-libraries?

**Answer**: Because the consumer of the protocol is an AI agent, not a human programmer. Markdown is the agent's native input format. Embedding the protocol in code would require:

1. The agent to read the code (slower, less context-efficient)
2. The agent to understand the implementation, not the contract (more error-prone)
3. Versioning the implementation alongside the contract (more brittle)

By keeping protocols as markdown specifications, we get:

1. Direct loadability into agent context
2. Human-readable contracts that match what the agent reads
3. Independent versioning of contract vs implementation
4. The ability to apply the same protocol across different language implementations (Python, R, Stata)

## What This Generalizes To

The Anu Suite was designed for empirical economic research, but the core patterns generalize to any AI-driven data construction:

- **Skills as protocol specifications** → Modular agent task definitions
- **Single source of truth** → Configuration-driven outputs
- **Hash audit trails** → Tamper-evident pipelines
- **Agent constructs, human verifies** → Reproducible AI assistance
- **Phase-gated execution** → Validation as a first-class concern

These patterns apply equally to climate data construction, regulatory filings analysis, biomedical literature mining, or any other domain where AI-constructed data outputs must be auditable.
