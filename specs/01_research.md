# Anu Research — Per-Series Research Dossier Protocol

## Purpose

Before constructing or extending any data series, an AI agent must deeply understand what the original author did. This skill produces a structured `S###_research.json` file that captures every quote, reference, footnote, methodology description, and piece of information relevant to constructing the series.

This research dossier is the **foundation** for all downstream outputs. Without it, the agent is reasoning from incomplete information.

## When to Use

Use Anu Research **before** any of the following:
- Creating a series decomposition
- Building loading or processing scripts
- Writing data provenance reports (DPRs) or extension provenance reports (EPRs)
- Populating the `series_registry.json`

## Workflow

### Step 1: Identify the Series
Determine the series ID, chapter, and associated figures. Check the `series_registry.json` if it exists, or the source CSV.

### Step 2: Search Knowledge Base
For the given series, systematically search:

1. **Appendix methodology text** — The primary source. Search for the series name, figure number, and data source names. Extract every sentence describing how the data was constructed.
2. **Chapter body text** — Search for figure references, data discussions, interpretive context.
3. **Footnotes and endnotes** — Source citations, data access dates, methodological notes.
4. **Other chapters** — Cross-references where this series or its sources appear elsewhere.
5. **Bibliography** — Full citations for all sources mentioned.

### Step 3: Classify Each Finding

| Type | Description |
|------|-------------|
| `methodology_description` | How the series was constructed |
| `source_citation` | Citation of an original data source |
| `figure_context` | How the series appears in a figure or narrative |
| `data_caveat` | Known limitations, adjustments, or warnings |
| `reindexing_note` | Why/how a base year change was made |
| `splicing_note` | How two series were combined |
| `units_note` | Unit definitions, conversions, or clarifications |
| `cross_reference` | Reference to another series sharing sources |
| `errata` | Known errors or corrections |

### Step 4: Record Confidence Level

| Confidence | Meaning |
|------------|---------|
| `exact_quote` | Verbatim text from the source |
| `paraphrase` | Faithful restatement of the source |
| `inference` | Deduced from context, not explicitly stated |

### Step 5: Compile Citations
For each original data source referenced, create a citation entry with citation_id, full_citation, short_cite, subseries (which subseries this source feeds), and type.

### Step 6: Write Methodology Summary
Synthesize all findings into a concise `methodology_summary` paragraph that explains the complete construction in plain language.

### Step 7: Write `S###_research.json`

## Output Format

```json
{
  "series_id": "S001",
  "series_name": "US Industrial Production Index",
  "chapter": 2,
  "figures": ["Fig2.1"],
  "research_date": "2026-04-26",
  "researcher": "agent",
  "kb_sources_searched": [
    "ch02_main_text.md",
    "ch18_appendices.md"
  ],
  "entries": [
    {
      "entry_id": "R001",
      "type": "methodology_description",
      "source_location": "Appendix 2, p. 843",
      "quote": "The industrial production index is constructed from...",
      "relevance": "primary_construction",
      "subseries_affected": ["S001-A", "S001-B"],
      "confidence": "exact_quote",
      "source_refs": ["SRC-BEA-LTEG", "SRC-FRB-INDPRO"],
      "kb_reference": "ch02_main_text.md#section-industrial-production"
    }
  ],
  "citations": [
    {
      "citation_id": "C001",
      "full_citation": "Bureau of Economic Analysis, Long Term Economic Growth, 1860-1970, Table A-15, p.185",
      "short_cite": "BEA LTEG 1966, TA15",
      "subseries": ["S001-A"],
      "type": "primary_source"
    }
  ],
  "methodology_summary": "The series is constructed by splicing two industrial production indices...",
  "known_issues": [],
  "cross_references": ["S007 (uses same BEA LTEG source for manufacturing)"]
}
```

## Why This Matters

A common failure mode in AI-assisted research is **hallucinated citations**: the agent confidently asserts that "according to Smith (2010), the data was constructed by..." when no such source exists. The Anu Research protocol prevents this by:

1. Forcing the agent to ground every claim in a specific KB location (page reference, section)
2. Requiring confidence labels (`exact_quote` vs `paraphrase` vs `inference`) so reviewers know what is verbatim and what is interpreted
3. Compiling citations as a separate, verifiable list

The output is a structured dossier that downstream skills consume programmatically — the methodology summary appears in Chopped CSV metadata, the citations populate the Extenbook Research sheet, the cross-references inform the Shiny app's series catalog.

## Quality Checklist

For each `S###_research.json`:
- [ ] At least one `methodology_description` entry exists
- [ ] All original data sources have corresponding `source_citation` entries
- [ ] `methodology_summary` accurately describes the full construction
- [ ] `citations` array has full bibliographic info for every source
- [ ] `cross_references` lists all related series
- [ ] `kb_sources_searched` lists all KB files consulted
- [ ] All reindexing operations have `reindexing_note` entries
- [ ] All splicing operations have `splicing_note` entries
- [ ] `confidence` is set correctly for every entry (prefer `exact_quote`)
