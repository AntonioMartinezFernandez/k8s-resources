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
      args: []
      ports:
        - containerPort: 80
      env:
        - name: foo
          value: bar
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
k exec -it <pod-name> -- sh
k exec -it -n <namespace> -c <pod-name> -- sh -c "clear; (bash || ash || sh)"
k exec --stdin --tty <pod-name> -- /bin/sh

# delete pod
k delete pod echo

# force pod deletion
k delete pod <pod-name> --force
```
