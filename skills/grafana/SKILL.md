---
name: grafana
description: Grafana dashboard JSON configuration and alerting. Use when creating or editing dashboard JSON, configuring panels programmatically, setting up Grafana alerting rules, or troubleshooting visualization issues.
---

# Grafana

## Dashboard JSON Gotchas

### Required Fields
```json
{
  "dashboard": {
    "id": null,           // null for new, existing ID for update
    "uid": "unique-id",   // Stable identifier for API/links
    "title": "Name",
    "panels": []
  },
  "overwrite": true       // Required at root level for updates
}
```

### Panel Positioning
`gridPos` uses 24-column grid. Panels auto-stack if positions overlap.
```json
"gridPos": {"h": 8, "w": 12, "x": 0, "y": 0}
```

### Data Source Reference
Always use UID, not name (names can change):
```json
"datasource": {"type": "prometheus", "uid": "prometheus-uid"}
```

## Template Variable Syntax

| Context | Syntax | Multi-value behavior |
|---------|--------|---------------------|
| PromQL label | `{ns=~"$var"}` | Pipe-joined: `ns=~"a\|b\|c"` |
| SQL | `'$var'` or `${var:csv}` | Depends on format |
| Lucene | `$var` | Space-joined |

### Variable Query Refresh
- `0`: Never (on dashboard load only)
- `1`: On dashboard load
- `2`: On time range change (use this for most cases)

## Alerting (Grafana 9+)

### Alert Rule Data Array
```json
"data": [
  {"refId": "A", "model": {"expr": "..."}},           // Query
  {"refId": "B", "reducer": "last", "expression": "A"}, // Reduce
  {"refId": "C", "type": "threshold", "expression": "B", "conditions": [...]} // Condition (must be last)
]
```
The `condition` field in the rule must reference the final refId (here "C").

### No Data / Error States
- `NoData`: Query returns empty
- `Alerting`: Treat no data as firing
- `OK`: Treat no data as resolved
- `Error`: Evaluation failed

## Common Panel Configs

### Thresholds
```json
"thresholds": {
  "mode": "absolute",  // or "percentage"
  "steps": [
    {"color": "green", "value": null},  // null = base
    {"color": "yellow", "value": 70},
    {"color": "red", "value": 90}
  ]
}
```

### Value Mappings
```json
"mappings": [
  {"type": "value", "options": {"0": {"text": "Down", "color": "red"}}},
  {"type": "range", "options": {"from": 1, "to": 100, "result": {"text": "OK"}}}
]
```

### Units
- `percentunit`: 0-1 displayed as 0%-100%
- `percent`: Already 0-100, displayed as-is
- `bytes` vs `decbytes`: Binary (1024) vs decimal (1000)
- `s`, `ms`, `µs`, `ns`: Auto-scales appropriately
