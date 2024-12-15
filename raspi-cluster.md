# Raspberry Pi Cluster

## How to install a Raspberry Pi cluster

1. Install [Raspberry Pi OS](https://www.raspberrypi.com/documentation/computers/getting-started.html#raspberry-pi-imager) without desktop on the Raspberry Pi's
2. Set [static IP address](https://raspberrypi-guide.github.io/networking/set-up-static-ip-address#setting-up-with-the-terminal) and connect to the internet

```bash
sudo nano /etc/dhcpcd.conf
```

- Add the following lines with the correct IP address, router and DNS

```bash
interface eth0
static ip_address=192.168.1.113/24
static routers=192.168.1.1
static domain_name_servers=1.1.1.1
```

3. Disable SWAP

```bash
# 1. Turn off swap temporary.
sudo swapoff -a

# 2. To turn of swap permanently we need to update the `CONF_SWAPSIZE` in `dphys-swapfile` file to `0`
sudo nano /etc/dphys-swapfile

# 3. set
  CONF_SWAPSIZE=0
```

4. Install k3s in the Raspberry Pi's (https://medium.com/@stevenhoang/step-by-step-guide-installing-k3s-on-a-raspberry-pi-4-cluster-8c12243800b9)

```bash
# MASTER NODE
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server --disable=traefik --disable=servicelb --flannel-backend=host-gw --tls-san=192.168.1.113 --bind-address=192.168.1.113 --advertise-address=192.168.1.113 --node-ip=192.168.1.113 --cluster-init" sh -s -

# WORKER NODES

# Get node-token from master node
sudo cat /var/lib/rancher/k3s/server/node-token
# The result is something likes this
  `THIS19937008cbde678aeaf200517f07c0ccd67dc80bdf4df6f746IS4780e15ebcd::server:40fc2cc2fnode81cdacc0b9bb1231token`

# Execute this to in each worker node
curl -sfL https://get.k3s.io | K3S_URL=https://192.168.1.113:6443 \
  K3S_TOKEN="THIS19937008cbde678aeaf200517f07c0ccd67dc80bdf4df6f746IS4780e15ebcd::server:40fc2cc2fnode81cdacc0b9bb1231token" sh -
```

## How to uninstall the Raspberry Pi's cluster

MASTER NODE

```bash
sudo /usr/local/bin/k3s-uninstall.sh
```

WORKER NODES

```bash
sudo /usr/local/bin/k3s-agent-uninstall.sh
```

## How to configure the Raspberry Pi's cluster

1. Install kubectl

```bash
brew install kubectl
```

2. Save the raspi kubeconfig (from cluster file `/etc/rancher/k3s/k3s.yaml`) in the `~/.kube/raspi-config` file
3. Add the following lines in the `~/.zshrc` file:

```bash
export KUBECONFIG="$HOME/.kube/config"
alias k="kubectl"
alias kraspi="kubectl --kubeconfig $HOME/.kube/raspi-config"
```

4. `source ~/.zshrc`
5. Check the cluster info

```bash
kraspi cluster-info
```

6. Config the access to the Raspberry Pi's cluster with [OpenLens](./install-openlens.md)
7. Install Helm

```bash
brew install helm
```

8. Install MetalLB

```bash
# Check latest MetalLB version. In this case is v0.14.8 (backup in the 00-metal-lb-install.yaml file)
kraspi apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.8/config/manifests/metallb-native.yaml
kraspi get pods -n metallb-system
```

9. Create MetalLB IP addresses pool and layer 2 advertisement

```bash
kraspi apply -f 01-ip-address-pool.yaml
```

10. Check the MetalLB IP addresses pool

```bash
kraspi apply -f 02-load-balancer-checker.yaml
kraspi get services -n lb-checker -o wide # Check that the EXTERNAL-IP is available
curl -X GET http://<EXTERNAL-IP>/
kraspi delete -f 02-load-balancer-checker.yaml
```

11. Install the Ingress Controller

```bash
# Check latest nginx ingress controller version for cloud (not for baremetal). In this case is v1.1.3 (backup in the 00-nginx-ingress-controller.yaml file)
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.3/deploy/static/provider/cloud/deploy.yaml

# Check the ingress-nginx-controller have an external IP address
kraspi -n ingress-nginx get svc ingress-nginx-controller
```

## How to install an example application

1. Apply the resources

```bash
kraspi apply -f 03-namespace.yaml # Create the application namespace
kraspi apply -f 04-deployment.yaml # Create the deployment with 2 replicas of the application
kraspi apply -f 05-service.yaml # Create the ClusterIP service with port 8080 pointing to the port 80 of the application
kraspi apply -f 06-ingress.yaml # Create the ingress with the host green-app.local (path "/") pointing to the service
```

2. Check the application IP address

```bash
kraspi get ingress green-ingress -n green-web-app -o wide
```

3. Access the application

```bash
curl -H 'Host: green-app.local' http://<EXTERNAL-IP>/
```
