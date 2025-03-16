## SECURITY - RBAC

Role Based Access Controll flow:

kubectl/apiClient -> request Authentication -> request Authorization (Role Based) -> do something with some resource

API Groups:

- The core (also called legacy) group is found at REST path /api/v1. The core group is not specified as part of the apiVersion field, for example, apiVersion: v1
- The named groups are at REST path /apis/$GROUP_NAME/$VERSION and use apiVersion: $GROUP_NAME/$VERSION (for example, apiVersion: batch/v1)

Kubernetes API reference: https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.32/

Authorization: allow authorized Users or ServiceAccounts to performs actions (VERBS) over the cluster resources

Kubernetes authorization types

- Node
- ABAC
- RBAC
- Webhook

```bash
# Create kubectl proxy to access directly to the kube api
k proxy

# Access kube api through kubectl proxy
curl -k -v http://localhost:8001

# Create users:
# https://github.com/onthedock/k8s-devops/blob/main/docs/seguridad/crear-usuarios-en-k8s/creacion-de-usuarios.md
# https://github.com/pablokbs/peladonerd/blob/master/kubernetes/18/comandos.txt

# Roles
k get roles
k describe role <role-name>

k get rolebindings
k describe rolebinding <rolebinding-name>

# Check Access
k auth can-i <verb> <resource>
k auth can-i <verb> <resource> --as=<user>

# Examples
k auth can-i create pods
k auth can-i delete secrests --as=developer

# Check cluster api resources
k api-resources --namespaced=true # Namespaced resources - mainly Role targets
k api-resources --namespaced=false # Cluster scoped resources - mainly ClusterRole targets

```

Simple Role:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: developer
rules:
  - apiGroups: ['', 'extensions', 'apps'] # "" indicates the core API group
    resources: ['pods']
    verbs: ['get', 'list', 'create', 'update', 'delete', 'watch'] # Equivalent to ["*"]
    resourceNames: ['red', 'green']
  - apiGroups: [''] # "" indicates the core API group
    resources: ['configmaps']
    verbs: ['create', 'update']
```

Simple ClusterRole:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # "namespace" omitted since ClusterRoles are not namespaced
  name: secret-reader
rules:
  - apiGroups: ['']
    # at the HTTP level, the name of the resource for accessing Secret
    # objects is "secrets"
    resources: ['secrets']
    verbs: ['get', 'watch', 'list']
```

Simple Role binding with Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "jane" to read pods in the "default" namespace.
# You need to already have a Role named "pod-reader" in that namespace.
kind: RoleBinding
metadata:
  name: developer-binding
  namespace: default
subjects:
  # You can specify more than one "subject"
  - kind: User
    name: john # "name" is case sensitive
    apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" specifies the binding to a Role / ClusterRole
  kind: Role #this must be Role or ClusterRole
  name: developer # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```

Simple Role binding with ClusterRole

```yaml
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "dave" to read secrets in the "development" namespace.
# You need to already have a ClusterRole named "secret-reader".
kind: RoleBinding
metadata:
  name: read-secrets
  # The namespace of the RoleBinding determines where the permissions are granted.
  # This only grants permissions within the "development" namespace.
  namespace: development
subjects:
  - kind: User
    name: dave # Name is case sensitive
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

Simple ClusterRole binding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
# This cluster role binding allows anyone in the "manager" group to read secrets in any namespace.
kind: ClusterRoleBinding
metadata:
  name: read-secrets-global
subjects:
  - kind: Group
    name: manager # Name is case sensitive
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```
