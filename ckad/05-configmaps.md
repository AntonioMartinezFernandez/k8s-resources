## ConfigMaps

```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
data:
  APP_FOO: bar
  APP_FIZZ: buzz
```

```yml
# Pod using ConfigMap
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
        - configMapRef:
          name: myapp-config
```

```
# file-with-values.properties

APP_FOO=BAR
APP_FIZZ=BUZZ
```

```bash
# Imperative
k create configmap \
  myapp-config --from-literal=APP_FOO=bar \
               --from-literal=APP_FIZZ=buzz

# From file
k create configmap \
  myapp-config --from-file=file-with-values.properties

# Declarative
k create -f manifest-file.yaml
```
