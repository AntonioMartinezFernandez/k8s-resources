# Install OpenLens

## Installation

1. Go to `https://github.com/MuhammedKalkan/OpenLens/releases` and download the installation package
2. Install
3. Add the [shell](https://medium.com/geekculture/fix-essential-functionality-missing-in-the-new-openlens-version-f9ac862e9e27) plugin

## Add Cluster

1. Execute `sudo cat /var/lib/k0s/pki/admin.conf` in the control plane and copy the output
2. Execute `ip a` in the control plane to know the IP address
3. Go to OpenLens
4. Go to `Menu Catalog > + > Add from kubeconfig`
5. Paste the output from the first step, and substitute the _localhost_ of the output with the real IP address of the control plane
6. Click 'Add Clusters'
7. Connect to the cluster
