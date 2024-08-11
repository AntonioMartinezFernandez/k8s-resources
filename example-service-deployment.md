# Example service deployment

## Create the example deployment

```bash
kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.39 -- /agnhost netexec --http-port=8080
kubectl get deployments
kubectl get pods
kubectl get events
kubectl config view
kubectl logs [pod-name-from-getpods]
```

## Create the example service

```bash
kubectl expose deployment hello-node --type=LoadBalancer --port=8080
kubectl get services
```

## Remove service and deployment

```bash
kubectl delete service hello-node
kubectl delete deployment hello-node
```
