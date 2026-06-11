# Nginx Web Server Repository

Bu repository, Nginx web server'ının OpenShift/Kubernetes yapılandırmalarını içerir. Uygulama, bir base bileşeni ve her ortam için ayrı overlay konfigürasyonları ile dağıtılır.

## Klasör Yapısı

```
repo-nginx/
├── components/
│   └── nginx/base/
│       ├── kustomization.yaml      # Base kaynak tanımı
│       ├── nginx-deployment.yaml   # Nginx deployment
│       ├── nginx-service.yaml      # ClusterIP Service
│       ├── nginx-pvc.yaml          # PersistentVolumeClaim
│       ├── nginx-route.yaml        # OpenShift Route
│       └── nginx-custom-config.yaml# Nginx konfigürasyonu
└── overlays/
    ├── test/                       # Test ortamı
    │   ├── kustomization.yaml
    │   └── index.html
    ├── pre-prod/                   # Pre-production ortamı
    │   ├── kustomization.yaml
    │   └── index.html
    └── prod/                       # Production ortamı
        ├── kustomization.yaml
        └── index.html
```

## Base Bileşen

### Base Kustomization (`components/nginx/base/kustomization.yaml`)

Base kustomization, Nginx bileşenini tanımlar ve aşağıdaki kaynakları içerir:

- `nginx-deployment.yaml`
- `nginx-service.yaml`
- `nginx-pvc.yaml`
- `nginx-route.yaml`
- `nginx-custom-config.yaml`

### Nginx Deployment

`components/nginx/base/nginx-deployment.yaml` dosyası:

- `registry.redhat.io/rhel8/nginx-120:latest` imajını kullanır
- `command: ["/usr/libexec/s2i/run"]` ile S2I uyumlu çalışma sağlar
- Container `8080` portunu `http` olarak sunar
- `index.html` dosyasını ConfigMap üzerinden `/opt/app-root/src/index.html` olarak mount eder
- Kaynak istekleri ve limitleri tanımlanmıştır

### Nginx Service

`components/nginx/base/nginx-service.yaml`:

- `ClusterIP` tipi Service
- Port 80 üzerinden HTTP trafiği hedefler
- `app: nginx` selector ile bağlantı kurar

### Nginx PVC

`components/nginx/base/nginx-pvc.yaml`:

- `nginx-html` PersistentVolumeClaim
- `ReadWriteOnce` erişim modu
- `1Gi` depolama talebi

### Nginx Route

`components/nginx/base/nginx-route.yaml`:

- OpenShift `Route` kaynağı tanımlar
- `nginx` servisinin `http` portuna yönlendirir

### Nginx Konfigürasyonu

`components/nginx/base/nginx-custom-config.yaml`:

- Nginx konfigürasyonunu `ConfigMap` olarak tutar
- `/tmp/html` kökünü ve `index.html` için root ayarlarını içerir

## Overlay Konfigürasyonları

### Test Ortamı (`overlays/test`)

- `resources: - ../../components/nginx/base`
- Namespace: `web-pay`
- `nginx` için 1 replika
- `commonLabels: environment: test`
- `configMapGenerator` ile `index.html` dosyası embed edilir
- `index.html` içeriği `Hello World from TEST Cluster` şeklindedir

### Pre-Prod Ortamı (`overlays/pre-prod`)

- `resources: - ../../components/nginx/base`
- Namespace: `web-pay`
- `nginx` için 2 replika
- `commonLabels: environment: pre-prod`
- `index.html` içeriği `Hello World from PRE-PROD Cluster` şeklindedir

### Prod Ortamı (`overlays/prod`)

- `resources: - ../../components/nginx/base`
- Namespace: `web-pay`
- `nginx` için 3 replika
- `commonLabels: environment: prod`
- `configMapGenerator` ile `index.html` dosyası embed edilir

## Deployment

### Test Ortamı

```bash
kubectl apply -k repo-nginx/overlays/test/
```

### Pre-Prod Ortamı

```bash
kubectl apply -k repo-nginx/overlays/pre-prod/
```

### Prod Ortamı

```bash
kubectl apply -k repo-nginx/overlays/prod/
```

## Önemli Notlar

- Base yapı `components/nginx/base` altında tanımlıdır.
- Overlay'ler ortam bazlı içerik ve replika sayısını yönetir.
- `nginx-route.yaml` OpenShift `Route` kaynağı sağlar.
