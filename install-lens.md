# Install Lens

## Installation

```bash
curl -fsSL https://downloads.k8slens.dev/keys/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/lens-archive-keyring.gpg > /dev/null
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/lens-archive-keyring.gpg] https://downloads.k8slens.dev/apt/debian stable main" | sudo tee /etc/apt/sources.list.d/lens.list > /dev/null
sudo apt update && sudo apt install lens
lens-desktop
```

## Add Cluster

1. Execute `sudo cat /var/lib/k0s/pki/admin.conf` in the control plane and copy the output
2. Execute `ip a` in the control plane to know the IP address
3. Go to Lens
4. Go to `Menu Catalog > + > Add from kubeconfig`
5. Paste the output from the first step, and substitute the _localhost_ of the output with the real IP address of the control plane
6. Click 'Add Clusters'
7. Connect to the cluster
