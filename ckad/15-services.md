## Services

Service types:

- NodePort: expose an specific port at NODE level
- ClusterIP: expose an specific port at CLUSTER level
- Load Balancer:

```bash
k get svc --all-namespaces
```

```yaml
# Deployment Example

apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: default
  name: amf-http-echo-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: amf-http-echo
  template:
    metadata:
      labels: # IMPORTANT! This labels are used by the SERVICES to find the pods
        app: amf-http-echo
    spec:
      containers:
        - name: amf-http-echo
          image: antoniomarfer/http-echo
          ports:
            - containerPort: 8080
```

```yaml
# NodePort SERVICE

apiVersion: v1
kind: Service
metadata:
  namespace: default
  name: amf-http-echo-service
spec:
  type: NodePort
  selector:
    app: amf-http-echo # From deployment above
  ports:
    - protocol: TCP
      targetPort: 8080 # Deployment port
      port: 8080 # Service port
      nodePort: 30080 # Node port. Allowed values: 30000-32767. Automatically assigned if not present
```

```yaml
# ClusterIP SERVICE

apiVersion: v1
kind: Service
metadata:
  namespace: default
  name: amf-http-echo-service
spec:
  type: ClusterIP
  selector:
    app: amf-http-echo # From deployment above
  ports:
    - protocol: TCP
      targetPort: 8080 # Deployment port
      port: 8080 # Service port
```
