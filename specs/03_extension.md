# Anu Extension — Faithful API Data Extension

## Purpose

Maximum Faithfulness Data Extension Framework. When extending an existing data series with modern API data, the extension must preserve the original construction methodology exactly.

## Core Principle

**Faithfulness over convenience.** A naive extension simply concatenates new API data to old historical data. A faithful extension reindexes the new data to match the original methodology at the splice point.

## The 10 Principles

1. **Use only public APIs** for extensions. Internal databases create reproducibility problems.
2. **Document the splice year explicitly** in `series_registry.json`.
3. **Reindex at the splice**, do not just concatenate. Formula:
   `S###-F[t] = prev_subsource[splice_year] * (API[t] / API[splice_year])`
4. **Preserve original units**. If the original was Index 1899=100, extension must reindex.
5. **Mark extensions clearly** with `is_extension: true` in registry.
6. **Track API vintage** (when data was fetched).
7. **Write Extension Provenance Report (EPR)** for each extended series.
8. **Validate splice quality**: continuity at splice point should be smooth.
9. **Test against reference values** from the original published source.
10. **Document every methodology decision** in DECISION_LOG.md.

## Workflow

### Step 1: Identify Splice Point
The splice year is where the old methodology ends and the new API data begins. Often determined by:
- Last year of original published data
- Methodology revision in the source agency
- Definitional change in the underlying classification

### Step 2: Choose Extension Method
| Method | When to Use |
|--------|-------------|
| `reindex_at_splice` | API data has different base year than original |
| `concatenate` | API data is identical methodology, just extended |
| `weighted_average` | Multiple competing API sources |
| `manual` | Cannot be automated; document M## adjustment |

### Step 3: Implement in P## Script
The processing script computes the extension:
```python
# Pseudocode
splice_year = registry["S001"]["extension"]["splice_year"]
ratio = original_data[splice_year] / api_data[splice_year]
extended = api_data[splice_year:] * ratio
```

### Step 4: Add Extension Columns
Both `S###-EXT` (raw API data) and `S###-F` (re-indexed) must be in the data_dict and Chopped CSV.

### Step 5: Validate
Check:
- Continuity at splice point (year-over-year change reasonable)
- Reference values match published values
- Extension overlap correlation > 0.95

### Step 6: Write EPR
Extension Provenance Report documents:
- Original methodology summary
- API source and access date
- Splice method and formula
- Validation results
- Decisions and assumptions

## Quality Checklist

- [ ] Splice year documented in registry
- [ ] Extension method documented
- [ ] Both raw and reindexed columns present in chopped CSV
- [ ] Continuity at splice within tolerance (typically <5% YoY change)
- [ ] Reference values pass at original endpoints
- [ ] EPR written and reviewed
