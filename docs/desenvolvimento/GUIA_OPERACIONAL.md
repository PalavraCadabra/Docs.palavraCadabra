# Guia Operacional — Palavra Cadabra

> Como rodar, deployar, debugar e manter o projeto em produção.

---

## URLs de Produção

| Serviço | URL |
|---|---|
| **Portal Web** | https://www.palavracadabra.com.br |
| **API** | https://api.palavracadabra.com.br |
| **API Docs (Swagger)** | https://api.palavracadabra.com.br/docs |
| **PWA iOS/Android** | https://app.palavracadabra.com.br |
| **APK Download** | https://downloads.palavracadabra.com.br/palavra-cadabra-latest.apk |
| **Página de Download** | https://www.palavracadabra.com.br/baixar |
| **Política de Privacidade** | https://www.palavracadabra.com.br/privacidade |
| **Termos de Uso** | https://www.palavracadabra.com.br/termos |

---

## Credenciais de Teste

### Admin
- Email: `admin@palavracadabra.edu.br`
- Senha: `Admin2026!`
- Role: `admin`

### Terapeuta Demo
- Email: `terapeuta@palavracadabra.com.br`
- Senha: `Palavra2026!`
- Role: `therapist`
- Paciente vinculado: **Pedro** (perfil `fe0b67a2-71a6-4e70-aa84-09b100dc4525`)

---

## Estrutura dos Repositórios

```
GitHub org: PalavraCadabra
├── App.palavraCadabra   — Flutter mobile app
├── Api.palavraCadabra   — FastAPI backend
├── Web.palavraCadabra   — Next.js portal + landing
├── Infra.palavraCadabra — Terraform AWS
└── Docs.palavraCadabra  — Documentação
```

Local (todos clonados em uma pasta única):
```
~/Github/palavraCadabra/
├── App/
├── Api/
├── Web/
├── Infra/
├── Docs/
└── .env (AWS credentials + ANTHROPIC_API_KEY)
```

---

## Desenvolvimento Local

### API (FastAPI + Docker)
```bash
cd ~/Github/palavraCadabra/Api
docker compose up -d         # Sobe Postgres + Redis + API
# API: http://localhost:8000
# Swagger: http://localhost:8000/docs

# Rodar migrations
docker compose exec api python -m scripts.create_tables

# Seed ARASAAC (uma vez)
docker compose exec api python -m scripts.seed_arasaac
```

### Portal Web (Next.js)
```bash
cd ~/Github/palavraCadabra/Web
npm install
npm run dev                   # http://localhost:3000
```

### App Flutter
```bash
cd ~/Github/palavraCadabra/App
flutter pub get
flutter run -d chrome         # Web (mais rápido para iterar)
flutter run -d macos          # macOS desktop
flutter run -d <android>      # Android device/emulator
```

---

## Deploy em Produção

### Pré-requisitos
- AWS credentials em `~/Github/palavraCadabra/.env`
- Docker rodando (para builds da API)
- JDK 17 + Android SDK (para APK)
- Node.js 18+ (para Web)
- Flutter 3.41+

### Variáveis de ambiente
```bash
export AWS_REGION=us-east-1
export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_ACCESS_KEY=...
```

### Deploy API
```bash
cd ~/Github/palavraCadabra/Api

# 1. Login no ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  237494380636.dkr.ecr.us-east-1.amazonaws.com

# 2. Build platform amd64
docker build --platform linux/amd64 -t palavracadabra-api .

# 3. Tag e push
docker tag palavracadabra-api:latest \
  237494380636.dkr.ecr.us-east-1.amazonaws.com/palavracadabra/api:latest
docker push 237494380636.dkr.ecr.us-east-1.amazonaws.com/palavracadabra/api:latest

# 4. Force new deployment
aws ecs update-service \
  --cluster palavracadabra-dev \
  --service palavracadabra-dev-api \
  --force-new-deployment
```

### Deploy Web Portal
```bash
cd ~/Github/palavraCadabra/Web
bash scripts/deploy.sh        # build + sync S3 + copia UUIDs + invalida CF
```

O script `deploy.sh` faz:
1. `npm run build` → gera `out/`
2. `aws s3 sync` → upload para bucket
3. Busca todos profile/board IDs da API
4. Copia placeholder HTML para cada UUID (workaround static export)
5. Invalida CloudFront

### Deploy PWA (Flutter Web)
```bash
cd ~/Github/palavraCadabra/App

flutter build web --release \
  --dart-define=API_BASE_URL=https://api.palavracadabra.com.br/api/v1 \
  --dart-define=PRODUCTION=true

aws s3 sync build/web/ s3://palavracadabra-app-pwa/ --delete
aws cloudfront create-invalidation --distribution-id EJ77N0ML8TM7K --paths "/*"
```

### Deploy APK
```bash
cd ~/Github/palavraCadabra/App

export JAVA_HOME=/opt/homebrew/opt/openjdk@17
export PATH=$JAVA_HOME/bin:$PATH
export ANDROID_HOME=/opt/homebrew/share/android-commandlinetools
export PATH=$ANDROID_HOME/cmdline-tools/latest/bin:$ANDROID_HOME/platform-tools:$PATH

flutter build apk --release \
  --dart-define=API_BASE_URL=https://api.palavracadabra.com.br/api/v1 \
  --dart-define=PRODUCTION=true

aws s3 cp build/app/outputs/flutter-apk/app-release.apk \
  s3://palavracadabra-downloads/palavra-cadabra-latest.apk \
  --content-type application/vnd.android.package-archive

aws cloudfront create-invalidation \
  --distribution-id E1AC3RN2ISH7DV --paths "/*"
```

---

## IDs de Recursos AWS

