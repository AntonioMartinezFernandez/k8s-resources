## Resource requirements

```yaml
# Limit resources at pod level for an specific namespace
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
spec:
  limits:
    - default:
        cpu: 100m
      defaultRequest:
        cpu: 100m
      max:
        cpu: '1'
      min:
        cpu: 50m
      type: Container
---
apiVersion: v1
kind: LimitRange
metadata:
  name: memory-resource-constraint
spec:
  limits:
    - default:
        memory: 128Mi
      defaultRequest:
        memory: 128Mi
      max:
        memory: 1Gi
      min:
        memory: 64Mi
      type: Container
```

```yaml
# Limit resources at whole cluster level for an specific namespace
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-resource-quota
spec:
  hard:
    request.cpu: 4
    request.memory: 4Gi
    limits.cpu: 10
    limits.memory: 8Gi
```
