# Anu Review — 12-Dimension Quality Audit

## Purpose

A systematic audit framework for reviewing the quality of an Anu Suite data construction project across all 10 functional skills.

## The 12 Dimensions

| # | Dimension | Skill Audited | Max Score |
|---|-----------|---------------|-----------|
| D1 | Adequacy | Anu Adequacy | 5 |
| D2 | Research Coverage | Anu Research | 10 |
| D3 | Ingestion Quality | Anu Ingestion | 10 |
| D4 | Extension Faithfulness | Anu Extension | 10 |
| D5 | Replicator Scripts | Anu Replicator | 10 |
| D6 | Chopped Validation | Anu Chopped | 5 |
| D7 | Extenbook Completeness | Anu Extenbook | 5 |
| D8 | Visualization Quality | Anu Shiny | 10 |
| D9 | Variant Tracking | Anu Variant | 5 |
| D10 | Ledger Coverage | Anu Ledger | 5 |
| D11 | Documentation Quality | (cross-cutting) | 10 |
| D12 | Pipeline Integrity | Anu Pipeline | 5 |
| | **Total** | | **90** |

## Grading

| Score | Grade | Meaning |
|-------|-------|---------|
| 81-90 | A | Exemplary — ready for publication |
| 70-80 | B | Good — minor improvements needed |
| 55-69 | C | Acceptable — several gaps to address |
| 40-54 | D | Needs significant work |
| < 40 | F | Failed — fundamental issues |

## Per-Dimension Audit

### D1: Adequacy (5 pts)
- Adequacy report exists for the chapter (1 pt)
- All 5 layers (KB, Series, Data Sources, Construction, Validation) covered (2 pts)
- Adequacy gates passed before pipeline execution (2 pts)

### D2: Research Coverage (10 pts)
- Research file exists for every series (2 pts)
- Methodology summary present for every series (2 pts)
- All sources cited (2 pts)
- Cross-references documented (2 pts)
- Confidence levels set correctly (2 pts)

### D3: Ingestion Quality (10 pts)
- Registry validates against schema (3 pts)
- All series have subsources defined (2 pts)
- All extensions configured (2 pts)
- Reference values exist for validation (2 pts)
- DPRs written for every series (1 pt)

### D4: Extension Faithfulness (10 pts)
- Splice methodology documented (2 pts)
- Reindexing applied correctly at splice (3 pts)
- Both -EXT and -F columns in chopped CSV (2 pts)
- EPR written (2 pts)
- Validation passes (1 pt)

### D5: Replicator Scripts (10 pts)
- L## scripts exist for every series (2 pts)
- P## scripts exist for every series (2 pts)
- V## validation scripts pass (2 pts)
- M## adjustments documented (1 pt)
- Master orchestrator runs end-to-end (3 pts)

### D6: Chopped Validation (5 pts)
- Chopped CSV exists for every series (2 pts)
- Row 1 metadata is well-formed (1 pt)
- Subsource metadata JSON generated (1 pt)
- Validation passes (1 pt)

### D7: Extenbook Completeness (5 pts)
- Extenbook exists for every series (1 pt)
- All 4 sheets present (2 pts)
- Cross-references correct (1 pt)
- Construction validation PASS (1 pt)

### D8: Visualization Quality (10 pts)
- All series accessible in app (2 pts)
- Multi-source traces visible (2 pts)
- Extension traces visually distinct (1 pt)
- Methodology panel populates (2 pts)
- Provenance panel populates (2 pts)
- App handles missing data gracefully (1 pt)

### D9: Variant Tracking (5 pts)
- Variants documented when methodology alternatives exist (2 pts)
- Canonical IDs assigned (2 pts)
- Variant rationale documented (1 pt)

### D10: Ledger Coverage (5 pts)
- Ledger exists (1 pt)
- All artifacts tracked (2 pts)
- Coverage matrix complete (2 pts)

### D11: Documentation Quality (10 pts)
- README explains project clearly (2 pts)
- Methodology documented (2 pts)
- Decision log maintained (2 pts)
- Version history present (2 pts)
- Citations and licenses correct (2 pts)

### D12: Pipeline Integrity (5 pts)
- All stages execute in correct order (2 pts)
- Hash audit trail present (2 pts)
- No circular dependencies (1 pt)

## Audit Workflow

```bash
# Run full audit
anu-review project_name

# Per-dimension drill-down
anu-review project_name --dimension D4

# Generate report
anu-review project_name --output AUDIT_REPORT.md
```

## Output: AUDIT_REPORT.md

```markdown
# Anu Suite Audit Report

**Project**: Capitalism Data
**Audit Date**: 2026-04-26
**Auditor**: Anu Review v2.0

## Overall Score: 78/90 (B)

## Dimension Scores
- D1 Adequacy: 5/5
- D2 Research Coverage: 9/10 (1 series missing methodology summary)
- D3 Ingestion Quality: 10/10
- D4 Extension Faithfulness: 9/10 (S037 missing EPR)
- ...

## Issues to Address
1. S013 missing methodology_summary in research file
2. S037 missing EPR
3. Shiny app does not render extension traces for chapter 6

## Strengths
- Hash audit trail complete
- All 105 series have valid registry entries
- Validation reports show 100% pass rate
```
