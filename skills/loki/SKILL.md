---
name: loki
description: Grafana Loki log aggregation and LogQL queries. Use when writing LogQL queries for log analysis, configuring Promtail scrape pipelines, debugging log ingestion issues, or creating Loki alerting rules.
---

# Loki

## LogQL Syntax Gotchas

### Stream Selector (required first)
```logql
{app="nginx", namespace=~"prod|staging"}
```
- At least one label matcher required
- `=~` is regex, not glob (use `.*` not `*`)

### Filter Order Matters
```logql
{app="api"} |= "error" | json | level="error" | line_format "{{.message}}"
```
Filters apply left-to-right. Put cheap filters (string match) before expensive ones (json parse).

### Parser Output
After `| json` or `| logfmt`, extracted fields become labels for filtering:
```logql
{app="api"} | json | status_code >= 400 | duration > 1s
```

### Line vs Label Filters
- `|=` `!=` `|~` `!~` filter on log line content (before parsing)
- Label matchers after parser filter on extracted fields

## Metric Queries

```logql
# Logs per second
rate({app="nginx"}[5m])

# Error rate percentage
sum(rate({app="api"} |= "error" [5m])) / sum(rate({app="api"}[5m]))

# Extract numeric value for aggregation
quantile_over_time(0.95, {app="api"} | json | unwrap duration [5m]) by (endpoint)
```

### unwrap Gotchas
- Requires parsed numeric field
- Add `| __error__=""` to filter parse failures
- Supports unit conversion: `unwrap duration(latency)`, `unwrap bytes(size)`

## Promtail Pipeline

```yaml
pipeline_stages:
  - cri: {}          # Parse container runtime format first
  - json:
      expressions:
        level: level
        msg: message
  - labels:
      level:         # Promote extracted field to label
  - timestamp:
      source: time
      format: RFC3339Nano
  - output:
      source: msg    # Replace log line with extracted field
```

### Stage Order
1. `cri` or `docker` (parse container format)
2. `multiline` (if needed)
3. `regex`/`json`/`logfmt` (extract fields)
4. `labels` (promote to index)
5. `timestamp` (set log timestamp)
6. `output` (modify final line)

## Cardinality Warning

Labels are indexed. High-cardinality labels (user IDs, trace IDs) cause:
- Index bloat
- Query performance degradation
- Ingestion rate limits

Keep extracted fields as line content unless you need to filter by them.

## Common Patterns

```logql
# Errors with context
{namespace="prod"} |= "error" | json | line_format "{{.timestamp}} [{{.service}}] {{.message}}"

# Logs missing (for alerting)
absent_over_time({app="critical"}[5m])

# Top error messages
topk(10, sum by (message) (count_over_time({app="api"} | json | level="error" [1h])))
```
