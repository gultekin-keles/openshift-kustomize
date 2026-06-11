# Product Catalog Repository

Bu repository, Product Catalog uygulamasının OpenShift/Kubernetes yapılandırmalarını içerir. Uygulama üç katmanlıdır: Database, Server (Backend) ve Client (Frontend).

## Klasör Yapısı

```
repo-product-catalog/
├── components/
│   ├── client/base/
│   │   ├── kustomization.yaml
│   │   ├── client-deployment.yaml
│   │   ├── client-route.yaml
│   │   ├── client-service.yaml
│   │   └── config/
│   │       └── config.js
│   ├── server/base/
│   │   ├── kustomization.yaml
│   │   ├── server-deployment.yaml
│   │   ├── server-route.yaml
│   │   ├── server-service.yaml
│   │   ├── default-view-rolebinding.yaml
│   │   └── config/
│   │       └── application.properties
│   └── database/base/
│       ├── kustomization.yaml
│       ├── db-deployment.yaml
│       ├── db-service.yaml
│       ├── db-pvc.yaml
│       ├── db-secret.yaml
│       └── config/
│           ├── 90-init-database.sh
│           ├── import.sql
│           └── schema.sql
└── overlays/
    ├── test/
    │   ├── kustomization.yaml
    │   └── patches/
    │       └── route-host.yaml
    ├── pre-prod/
    │   └── kustomization.yaml
    └── prod/
        ├── kustomization.yaml
        └── patches/
            ├── db-pvc-size.yaml
            └── secure-route.yaml

└── kustomization.yaml
```

## Root Kustomization

`repo-product-catalog/kustomization.yaml` dosyası şu kaynakları içerir:

- `components/client/base`
- `components/server/base`
- `components/database/base`

Root kustomization ayrıca `server` ve `client` için default replika değerlerini tutar.

## Base Bileşenler

### Client Base

`components/client/base/kustomization.yaml`:

- `client-service.yaml`
- `client-route.yaml`
- `client-deployment.yaml`

Bu yapı frontend için ClusterIP Service, Route ve Deployment sağlar.

### Server Base

`components/server/base/kustomization.yaml`:

- `default-view-rolebinding.yaml`
- `server-service.yaml`
- `server-route.yaml`
- `server-deployment.yaml`
- `config/application.properties` dosyasını `ConfigMap` olarak oluşturur

### Database Base

`components/database/base/kustomization.yaml`:

- `db-pvc.yaml`
- `db-secret.yaml`
- `db-service.yaml`
- `db-deployment.yaml`
- `config/90-init-database.sh`, `config/import.sql` ve `config/schema.sql` dosyalarını `ConfigMap` olarak oluşturur

## Overlay Konfigürasyonları

### Test Ortamı (`overlays/test`)

- `resources: - ../../`
- Namespace: `product-catalog`
- `server` ve `client` için 1 replika
- `commonLabels: environment: test`
- `patchesStrategicMerge` ile `patches/route-host.yaml` uygulanır

`route-host.yaml` içinde test ortamı için aşağıdaki hostlar tanımlıdır:

- `client.apps.baremetal.konsalt.info`
- `server.apps.baremetal.konsalt.info`

### Pre-Production Ortamı (`overlays/pre-prod`)

- `resources: - ../../`
- Namespace: `product-catalog`
- `server` ve `client` için 2 replika
- `commonLabels: environment: pre-prod`

### Production Ortamı (`overlays/prod`)

- `resources: - ../../`
- Namespace: `product-catalog`
- `server` ve `client` için 4 replika
- `commonLabels: environment: prod`
- `patchesStrategicMerge` ile:
  - `patches/db-pvc-size.yaml`
  - `patches/secure-route.yaml`

`db-pvc-size.yaml` prod ortamında database PVC boyutunu `10Gi` olarak artırır.
`secure-route.yaml` prod ortamı için `Route` TLS terminasyonunu `passthrough` olarak ayarlar.

## Deployment

### Base Deployment

```bash
kubectl apply -k repo-product-catalog/
```

### Test Ortamı

```bash
kubectl apply -k repo-product-catalog/overlays/test/
```

### Pre-Production Ortamı

```bash
kubectl apply -k repo-product-catalog/overlays/pre-prod/
```

### Production Ortamı

```bash
kubectl apply -k repo-product-catalog/overlays/prod/
```
