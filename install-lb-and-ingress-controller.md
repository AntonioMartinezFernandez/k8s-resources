# Install Load Balancer and Ingress Controller

## Install MetalLB (Load Balancer)

1. Open the `assets` folder of this repository in the client terminal
2. Execute:

```bash
# Check latest MetalLB version. In this case is v0.14.8
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.8/config/manifests/metallb-native.yaml
kubectl get pods -n metallb-system

# Stablish the MetalLB IP addresses pool in the ip-addresses.yaml file before execute the next command
kubectl apply -f ip-addresses.yaml

# Stablish the network interface in the l2-advertisement.yaml file before execute the next command
kubectl apply -f l2-advertisement.yaml
```

## Verify Load Balancer

```bash
kubectl apply -f load-balancer.yaml
# Check that is an available EXTERNAL-IP
kubectl get service load-balancer
# Remove load balancer
kubectl delete -f load-balancer.yaml
```

## Install NGINX ingress controller

```bash
### Baremetal ###
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.3/deploy/static/provider/baremetal/deploy.yaml
### Cloud ###
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.3/deploy/static/provider/cloud/deploy.yaml

kubectl get pods -n ingress-nginx
kubectl get services -n ingress-nginx
kubectl -n ingress-nginx get ingressclasses
kubectl -n ingress-nginx annotate ingressclasses nginx ingressclass.kubernetes.io/is-default-class="true" # Only if there is an only instance of the ingress nginx controller
kubectl edit service ingress-nginx-controller -n ingress-nginx # Or go to OpenLens, Menu Network > Services > ingress-nginx-controller > edit ...
# Change all "NodePort" to "LoadBalancer"
kubectl get services -n ingress-nginx # Check that the TYPE is LoadBalancer
curl <EXTERNAL-IP>

```

## Deploy an example service

1. Open the `assets` folder of this repository in the client terminal
2. Execute:

```bash
kubectl apply -f simple-web-server-with-nginx-ingress.yaml
kubectl get pods -n example-web # Get pods with example-web namespace
kubectl get services -n example-web # Get services with example-web namespace
kubectl get services -n ingress-nginx # Check the EXTERNAL-IP address of the ingress-nginx-controller
curl <EXTERNAL-IP> -H 'Host: simple-web-example.local.com'
```

## Remove the example service

1. Open the `assets` folder of this repository in the client terminal
2. Execute:

```bash
kubectl delete -f simple-web-server-with-nginx-ingress.yaml
```
