## SECURITY - Admission Controllers

Role Based Access Control WITH admission controllers flow:

kubectl/apiClient
-> request Authentication
-> request Authorization (Role Based)
-> admission controllers (mutating -> validating)
-> do something with some resource

```bash
# Get admission plugins enabled
cat /etc/kubernetes/manifests/kube-apiserver.yaml

# Interact with kube-apiserver (deployed as Pod)
k exec kube-apiserver-controlplane -n kube-system -- kube-apiserver -h
```

ValidatingWebhookConfiguration example:

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: 'pod-policy.example.com'
webhooks:
  - name: 'pod-policy.example.com'
    rules:
      - apiGroups: ['']
        apiVersions: ['v1']
        operations: ['CREATE']
        resources: ['pods']
        scope: 'Namespaced'
    clientConfig:
      service:
        namespace: 'example-namespace'
        name: 'example-service'
      caBundle: <CA_BUNDLE>
    admissionReviewVersions: ['v1']
    sideEffects: None
    timeoutSeconds: 5
```
