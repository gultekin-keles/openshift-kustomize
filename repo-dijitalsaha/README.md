# repo-dijitalsaha Kustomize Template

Dijitalsaha uygulaması için OpenShift Kustomize template'i.

## Yapı

```
repo-dijitalsaha/
├── README.md
├── kustomization.yaml
├── components/
│   ├── saha-manager/
│   │   └── base/
│   │       ├── deployment.yaml
│   │       ├── service.yaml
│   │       ├── route.yaml
│   │       └── kustomization.yaml
│   ├── saha-web/
│   │   └── base/
│   │       ├── deployment.yaml
│   │       ├── service.yaml
│   │       ├── route.yaml
│   │       └── kustomization.yaml
│   ├── saha-sync/
│   │   └── base/
│   │       ├── deployment.yaml
│   │       ├── service.yaml
│   │       ├── route.yaml
│   │       └── kustomization.yaml
│   └── saha-pars/
│       └── base/
│           ├── deployment.yaml
│           ├── service.yaml
│           ├── route.yaml
│           └── kustomization.yaml
└── overlays/
    └── dijitalsaha/
        ├── test/
        │   └── kustomization.yaml
        ├── pre-prod/
        │   └── kustomization.yaml
        └── prod/
            └── kustomization.yaml
```

## Bileşenler

- **saha-manager**: Yönetim servisi (2 replika)
- **saha-web**: Web arayüzü (1 replika)
- **saha-sync**: Senkronizasyon servisi (2 replika)
- **saha-pars**: Parser servisi (2 replika)

## Kullanım

```bash
# Kök kustomize oluştur
cd repo-dijitalsaha
kubectl kustomize .

# Test overlay
kubectl kustomize overlays/dijitalsaha/test

# Pre-prod overlay
kubectl kustomize overlays/dijitalsaha/pre-prod

# Prod overlay
kubectl kustomize overlays/dijitalsaha/prod
```

## Ortam Özellikleri

- `test`: Minimal yapılandırma, 1 replika
- `pre-prod`: Orta seviye yapılandırma, 2 replika
- `prod`: Yüksek kullanılabilirlik, 3 replika

## S2I Temizleme

Tüm S2I geçiş ürünleri temizlenmiştir:
- Gereksiz annotations kaldırıldı
- Basit, temiz YAML yapısı oluşturuldu
- Kustomize ile yönetim yapısı uygulandı
- Ortam-spesifik özelleştirmeler overlays'de tanımlandı
