## Multicontainer pods

Related Design Patterns:

- Sidecar: container deployed in the same pod to run an specific auxiliar task (example, send logs)
- Adapter: process which transform data before send to an external service (example, parse logs before send them to the logs collector)
- Ambassador: outsource 'switch' logic based on some specific factor (example, connect to different database depending on the environment where the app is running)

**initContainers**:
When a POD is first created the initContainer is run, and the process in the initContainer must run to a completion before the real container hosting the application starts

```yaml
# sidecar example
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
# initContainer example
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
