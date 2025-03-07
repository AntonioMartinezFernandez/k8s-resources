## Volumes, Persisteng Volumes and Persistent Volume Claims

Volumes are storage resources that are used by containers to store data. They exist as long as the pod exists and are erased when the pod is deleted.

Persistent Volumes (PVs) are storage resources in the cluster that exist independently of pods. They are created by the administrator and can be used by pods to store data persistently.

Persistent Volume Claims (PVCs) are requests for storage by users. A PVC specifies the size, access modes, and other requirements for a PV, and Kubernetes will try to match it with an available PV.

A StorageClass in Kubernetes defines the type of storage that should be used for a Persistent Volume (PV). It provides a way to dynamically provision PVs with different characteristics, such as performance, cost, or backup policies.

A StatefulSet is a Kubernetes controller used to manage stateful applications. Unlike a Deployment (which is used for stateless applications), a StatefulSet ensures that each pod has a unique identity and stable, persistent storage, which is essential for applications that require stable network identities and storage over time (like databases or distributed systems).

```yaml
# Pod with HOST PATH volume

apiVersion: v1
kind: Pod
metadata:
  name: hostpath-example-linux
spec:
  os: { name: linux }
  nodeSelector:
    kubernetes.io/os: linux
  containers:
    - name: example-container
      image: registry.k8s.io/test-webserver
      volumeMounts:
        - mountPath: /foo
          name: example-volume
          readOnly: true
  volumes:
    - name: example-volume
      # mount /data/foo, but only if that directory already exists
      hostPath:
        path: /data/foo # directory location on host
        type: Directory # this field is optional
```

```yaml
# Persistent volume

apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce # ReadWriteOnce / ReadOnlyMany / ReadWriteMany / ReadWriteOncePod
  persistentVolumeReclaimPolicy: Recycle # Retain / Delete / Recycle
  storageClassName: slow
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /tmp
    server: 172.17.0.2
```

```yaml
# Persistent volume claim

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 1Gi
  storageClassName: slow
  selector:
    matchLabels:
      release: 'stable'
    matchExpressions:
      - { key: environment, operator: In, values: [dev] }
```

```yaml
# Pod using PVC

apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
        - mountPath: '/var/www/html'
          name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```

```yaml
# StorageClass
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: delayed-volume-sc
  annotations:
    storageclass.kubernetes.io/is-default-class: 'false'
provisioner: kubernetes.io/no-provisioner
reclaimPolicy: Delete # default value is Delete
allowVolumeExpansion: true
mountOptions:
  - discard # this might enable UNMAP / TRIM at the block storage layer
volumeBindingMode: WaitForFirstConsumer
```

```yaml
# StatefulSet (with Headless Service -service without clusterIP-)

apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
    - port: 80
      name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: 'nginx' # Headless Service name
  replicas: 3 # by default is 1
  minReadySeconds: 10 # by default is 0
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
        - name: nginx
          image: registry.k8s.io/nginx-slim:0.24
          ports:
            - containerPort: 80
              name: web
          volumeMounts:
            - name: www
              mountPath: /usr/share/nginx/html
  volumeClaimTemplates: # Equivalent to an independent PersistentVolumeClaim
    - metadata:
        name: www
      spec:
        accessModes: ['ReadWriteOnce']
        storageClassName: 'my-storage-class' # StorageClass associated
        resources:
          requests:
            storage: 1Gi
```
