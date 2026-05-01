# Anu Adequacy — Statistical Adequacy Testing Framework

## Purpose

A pre-flight check that gates the Anu pipeline. Before any agent invests effort in research, ingestion, or processing, the Adequacy skill verifies that sufficient material exists to construct the project rigorously.

This is **Stage 0** of the pipeline. Failed adequacy blocks all downstream work.

## The 5 Adequacy Layers

| Layer | What's Checked | Pass Criterion |
|-------|----------------|----------------|
| **L1: Knowledge Base** | KB pages, methodology text, footnotes | ≥ 1 KB file per chapter, methodology section present |
| **L2: Series Inventory** | Identified series, naming consistency | All figures mapped to series IDs |
| **L3: Data Sources** | Primary sources accessible | At least 1 verified source per series |
| **L4: Construction Logic** | Methodology reconstructable | Construction steps derivable from KB |
| **L5: Validation** | Reference values exist | At least 1 reference value per series |

## Adequacy Ratings

| Rating | Meaning |
|--------|---------|
| EXEMPLARY | All 5 layers pass with high confidence |
| ADEQUATE | All 5 layers pass; some have low confidence |
| INSUFFICIENT | One or more layers fail; project can proceed with documented gaps |
| BLOCKED | Critical layer fails; project cannot proceed |

## Adequacy Workflow

### Step 1: Run Per-Layer Audit
For each layer, verify the criterion is met. Document findings in `Technical/docs/chapters/CH{N}_ADEQUACY_REPORT.json`.

### Step 2: Compute Overall Adequacy
```
overall_adequacy = min(layer_adequacy for layer in [L1, L2, L3, L4, L5])
```

The chain is only as strong as its weakest link.

### Step 3: Document Gaps
For any layer that doesn't pass:
- What is missing?
- Can it be remediated?
- What is the remediation cost?
- Should we proceed with documented gaps or stop?

### Step 4: Gate Decision
- EXEMPLARY or ADEQUATE → proceed to Stage 1 Research
- INSUFFICIENT → require user/researcher decision
- BLOCKED → halt pipeline; remediate or descope

## Output Format: CH{N}_ADEQUACY_REPORT.json

```json
{
  "chapter": 2,
  "audit_date": "2026-03-15",
  "auditor": "agent",

  "layers": {
    "L1_knowledge_base": {
      "status": "ADEQUATE",
      "kb_files_found": ["ch02_main.md", "ch18_appendices.md"],
      "methodology_sections": 3,
      "footnotes_extracted": 47,
      "confidence": "HIGH"
    },
    "L2_series_inventory": {
      "status": "ADEQUATE",
      "series_identified": ["S001", "S002", ...],
      "figures_mapped": 12,
      "confidence": "HIGH"
    },
    "L3_data_sources": {
      "status": "ADEQUATE",
      "primary_sources": ["BEA LTEG 1966", "FRED INDPRO"],
      "api_accessibility": {"FRED": "OK", "BEA": "OK"},
      "confidence": "HIGH"
    },
    "L4_construction_logic": {
      "status": "ADEQUATE",
      "construction_steps_derivable": 12,
      "ambiguous_steps": 1,
      "confidence": "MEDIUM"
    },
    "L5_validation": {
      "status": "ADEQUATE",
      "reference_values_per_series": {"S001": 3, "S002": 2, ...},
      "confidence": "HIGH"
    }
  },

  "overall_adequacy": "ADEQUATE",
  "ready_for_pipeline": true,

  "documented_gaps": [
    "S003 has only one reference value (acceptable but minimal)",
    "L4 has 1 ambiguous construction step requiring researcher decision"
  ],

  "remediation_actions": []
}
```

## Why a Pre-Flight Check?

Without adequacy gating, agents proceed to research and construction on incomplete material, producing low-quality output that has to be redone. The Adequacy skill catches this early — typically within the first hour of project setup.

This is analogous to **fail-fast** in software engineering: discover the project is non-viable as early as possible, not after weeks of work.

## Adequacy Refs

The Adequacy report becomes a reference for downstream skills. Research entries can include `adequacy_refs` linking to specific KB files identified at L1:

```json
"adequacy_refs": {
  "L1_kb_pages": ["page_120_methodology.md", "page_310_table_E2.csv"],
  "L3_data_sources": ["BEA NIPA Table 6.2D", "BEA LTEG Table A-15"],
  "L5_validation": ["Mohun_2013_exploitation_rates.csv"]
}
```

This creates a traceable chain from final outputs all the way back to the adequacy-verified source materials.

## Quality Checklist

- [ ] Adequacy report exists for every chapter/module
- [ ] All 5 layers audited
- [ ] Overall adequacy ≥ ADEQUATE
- [ ] Documented gaps have remediation plans (or are accepted)
- [ ] Pipeline did not start before Stage 0 completion
