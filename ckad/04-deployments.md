## Deployments

**Deployment strategies**

- RECREATE STRATEGY: stop all the pods of an application and start new pods (with downtime between stop&start)
- ROLLING UPDATE STRATEGY: take down and bring up application pods one by one. Is de DEFAULT deployment strategy
- BLUE-GREEN STRATEGY: bring up all the new pods (green), redirect all the traffic to the new pods, and take down all the old pods (blue). We can do it with native kubernetes deploying a new deployment, pointing the service to the new deployment (e.g. using the 'version' matchLabel selector), and taking down the old deployment
- CANARY STRATEGY: bring up some pod of the new service version, redirect a percentage of the traffic to this new version, and if there is no errors, increase the percentage of traffic to the new version up to 100% while decreasing the traffic to the old version to 0%. We can do it with native kubernetes using a common service pointing to both versions (deployments with same serviceName matchLabel selector but with different 'version' metadata label), and increasing/decreasing the number of replicas of each deployment

**Terminology**
Rollout: deploying the containers in the cluster
Istio: service mesh for kubernetes which facilitates deployment strategies like blue-green or canary
Argo Rollouts: kubernetes controller which provide advanced deployment capabilities. Have integrations with service mesh like Istio

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
      version: v1
  template:
    metadata:
      labels:
        app: nginx
        version: v1
        type: backend
    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
            - containerPort: 80
```

```bash
k create deployment nginx --image=<image-name>:<image-version>
k create -f <deployment-definition-file>
k get deployments

# ROLLOUTS
k rollout status deployment/<deployment-name> # Check rollout status
k rollout history deployment/<deployment-name> # Check rollout history
k rollout undo deployment/<deployment-name> # Undo the rollout
k rollout history deployment <deployment-name> --revision=1 # Check revision 1 of the rollout
k rollout undo deployment nginx --to-revision=1 # rollout to revision 1

# SET DEPLOYMENT IMAGE ON THE FLY
k set image deployment/<deployment-name> <container-name>=<image-name>:<image-version>
k set image deployment/<deployment-name> <container-name>=<image-name>:<image-version> --record=true # save the command in the revision-change cause

# UPDATE DEPLOYMENT (ROLLING UPDATE)
k apply -f <deployment-definition-file>
```
