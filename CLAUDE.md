# Anu Data Framework — For Code Reviewers

## What This Is

A 14-skill protocol framework for AI agents to construct empirical research data with full reproducibility, provenance tracking, and human-verifiability. The skills are Markdown specifications (in `specs/`), not compiled code.

## Start Here

- **`specs/00_rules.md`** — The 8 mandatory invariants. Read this first to understand what the framework prohibits (synthetic data, proxies, lazy splices).
- **`specs/06_replicator.md`** — The 4-phase replication architecture (Loading → Processing → Validation → Manual Adjustment). This is the core pattern.
- **`specs/01_research.md`** — How agents build per-series methodology dossiers from primary sources before writing any code.
- **`schemas/series_registry_schema.json`** — The single source of truth schema.

## Key Insight

Three patterns that generalize beyond economic data:

1. **Skills as protocol specifications**: Markdown defines what the agent does, not Python. This is auditable and version-controlled.
2. **Registry-driven outputs**: A single JSON file is the source of truth. Every artifact derives from it. No drift between outputs.
3. **Hash audit trails**: Every input and output is hashed. Reproducibility is a measurable invariant, not an aspiration.

## See It Working

The [Measuring the Wealth of Nations](https://github.com/andenick/measuring-the-wealth-of-nations) replication package was built using this framework — 59 data series, 85 Python scripts, 15 automated validators, 0 failures.

## Try It

```bash
claude "Read specs/03_extension.md and extend this GDP series
        from 1990 to 2024 using FRED API data"
```

The specs work with Claude Code, Cursor, or any agent that can read Markdown instructions.
