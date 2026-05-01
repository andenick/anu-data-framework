# Anu Extenbook — Human-Readable 4-Sheet Excel Workbook

## Purpose

For each data series, produce a single Excel workbook with four sheets that make the series fully understandable to a human reviewer. The Extenbook is the human counterpart to the Chopped CSV.

## The Four Sheets

### Sheet 1: Data
The actual time series data, with subsources as columns:

| Year | S001-A (BEA 1860-1932) | S001-B (FRED 1933-2025) | S001-EXT (FRED 2011-2025) | S001-F (Reindexed) |
|------|-------|-------|-------|-------|
| 1860 | 2.1 | | | |
| 1861 | 2.2 | | | |
| ... | ... | ... | ... | ... |

Plus: column headers explain what each column is. Cell formatting indicates extension columns visually.

### Sheet 2: Provenance
For every observation, full source citation:

| Subsource | Period | Source Citation | Access Date | URL |
|-----------|--------|-----------------|-------------|-----|
| S001-A | 1860-1932 | BEA, *Long Term Economic Growth, 1860-1970*, Table A-15 | 2026-03-15 | (offline) |
| S001-B | 1933-2025 | FRED INDPRO | 2026-04-01 | https://fred.stlouisfed.org/series/INDPRO |
| ... | ... | ... | ... | ... |

### Sheet 3: Research
Quotes and methodology notes from the original author, extracted by Anu Research:

| Entry ID | Type | Source Location | Quote / Note |
|----------|------|-----------------|--------------|
| R001 | methodology_description | Appendix 2, p.843 | "The industrial production index is constructed from..." |
| R002 | source_citation | Appendix 2, footnote 12 | BEA 1966 Table A-15 |
| ... | ... | ... | ... |

This sheet makes the methodology *legible*. A reviewer can see exactly what the original author said about how the series was built.

### Sheet 4: Construction
The recipe for reconstructing the series from raw data:

```
Step 1: Load S001-A from BEA LTEG Table A-15 (data/user-inputs/bea_lteg.csv)
Step 2: Load S001-B from FRED INDPRO via API
Step 3: Reindex S001-A from 1899=100 to 2017=100 base
        Formula: S001-A_reindexed[t] = S001-A[t] * (S001-B[1933] / S001-A[1933])
Step 4: Splice S001-A and S001-B at 1933
Step 5: Validate against reference value: 1929 = 25.4 (book Fig 2.1)
```

Plus: Construction validation status (PASS/FAIL on reference values).

## Why This Format?

The four sheets correspond to the four questions a reviewer asks:

1. **What is the data?** → Data sheet
2. **Where did it come from?** → Provenance sheet
3. **What did the original author say?** → Research sheet
4. **How was it built?** → Construction sheet

Together they make a series transparent. A reader can verify methodology against the source, recompute values, and identify potential issues.

## Generation

```python
from anu_extenbook import write_extenbook

write_extenbook(
    series_id="S001",
    data_dict={...},                  # subseries data
    research=research_dict,           # from Anu Research output
    registry=registry,                # from series_registry.json
    output_path="data/final-data/extenbooks/S001.xlsx"
)
```

The writer auto-populates all four sheets from the registry and research files.

## File Naming

```
S001_industrial_production.xlsx
S037_profit_rate.xlsx
S047_corporate_profits.xlsx
```

Format: `S{NNN}_{snake_case_series_name}.xlsx`

## Quality Checklist

- [ ] All four sheets present
- [ ] Data sheet has every subsource as a column
- [ ] Provenance sheet has every source cited
- [ ] Research sheet has all entries from `S###_research.json`
- [ ] Construction sheet has every step documented
- [ ] Reference values validate (PASS in Construction sheet footer)
