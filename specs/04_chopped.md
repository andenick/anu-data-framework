# Anu Chopped — Machine-Readable CSV Format

## Purpose

A standardized machine-readable CSV format for empirical data series. Optimized for downstream consumption by visualization apps, replication scripts, and analytical pipelines.

## Format Specification

| Row | Content |
|-----|---------|
| 1 | Metadata (source, units, period, methodology summary) |
| 2 | Column IDs (S001-A, S001-B, S001-EXT, S001-F, etc.) |
| 3+ | Data rows (Year, value, value, ...) |

### Row 1: Metadata
Pipe-delimited fields for each column:
```
Year|S001-A: BEA LTEG 1966 [1860-1932] Index 1899=100|S001-B: FRED INDPRO [1933-2025] Index 2017=100|...
```

### Row 2: Column IDs
Clean column identifiers for programmatic use:
```
year,S001-A,S001-B,S001-EXT,S001-F
```

### Row 3+: Data
Standard CSV data rows:
```
1860,2.1,,,
1861,2.2,,,
...
1932,18.7,,,
1933,,17.1,,18.5
1934,,18.3,,
...
2025,,108.2,108.2,
```

## Why This Format?

1. **Self-documenting**: Row 1 metadata travels with the data; no separate schema file needed
2. **Programmatic-friendly**: Row 2 column IDs are valid Python/R identifiers
3. **Multi-source visible**: Each subsource gets its own column; readers see methodology boundaries
4. **Extension-aware**: Extension columns (`-EXT`, `-F`) are explicit, not hidden

## Generation

The Chopped writer auto-generates Row 1 metadata from `series_registry.json`:

```python
from anu_chopped import write_chopped

write_chopped(
    series_id="S001",
    data_dict={
        "S001-A": pd.Series(...),
        "S001-B": pd.Series(...),
        "S001-EXT": pd.Series(...),
        "S001-F": pd.Series(...),
    },
    registry=registry,
    output_path="data/final-data/chopped/S001.csv"
)
```

## Subsource Metadata Generation

Alongside each Chopped CSV, the writer generates `SUBSOURCE_METADATA.json`:

```json
{
  "S001": {
    "subseries": {
      "S001-A": {
        "viz_label": "Industrial Production (Historical)",
        "role": "primary_historical",
        "period": "1860-1932",
        "units": "Index 1899=100",
        "is_extension": false
      },
      "S001-EXT": {
        "viz_label": "Industrial Production (Extension)",
        "role": "extension",
        "period": "2011-2025",
        "is_extension": true
      }
    }
  }
}
```

This metadata feeds the Shiny app's series catalog.

## Validation

A valid Chopped CSV must:
- [ ] Row 1 has metadata for every data column
- [ ] Row 2 has column IDs matching the registry
- [ ] Year column is monotonically increasing
- [ ] No NaN in the year column
- [ ] All numeric columns parse as floats

Run validation with:
```python
from anu_chopped import validate_chopped
result = validate_chopped("data/final-data/chopped/S001.csv")
```

## Concurrent Series (CS) Columns

For ratio/rate series, the Chopped writer also includes Concurrent Series components:

```
year,S026-A,S026-B,CS026-N,CS026-D
```

Where:
- `S026-A`, `S026-B`: The ratio subsources
- `CS026-N`: Numerator component (e.g., corporate profits)
- `CS026-D`: Denominator component (e.g., capital stock)

CS columns are filtered out in standard views and shown only when "Show Components" is enabled in the Shiny app.
