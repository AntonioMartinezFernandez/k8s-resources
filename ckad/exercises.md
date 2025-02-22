## CKAD Practice #1 (PODS)

```
alias k=kubectl
k get pods
k run nginx --image nginx
k get pods
k describe pod <pod-name>
k get pod <pod-name> -o yaml > pod-definition.yaml # EXTRACT DEPLOYED POD DEFINITION INTO A YAML FILE

# Create an incorrect Redis pod via YAML definition (redis.yaml)
k run redis --image=redis123 --dry-run=client -o yaml > redis.yaml

# FILE CONTENT:
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: redis
  name: redis
spec:
  containers:
  - image: redis123
    name: redis
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

# EXECUTE:
k create -f redis.yaml

# FIX THE DEFINITION AND APPLY AGAIN
k apply -f redis.yaml

# get resources with extra info
k get all -o wide
```

## CKAD Practice #2 (REPLICASETS)

```
alias k=kubectl

k get pods
k get all
k get replicasets
k get replicasets --all-namespaces
k describe replicaset <replicaset-name>
k describe pod <pod-name>
k delete pod <pod-name>

# Find a bug in several ReplicaSet YAML definitions and deploy it:
# - wrong apiVersion field: v1 -> apps/v1
# - field spec.selector.matchLabels incorrect: frontend -> nginx

k create -f <replicaset-definition-file>
k delete -f <replicaset-definition-file>

k edit replicaset <replicaset-name>

k scale replicaset --replicas=5 <replicaset-name>
```

## CKAD Practice #3 (DEPLOYMENTS)

```
k describe deployments frontend-deployment
k create deployment --help
k create deployment <deployment-name> --image=<docker-image> --replicas=<number-of-replicas> --port=<port-exposed>
```

## CKAD Practice #4 (NAMESPACES)

```
k get namespaces
k get pods -n <namespace-name>
k get pods --all-namespaces
k run redis --image=redis --namespace=finance
```

## CKAD Practice #5 (IMPERATIVE MANAGEMENT)

```pod.yaml
# k run redis --image=redis:alpine --dry-run=client -o yaml > redis.yaml
# (and add the tier=db label)

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: redis
    tier: db
  name: redis
spec:
  containers:
  - image: redis:alpine
    name: redis
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```service.yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-service
spec:
  selector:
    tier: db
  ports:
    - protocol: TCP
      port: 6379
      targetPort: 6379
```

```deployment.yaml
# k create deployment webapp --image=kodekloud/webapp-color --replicas=3 --dry-run=client -o yaml > webapp-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: webapp
  name: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: webapp
    spec:
      containers:
      - image: kodekloud/webapp-color
        name: webapp-color
        resources: {}
status: {}
```

```custom-nginx.yaml
# k run custom-nginx --image=nginx --port=8080 -o yaml --dry-run=client > custom-nginx.yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: custom-nginx
  name: custom-nginx
spec:
  containers:
  - image: nginx
    name: custom-nginx
    ports:
    - containerPort: 8080
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```custom-webapp.yaml
# k run httpd --image=httpd:alpine --port=80 --expose=true

apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  name: httpd
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: httpd
status:
  loadBalancer: {}
---
---
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: httpd
  name: httpd
spec:
  containers:
  - image: httpd:alpine
    name: httpd
    ports:
    - containerPort: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

## CKAD Practice #6 (ConfigMaps)

```bash
k get pods webapp -o yaml > pod.yaml

k create configmap \
  webapp-config --from-literal=APP_COLOR=blue \
               --from-literal=APP_FIZZ=buzz
```

```yml
# Pod using ConfigMap
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
    - name: webapp
      image: webapp
      ports:
        - containerPort: 8080
      env:
        - name: APP_COLOR
          valueFrom:
            configMapKeyRef:
              name: webapp-config
              key: APP_COLOR
```

## CKAD Practice #7 (Secrets)

```bash
k describe secrets secret-name
k get secrets secret-name -o yaml
echo -n 'text_to_encode' | base64
k create secret generic \
  secret-name --from-literal=KEY=value
```

## CKAD Practice #8 (Security Contexts)

```bash
k describe pods <pod-name>
k exec -it <pod-name> -- sh
ps aux
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
  namespace: default
spec:
  containers:
    - command:
        - sleep
        - '4800'
      image: ubuntu
      name: ubuntu
      securityContext:
        runAsUser: 1010
```
