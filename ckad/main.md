# CKAD cheatsheets and resources

## Topics

- [Docker Commands](./00-docker-commands.md)
- [CKAD Exercises](./00-exercises.md)
- [Pods](./01-pods.md)
- [Jobs and Cronjobs](./02-jobs-and-cronjobs.md)
- [Replicasets](./03-replicasets.md)
- [Deployments](04-deployments.md)
- [Namespaces](./05-namespaces.md)
- [ConfigMaps](./06-configmaps.md)
- [Secrets](./07-secrets.md)
- [Security Contexts](./08-security-contexts.md)
- [Service Accounts](./09-service-accounts.md)
- [Resource Requirements](./10-resource-requirements.md)
- [Taints and Tolerations](./11-taints-and-tolerations.md)
- [Node Selectors and Affinity](./12-node-selectors-and-affinity.md)
- [Multicontainer Pods](./13-multicontainer-pods.md)
- [Observability](./14-observability.md)
- [Services](./15-services.md)
- [Ingress](./16-ingress.md)
- [Network Policies](./17-network-policies.md)

## Exam Resources

- https://www.linkedin.com/pulse/my-ckad-exam-experience-atharva-chauthaiwale/
- https://medium.com/@harioverhere/ckad-certified-kubernetes-application-developer-my-journey-3afb0901014

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

### Dockerfile vs K8s Pod

**Dockerfile**

```yml
FROM ubuntu

ENTRYPOINT["sleep"]
CMD["5"]
```

**Pod**

_commands_ will replace the ENTRYPOINT values of the Dockerfile

_args_ will replace the CMD values of the Dockerfile

```yml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      command: ['sleep2']
      args: ['10']
      ports:
        - containerPort: 80
```

## Encode/Decode base64 from terminal

```bash
# Encode
echo -n 'text_to_encode' | base64

# Decode
echo -n 'dGV4dF90b19lbmNvZGU=' | base64 -d
```

## Encryption at rest for etcd data

- [Encrypt secrets data at rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)

## Count output lines

```bash
# pipe to "wc -l" return the number of lines
k get clusterroles.rbac.authorization.k8s.io --all-namespaces | wc -l
```
