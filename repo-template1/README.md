# repo-template1

Bu proje, çoklu servis / çoklu deployment içeren bir OpenShift Kustomize template örneğidir.
Amaç: hiç bilmeyen bir kişinin dahi adım adım düzenleyip kendi uygulamasını kurabileceği örnek bir şablon sağlamak.

## Dizin Yapısı

```
repo-template1/
├── README.md
├── kustomization.yaml
├── components/
│   ├── service-a/base/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── route.yaml
│   │   └── kustomization.yaml
│   ├── service-b/base/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── route.yaml
│   │   └── kustomization.yaml
│   └── database/base/
│       ├── deployment.yaml
│       └── kustomization.yaml
└── overlays/
    └── app/
        ├── patches/
        │   ├── database-resources.yaml
        │   └── secure-route.yaml
        ├── pre-prod/
        │   └── kustomization.yaml
        ├── prod/
        │   └── kustomization.yaml
        └── test/
            └── kustomization.yaml
```

## Nasıl Kullanılır?

### 1. Kopyala
```bash
cp -r repo-template1 repo-myapp
cd repo-myapp
```

### 2. YAML dosyalarını düzenle

#### `components/service-a/base/deployment.yaml`
- `name`: Uygulama adı
- `image`: Kendi image path'iniz
- `ports`: Container'ın dinlediği port
- `env`: Ortam değişkenleri
- `resources`: CPU ve bellek talepleri
- `livenessProbe` / `readinessProbe`: Sağlık kontrolleri

#### `components/service-a/base/service.yaml`
- `selector`: Deployment içindeki label ile aynı olmalı
- `port` / `targetPort`: Deployment port'u ile eşleşmeli

#### `components/service-a/base/route.yaml`
- `name`: Route adı
- `tls.termination`: `edge` veya `reencrypt`
- `targetPort`: Service port adı

#### `components/database/base/deployment.yaml`
- `POSTGRES_USER` ve `POSTGRES_PASSWORD` secret'tan alınır
- `PVC`: Veritabanı için kalıcı depolama

---

## Örnek Senaryolar

### `test`
- `1` adet `service-a`
- `1` adet `service-b`
- `database` ve basit ortam
- `test-` prefix / `-test` suffix kullanılır

### `pre-prod`
- `2` adet `service-a`
- `2` adet `service-b`
- `preprod-` prefix / `-preprod` suffix kullanılır

### `prod`
- `3` adet `service-a`
- `3` adet `service-b`
- `patchesStrategicMerge` ile production ayarları uygulanır

---

## Kustomize Komutları

```bash
kubectl kustomize overlays/app/test
kubectl kustomize overlays/app/pre-prod
kubectl kustomize overlays/app/prod
```

## Patch Örnekleri

- `patches/secure-route.yaml`: production için Route TLS yapılandırmasını değiştirir
- `patches/database-resources.yaml`: production veritabanı kaynak limitlerini artırır

---

## Notlar

- `components/*/base/kustomization.yaml` dosyaları sadece ilgili component kaynaklarını tanımlar.
- `overlays/app/*/kustomization.yaml` environment bazlı değişiklikleri içerir.
- Her YAML dosyasında kullanıcıya yönelik açıklamalar vardır.
- Kullanıcı kendi uygulamasını buradan kopyalayıp düzenleyebilir.
