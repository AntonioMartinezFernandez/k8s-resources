## Taints and tolerations

Taints are set on NODEs and Tolerations are set on PODs.
Taint define a deployment rule for Toleration (NoSchedule|PreferNoSchedule|NoExecute)

```bash
# Add taint to node
k taint nodes <node-name> key=value:<taint-effect>

# Example: not allow schedule pods with toleration app=blue in the my-node node
k taint nodes my-node app=blue:NoSchedule

# Remove taint from node
k taint node <node-name> <taint>-

# Example: remove 'node-role.kubernetes.io/control-plane:NoSchedule' taint from 'controlplane' node
k taint node controlplane node-role.kubernetes.io/control-plane:NoSchedule-
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
    - name: webapp
      image: webapp

  tolerations:
    - key: "app"
      operator: "Equal"
      value: "blue"
      effect: "NoSchedule"
```
