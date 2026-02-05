---
name: kubernetes
description: Kubernetes resource configuration and troubleshooting. Use when debugging pod failures, configuring probes and resource limits, setting up RBAC or NetworkPolicies, or resolving common Kubernetes errors like CrashLoopBackOff or ImagePullBackOff.
---

# Kubernetes

## Pod Failure Troubleshooting

| Status | Common Causes | Debug Steps |
|--------|---------------|-------------|
| CrashLoopBackOff | App crash, bad entrypoint, missing deps | `kubectl logs <pod> --previous` |
| ImagePullBackOff | Wrong image/tag, no auth, registry down | Check image name, `kubectl get events` |
| Pending | No resources, node selector mismatch, PVC pending | `kubectl describe pod`, check node capacity |
| OOMKilled | Memory limit exceeded | Increase `limits.memory` or fix leak |
| Evicted | Node disk/memory pressure | Check node conditions, clean up |
| CreateContainerError | Bad securityContext, missing configmap/secret | `kubectl describe pod` for specific error |

## Resource Configuration Gotchas

### Requests vs Limits
- **Requests**: Scheduling guarantee. Pod won't schedule if node lacks capacity.
- **Limits**: Hard ceiling. Container killed (OOM) or throttled (CPU) if exceeded.
- No limits = unbounded (can consume entire node)
- `requests` > `limits` is invalid

### Probe Timing
```yaml
livenessProbe:
  initialDelaySeconds: 10  # Wait before first check
  periodSeconds: 5         # Check interval
  timeoutSeconds: 1        # Max wait for response
  failureThreshold: 3      # Failures before action
```
- Liveness failure → container restart
- Readiness failure → removed from service endpoints
- StartupProbe disables other probes until success (use for slow-starting apps)

### Security Context Inheritance
Pod-level `securityContext` applies to all containers but container-level overrides it:
```yaml
spec:
  securityContext:
    runAsNonRoot: true      # Pod default
  containers:
    - securityContext:
        runAsUser: 1000     # Container override
```

## RBAC Patterns

### Minimal Role for Pod Logs
```yaml
rules:
  - apiGroups: [""]
    resources: ["pods", "pods/log"]
    verbs: ["get", "list"]
```

### Common API Groups
- `""` (empty): Core resources (pods, services, configmaps)
- `apps`: Deployments, StatefulSets, DaemonSets
- `networking.k8s.io`: Ingress, NetworkPolicy
- `rbac.authorization.k8s.io`: Roles, bindings

## NetworkPolicy Gotchas

- No NetworkPolicy = all traffic allowed
- Any NetworkPolicy selecting a pod = default deny for that direction
- Empty `podSelector: {}` selects all pods in namespace
- `namespaceSelector: {}` selects all namespaces
- Combine selectors with `- ` (OR) vs nested (AND)

```yaml
ingress:
  - from:
      - podSelector: {matchLabels: {app: frontend}}  # AND
        namespaceSelector: {matchLabels: {env: prod}}
  - from:  # OR (separate rule)
      - podSelector: {matchLabels: {app: monitoring}}
```
