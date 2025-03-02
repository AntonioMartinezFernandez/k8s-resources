## Network Policies

Set ingress and egress traffic rules among pods.

A Container Network Interface (CNI) plugin is required to implement the Kubernetes network model. Docs: https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/

```bash
k get networkpolicies --all-namespaces
k delete networkpolicies <policy-name>
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: database-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: redis

  # Policies to use
  policyTypes:
    - Ingress
    - Egress

  # Allowed ingress traffic
  ingress:
    - from:
        - ipBlock:
            cidr: 172.17.0.0/16
            except:
              - 172.17.1.0/24
        - namespaceSelector:
            matchLabels:
              project: app-namespace
        - podSelector:
            matchLabels:
              role: api-service
      ports:
        - protocol: TCP
          port: 6679

  # Allowed egress traffic
  egress:
    - to:
        - ipBlock:
            cidr: 10.0.0.0/24
      ports:
        - protocol: TCP
          port: 5978
```
