# 📋 REPO TEMPLATE - Kustomize Uygulaması Şablonu

Bu template, OpenShift üzerinde Kubernetes uygulamalarını Kustomize ile dağıtmak isteyenler için **başlangıç noktası**dır.

## 🎯 Amaç

Yeni bir uygulama deploy etmek istediğinizde bu template'i kopyalayıp, YAML dosyalarını kendi uygulamanız için özelleştirin.

---

## 📁 Dizin Yapısı

```
repo-template/
│
├── README.md                           # Bu dosya
├── kustomization.yaml                  # Root Kustomization
│
├── components/
│   └── app/base/                       # Temel kaynaklar (değişmez)
│       ├── deployment.yaml             # ✏️  BURADAN BAŞLA: Uygulamanı tanımla
│       ├── service.yaml                # Ağ erişimi
│       ├── route.yaml                  # Internet'e açma (HTTPS)
│       ├── configmap.yaml              # Konfigürasyon (non-sensitive)
│       ├── secret.yaml                 # Hassas bilgiler (password, API key)
│       └── kustomization.yaml
│
└── overlays/
    └── app/
        ├── test/                       # Test ortamı (1 replica, test- prefix)
        │   └── kustomization.yaml
        ├── pre-prod/                   # Pre-prod ortamı (2 replica, preprod- prefix)
        │   └── kustomization.yaml
        └── prod/                       # Production ortamı (3 replica, standart isimler)
            └── kustomization.yaml
```

---

## 🚀 Başlangıç Rehberi

### Adım 1: Template'i Klonla
```bash
# Dosyaları kopyala
cp -r repo-template/ repo-myapp/
cd repo-myapp/

# Ya da Git'te:
git clone <repo-template-url> my-new-app
cd my-new-app
```

### Adım 2: YAML Dosyalarını Düzenle

#### 2.1 Deployment Yapılandırması (`components/app/base/deployment.yaml`)
```
Açılacak alan:
- name: İlk satırda "app" → "myapp" yaz
- image: Docker image'ınızı yazın (registry, image, tag)
- containerPort: Uygulamanın dinlediği port
- env: Ortam değişkenleri
- resources: CPU/Memory taleplerini ayarla
- livenessProbe/readinessProbe: Health check endpoint'lerini ayarla
```

Örnek:
```yaml
name: myapp                                    # Değiştir: app → myapp
image: registry.example.com/myapp:1.0        # Değiştir: kendi image'ın
ports:
  - containerPort: 3000                       # Değiştir: kendi portu
env:
  - name: DATABASE_HOST
    value: "mysql.default.svc.cluster.local"  # Değiştir: kendi host'u
```

#### 2.2 Service Yapılandırması (`components/app/base/service.yaml`)
```
Genellikle deployment'tan sonra çalışır, minimal değişiklik gerekir.
- name: "app" → "myapp" (opsiyonel, consistency için)
- ports: Deployment'taki port'larla match et
```

#### 2.3 Route Yapılandırması (`components/app/base/route.yaml`)
```
OpenShift'e internet erişimi sağlar.
- name: "app" → "myapp"
- tls.termination: "edge" (TLS koruması için önerilir)
- Otomatik DNS name: myapp.default.apps.cluster.example.com
```

#### 2.4 ConfigMap Yapılandırması (`components/app/base/configmap.yaml`)
```
Non-sensitive konfigürasyon (password yok, API key yok)
Kendi uygulama config'lerini ekle:
- DATABASE_HOST
- LOG_LEVEL
- CACHE_SIZE
- vb.
```

#### 2.5 Secret Yapılandırması (`components/app/base/secret.yaml`)
```
⚠️  Sensitive bilgiler: password, API key, certificate
Değerleri base64 ile encode et:
  echo -n "mypassword" | base64  # mypassword → bXlwYXNzd29yZA==

⚠️  UYARI: Secret'ları Git'e commit ETME! (özel güvenlik gerekli)
  - Sealed Secrets (https://sealed-secrets.netlify.app/)
  - HashiCorp Vault
  - External Secrets Operator
  kullan.
```

### Adım 3: Kustomization Dosyalarını Düzenle

