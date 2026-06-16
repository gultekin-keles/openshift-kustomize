# repo-example Kustomize Template

Bu proje, `repo-product-catalog` yapısına benzer çoklu deployment içeren bir Kustomize template örneğidir.
Amaç: kullanıcılar için açık, çoklu servis/deployment içeren bir Kustomize şablonu sunmak.

## Dizin Yapısı

```
repo-example/
├── README.md
├── kustomization.yaml
├── components/
│   ├── api/base/
│   ├── db/base/
│   └── web/base/
└── overlays/
    └── repo-example/
        ├── pre-prod/
        ├── prod/
        └── test/
```

## Ne içerir?

- `web` (frontend) deployment + service + route
- `api` (backend) deployment + service + route
- `db` (database) deployment + service + pvc
- `overlays` ile ortam bazlı özelleştirmeler
- `patches` ile production senaryolarında ek değişiklikler

## Örnek Senaryolar

### test
- Tek bir frontend ve backend
- `test-` prefix / `-test` suffix
- `environment: test` label

### pre-prod
- 2 frontend ve 2 backend
- `preprod-` prefix / `-preprod` suffix
- `environment: pre-prod` label

### prod
- 3 frontend ve 3 backend
- `patchesStrategicMerge` ile route ve db kaynakları güncellenir
- `environment: prod` label

## Kustomize ile çalıştırma

```bash
cd repo-example
kubectl kustomize overlays/repo-example/test
kubectl kustomize overlays/repo-example/pre-prod
kubectl kustomize overlays/repo-example/prod
```

## Notes

- `components/*/base/kustomization.yaml` her bir bileşeni kendi kaynaklarıyla tanımlar.
- `overlays/repo-example/*/kustomization.yaml` ortam bazlı repo-wide override sağlar.
- `patches/secure-route.yaml` ve `patches/db-pvc-size.yaml` production için örnek Kustomize patch kullanımını gösterir.
