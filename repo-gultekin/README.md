# repo-gultekin

OpenShift üzerinde Apache web sunucusu çalıştırmak için basit bir Kustomize şablonudur.

## Yapı

- `components/apache/base`: Apache deployment, service, route ve configmap.
- `overlays/test`: test ortamı için 1 replika.
- `overlays/pre-prod`: pre-prod ortamı için 2 replika.
- `overlays/prod`: prod ortamı için 3 replika.

## Kullanım

```bash
cd repo-gultekin
kubectl apply -k overlays/test
kubectl apply -k overlays/pre-prod
kubectl apply -k overlays/prod
```