#### 3.1 `overlays/app/test/kustomization.yaml`
```yaml
namePrefix: test-        # Resource isimlerine "test-" ekle
nameSuffix: -test        # Resource isimlerine "-test" ekle
replicas:
  - name: app            # "app" → "myapp" (deployment name)
    count: 1             # Test: 1 replica
```

#### 3.2 `overlays/app/pre-prod/kustomization.yaml`
```yaml
namePrefix: preprod-
nameSuffix: -preprod
replicas:
  - name: app
    count: 2             # Pre-prod: 2 replica
```

#### 3.3 `overlays/app/prod/kustomization.yaml`
```yaml
# Prefix/suffix yok (standart isimler)
replicas:
  - name: app
    count: 3             # Production: 3 replica (High Availability)
```

---

## 🔨 Komutlar

### Build (YAML'leri görmek istersen)
```bash
# Test ortamını build et
kubectl kustomize overlays/app/test

# Pre-prod ortamını build et
kubectl kustomize overlays/app/pre-prod

# Production ortamını build et
kubectl kustomize overlays/app/prod
```

### Deploy
```bash
# Test ortamına deploy et
kubectl apply -k overlays/app/test

# Pre-prod ortamına deploy et
kubectl apply -k overlays/app/pre-prod

# Production ortamına deploy et
kubectl apply -k overlays/app/prod
```

### Status Kontrol
```bash
# Deployment'ı kontrol et
kubectl get deployment -l app=myapp

# Pod'ları görmek
kubectl get pods -l app=myapp

# Service'i test et
kubectl port-forward svc/myapp 8080:8080
# Sonra: http://localhost:8080

# Route'u görüntüle (OpenShift)
kubectl get routes -l app=myapp
```

### Update/Restart
```bash
# ConfigMap güncelledikten sonra pod'ları restart et
kubectl rollout restart deployment/test-myapp-test

# Yeni image version deploy etmek
kubectl set image deployment/myapp myapp=registry.example.com/myapp:2.0
```

---

## 📝 Detaylı Açıklamalar

### Deployment YAML Nedir?
Pod'lar nasıl çalışması gerektiğini tanımlar:
- Kaç pod (replicas)
- Hangi image
- Hangi port'lar
- Kaç CPU/Memory
- Health check'ler (liveness, readiness)

### Service YAML Nedir?
Pod'ların ağ üzerinden erişilmesini sağlar:
- Sabit DNS adı (`myapp.default.svc.cluster.local`)
- Pod'lar silindi çıksa da service adı aynı kalır
- Pod'lar ölürse, diğer pod'lar service'e bağlanır

### Route YAML Nedir?
OpenShift'e özel, internet erişimi sağlar:
- HTTPS (TLS) otomatik
- DNS adı (myapp.default.apps.cluster.example.com)
- Load balancer gerekli değil

### ConfigMap Nedir?
Uygulamanın dış konfigürasyonunu depolar (password yok):
- Database host, port, log level vb
- Pod'lar ConfigMap'ı okur
- ConfigMap değişirse pod'ları restart et

### Secret Nedir?
Hassas bilgiler (password, API key, certificate):
- Base64 ile encoded (şifrelenmiş DEĞİL, dikkat!)
- Git'e commit ETME
- Pod'lar Secret'ı ortam değişkeni ya da file olarak okur

---

## ⚙️ Gelişmiş Ayarlar

### Custom Namespace
```yaml
# overlays/app/test/kustomization.yaml
namespace: my-test-ns

# Sonra: kubectl create namespace my-test-ns
```

### Image Değişikliği (Overlay'da)
```yaml
# overlays/app/prod/kustomization.yaml
images:
- name: app
  newName: registry.example.com/myapp
  newTag: "1.0.0"
```

### Environment Değişkeni Ekleme
```yaml
# overlays/app/prod/kustomization.yaml
patches:
- target:
    kind: Deployment
    name: app
  patch: |
    - op: add
      path: /spec/template/spec/containers/0/env/-
      value: {name: PRODUCTION_FLAG, value: "true"}
```

### Resource Limits Override
```yaml
# overlays/app/prod/kustomization.yaml
patches:
- target:
    kind: Deployment
    name: app
  patch: |
    - op: replace
      path: /spec/template/spec/containers/0/resources/limits/memory
      value: 1Gi
```

