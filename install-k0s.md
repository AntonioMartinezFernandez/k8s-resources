# Install k0s

## Install and start Control Plane (Master)

```bash
curl -sSLf https://get.k0s.sh | sudo sh
mkdir -p /etc/k0s
k0s config create > /etc/k0s/k0s.yaml
sudo k0s install controller -c /etc/k0s/k0s.yaml
sudo k0s start
```

## Create token (in Control Plane)

```bash
k0s token create --role=worker
```

## Workers

Create the file **/etc/k0s/token** and paste the previously created token

```bash
curl -sSLf https://get.k0s.sh | sudo sh
sudo k0s install worker --token-file /etc/k0s/token
sudo k0s start
```

## Notes

**Check Status**

```bash
sudo k0s status
```

**Copying the kubeconfig for pasting it to Lens**

```bash
sudo cat /var/lib/k0s/pki/admin.conf
```

**Extensions**

- [MetalLB Load Balancer](https://docs.k0sproject.io/v1.23.3+k0s.0/examples/metallb-loadbalancer/)
- [Traefik Ingress Controller](https://docs.k0sproject.io/v1.23.3+k0s.0/examples/traefik-ingress/)
