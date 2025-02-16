# CKAD cheatsheets and resources

## Resources

- https://kubernetes.io/docs/reference/kubectl/quick-reference/


During the exam, candidates may:

- review the Exam content instructions that are presented in the command line terminal.

- review Documents installed by the distribution (i.e. /usr/share and its subdirectories)

- use Packages that are part of the distribution (may also be installed by Candidate if not available by default)

-use the browser within the VM to access the following documentation: 
https://kubernetes.io/docs/, https://kubernetes.io/blog/ . This includes all available language translations of these pages (e.g. https://kubernetes.io/zh/docs/)

- CKAD ONLY: candidates can use the browser within the VM to access https://helm.sh/docs  

- use the search function provided on https://kubernetes.io/docs/ however, they may only open search results that have a domain matching the sites listed above

### Output Formats and useful options
```
-o json --- Output a JSON formatted API object.

-o name --- Print only the resource name and nothing else.

-o wide --- Output in the plain-text format with any additional information.

-o yaml --- Output a YAML formatted API object.

--dry-run=client --- By default, as soon as the command is run, the resource will be created. If you simply want to test your command, use the --dry-run=client option. This will not create the resource. Instead, tell you whether the resource can be created and if your command is right.
```

### Internal DNS

```
<object-name>.<namespace>.<object-type>.cluster.local

# Example:
db-service.prod.svc.cluster.local
```

## Pods

```yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

```bash
# Pod Docs
k explain pod

# run pod
k run echo --image hashicorp/http-echo:1.0.0

# obtain the pod info
k describe pod echo

# port forwarding a pod port
k port-forward echo 5678:5678 -n default
curl localhost:5678

# create a pod and automatically create an associated service
k run httpd --image=httpd:alpine --port=80 --expose=true

# open a shell into a pod
k exec --stdin --tty <pod-name> -- /bin/sh

# delete pod
k delete pod echo

# force pod deletion
k delete pod <pod-name> --force
```

## ReplicaSet

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: us-docker.pkg.dev/google-samples/containers/gke/gb-frontend:v5
```

```bash
# ReplicaSet Docs
k explain replicaset

# Create ReplicaSet
k create -f replica-set-file.yaml

# Get ReplicaSet info
k get replicaset

# Edit replicaset on-the-fly
k edit replicaset <replicaset-name>

# Apply a change in the number of replicas of a ReplicaSet after update the replica-set-file.yaml
k replace -f replica-set-file.yaml

# Directly scale the number of replicas of a ReplicaSet (updating the definition file)
k scale --replicas=6 -f replica-set-file.yaml

# Directly scale the number of replicas of a ReplicaSet WITHOUT update the definition file
k scale --replicas=6 replicaset <replica-set-name>

# Delete ReplicaSet
k delete replicaset <replica-set-name>
```

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

## Namespaces

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: development
  labels:
    name: development
```

```bash
k get namespaces
k get pods -n <namespace-name>
k get pods --all-namespaces
k run redis --image=redis --namespace=finance
```


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