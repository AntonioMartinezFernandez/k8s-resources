## Secrets

```yml
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secrets
data:
  APP_FOO_SECRET: YmFy
  APP_FIZZ_SECRET: YnV6eg==
```

```yml
# Pod using Secret
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx:1.14.2
      ports:
        - containerPort: 80
      envFrom:
        - secretRef:
          name: myapp-secrets
```

```
# file-with-values.properties

APP_FOO_SECRET=YmFy
APP_FIZZ_SECRET=YnV6eg==
```

```bash
# Imperative
k create secret generic \
  myapp-secrets --from-literal=APP_FOO_SECRET=YmFy \
                --from-literal=APP_FIZZ_SECRET=YnV6eg==

# From file
k create secret generic \
  myapp-secrets --from-file=file-with-values.properties

# Declarative
k create -f manifest-file.yaml
```
