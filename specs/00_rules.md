# Anu Data Framework — Mandatory Rules

These rules apply across all 14 skills. They are non-negotiable invariants that every data construction project must satisfy.

---

## 1. Single Source of Truth

`series_registry.json` is the canonical data registry. ALL output formats (CSVs, Excel workbooks, visualization dashboards, provenance records) read from this file. Never create output files that bypass the registry.

## 2. No Synthetic Data, Placeholders, Approximations, or Freezes

- NEVER generate synthetic, estimated, placeholder, approximated, or frozen data to fill gaps
- Every value in every CSV must trace to either: (1) a faithful replication of a published source, or (2) documented analytical methodology with full provenance
- If data truly cannot be obtained, mark the series as `"status": "data_unavailable"` with an empty CSV — do not fabricate values
- `np.random` in a data construction script is a code smell — investigate and remove
- Before giving up: (1) digitize from figures, (2) fetch from public APIs/statistical agencies, (3) contact authors

## 3. No Proxies

- NEVER substitute a proxy series when the original's exact source is publicly available
- CPI is not PPI. Earnings are not compensation. Yield is not total return.
- If the original used Series X from Agency Y, the extension must use Series X from Agency Y
- If the exact series is discontinued, document the substitution with a "Concept Match Justification"
- Every substitution must be flagged in the registry as `"proxy": true` with `"proxy_justification": "..."`

## 4. No Lazy Splices on Derived Quantities

- Growth-rate splice is ONLY valid when the original series was itself a directly-observed time series
- If the original computed a formula (r = NOS/K, ratio = X/Y), the extension must compute the same formula with extended component data
- The registry should classify each series: `"construction": "direct" | "formula" | "composite"`
- Formula-type series should have `"formula"` and `"components"` fields listing all inputs

## 5. Unit Documentation

- Every series must have a `"units"` field (e.g., `"billions_usd"`, `"index_2017=100"`, `"percent"`, `"ratio"`)
- Every loading script must validate that fetched data units match expected units
- Processing scripts that combine series must include a dimensional analysis comment if units differ

## 6. Content Type Classification

- Every series must have `"content_type": "time_series" | "cross_sectional" | "theoretical" | "derived"`
- Extensions ONLY apply to `time_series` type
- The pipeline must skip extension for non-time-series automatically

## 7. Source Material Reading (for reviews)

- Before scoring any review dimension, the reviewer MUST read the original source material
- Every review must verify that the implementation matches the author's stated methodology, not just that code runs
- Cross-check actual data values against published figures and tables
- The Knowledge Base is the ground truth for what the author intended — code must conform to it

## 8. Pipeline Stage Ordering

```
1. anu-research     — Mine quotes and methodology from Knowledge Base
2. anu-adequacy     — Post-research readiness gate
3. anu-ingestion    — Build series_registry.json, decompose series
4. anu-extension    — Extend with modern APIs
5. anu-replicator   — Self-contained reproduction package
6. anu-chopped / anu-extenbook / anu-visualize — Output formats
   anu-review       — Quality audit (FLOATING — can run at any stage)
   anu-data         — AnuData Architecture format standard
   anu-ledger / anu-variant — Cross-cutting infrastructure
7. anu-publish      — Publication packaging
```

---

*These rules were learned from building two large-scale replication projects (105 and 59 series respectively). Each rule exists because its absence caused a specific, documented failure.*
