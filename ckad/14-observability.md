## Observability

LIVE (Healthcheck): The pod is running, but not necessarily ready to receive traffic. If 'liveness probe' fails, the pod is restarted
READY: The pod is ready to receive traffic. If 'readiness probe' fails, k8s stop sending traffic to the pod AND the container inside the pod is restarted

```bash
# See the current status of the pod in the 'Conditions' section of the pod details
k describe pod -n <namespace> <pod-name>
k get pods -w -o wide
k delete pod --all

# See pod logs
k logs -n <namespace> -f <pod-name>
k logs -n <namespace> -f <pod-name> <pod-container>

# Monitoring cluster (metrics-server addon is needed! Docs: https://kubernetes-sigs.github.io/metrics-server)
k apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml # Install metric-server
k top node # k8s nodes info
k -n <namespace> top pod <pod-name> # pod info
```

```yaml
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

      # READINESS PROBE ALTERNATIVES
      readinessProbe:
        # Option 1
        httpGet:
          path: /api/ready
          port: 80
        # Option 2
        tcpSocket:
          port: 3306
        # Option 3
        exec:
          command:
            - cat
            - /app/is_ready
        # Define backoff policy for readiness probe
        initialDelaySeconds: 10
        periodSeconds: 5
        failureThreshold: 10 # default: 3

      # LIVENESS PROBE ALTERNATIVES
      livenessProbe:
        # Option 1
        httpGet:
          path: /api/ready
          port: 80
        # Option 2
        tcpSocket:
          port: 3306
        # Option 3
        exec:
          command:
            - cat
            - /app/is_ready
        # Define backoff policy for readiness probe
        initialDelaySeconds: 10
        periodSeconds: 5
        failureThreshold: 10 # default: 3
```
