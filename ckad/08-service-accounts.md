## Service Accounts

```bash
k get serviceaccount

k create serviceaccount <serviceaccount-name>

k create token <serviceaccount-name>

k describe serviceaccount <serviceaccount-name>
```

All deployed pods have a mounted volume in `/var/run/secrets/kubernetes.io/serviceaccount` with the DEFAULT service account secret. This folder contains 3 files, one of them is the `token`.

To mount a different service account:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-service
spec:
  containers:
    - name: my-service
      image: my-service:1.0.0
  serviceAccountName: serviceaccount-name
```

To avoid automount service account:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-service
spec:
  containers:
    - name: my-service
      image: my-service:1.0.0
  automountServiceAccountToken: false
```
