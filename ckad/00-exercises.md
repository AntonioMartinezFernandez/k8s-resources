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

```bash
k exec webapp -- cat /log/app.log

k delete pvc claim-log-1 # Recreate it with correct access mode
```

```yaml
# Pod with volume
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
    - name: webapp
      image: kodekloud/event-simulator
      volumeMounts:
        - mountPath: /log
          name: mylogs
  volumes:
    - name: mylogs
      hostPath:
        path: /var/log/webapp
        type: DirectoryOrCreate
```

```yaml
# PV
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-log
spec:
  capacity:
    storage: 100Mi
  storageClassName: manual
  persistentVolumeReclaimPolicy: Retain
  accessModes:
    - ReadWriteMany
  hostPath:
    path: '/pv/log'
```

```yaml
# PVC
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim-log-1
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Mi
```

```yaml
# Pod with PVC attached
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
    - name: webapp
      image: kodekloud/event-simulator
      volumeMounts:
        - mountPath: /log
          name: mylogs
  volumes:
    - name: mylogs
      persistentVolumeClaim:
        claimName: claim-log-1
```

## CKAD Practice #20 (Kubeconfig)

```bash
k config view
k config set-context develop --kubeconfig=/root/my-kube-config

vi .bashrc
# include line:
# export KUBECONFIG=/root/my-kube-config
source ~/.bashrc
```

## CKAD Practice #21 (RBAC)

```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml
k get roles --all-namespaces
k -n kube-system describe role kube-proxy
k edit roles dev-user

k get clusterroles.rbac.authorization.k8s.io --all-namespaces | wc -l

k describe clusterrolebindings.rbac.authorization.k8s.io cluster-admin


```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: blue
  name: developer
rules:
  - apiGroups: [''] # "" indicates the core API group
    resources: ['pods']
    verbs: ['get', 'create', 'update', 'delete', 'watch']
  - apiGroups: ['apps'] # "" indicates the core API group
    resources: ['deployments']
    verbs: ['create']
```

## CKAD Practice #22 (Admission Controllers)

```bash
k exec -n kube-system kube-apiserver-controlplane -- kube-apiserver -h

cat /etc/kubernetes/manifests/kube-apiserver.yaml
vi /etc/kubernetes/manifests/kube-apiserver.yaml
# adding line: - --enable-admission-plugins=NamespaceAutoProvision
# adding line: - --disable-admission-plugins=DefaultStorageClass

k run --image=nginx nginx -n blue # blue namespace auto-created
# NOTE that NamespaceAutoProvision is deprecated!!!
# Now is used NamespaceLifecycle and namespace auto-creation is not supported (and default namespaces can't be deleted)

ps -ef | grep kube-apiserver | grep admission-plugins # check the processes to see enabled and disabled plugins
```

## CKAD Practice #23 (Validating and Mutating Admission Controllers)

```bash
kubectl create secret tls tls-secret --cert=path/to/tls.crt --key=path/to/tls.key -n webhook-demo
k get secrets -n webhook-demo


k edit pods pod-with-defaults # for checking security context
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webhook-server
  namespace: webhook-demo
  labels:
    app: webhook-server
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webhook-server
  template:
    metadata:
      labels:
        app: webhook-server
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 1234
      containers:
        - name: server
          image: stackrox/admission-controller-webhook-demo:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 8443
              name: webhook-api
          volumeMounts:
            - name: webhook-tls-certs
              mountPath: /run/secrets/tls
              readOnly: true
      volumes:
        - name: webhook-tls-certs
          secret:
            secretName: webhook-server-tls
---
apiVersion: v1
kind: Service
metadata:
  name: webhook-server
  namespace: webhook-demo
spec:
  selector:
    app: webhook-server
  ports:
    - port: 443
      targetPort: webhook-api
---
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
metadata:
  name: demo-webhook
