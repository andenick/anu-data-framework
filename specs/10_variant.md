# Anu Variant — Methodology Variant Tracking

## Purpose

Track alternative construction methodologies for the same underlying concept, with canonical IDs that preserve the comparison across variants.

## The Variant Problem

The same economic concept can be measured multiple ways:

- **Profit rate**: r = SP/(K×u) [Shaikh] vs r = s'/(1+c') [textbook] vs r = NOS/K [BEA]
- **Output gap**: HP filter vs BK filter vs CBO official vs production function
- **Real wages**: nominal/CPI vs nominal/PCE vs nominal/GDP deflator

Variants are **not errors**. They are legitimate alternative approaches. The Anu Variant skill makes them tractable.

## Canonical Variant IDs

Each variant gets a stable ID:

```
S037-V01: Shaikh formula r = SP/(K*u)
S037-V02: Textbook formula r = s'/(1+c')
S037-V03: BEA-derived r = NOS/Capital
```

The canonical series (`S037`) is the project's primary choice. Variants are documented but separate.

## Variant Registry Entry

In `series_registry.json`:

```json
"S037": {
  "name": "Industrial Profit Rate",
  "canonical_variant": "V01",
  "variants": {
    "V01": {
      "name": "Shaikh formula",
      "formula": "r = SP/(K*u)",
      "rationale": "Theoretical consistency with Shaikh 2016 framework",
      "subseries": {...}
    },
    "V02": {
      "name": "Textbook formula",
      "formula": "r = s'/(1+c')",
      "rationale": "Standard classical political economy textbook approach",
      "subseries": {...}
    },
    "V03": {
      "name": "BEA-derived",
      "formula": "r = NOS/Capital",
      "rationale": "Direct from national accounts; no theoretical adjustments",
      "subseries": {...}
    }
  }
}
```

## When to Create a Variant

Create a variant when:

1. Multiple legitimate approaches exist in the literature
2. The choice between them is non-obvious or contested
3. Reviewers might reasonably prefer a different choice
4. Comparison across variants is itself analytically interesting

Do NOT create variants for:

1. Computational variations of the same approach (just use the canonical)
2. Earlier draft versions (use git history instead)
3. Sensitivity analyses (use exploration scripts)

## Variant Comparison Outputs

The Shiny app supports variant comparison mode:

```
Industrial Profit Rate
─────────────────────────────────────────
[x] V01 Shaikh formula
[x] V02 Textbook formula
[ ] V03 BEA-derived

[Comparison chart with both V01 and V02 traces]
```

Variant differences become visually obvious. A reviewer can see why methodology choice matters.

## Documentation Per Variant

Each variant has:

- **Formula** (mathematical definition)
- **Rationale** (why this approach)
- **Limitations** (what it doesn't capture)
- **Reference** (paper/methodology source)
- **Reference values** (separate validation per variant)

## Quality Checklist

- [ ] Canonical variant explicitly named in registry
- [ ] All variants have formula documented
- [ ] All variants have rationale documented
- [ ] All variants have separate validation
- [ ] Variant comparison is visualizable in Shiny app
- [ ] DECISION_LOG.md explains why canonical was chosen