---

## 🔒 Güvenlik İpuçları

### ✅ Yapılması Gerekenler
- [ ] Secret'ları Git'e commit ETME
- [ ] Sealed Secrets veya Vault kullan
- [ ] RBAC politikaları koy
- [ ] Network Policies koy (pod'lar arası iletişim kısıtla)
- [ ] Liveness/Readiness probe'ları yapılandır
- [ ] Resource requests/limits koy
- [ ] etcd encryption'ı etkinleştir

### ❌ Yapılmaması Gerekenler
- Secret'ları plaintext olarak log'lara yazma
- Password'leri image'a bake etme
- Tüm pod'lara full admin yetkisi verme
- External API key'leri kaynak koda koy

---

## 🐛 Troubleshooting

### Pod başlamıyor?
```bash
# Log'ları kontrol et
kubectl logs deployment/myapp

# Pod'un tanımını kontrol et
kubectl describe pod <pod-name>

# Event'leri görmek
kubectl get events --sort-by='.lastTimestamp'
```

### ConfigMap/Secret değiştirdim ama etki yok?
```bash
# Pod'ları yeniden başlat
kubectl rollout restart deployment/myapp
```

### Image bulunamıyor?
```bash
# Image pull policy kontrol et
# Eğer private registry ise: imagePullSecrets koy
```

---

## 📚 Kaynaklar

- [Kubernetes Resmi Docs](https://kubernetes.io/docs/)
- [Kustomize GitHub](https://github.com/kubernetes-sigs/kustomize)
- [OpenShift Resmi Docs](https://docs.openshift.com/)
- [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets)
- [12 Factor App](https://12factor.net/)

---

## 💡 İpuçları

1. **Sürüm kontrolü**: Her ortam (test/pre-prod/prod) için ayrı image tag kullan
   ```yaml
   image: registry.example.com/myapp:v1.0.0  # Production
   image: registry.example.com/myapp:v1.0.0-rc1  # Pre-prod
   image: registry.example.com/myapp:latest  # Test
   ```

2. **Namespace ayrımı**: Her ortam kendi namespace'ini kullan
   ```bash
   kubectl create namespace test
   kubectl create namespace pre-prod
   kubectl create namespace production
   ```

3. **Monitoring**: Her ortama uygun alert'ler koy
   - Test: Alert'ler sessiz olabilir
   - Pre-prod: Bazı alertler etkinleştir
   - Prod: Tüm alertler etkinleştir

---

## 🎓 Örnek: "MyApp" Projesi

Aşağıda gerçek bir örnek:

### 1. Template'i klonla
```bash
cp -r repo-template repo-myapp
cd repo-myapp
```

### 2. deployment.yaml'ı düzenle
```yaml
name: myapp                                          # app → myapp
image: registry.example.com/myapp:1.0              # Kendi image'ın
containerPort: 3000                                 # Node.js portu
env:
  - name: NODE_ENV
    value: "production"
  - name: DATABASE_URL
    valueFrom:
      secretKeyRef:
        name: app-secrets
        key: db-connection-string
```

### 3. service.yaml'ı düzenle (minimal, genellikle önemsiz)
```yaml
name: myapp
ports:
  - port: 3000
    targetPort: 3000
```

### 4. route.yaml'ı düzenle
```yaml
name: myapp
# Host opsiyonel, otomatik: myapp.default.apps.cluster.example.com
```

### 5. configmap.yaml'ı düzenle
```yaml
data:
  NODE_ENV: "production"
  LOG_LEVEL: "info"
  CACHE_TTL: "3600"
```

### 6. secret.yaml'ı düzenle
```yaml
stringData:
  db-password: "real-db-password-here"  # Base64'leme gerekli
  # echo -n "real-db-password-here" | base64
  # cmVhbC1kYi1wYXNzd29yZC1oZXJl
  db-password: cmVhbC1kYi1wYXNzd29yZC1oZXJl
```

### 7. Deploy et
```bash
# Test ortamına
kubectl apply -k overlays/app/test

# Production'a
kubectl apply -k overlays/app/prod
```

---

**Sorularınız olursa, her YAML dosyasındaki açıklamları okuyunuz! 📖**
