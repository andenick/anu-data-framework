# Anu Pipeline — Multi-Agent Orchestrator

## Purpose

A 10-stage orchestrator that sequences the Anu Suite skills into a complete data construction workflow. Tracks pipeline state, manages agent handoffs, and enforces prerequisite ordering.

## The 10 Stages

| Stage | Skill | Output |
|-------|-------|--------|
| 0 | Adequacy | `CH{N}_ADEQUACY_REPORT.json` |
| 1 | Research | `S###_research.json` per series |
| 2 | Ingestion | `series_registry.json`, decompositions, DPRs |
| 3 | Extension | EPRs, extension methodology |
| 4 | Replicator | L##/P##/V##/M## scripts |
| 5 | Chopped + Extenbook | CSVs and Excel workbooks |
| 5b | Visualization Export | Chapter CSVs, catalog, SUBSOURCE_METADATA |
| 6 | Variant | Methodology variant tracking |
| 7 | Ledger | Coverage tracking |
| 8 | Review | 12-dimension audit |
| 9 | Pipeline Validation | End-to-end checks |

## Prerequisite Graph

```
Stage 0 (Adequacy) ─┬─ blocks all subsequent stages
                    │
Stage 1 (Research) ─┴─ required for Stage 2, 3, 5
                    │
Stage 2 (Ingestion) ── produces registry, required for all data stages
                    │
Stage 3 (Extension) ── required for series with extensions
                    │
Stage 4 (Replicator) ── produces all data, required for Stage 5+
                    │
Stages 5, 5b ───────── output generation, parallelizable
                    │
Stages 6, 7 ────────── tracking, can run anytime after Stage 4
                    │
Stage 8 (Review) ───── final audit, requires all prior stages
                    │
Stage 9 (Validation) ─ end-to-end pipeline integrity check
```

## Pipeline State

The orchestrator maintains `PIPELINE_STATE.json`:

```json
{
  "project_name": "Capitalism Data",
  "current_stage": 4,
  "stages": {
    "0_adequacy": {"status": "COMPLETE", "completion_date": "2026-03-15"},
    "1_research": {"status": "COMPLETE", "completion_date": "2026-03-20"},
    "2_ingestion": {"status": "COMPLETE", "completion_date": "2026-03-25"},
    "3_extension": {"status": "COMPLETE", "completion_date": "2026-04-01"},
    "4_replicator": {"status": "IN_PROGRESS"},
    "5_chopped_extenbook": {"status": "PENDING"},
    ...
  },
  "blocking_issues": []
}
```

## Agent Handoff Pattern

Each stage produces deliverables consumed by the next stage. Agents check upstream deliverables before starting:

```python
# Pseudocode for Stage 3 (Extension) agent
prerequisites = ["S###_research.json (Stage 1)", "series_registry.json (Stage 2)"]
for prereq in prerequisites:
    if not exists(prereq):
        raise PrerequisiteError(f"Missing: {prereq}")

# Proceed with extension work
```

## Commands

```bash
# Start a new project at Stage 0
anu-pipeline init project_name

# Advance one stage
anu-pipeline next

# Run a specific stage
anu-pipeline run --stage 4

# Check current status
anu-pipeline status

# Validate dependencies before advancing
anu-pipeline validate
```

## Pipeline State Transitions

```
PENDING → IN_PROGRESS → COMPLETE
                    \
                     → BLOCKED → (resolve issue) → IN_PROGRESS
                    \
                     → FAILED → (debug) → IN_PROGRESS
```

State transitions are tracked with timestamps and (optional) agent identifiers.

## Why a Pipeline?

Without explicit pipeline orchestration, agent-driven data projects fail in two ways:

1. **Skipped stages**: Agent jumps to extension before research is complete; methodology is invented rather than derived from sources.
2. **Hidden dependencies**: A change in registry doesn't propagate to chopped CSVs because no one knew the dependency existed.

The pipeline makes the dependency graph explicit, gates progression on prerequisite completion, and produces a single state file that any agent (or human) can read to understand current status.

## Quality Checklist

- [ ] PIPELINE_STATE.json exists and is current
- [ ] Stage 0 Adequacy was run before any other stage
- [ ] No stages started before prerequisites complete
- [ ] All stage transitions are timestamped
- [ ] Blocking issues are documented when present
- [ ] Stage 8 Review run before declaring project complete
