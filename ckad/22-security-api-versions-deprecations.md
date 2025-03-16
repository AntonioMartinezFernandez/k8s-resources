## SECURITY - API Versions / Deprecations

```bash
# Update deprecated apiVersion in the file (apps/v1alpha1 for example) with the new apps/v1
# Installation docs: https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/#install-kubectl-convert-plugin
k convert -f deprecated-deployment.yaml --output-version apps/v1

k api-resources

k proxy
curl localhost:8001/apis/rbac.authorization.k8s.io
```
