## CKAD Practice #1 (PODS)

```bash
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

```bash
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

```bash
k describe deployments frontend-deployment
k create deployment --help
k create deployment <deployment-name> --image=<docker-image> --replicas=<number-of-replicas> --port=<port-exposed>
```

## CKAD Practice #4 (NAMESPACES)

```bash
k get namespaces
k get pods -n <namespace-name>
k get pods --all-namespaces
k run redis --image=redis --namespace=finance
```

## CKAD Practice #5 (IMPERATIVE MANAGEMENT)

```yaml
# pod.yaml

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

```yaml
# service.yaml

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

```yaml
# deployment.yaml

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

```yaml
# custom-nginx.yaml

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

```yaml
# custom-webapp.yaml

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

## CKAD Practice #9 (Service Accounts)

```bash
k get serviceaccount

k create serviceaccount <serviceaccount-name>

k create token <serviceaccount-name>

k get deployment <deployment-name> -o yaml > deployment.yaml

k delete deployment <deployment-name>

k create -f deployment.yaml

k exec -it <pod-name> -- sh

cat /var/run/secrets/kubernetes.io/serviceaccount/token
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: http-echo
  namespace: http-echo
spec:
  template:
    metadata:
      labels:
        app: http-echo
    spec:
      containers:
        - image: antoniomarfer/http-echo
          imagePullPolicy: Always
          name: http-echo
          ports:
            - containerPort: 8080
              protocol: TCP
          serviceAccountName: my-custom-service-account
        resources:
          limits:
            cpu: 100m
            memory: 128Mi
          requests:
            cpu: 100m
            memory: 128Mi
```

## CKAD Practice #10 (Resource requirements)

```bash
k describe pod <pod-name>
k delete pod <pod-name>
k get pod <pod-name> -o yaml > deployment.yml
k create -f deployment.yml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: http-echo
  namespace: http-echo
spec:
  template:
    metadata:
      labels:
        app: http-echo
    spec:
      containers:
        - image: antoniomarfer/http-echo
          imagePullPolicy: Always
          name: http-echo
          ports:
            - containerPort: 8080
              protocol: TCP
          resources:
            limits:
              cpu: 100m
              memory: 128Mi
            requests:
              cpu: 100m
              memory: 128Mi
```

## CKAD Practice #11 (Taints and tolerations)

```bash
k get nodes

k describe nodes node01 | grep 'Taint'

kubectl taint nodes node01 spray=mortein:NoSchedule

k get pods -w

k describe nodes controlplane | grep 'Taint'

k taint node controlplane node-role.kubernetes.io/control-plane:NoSchedule-
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: bee
  labels:
    env: test
spec:
  containers:
    - name: bee
      image: nginx
  tolerations:
    - key: 'spray'
      operator: 'Equal'
      value: 'mortein'
      effect: 'NoSchedule'
```

## CKAD Practice #12 (Node Selectors and Affinity)

```bash
k describe nodes node01

k label nodes node01 color=blue

k create deployment blue --image=nginx --replicas=3 --dry-run=client -o yaml > blue-deployment.yaml

k create -f blue-deployment.yaml

k delete deployment blue

k get pods -o wide


```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: blue
  name: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: blue
  template:
    metadata:
      labels:
        app: blue
    spec:
      containers:
        - image: nginx
          name: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: color
                    operator: In
                    values:
                      - blue
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: red
  name: red
spec:
  replicas: 2
  selector:
    matchLabels:
      app: red
  template:
    metadata:
      labels:
        app: red
    spec:
      containers:
        - image: nginx
          name: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: node-role.kubernetes.io/control-plane
                    operator: Exists
```

## CKAD Practice #13 (Multicontainers)

```bash
k get pods yellow -o wide

k get all -n elastic-stack

k logs -n elastic-stack kibana

k exec -it -n elastic-stack app -- cat /log/app.log

k create -f elastic-app.yaml
```

```yaml
# sidecar

apiVersion: v1
kind: Pod
metadata:
  labels:
    name: app
  name: app
  namespace: elastic-stack
spec:
  volumes:
    - name: log-volume
      emptyDir: {}
  containers:
    - image: kodekloud/event-simulator
      name: app
      terminationMessagePath: /dev/termination-log
      terminationMessagePolicy: File
      volumeMounts:
        - mountPath: /log
          name: log-volume
    - image: kodekloud/filebeat-configured
      name: sidecar
      volumeMounts:
        - mountPath: /var/log/event-simulator/
          name: log-volume
```

```yaml
# initContainers

apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
    - name: myapp-container
      image: busybox:1.28
      command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
    - name: init-myservice
      image: busybox
      command: ['sh', '-c', 'git clone <some-repository-that-will-be-used-by-application> ;']
```

## CKAD Practice #14 (Observability)

```bash
k logs -f pods/<pod-name>
k top node
k -n <namespace> top pod
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: bee
  labels:
    env: test
spec:
  containers:
    - name: bee
      image: nginx
      readinessProbe:
        httpGet:
          path: /api/ready
          port: 80
      livenessProbe:
        httpGet:
          path: /api/live
          port: 80
```

## CKAD Practice #15 (POD design)

```bash
k describe deployments frontend
k edit deployments frontend # to change the number of replicas on-the-fly
```

## CKAD Practice #16 (Jobs and Cronjobs)

```bash
k create -f job.yaml
k delete job throw-dice-job

k create -f cronjob.yaml
```

```yaml
# job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: throw-dice-job
spec:
  completions: 3
  parallelism: 3
  backoffLimit: 100
  template:
    spec:
      containers:
        - name: throw-dice
          image: kodekloud/throw-dice
      restartPolicy: Never
```

```yaml
# cronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: throw-dice-cron-job
spec:
  schedule: '30 21 * * *'
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: throw-dice
              image: kodekloud/throw-dice
          restartPolicy: OnFailure
```

## CKAD Practice #17 (Service and Ingress)

```bash
k create ns ingress-nginx
k -n ingress-nginx create configmap ingress-nginx-controller
k -n ingress-nginx create serviceaccount ingress-nginx
k -n ingress-nginx create serviceaccount ingress-nginx-admission

k -n app-space edit ingress <ingress-name>
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: pay-ingress
  namespace: critical-space
  annotations:
    # Docs: https://kubernetes.github.io/ingress-nginx/examples/rewrite/
    nginx.ingress.kubernetes.io/rewrite-target: / # It redirect the 'path' used in the rules to the root path ('/') of the service
spec:
  rules:
    - http:
        paths:
          - path: /pay
            pathType: Prefix
            backend:
              service:
                name: pay-service
                port:
                  number: 8282
```

## CKAD Practice #18 (Network Policies)

```bash
k get networkpolicies --all-namespaces
k delete networkpolicies <policy-name>
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      name: internal
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              name: external
      ports:
        - protocol: TCP
          port: 8080
  egress:
    - to:
        - podSelector:
            matchLabels:
              name: external
      ports:
        - protocol: TCP
          port: 8080
    - to:
        - podSelector:
            matchLabels:
              name: database
      ports:
        - protocol: TCP
          port: 3306
```

## CKAD Practice #19 (Volumes)
