# Install kubectl

## Install latest version of kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
rm kubectl
```

## Add kubeconfig

1. Execute `sudo cat /var/lib/k0s/pki/admin.conf` in the control plane and copy the output
2. Execute `ip a` in the control plane to know the IP address
3. Execute `nano ~/.kube/config` in the client machine
4. Paste the output from the first step, and substitute the _localhost_ of the output with the real IP address of the control plane
5. Save the file
6. Execute `kubectl get nodes` in the client to check if is working
