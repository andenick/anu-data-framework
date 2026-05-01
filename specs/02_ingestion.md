# Anu Ingestion — Knowledge Base, Decomposition, Provenance

## Purpose

Comprehensive data intake protocol covering Knowledge Base construction, raw data import, absorption into a definitive internal format, series decomposition, and provenance documentation.

This is the second skill in the Anu pipeline (after Research) and produces the central artifact of the entire suite: `series_registry.json`.

## When to Use

- Setting up a new data project
- Importing raw data sources
- Decomposing complex series into subsources
- Documenting provenance for academic or regulatory work

## Workflow

### Step 1: Knowledge Base Construction
Place HDARP-extracted content (text, tables, equations, figures) into a structured KB directory. This is the source material agents consult during construction.

### Step 2: Data Import
For each data source identified in research:
1. Identify primary source (book appendix, agency CSV, API endpoint)
2. Acquire raw data (download, parse, normalize)
3. Write to `data/user-inputs/[source]/`
4. Hash file with SHA-256

### Step 3: Series Decomposition
Decompose each series into subsources by period:
- Identify methodology transitions (e.g., 1947 NIPA boundary)
- Document each subsource with period, source, units, base year
- Record splice methodology (reindex, concatenate, weighted average)

Output: `Technical/decompositions/S###_DECOMPOSITION.md`

### Step 4: Registry Population
Populate `series_registry.json` with:
- Series metadata (name, chapter, figures)
- Subsource definitions
- Construction steps (load, reindex, splice, aggregate)
- Reference values for validation
- Extension configuration if applicable

### Step 5: Provenance Documentation (DPRs)
For each series, write a Data Provenance Report:
- Original sources cited
- Construction methodology
- Known issues and caveats
- Cross-references to related series

## Output Format: series_registry.json

The registry is the single source of truth for the entire project. See `schemas/series_registry_schema.json` for full specification.

```json
{
  "registry_version": "2.0.0",
  "project_name": "...",
  "sources": { "SRC-XXX": {...} },
  "series": { "S001": {...} }
}
```

## Quality Checklist

- [ ] Every series has at least one subsource defined
- [ ] Every subsource has period, source, units
- [ ] All extension methodologies are specified
- [ ] Reference values exist for validation
- [ ] DPRs written for every series
- [ ] All raw inputs hashed and tracked

## Integration

| Skill | Relationship |
|-------|--------------|
| Anu Research | Provides research dossiers consumed during ingestion |
| Anu Extension | Reads extension config from registry |
| Anu Replicator | Consumes registry as single source of truth |
| Anu Chopped | Generates Row 1 metadata from registry |
| Anu Extenbook | Pulls provenance from registry into Excel sheets |
