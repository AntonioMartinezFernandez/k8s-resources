## Node selectors & Affinity

We can set labels to the cluster NODEs, and then:

- Set POD 'nodeSelector': this POD will be deployed only in NODEs where the nodeSelector match with the NODE label
- Set POD 'affinity' rules: this POD will be deployed following the defined rules

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