webhooks:
  - name: webhook-server.webhook-demo.svc
    clientConfig:
      service:
        name: webhook-server
        namespace: webhook-demo
        path: '/mutate'
      caBundle: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURQekNDQWllZ0F3SUJBZ0lVZUo4VFJUK0JOMFE3MUppMi9NVmV4dnFUUkdnd0RRWUpLb1pJaHZjTkFRRUwKQlFBd0x6RXRNQ3NHQTFVRUF3d2tRV1J0YVhOemFXOXVJRU52Ym5SeWIyeHNaWElnVjJWaWFHOXZheUJFWlcxdgpJRU5CTUI0WERUSTFNRE14TmpFeU16VXhObG9YRFRJMU1EUXhOVEV5TXpVeE5sb3dMekV0TUNzR0ExVUVBd3drClFXUnRhWE56YVc5dUlFTnZiblJ5YjJ4c1pYSWdWMlZpYUc5dmF5QkVaVzF2SUVOQk1JSUJJakFOQmdrcWhraUcKOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQW1BV2FZL25MN2gyWXp2YTBrcG5Gcmd4Si9leVpIYm9ZQnVwRApiWkhpTk9IbmN6QytIQzV1M0ErRzFEUnVTM2hjSzhYdy9xNnQ1a01xdU5Ld3dmVFE5SXlMdWppSnhsL2pvQ0xuClJUckRyK2hOSGdwaDQrZHYxb0FNa3hRQ2ZlLzhOb2wxZjhBb1RBUmo2RzN2cmlrSnczTzlvUFJva3g0dVVzNWgKeGQwNVhwdnRIZ0o5MDhlaEdVdGJXS2dqY0dyVFJrM2tRYWw1RGUzVEVrNkNHM3dUY1p1Y2tsSGhEa1ZYbHVpWQp1Q0dVMENQblpFM3AvVTJDNWpxS09aTnJGUnEra2VoanMzQ25CazRod1MrQkZoWHgvdnJDdldqTEFJRnBURDNICkZZY1dDNGtPZDFYVU1WamhRbEtOQnQzcnkrNXBobXVIMFpjUWZwWStyZTZYVkI1YnZRSURBUUFCbzFNd1VUQWQKQmdOVkhRNEVGZ1FVOFBiQnZxeXNweERnOU1keHpNU3NWUkI3TG1Zd0h3WURWUjBqQkJnd0ZvQVU4UGJCdnF5cwpweERnOU1keHpNU3NWUkI3TG1Zd0R3WURWUjBUQVFIL0JBVXdBd0VCL3pBTkJna3Foa2lHOXcwQkFRc0ZBQU9DCkFRRUFnckJrMGhSZHhKTms2STAyaUE3cFlwbEllZUFlNW9pYm0wZlJFQXpINkpjblR0TFlTOGYwVDlTSW92NEEKTXdreDRTNTFvT3pDQ1hPSzRUMHMwaDR5SU8rdi9tUlF6UitYYThjdlAydjJsZzFIamZmTnAyd3dUY2E3VW1GQgpoWnMzdU8zUzBkTUpTZGhKK0hTMUhGUHRpNFVzRTBGenNranNPQ1dPSG11aXBISlNvN1hSUTNIQjVReXAyMkgxCnQvWXVGMXdxRkJFM2crbjFCWXprUlc2NjVKbUdLUWlCRklTcW01cnAxWC8vTHJGSTk5Ym8wcXZENk5yY2UzZ1oKMEd3dzR1NVFTZmJUdlJwRlJYaCtBVk04S1BZYURQR2M0Z0k4YUwvbmFTKzYxU2I1NkF2VURYQ1BWWjJlR3FTWApvWkprcVJYbkNlVldBNTNMTjVKSmx5NGE4dz09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    rules:
      - operations: ['CREATE']
        apiGroups: ['']
        apiVersions: ['v1']
        resources: ['pods']
    admissionReviewVersions: ['v1beta1']
    sideEffects: None
---
```

## CKAD Practice #24 (API Versions / Deprecations)

```bash
k api-resources

k proxy
curl localhost:8001/apis/rbac.authorization.k8s.io

vi /etc/kubernetes/manifests/kube-apiserver.yaml # Add "- --runtime-config=rbac.authorization.k8s.io/v1alpha1

# Install kubectl convert: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-kubectl-convert-plugin

k convert -f ingress-old.yaml -o yaml > new.yaml
k create -f new.yaml
```

## CKAD Practice #25 (Helm)

```bash
cat /etc/os-release # Discover operating system installed

# Install Helm
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh

# Get environment variables
helm env

# Get helm client version
helm version
```
