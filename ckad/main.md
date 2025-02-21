# CKAD cheatsheets and resources

## Topics

- [Docker Commands](./00-docker-commands.md)
- [CKAD Exercises](./exercises.md)
- [Pods](./01-pods.md)
- [Replicasets](./02-replicasets.md)
- [Deployments](03-deployments.md)
- [Namespaces](./04-namespaces.md)

## Exam Resources

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

*commands* will replace the ENTRYPOINT values of the Dockerfile

*args* will replace the CMD values of the Dockerfile

```yml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
  - name: ubuntu-sleeper
    image: ubuntu-sleeper
    command: ["sleep2"]
    args: ["10"]
    ports:
    - containerPort: 80
```
