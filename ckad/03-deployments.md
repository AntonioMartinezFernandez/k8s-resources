## Deployments

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
    type: backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
        type: backend
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

```bash
k create -f <deployment-definition-file>
k get deployments

# INVESTIGATE COMMANDS AND OPTIONS FOR ROLLOUTS AND ROLLBACKS!!!
```