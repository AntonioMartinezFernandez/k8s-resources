# Install k0s

## Install and start Control Plane (Master)

0. Configure a static IP in the server (this server could be a VM, VPS, AWS EC2 instance... with a debian based distro installed -i.e. ubuntu server-)
1. Install

```bash
sudo apt install curl
curl -sSLf https://get.k0s.sh | sudo sh
sudo mkdir -p /etc/k0s
sudo k0s config create > /etc/k0s/k0s.yaml
sudo k0s install controller -c /etc/k0s/k0s.yaml
sudo k0s start
```

## Create token (in Control Plane)

```bash
sudo k0s token create --role=worker
```

## Configure Workers

0. Configure a static IP in the server
1. Create the file `/etc/k0s/token` and paste the previously created token
2. Install k0s

```bash
sudo apt install curl
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

**Reset k0s state node (control plane or worker)**

```bash
sudo k0s reset
```

**Extensions**

- [MetalLB Load Balancer](https://docs.k0sproject.io/v1.23.3+k0s.0/examples/metallb-loadbalancer/)
- [Traefik Ingress Controller](https://docs.k0sproject.io/v1.23.3+k0s.0/examples/traefik-ingress/)
