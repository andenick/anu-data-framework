# Anu Shiny — Interactive Visualization Standard

## Purpose

A standard for building registry-driven interactive data dashboards using R Shiny + Plotly. Designed so the same registry that drives Chopped CSVs and Extenbooks also drives the visualization app — with no manual configuration drift.

## Core Features

### Multi-Source Charts
Every series is plotted with multiple traces — one per subsource, plus extension traces:

```
Industrial Production Index
─────────────────────────────────────────
●  S001-A (BEA LTEG, 1860-1932)
●  S001-B (FRED INDPRO, 1933-2010)
●  S001-EXT (Extension, 2011-2025)  [dashed line]
```

Users see methodology boundaries directly. A splice in 1933 is visible because two different colored traces meet there.

### Methodology Panel
Click a series, see the methodology:

```
─────────────────────────────────────────
Series: S001 — US Industrial Production
─────────────────────────────────────────
Methodology summary (from Anu Research):
"The series is constructed by splicing two
industrial production indices..."

Sources:
- BEA Long Term Economic Growth 1966 (1860-1932)
- FRED INDPRO (1933-2025)

Splice: 1933 (reindexed to 2017=100 base)
─────────────────────────────────────────
```

### Author Quotes
The Research sheet from Extenbooks is loaded into the Shiny app as a "Notes" panel. Users can read what the original author said about each series.

### Extension Visibility
Extension data uses different visual encoding (typically dashed line) so users know which observations are post-original-publication and from public APIs.

### Dual-Axis Support
For series with concurrent components (CS columns), users can toggle "Show Components" to see the numerator and denominator alongside the ratio.

## Architecture

```
                       series_registry.json
                              │
                              ▼
┌─────────────────────────────────────────────────────┐
│  Shiny App (R + Plotly)                              │
│                                                       │
│  global.R                                             │
│    Loads:                                             │
│      - series_registry.json                          │
│      - SUBSOURCE_METADATA.json (per-column metadata)  │
│      - chopped CSVs (one per chapter)                │
│      - research JSONs (for methodology panel)         │
│                                                       │
│  app.R                                                │
│    UI:                                                │
│      - sidebar: chapter/series selector              │
│      - main: Plotly chart with all subsources        │
│      - tabs: Methodology | Provenance | Research     │
│                                                       │
│  Server:                                              │
│      - reactive: filter by selected series            │
│      - renderPlotly: build multi-source chart        │
│      - render UI panels from registry data            │
└─────────────────────────────────────────────────────┘
```

## L00 / P00 Data Pipeline

The Shiny app expects clean inputs from the Anu Replicator's visualization export phase:

```
data/final-data/shiny/
  ├── chapter_01.csv     # Wide CSV: Year + aliased columns
  ├── chapter_02.csv
  ├── ...
  ├── series_catalog.json
  └── SUBSOURCE_METADATA.json
```

No manual data preparation. The Shiny app reads these files and renders.

## Column Aliasing

The registry's `viz_column` field controls chart labels:

```json
"S026": {
  "subseries": {
    "S026-A": {
      "viz_column": "rcorp",
      "is_component": false
    },
    "CS026-N": {
      "viz_column": "Pcorp",
      "is_component": true
    }
  }
}
```

The Shiny app reads viz_column as the trace name. Components (`is_component: true`) are filtered out by default and shown only in "Show Components" mode.

## Implementation Pattern

```r
# global.R
registry <- jsonlite::fromJSON("data/series_registry.json")
metadata <- jsonlite::fromJSON("data/SUBSOURCE_METADATA.json")
chapter_data <- lapply(list.files("data/", pattern = "chapter_.*\\.csv"), read.csv)

# app.R
ui <- fluidPage(
  selectInput("series_id", "Series:", choices = names(registry$series)),
  plotlyOutput("chart"),
  tabsetPanel(
    tabPanel("Methodology", textOutput("methodology")),
    tabPanel("Provenance", DT::dataTableOutput("provenance")),
    tabPanel("Research", DT::dataTableOutput("research"))
  )
)

server <- function(input, output) {
  output$chart <- renderPlotly({
    series <- registry$series[[input$series_id]]
    # Build multi-source plotly chart from subseries
    ...
  })
}
```

## Quality Checklist

- [ ] All series in registry are accessible from app
- [ ] Multi-source traces are visible for series with multiple subsources
- [ ] Extension traces are visually distinct (e.g., dashed)
- [ ] Methodology panel populates from research files
- [ ] Provenance panel shows all sources from registry
- [ ] App handles missing data gracefully (no broken charts)
- [ ] Mobile-responsive layout (or at minimum, gracefully degrades)
