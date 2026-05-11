# Changelog — Palavra Cadabra

> Registro de mudanças significativas no projeto.

---

## [Unreleased]

### Adicionado
- **Onboarding tutorial** (5 páginas) na primeira abertura do app
  - Persistência via SharedPreferences (`onboarding_completed_v1`)
  - Páginas: Welcome, Toque para falar, Forme frases, IA, Aprender
  - Botão "Pular" disponível
- **Página "Como começar"** no portal (`/dashboard/comecar`)
  - 5 passos com links diretos para cada seção
  - Dicas importantes (offline, IA, cores, caixa alta, pronúncia)
  - Link para baixar app
- **9 novas atividades de letramento**:
  - Stage 2: Palavras Fáceis, Vogais, Famílias Silábicas
  - Stage 3: Animais/Alimentos/Ações → Texto, Misturado classes
  - Stage 4: Leitura Funcional, Palavras Compostas
- **Documentação detalhada** em `Docs/desenvolvimento/`:
  - HISTORICO_EVOLUCAO.md — linha do tempo completa
  - GUIA_OPERACIONAL.md — como rodar, deployar, debugar
  - CHANGELOG.md — este arquivo

### Modificado
- App.dart agora tem fluxo: Splash → Onboarding (1x) → Auth → Profile → Shell
- Sidebar do portal: adicionado item "Como começar" (Rocket icon)
- widget_test atualizado para mockar `onboarding_completed_v1`

---

## [2026-05-05] — Fix Navegação Portal

### Corrigido
- **Patient detail redirecionava para landing page**
  - Causa: Next.js static export + UUID não estava em `generateStaticParams`
  - Solução em camadas: CloudFront Function + deploy script copiando placeholder + `useParams` fallback
- **CloudFront Function quebrada** (`palavracadabra-url-rewrite`)
  - Substituída por `palavracadabra-spa-router` testada e funcional
  - Função adiciona trailing slash via 301 + rewrite UUID routes
- **`useParams()` retornava `placeholder`** em vez do UUID real
  - Fallback: ler de `window.location.pathname` em client component

### Adicionado
- Script `Web/scripts/deploy.sh` — automatiza build + sync + copia UUIDs + invalidação
- Subdomínios SSL: `app.palavracadabra.com.br`, `downloads.palavracadabra.com.br`

---

## [2026-05-04] — Pronúncia + IA Integrada

### Adicionado
- **Pronúncia automática** ao tocar qualquer símbolo (TTS nativo pt-BR)
- **Botão Falar com IA** integrada:
  - Online + múltiplas palavras → expande via Claude e fala frase natural
  - Online + 1 palavra → fala direto (sem chamar IA)
  - Offline → fala telegráfico
- **Timeout de 8s** na chamada IA com fallback automático
- **Botão Magic Wand** mantém bottom sheet com alternativas

### Modificado
- `_speakAndLog()` agora chama `_tts.stop()` antes do `speak()` (evita sobreposição)

---

## [2026-05-03] — Conteúdo Pedro + Imagens

### Adicionado
- **7 pranchas para Pedro** (123 células):
  - Principal 5×6, Comida 4×5, Ações 4×5, Corpo 4×5
  - Sentimentos 3×5, Animais 3×5, Escola 3×5
  - Navegação hierárquica (Principal como pai)
- **6 atividades de letramento com imagens ARASAAC**:
  - Symbol Matching: animais, alimentos, objetos, ações
  - Sight Words: palavras frequentes
  - Symbol to Text: ponte símbolo→texto
- **56 símbolos ARASAAC adicionais** no banco (brincar, cabeça, mão, médico, etc.)
- **DELETE endpoint** para literacy activities (`/api/v1/literacy/activities/{id}`)
- **POST /symbols/batch** — fetch múltiplos símbolos por IDs

### Corrigido
- **Stage IDs em idiomas diferentes**: Flutter usava pt, backend usa en
  - Padronizado para inglês no Flutter, labels pt apenas na UI
- **Symbol matching options re-shuffle**: `_options` agora field, não getter

### Modificado
- Atividades agora têm `image_url` em cada item
- Widgets de atividade renderizam `CachedNetworkImage` com fallback
- Parsers auto-geram `missing_index`, `options`, `correct_word`

