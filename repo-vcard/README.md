# VCard Application - Kustomize Structure

Bu yapı, VCard uygulamasının OpenShift üzerinde Kustomize ile dağıtılması için tasarlanmıştır.

## Dizin Yapısı

```
repo-vcard/
├── components/
│   └── vcard/
│       └── base/              # Temel Kubernetes kaynakları
│           ├── vcard-deployment.yaml
│           ├── vcard-service.yaml
│           ├── vcard-route.yaml
│           └── kustomization.yaml
├── overlays/
│   └── vcard/
│       ├── test/              # Test ortamı (1 replica, test- prefix)
│       ├── pre-prod/          # Pre-prod ortamı (2 replica, preprod- prefix)
│       └── prod/              # Prod ortamı (3 replica, standart isimler)
├── kustomization.yaml         # Root Kustomization
└── README.md
```

## Ortamlar

### Test
```bash
kubectl kustomize overlays/vcard/test
kubectl apply -k overlays/vcard/test
```
- 1 replica
- namePrefix: `test-`, nameSuffix: `-test`
- Label: `environment: test`

### Pre-Production
```bash
kubectl kustomize overlays/vcard/pre-prod
kubectl apply -k overlays/vcard/pre-prod
```
- 2 replica
- namePrefix: `preprod-`, nameSuffix: `-preprod`
- Label: `environment: pre-prod`

### Production
```bash
kubectl kustomize overlays/vcard/prod
kubectl apply -k overlays/vcard/prod
```
- 3 replica
- Standart isimler (prefix/suffix yok)
- Label: `environment: prod`

## Kaynaklar

- **Deployment**: Java uygulaması (OpenJDK 17), 2 port (8080, 8443)
- **Service**: ClusterIP türü, her iki porta erişim
- **Route**: Edge TLS encryption ile HTTPS desteği

## Build ve Deploy

```bash
# Belirli bir ortamı build et
kubectl kustomize overlays/vcard/test

# Deploy et
kubectl apply -k overlays/vcard/prod
```
