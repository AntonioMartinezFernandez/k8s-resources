## Namespaces

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: development
  labels:
    name: development
```

```bash
k get namespaces
k get pods -n <namespace-name>
k get pods --all-namespaces
k run redis --image=redis --namespace=finance

# Delete all resources in a namespace (and the namespace itself)
k delete all -n <namespace> --all
k delete ns <namespace>
```
