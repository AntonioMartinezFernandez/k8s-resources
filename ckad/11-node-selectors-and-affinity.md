## Node selectors & Affinity

```bash
# Label a node
k label nodes <node-name> <label-key>=<label-value>

# Example
k label nodes my-node size=Large
```

```yaml
# Example POD with node selector

apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
    - name: webapp
      image: webapp
  nodeSelector:
    size: Large
```

```yaml
# Example POD with affinity

apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
    - name: webapp
      image: webapp
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution: # requiredDuringSchedulingIgnoredDuringExecution|preferredDuringSchedulingIgnoredDuringExecution|requiredDuringSchedulingRequiredDuringExecution
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: In # In|NotIn|Exists
            values:
            - Large
            - Medium
```