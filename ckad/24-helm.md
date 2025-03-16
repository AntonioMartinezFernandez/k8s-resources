## Helm

```bash
helm search hub wordpress # Search in the artifact Hub

helm repo add bitnami https://charts.bitnami.com/bitnami # Add bitnami repository

helm repo update bitnami # Update the bitnami repository

helm search repo wordpress # Search in local repositories

helm repo list # List repositories

helm install my-wordpress bitnami/wordpress # Install wordpress from bitnami repository

helm list # List helm charts installed

helm uninstall my-wordpress # Uninstall helm chart

helm pull --untar bitnami/wordpress # Download but not install

ls wordpress

helm install my-other-wordpress ./wordpress # Install helm chart from downloaded package
```
