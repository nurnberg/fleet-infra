Create cluster in k3d:

```sh
k3d cluster create staging \
  -p "80:80@loadbalancer" \
  -p "443:443@loadbalancer" \
  -p "8000:8000@loadbalancer" \
  --k3s-arg "--disable=traefik@server:0"
```

HTTPRoute template:
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: my-app-route
  namespace: default       # Can be any namespace where your application service lives
spec:
  parentRefs:
    - name: traefik-gateway # Binds straight to the manual Gateway you defined in infra-configs
      namespace: kube-system # The namespace where your modern Traefik operator lives
  hostnames:
    - "my-app.docker.localhost" # k3d natively loops *.localhost routing straight to your host machine
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /
      backendRefs:
        - name: my-app-service # Your standard K8s ClusterIP Service name
          port: 80             # The port exposed on that Service resource
```