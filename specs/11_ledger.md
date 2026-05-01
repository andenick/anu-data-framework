# Anu Ledger — Data Checkout/Checkin Coverage Tracking

## Purpose

Track which artifacts (chopped CSVs, extenbooks, charts, validation reports) have been generated for which series. Provides a coverage matrix showing what exists, what's missing, and what's stale.

## The Ledger

A single JSON file tracking the state of all artifacts:

```json
{
  "ledger_version": "2.2",
  "project_name": "Capitalism Data",
  "last_updated": "2026-04-26T12:00:00Z",
  "series": {
    "S001": {
      "research": {"exists": true, "hash": "a7f3...", "modified": "2026-03-20"},
      "registry_entry": {"exists": true, "hash": "b9e2...", "modified": "2026-03-25"},
      "decomposition": {"exists": true, "hash": "c1d4...", "modified": "2026-03-25"},
      "dpr": {"exists": true, "hash": "d2e5...", "modified": "2026-03-26"},
      "epr": {"exists": true, "hash": "e3f6...", "modified": "2026-04-01"},
      "L_script": {"exists": true, "hash": "f4g7...", "modified": "2026-04-05"},
      "P_script": {"exists": true, "hash": "g5h8...", "modified": "2026-04-05"},
      "V_validation": {"exists": true, "passed": true, "modified": "2026-04-06"},
      "chopped_csv": {"exists": true, "hash": "h6i9...", "modified": "2026-04-07"},
      "extenbook": {"exists": true, "hash": "i7j0...", "modified": "2026-04-07"},
      "shiny_chapter": {"exists": true, "modified": "2026-04-08"}
    },
    "S002": {...},
    ...
  }
}
```

## Coverage Matrix

The ledger generates a coverage matrix showing what's complete:

```
Series  Research  Registry  L##  P##  V##  Chopped  Extenbook  Shiny
S001    OK        OK        OK   OK   OK   OK       OK         OK
S002    OK        OK        OK   OK   OK   OK       OK         OK
S003    OK        OK        OK   OK   --   OK       OK         OK     <- V## missing
S004    OK        OK        --   --   --   --       --         --     <- Stuck at registry
...

Coverage: 89/105 series fully complete (84.8%)
```

## Stale Detection

If a downstream artifact's hash differs from what's recorded as the source's hash, the artifact is stale:

```
S003: registry hash changed (b9e2... → c1d4...)
  - Stale: chopped_csv (built from old registry)
  - Stale: extenbook (built from old registry)
  - Action: Re-run P03_process.py
```

Stale propagation makes data drift visible.

## Checkout/Checkin Pattern

When an agent works on a series:

```python
# Checkout
ledger.checkout("S001", agent="claude-sonnet-4")
# This locks the series and records who is working on it

# ... do work ...

# Checkin
ledger.checkin("S001", artifacts={
    "research": "S001_research.json",
    "registry_entry": "S001 in registry",
    ...
}, agent="claude-sonnet-4")
# This updates hashes and unlocks the series
```

The checkout/checkin pattern prevents two agents from concurrently modifying the same series.

## Commands

```bash
# Show coverage
anu-ledger coverage

# Show stale artifacts
anu-ledger stale

# Drill into one series
anu-ledger show S001

# Force regenerate coverage from disk
anu-ledger regenerate

# Find blocking issues
anu-ledger blockers
```

## Quality Checklist

- [ ] Ledger exists and is current
- [ ] All 105+ series have ledger entries
- [ ] Coverage matrix complete (all artifacts present)
- [ ] No stale artifacts
- [ ] Hash verification passes for all artifacts
- [ ] Last-updated timestamps are recent
