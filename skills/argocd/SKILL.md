---
name: argocd
description: ArgoCD GitOps continuous delivery for Kubernetes. Use when creating or debugging ArgoCD Application/ApplicationSet manifests, configuring sync policies, troubleshooting OutOfSync or degraded states, or integrating Helm/Kustomize sources.
---

# ArgoCD

## Critical Gotchas

### Sync Behavior
- `selfHeal: true` reverts manual cluster changes every 3 minutes (default)
- `prune: true` deletes resources removed from Git—enable only when certain
- `replace: true` in syncOptions does full replacement instead of patch (destructive)
- `ServerSideApply=true` required for CRDs with large specs to avoid annotation size limits

### Application Targeting
- `destination.server` must match exactly what's registered in ArgoCD (check `argocd cluster list`)
- Use `https://kubernetes.default.svc` for in-cluster, not `kubernetes.default`
- `destination.namespace` doesn't auto-create unless `CreateNamespace=true` in syncOptions

### Health Assessment
- ArgoCD has built-in health checks for standard resources
- Custom resources show "Progressing" indefinitely without custom health check in `argocd-cm`
- Ingress health requires actual endpoint check, not just resource existence

## Non-Obvious Patterns

### Sync Waves
```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "-1"  # Negative = earlier
```
- Waves are strings sorted numerically ("-1" < "0" < "1")
- Resources in same wave sync in parallel
- Use for: namespaces → secrets → deployments → ingress

### ApplicationSet Generator Precedence
```yaml
generators:
  - matrix:
      generators:
        - clusters: {}        # Outer loop
        - list:               # Inner loop
            elements: [{env: prod}, {env: staging}]
```
- Matrix multiplies generators (clusters × environments)
- Merge combines generators with override precedence (later wins)

### Helm Value Precedence
`values` (inline) < `valueFiles` (in order) < `parameters` (highest priority)

## Troubleshooting

| Symptom | Likely Cause |
|---------|--------------|
| OutOfSync but no diff | Ignored differences, resource hooks, or server-side defaulting |
| Sync succeeds but unhealthy | Missing health check, resource not ready, CRD issue |
| "already exists" error | Resource managed by another Application or created manually |
| Stuck "Progressing" | No health check for CRD, or resource genuinely not ready |

```bash
argocd app get <app> --hard-refresh  # Force manifest re-read
argocd app diff <app> --local <path>  # Compare local to live
```
