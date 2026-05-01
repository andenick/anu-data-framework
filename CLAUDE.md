# Anu Suite — For Claude Code Reviewers

## What This Is

The Anu Suite is a 12-skill framework for AI agents to construct empirical research data with full reproducibility, provenance tracking, and human-verifiability.

**Key insight**: The skills are markdown specifications (in `specs/`), not compiled libraries. Agent behavior is governed by readable, auditable text.

## Quick Orientation

The interesting thinking is in:

- **`specs/06_replicator.md`** — Start here. The 4-phase replication architecture (Loading → Processing → Validation → Manual Adjustment) is the core pattern. Demonstrates "agent constructs, human verifies."
- **`specs/01_research.md`** — Shows how agents build per-series dossiers from primary sources before any code is written. Solves the "AI hallucinates citations" problem through structured grounding.
- **`schemas/series_registry_schema.json`** — The single source of truth. Every output derives from this.

## Try It

The Anu Suite is meant to be loaded as agent instructions:

```bash
# Have an agent read the spec and apply it
claude "Read specs/02_ingestion.md and apply this protocol to construct \
        a series_registry.json for the Maddison world GDP database"
```

Or use it as a reference for building your own agent-driven data project.

## Why This Matters

Three patterns that generalize beyond economic data:

1. **Skills as protocol specifications**: Markdown defines what the agent does, not Python. This is auditable.
2. **Registry-driven outputs**: A single JSON file is the source of truth. Every artifact derives from it. No drift.
3. **Hash audit trails**: Every input and output is hashed. Reproducibility is not a goal — it's a measurable invariant.

## Context

One of four repositories demonstrating AI-driven economic research:
1. **[hdarp](https://github.com/andenick/hdarp)** — Input layer
2. **anu-suite** (this repo) — Protocol layer
3. **[nickydata](https://github.com/andenick/nickydata)** — Reproducibility layer
4. **[capitalism-data](https://github.com/andenick/capitalism-data)** — Demonstration at scale
