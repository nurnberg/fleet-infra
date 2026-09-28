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

Master Clear:
```sh
# 1. Hard-delete the stuck Canary object to completely wipe Flagger's failure history
kubectl delete canary podinfo -n podinfo

# 2. Restart the loadtester pod to guarantee its web server hooks into port 8080 cleanly
kubectl rollout restart deployment flagger-loadtester -n kube-system

# 3. Wait a few seconds for the pod to shift to a stable "Running" state
kubectl rollout status deployment flagger-loadtester -n kube-system

# 4. Tell Flux to instantly re-hydrate the clean, updated Canary definitions
flux reconcile source git flux-system
flux reconcile kustomization apps
```