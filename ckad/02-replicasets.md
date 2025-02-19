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