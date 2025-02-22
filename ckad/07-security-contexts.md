## Security contexts

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000 # for any Containers in the Pod, all processes run with user ID 1000
    runAsGroup: 3000 # specifies the primary group ID of 3000 for all processes within any containers of the Pod
    fsGroup: 2000 # all processes of the container are also part of the supplementary group ID 2000
    supplementalGroups: [4000] # all processes of the container are also part of the specified groups
  volumes:
    - name: sec-ctx-vol
      emptyDir: {}
  containers:
    - name: sec-ctx-demo
      image: busybox:1.28
      command: ['sh', '-c', 'sleep 1h']
      volumeMounts:
        - name: sec-ctx-vol
          mountPath: /data/demo
      securityContext: # security context at container level overrides the security context at pod level
        allowPrivilegeEscalation: false
        runAsUser: 1010
        capabilities:
          add: ['NET_ADMIN', 'SYS_TIME'] # extend the allowed capabilities for the container
```
