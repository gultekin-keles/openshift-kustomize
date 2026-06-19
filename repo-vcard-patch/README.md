# VCard Application with Patches - Kustomize Structure

Bu yapı, VCard uygulamasının OpenShift üzerinde Kustomize ile dağıtılması için tasarlanmıştır. **Patch örnekleri** içerir.

## Dizin Yapısı

```
repo-vcard-patch/
├── components/
│   └── vcard/
│       └── base/              # Temel Kubernetes kaynakları
│           ├── vcard-deployment.yaml
│           ├── vcard-service.yaml
│           ├── vcard-route.yaml
│           └── kustomization.yaml
├── overlays/
│   └── vcard/
│       ├── test/              # Test ortamı - Patch örnekleri
│       │   ├── deployment-patch.yaml
│       │   └── kustomization.yaml
│       ├── pre-prod/          # Pre-prod ortamı - Patch örnekleri
│       │   ├── deployment-patch.yaml
│       │   └── kustomization.yaml
│       └── prod/              # Prod ortamı - Patch örnekleri
│           ├── deployment-patch.yaml
│           └── kustomization.yaml
├── kustomization.yaml         # Root Kustomization
└── README.md
```

## Patch Örnekleri

### Test Cluster Patchleri (`overlays/vcard/test/deployment-patch.yaml`)

Test ortamında aşağıdaki değişiklikler uygulanır:

```yaml
# Image: Test registry'sinden image kullanır (overlay kustomization içinde images: ile değiştirilir)
image: test-registry.openshift-image-registry.svc:5000/vcard/vcard:v1.0-test

# Resource Limits (Minimum):
resources:
  requests:
    memory: "128Mi"
    cpu: "50m"
  limits:
    memory: "256Mi"
    cpu: "250m"

# Replica: 1 adet (namePrefix: test-, nameSuffix: -test)
# Labels: environment=test, cluster=test-cluster
```

**Kullanım:**
```bash
kubectl kustomize overlays/vcard/test
kubectl apply -k overlays/vcard/test
```

### Pre-Prod Cluster Patchleri (`overlays/vcard/pre-prod/deployment-patch.yaml`)

Pre-prod ortamında aşağıdaki değişiklikler uygulanır:

```yaml
# Image: Pre-prod registry'sinden image kullanır (overlay kustomization içinde images: ile değiştirilir)
image: preprod-registry.openshift-image-registry.svc:5000/vcard/vcard:v1.0-preprod

# Resource Limits (Orta):
resources:
  requests:
    memory: "512Mi"
    cpu: "200m"
  limits:
    memory: "1Gi"
    cpu: "1000m"

# Replica: 2 adet (namePrefix: preprod-, nameSuffix: -preprod)
# Labels: environment=pre-prod, cluster=pre-prod-cluster
```

**Kullanım:**
```bash
kubectl kustomize overlays/vcard/pre-prod
kubectl apply -k overlays/vcard/pre-prod
```

### Production Cluster Patchleri (`overlays/vcard/prod/deployment-patch.yaml`)

Prod ortamında aşağıdaki değişiklikler uygulanır:

```yaml
# Image: Production registry'sinden image kullanır (overlay kustomization içinde images: ile değiştirilir)
image: prod-registry.openshift-image-registry.svc:5000/vcard/vcard:v1.0

# Resource Limits (Maksimum):
resources:
  requests:
    memory: "1Gi"
    cpu: "500m"
  limits:
    memory: "2Gi"
    cpu: "2000m"

# Rolling Update Strategy (Prod için optimize):
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 50%
    maxUnavailable: 0

# Replica: 3 adet (standart isimler)
# Labels: environment=prod, cluster=prod-cluster
# imagePullPolicy: Always
```

**Kullanım:**
```bash
kubectl kustomize overlays/vcard/prod
kubectl apply -k overlays/vcard/prod
```

## Ortamlar Özeti

| Ortam | Replicas | Memory Requests | CPU Requests | Memory Limits | CPU Limits | Image Registry |
|-------|----------|-----------------|--------------|---------------|-----------|---------------|
| Test | 1 | 128Mi | 50m | 256Mi | 250m | test-registry |
| Pre-Prod | 2 | 512Mi | 200m | 1Gi | 1000m | preprod-registry |
| Prod | 3 | 1Gi | 500m | 2Gi | 2000m | prod-registry |

## Patching Mekanizması

Bu yapı `patchesJson6902` kullanarak deployment yapılandırmalarını patchler ve `images:` transformasyonu ile image adresini override eder:

- **Base:** Ortak yapılandırma ve kaynaklar
- **Overlay Kustomization:** Her ortam için image override, replica sayısı ve etiketi ayarlar
- **Overlay Patchleri:** Her ortama özel resource limits, imagePullPolicy ve rolling update stratejileri

## Kustomize Komutları

### Tüm kaynakları görüntüle
```bash
# Test
kubectl kustomize overlays/vcard/test

# Pre-Prod
kubectl kustomize overlays/vcard/pre-prod

# Prod
kubectl kustomize overlays/vcard/prod
```

### Dağıt
```bash
# Test ortamına dağıt
kubectl apply -k overlays/vcard/test

# Pre-prod ortamına dağıt
kubectl apply -k overlays/vcard/pre-prod

# Prod ortamına dağıt
kubectl apply -k overlays/vcard/prod
```

### Kaldır
```bash
# Test ortamından kaldır
kubectl delete -k overlays/vcard/test

# Pre-prod ortamından kaldır
kubectl delete -k overlays/vcard/pre-prod

# Prod ortamından kaldır
kubectl delete -k overlays/vcard/prod
```

## Notlar

- Her ortam için farklı container image adresleri (test, pre-prod, prod registryleri)
- Resource requests ve limits ortamların gereksinimlerine göre ayarlanmıştır
- namePrefix ve nameSuffix ile resource isimleri ayırılmıştır
- Labels ile ortam tanımlaması yapılmıştır
- Prod ortamında `imagePullPolicy: Always` ile her deployment'da en son image çekilir

PR: add repo-vcard-patch for review (generated)
