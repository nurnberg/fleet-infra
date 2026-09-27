Create cluster in k3d:

```sh
k3d cluster create staging \
  -p "80:80@loadbalancer" \
  -p "443:443@loadbalancer" \
  -p "8000:8000@loadbalancer" \
  --k3s-arg "--disable=traefik@server:0"
```