---

## [2026-05-02] — Caixa Alta + Bug Fix Pranchas

### Adicionado
- **Labels em CAIXA ALTA** em todo o app (recomendação fonoaudiológica)
  - Symbol cells, message bar, prediction chips, atividades
  - `fontWeight: 700`, `letterSpacing: 0.3-0.5`

### Corrigido
- **Criação de pranchas no portal não funcionava**
  - Causa: POST `/boards` (sem trailing slash) → 307 redirect para HTTP
  - Solução: trailing slashes em todos endpoints + `redirect: 'error'` no fetch
- **Modal de criação** substituiu auto-create "Nova Prancha"
  - Campos: nome, paciente, tipo, dimensões, template toggle

---

## [2026-05-01] — Distribuição e Deploy

### Adicionado
- **APK Android** assinado e público em `downloads.palavracadabra.com.br`
- **PWA iOS** em `app.palavracadabra.com.br` (Service Worker + manifest)
- **Página `/baixar`** com instruções para Android e iPhone
- **Subdomínios SSL** via CloudFront + ACM wildcard

### Configurado
- **Android signing**: keystore release + key.properties + build.gradle.kts
- **App icon**: Pa + varinha mágica + estrelas (Material You compatible)
- **Splash screen**: roxo brand via `flutter_native_splash`

---

## [2026-04-30] — Anthropic API Configurada

### Adicionado
- **ANTHROPIC_API_KEY** na ECS task definition
- **3 endpoints de IA** funcionando em produção:
  - `/ai/expand-language` — Expande telegráfico
  - `/ai/generate-board` — Gera prancha personalizada
  - `/ai/clinical-insights` — Análise clínica

### Testado
- Expansão: "eu querer água frio" → "Eu quero água gelada."
- Generation: Prancha 4×5 para hora do lanche
- Insights: relatório estruturado de uso

---

## [2026-04-29] — Infraestrutura AWS Provisionada

### Adicionado
- **48 recursos AWS** via Terraform:
  - VPC 10.0.0.0/16, 2 subnets públicas + 2 privadas
  - NAT Gateway, IGW, route tables, security groups
  - RDS PostgreSQL 16 + TimescaleDB
  - ElastiCache Redis 7
  - ECS Fargate cluster + service + task definition
  - ALB com HTTPS listener + ACM cert
  - S3 (assets + web buckets)
  - CloudFront distribution
  - Cognito User Pool + 2 App Clients
  - ECR repository
- **Custo total**: ~$98/mês

### Configurado
- Domínio `palavracadabra.com.br` (registro.br)
- DNS CNAMEs: www, api, app, downloads, _validation
- SSL wildcard `*.palavracadabra.com.br` ISSUED
- HTTP → HTTPS redirect (ALB + CloudFront)

---

## [2026-04-15] — MVP Phase 7 Complete

7 fases do MVP concluídas conforme PLANEJAMENTO_PROJETO.md:
1. ✅ Fundação (repos, infra base, modelos)
2. ✅ CAA Core (pranchas, símbolos, TTS, predição local)
3. ✅ Predição v1 (frequência + bigrama + horário + recência)
4. ✅ Sync & Cloud (CRDT LWW, push/pull)
5. ✅ Portal Web v1 (Next.js + sidebar + analytics)
6. ✅ IA Enhancement (Claude API stubs)
7. ✅ Polish & Launch (LGPD, segurança, App icon, store metadata)

Plus 4 itens adicionais do planejamento original:
- ✅ Frontend Adaptativo (UI adapta por perfil)
- ✅ Gestão Clínica (coordenador→tutor→aluno→família)
- ✅ Acessibilidade Avançada (switch scanning + alto contraste)
- ✅ Plataforma de Pesquisa (dashboard anonimizado)

---

## Convenções de Versionamento

- **Tags Git**: `v0.X.Y` (MVP em 0.1.x, beta em 0.2.x)
- **APK versionCode**: cresce monotônico
- **Backend**: deploy contínuo via ECR/ECS
- **Web**: deploy contínuo via S3 + CloudFront