### CloudFront Distributions
| ID | Domínio | Origin |
|---|---|---|
| `E1JD6C88YRK9YO` | www.palavracadabra.com.br | S3 Web |
| `EJ77N0ML8TM7K` | app.palavracadabra.com.br | S3 PWA |
| `E1AC3RN2ISH7DV` | downloads.palavracadabra.com.br | S3 Downloads |

### CloudFront Function
- **Nome**: `palavracadabra-spa-router`
- **Função**: Trailing slash + rewrite de UUID routes para `placeholder/index.html`
- **Associada a**: E1JD6C88YRK9YO (portal)

### ECS
- **Cluster**: `palavracadabra-dev`
- **Service**: `palavracadabra-dev-api`
- **Task Definition**: `palavracadabra-dev-api` (revision atual: 4+)
- **CPU**: 256 / **Memory**: 512 MB

### RDS
- **Identifier**: `palavracadabra-dev-postgres`
- **Engine**: PostgreSQL 16.x
- **Class**: db.t3.micro
- **Storage**: 20 GB gp3

### ElastiCache
- **Cluster**: `palavracadabra-dev-redis`
- **Engine**: Redis 7.0
- **Node**: cache.t3.micro

### S3 Buckets
- `palavracadabra-dev-assets` — Símbolos, áudio (privado, CDN apenas)
- `palavracadabra-dev-web` — Portal Next.js (público via CF)
- `palavracadabra-app-pwa` — Flutter Web (público via CF)
- `palavracadabra-downloads` — APK e arquivos download (público via CF)

### Cognito
- **User Pool**: `us-east-1_O1Hj5yDLw`
- **App Client (Mobile)**: `4cdcdhqm9sip0ud0vad1m8rgab`
- **App Client (Web)**: `3c9c32a6snafg5enhpc5d2sbtr`

### ACM Certificate
- **ARN**: `arn:aws:acm:us-east-1:237494380636:certificate/3e40e8ae-c38f-45b7-96a5-6f8b3acee712`
- **Domínio**: `*.palavracadabra.com.br`

---

## Debug Comum

### API retorna 503
1. Verificar se a task ECS está running:
   ```bash
   aws ecs describe-services --cluster palavracadabra-dev --services palavracadabra-dev-api \
     --query 'services[0].{Running:runningCount,Desired:desiredCount}'
   ```
2. Ver logs da task:
   ```bash
   STREAM=$(aws logs describe-log-streams --log-group-name "/ecs/palavracadabra-dev/api" \
     --order-by LastEventTime --descending --limit 1 --query 'logStreams[0].logStreamName' --output text)
   aws logs get-log-events --log-group-name "/ecs/palavracadabra-dev/api" \
     --log-stream-name "$STREAM" --limit 20 --query 'events[].message' --output text
   ```

### Cliques no portal mostram landing page
- Significa que o CloudFront caiu no fallback 404
- Verificar se o UUID está no S3:
  ```bash
  aws s3 ls s3://palavracadabra-dev-web/dashboard/patients/UUID_AQUI/
  ```
- Se vazio, rodar `bash Web/scripts/deploy.sh` para copiar pastas dinâmicas

### IA retorna 503 "AI service not configured"
- ANTHROPIC_API_KEY não está na task definition
- Verificar:
  ```bash
  aws ecs describe-task-definition --task-definition palavracadabra-dev-api \
    --query 'taskDefinition.containerDefinitions[0].environment'
  ```
- Se faltar, registrar nova task def com env var

### App Flutter não conecta na API
- Web: verificar CORS no `app/main.py` (ALLOWED_ORIGINS)
- Android emulator: usar `http://10.0.2.2:8000` em vez de `localhost`
- iOS Simulator: `localhost:8000` funciona
- Device físico: usar IP da rede local

---

## Seed de Dados

### Símbolos ARASAAC
```bash
# Core vocabulary apenas (~900 símbolos)
docker compose exec api python -m scripts.seed_arasaac --core-only

# Todos os 13.733 pictogramas pt-BR
docker compose exec api python -m scripts.seed_arasaac
```

### Pranchas Template
```bash
docker compose exec api python -m scripts.seed_core_boards
```

### Atividades de Letramento
- Atividades atuais estão em prod, criadas via script Python que chama a API
- Para adicionar mais, criar payload em `/tmp/seed_more_activities.py` e rodar

---

## Monitoramento

### Custos AWS
```bash
aws ce get-cost-and-usage \
  --time-period Start=2026-05-01,End=2026-05-31 \
  --granularity MONTHLY \
  --metrics UnblendedCost
```

### Logs em tempo real
```bash
aws logs tail /ecs/palavracadabra-dev/api --follow
```

### Health Check completo
```bash
curl -s https://api.palavracadabra.com.br/health
curl -s https://www.palavracadabra.com.br | grep -o '<title>[^<]*</title>'
curl -s https://app.palavracadabra.com.br | head -5
curl -sI https://downloads.palavracadabra.com.br/palavra-cadabra-latest.apk | grep content-length
```

---

## Senhas e Secrets

> ⚠️ Não commitar nunca

Localizados em `~/Github/palavraCadabra/.env`:
- AWS credentials (admin)
- ANTHROPIC_API_KEY

Localizados em `~/Github/palavraCadabra/.env.ci`:
- CI/CD credentials (palavracadabra-ci user — permissões scoped)

Android keystore: `~/Github/palavraCadabra/App/android/app/upload-keystore.jks` (com `key.properties`)

---

## Contato e Suporte

- **DPO (LGPD)**: dpo@palavracadabra.com.br
- **Suporte**: contato@palavracadabra.com.br
- **Repos**: https://github.com/PalavraCadabra
