## SECURITY - Authentication

Users can be "Admins" or "Developers".
Service Accounts are for "Bots".

KubeConfig:

- Clusters: Different kubernetes clusters (e.g. prodcluster, devcluster, localcluster)
- Users: Different users (e.g. admin, developer)
- Contexts: Cluster/User relation (e.g. admin@prodcluster, developer@devcluster)

```bash
# Service Accounts
k create serviceaccount <sa-name>
k get serviceaccounts

# Accessing kube-api
## Obtain control plane URL
k cluster-info

## NOT RECOMMENDED!!! To be able to access tthe kube-api with token authentication, we need to specify the user-token-details.csv file with this information and configure the cluster with `kubeadm` or `kube-apiserver` defining the `token-auth-file` parameter

curl -v -k https://localhost:6443/api/v1/pods --header "Authorization: Bearer Kpj72c9dfgls9dkgsCS9v7s2dgddks4B"

## KubeConfig - Certificates
curl -v -k https://localhost:6443/api/v1/pods \
--key admin.key \
--cert admin.crt \
--cacert ca.crt

# Equivalent
k get pods --kubeconfig <config-file>

# Select KubeConfig context
k config set-context <context-name> --kubeconfig=<kubeconfig-filepath>

# Check current KubeConfig
k config view
```

KubeConfig example

```yaml
apiVersion: v1
clusters:
  - cluster:
      certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSBase64encodedCA...
      server: https://dev-cluster:6443
    name: development
  - cluster:
      certificate-authority: /etc/kubernetes/pki/ca.crt
      server: https://prod-cluster:6443
    name: production
users:
  - name: admindevelopment
    user:
      client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURBase64encodedCert...
      client-key-data: LS0tLS1CRUdJTiBFQyBQUBase64encodedKey...
  - name: adminproduction
    user:
      client-certificate: /etc/kubernetes/pki/users/admin.crt
      client-key: /etc/kubernetes/pki/users/admin.key
contexts:
  - context:
      cluster: production
      user: adminproduction
    name: prod
  - context:
      cluster: development
      user: admindevelopment
    name: dev
current-context: dev
kind: Config
preferences: {}
```
