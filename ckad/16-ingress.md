## Ingress

- Ingress Controller: proxy service deployed to manage the traffic (nginx, traefik, haproxy, ...). There is no a default ingress controller installed in kubernetes, so it have to be installed.
- Ingress Resources: ingres controller configuration.
- Gateway: successor to the ingress. Docs: https://kubernetes.io/docs/concepts/services-networking/gateway/

```bash
# Create ingress in the imperative way
k create ingress <ingress-name> --rule="host/path=service:port"

# Example
k create ingress ingress-host --rule="foo.bar.com/fizz=service1:80"

# Edit the Ingress definition on-the-fly
k edit ingress <ingress-name>
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-host
spec:
  rules:
    - host: 'foo.bar.com' # If the 'host' parameter is not defined, is equivalent to "*" (all hosts)
      http:
        paths:
          - pathType: Prefix
            path: '/fizz'
            backend:
              service:
                name: service1
                port:
                  number: 80
```
