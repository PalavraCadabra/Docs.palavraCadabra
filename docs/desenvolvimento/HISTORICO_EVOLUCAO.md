# Histórico de Evolução — Palavra Cadabra

> **Atualizado em**: 08/05/2026
> **Status do projeto**: MVP completo + Beta-ready
> **URL produção**: https://www.palavracadabra.com.br

---

## Sumário Executivo

O Palavra Cadabra é uma plataforma de **CAA (Comunicação Aumentativa e Alternativa)** para pessoas não verbais, integrada com **alfabetização estruturada** e **pesquisa científica**. Este documento registra a evolução do projeto além do MVP original (7 fases), com foco nas correções, refinamentos e novas funcionalidades implementadas.

### Arquitetura Atual
```
┌──────────────────────────────────────────────────────────┐
│                   CLIENTES                                │
│  ┌──────────────┐  ┌──────────────────┐  ┌────────────┐  │
│  │ App Flutter  │  │ PWA (Safari iOS) │  │ Portal Web │  │
│  │ APK Android  │  │ app.palavra...   │  │ www.pal... │  │
│  └──────┬───────┘  └────────┬─────────┘  └─────┬──────┘  │
└─────────┼───────────────────┼──────────────────┼─────────┘
          └──────────────┬────┴──────────────────┘
                         │
┌────────────────────────▼─────────────────────────────────┐
│                  AWS (us-east-1)                          │
│                                                           │
│  CloudFront (3 distributions) → S3 + ALB → ECS Fargate   │
│                                          ↓                │
│                        RDS PostgreSQL 16 + ElastiCache    │
│                                                           │
│  Cognito • ECR • ACM SSL • CloudWatch                    │
└──────────────────────────────────────────────────────────┘
```

---

## Linha do Tempo das Evoluções

### 1. Fundação do Projeto (Abril 2026)
- 7 fases do MVP implementadas conforme PLANEJAMENTO_PROJETO.md
- 5 repositórios criados (App, Api, Web, Infra, Docs)
- 13.733 pictogramas ARASAAC indexados
- 8 pranchas template Core Vocabulary pt-BR
- API com 26 endpoints iniciais → expandida para 43

### 2. Deploy em Produção (Abril 2026)
- **Infraestrutura AWS** provisionada via Terraform (48 recursos)
- **Domínio** palavracadabra.com.br com SSL wildcard via ACM
- **Subdomínios** configurados:
  - `api.palavracadabra.com.br` → ALB → ECS
  - `www.palavracadabra.com.br` → CloudFront → S3 (Next.js)
  - `app.palavracadabra.com.br` → CloudFront → S3 (Flutter PWA)
  - `downloads.palavracadabra.com.br` → CloudFront → S3 (APK)
- **CI/CD User** `palavracadabra-ci` com permissões scoped para ECR/ECS

### 3. App Icon Personalizado
- Conceito visual: balão de fala com "Pa" + varinha mágica + estrelas
- Representa "Palavra" (letra P) + "Cadabra" (magia/encantamento)
- Cores: roxo brand (#6750A4), teal (#00BFA5), dourado
- Gerado em todos os tamanhos para iOS + Android via `flutter_launcher_icons`
- Splash screen com cor brand configurada via `flutter_native_splash`

### 4. Integração Claude API (Anthropic)
- **API Key** configurada como env var na ECS task definition
- **3 endpoints de IA** funcionando em produção:
  - `POST /api/v1/ai/expand-language` — Expande sequência telegráfica em frase natural
  - `POST /api/v1/ai/generate-board` — Gera pranchas personalizadas baseado no perfil
  - `POST /api/v1/ai/clinical-insights` — Análise clínica de usage_logs
- **Comportamento integrado** no botão Falar:
  - Online + 1+ palavras → IA expande e fala a frase natural
  - Online + 1 palavra → fala direto (sem chamar IA)
  - Offline → fala telegráfico
  - Timeout 8s com fallback automático

### 5. Distribuição Sem App Stores

#### Android — APK Direto
- **URL**: https://downloads.palavracadabra.com.br/palavra-cadabra-latest.apk
- Build assinado com keystore release (RSA 2048, validade 10000 dias)
- Tamanho: ~62 MB
- Instalação: habilitar "fontes desconhecidas" → instalar

#### iOS — PWA (Progressive Web App)
- **URL**: https://app.palavracadabra.com.br
- Funciona offline via Service Worker
- Instalação: Safari → Compartilhar → "Adicionar à Tela de Início"
- Manifest configurado com theme color e ícones (192px, 512px, 180px touch icon)

#### Página /baixar
- Instruções passo-a-passo para Android e iPhone
- Links diretos para APK e PWA
- Lista de features e aviso de versão beta

### 6. Refinamentos de UX

#### Acessibilidade (CRÍTICO)
- **Touch targets** aumentados: 64dp → 80dp mínimo (AAC best practice)
- **Cell size**: 80dp → 88dp
- **Labels em 2 linhas** com text-wrap (português tem palavras longas)
- **Semantics** (`Semantics(label, button: true)`) em cada célula

#### Feedback Visual
- Tap animation: scale 0.92 → 0.90 + flash de cor
- Pulse do botão Falar: amplitude 1.15
- Botão Falar muda de cor durante fala (secondary → primary)
- Predições com fade-in animado

#### Caixa Alta nos Labels
> Recomendação fonoaudiológica: aprendizes em fase inicial reconhecem caixa alta mais facilmente
- Symbol cells: bold + letter-spacing 0.3
- Message bar: caixa alta
- Prediction chips: caixa alta
- Atividades de letramento: opções em caixa alta

### 7. Pronúncia Automática
- **Toque em qualquer símbolo** → fala imediatamente via TTS nativo pt-BR
- Stop automático em toques rápidos (sem sobreposição)
- Funciona online e offline
- Log de evento mantido para relatórios

### 8. Sistema de Pranchas Hierárquicas

Para o perfil **Pedro** (paciente demo), criadas 7 pranchas temáticas:

| Prancha | Dimensões | Células | Tipo |
|---|---|---|---|
| Principal | 5×6 | 27 | core (raiz) |
| Comida | 4×5 | 18 | category |
| Ações | 4×5 | 19 | category |
| Corpo | 4×5 | 17 | category |
| Sentimentos | 3×5 | 12 | category |
| Animais | 3×5 | 15 | category |
| Escola | 3×5 | 15 | category |

**Total**: 123 células com símbolos ARASAAC reais, navegação hierárquica.

### 9. Módulo de Letramento Expandido

**15 atividades** distribuídas pelas 4 etapas:

#### Stage 1 — Fundamentos (4 atividades)
- Parear: Animais, Alimentos, Objetos, Ações
- Tipo: `symbol_matching` com imagens ARASAAC

#### Stage 2 — Emergente (3 atividades)
- Palavras Fáceis (eu, tu, sim, não, água, casa)
- Vogais (palavras com vogal faltando)
- Famílias Silábicas (CA-CE-CI palavras)

#### Stage 3 — Desenvolvimento (4 atividades)
- Animais — Símbolo ao Texto
- Alimentos — Símbolo ao Texto
- Ações — Verbos em Texto
- Misturado — Classes Gramaticais

#### Stage 4 — Convencional (2 atividades)
- Leitura Funcional
- Palavras Compostas

### 10. Imagens ARASAAC em Atividades
- **Symbol Matching**: imagem pequena (64×64) ao lado da palavra alvo + imagens grandes (96×96) em cada opção
- **Sight Words**: imagem 120×120 acima da palavra com letra faltando
- **Symbol to Text**: imagem 160×160 no topo
- Auto-geração de `missing_index`, `options` e `correct_word` no parser quando não fornecidos

### 11. Onboarding Tutorial (App)
Tutorial guiado de 5 páginas na primeira abertura do app:

1. **Welcome** — Apresentação do Palavra Cadabra
2. **Toque para falar** — Demo de SymbolCell amarelo com "EU"
3. **Forme frases** — Demo de message bar com EU+QUERER+ÁGUA
4. **Inteligência artificial** — Exemplo de expansão
5. **Aprender brincando** — Apresentação do módulo de letramento

**Persistência**: SharedPreferences com chave `onboarding_completed_v1`
**Reset**: `OnboardingService.reset()` para mostrar novamente em testes

### 12. Página "Como Começar" (Portal)
Guia 5 passos para terapeutas no portal web:
1. Cadastre seu paciente
2. Crie ou personalize pranchas
3. Defina o programa de letramento
4. Convide a família
5. Acompanhe o progresso

Plus seção de **dicas importantes**:
- Funciona offline
- IA para frases naturais
- Cores Fitzgerald Key
- Letras maiúsculas
- Pronúncia automática
- Como compartilhar app com famílias

---

## Correções Críticas

### Bug 1: Criação de Pranchas no Portal (Abril)
**Sintoma**: Clicar em "Nova Prancha" não fazia nada.

**Causa raiz**: POST para `/boards` (sem trailing slash) → FastAPI redirecionava com 307 para `/boards/`, mas o redirect ia para HTTP (não HTTPS), perdendo o body POST e o auth header.

**Solução**:
- Todos endpoints no `api.ts` agora usam trailing slash (`/boards/`, `/profiles/`)
- `fetch` com `redirect: 'error'` para falhar explicitamente
- Modal de criação substituindo o auto-create de "Nova Prancha"

### Bug 2: Stage IDs em Idiomas Diferentes (Abril)
**Sintoma**: Atividades de letramento não carregavam, mostrava fallback demo sem imagens.

**Causa raiz**:
- Flutter usava `fundamentos`, `emergente`, `desenvolvimento`, `convencional` (pt)
- Backend usa `foundations`, `emerging`, `developing`, `conventional` (en)
- API retornava 0 atividades, caindo no fallback demo

**Solução**: IDs em inglês no Flutter, labels em português apenas na UI via `stageLabels` map.

### Bug 3: Symbol Matching Options Re-shuffle (Abril)
**Sintoma**: Ao tocar uma opção, os botões re-embaralhavam visualmente.

**Causa raiz**: `_options` era um getter com `shuffle()` chamado a cada rebuild.

**Solução**: `_options` agora é um field construído uma vez por pergunta com `_buildOptions()`.

### Bug 4: Navegação para Patient Detail (Maio)
**Sintoma**: Clicar no card do paciente redirecionava para landing page.

**Causa raiz**: Next.js static export gera HTML por rota, mas rotas dinâmicas `[id]` só geram para os `generateStaticParams`. UUIDs reais retornavam 404 → CloudFront servia `/index.html` (landing).

**Solução em camadas**:
1. **CloudFront Function** (`palavracadabra-spa-router`):
   - Adiciona trailing slash via 301
   - Rotas com UUID rewritten para `placeholder/index.html`
   - Outras rotas appendam `/index.html`

2. **Deploy script** (`scripts/deploy.sh`):
   - Após build, busca todos profile/board IDs da API
   - Copia placeholder HTML para cada UUID

3. **Fix no client component**:
   - `useParams()` retornava `placeholder` da página estática
   - Solução: ler ID diretamente de `window.location.pathname`

```typescript
const id = (() => {
  const paramId = params.id;
  if (paramId && paramId !== 'placeholder') return paramId;
  if (typeof window !== 'undefined') {
    const parts = window.location.pathname.split('/').filter(Boolean);
    const idx = parts.indexOf('patients');
    if (idx >= 0 && idx + 1 < parts.length) return parts[idx + 1];
  }
  return paramId;
})();
```

---

## Endpoints da API (43 total)

### Auth
- POST /auth/register, /auth/login
- GET /auth/me

### Profiles
- GET, POST /profiles/
- GET, PATCH, DELETE /profiles/{id}

### Boards
- GET, POST /boards/
- GET, PATCH, DELETE /boards/{id}
- POST /boards/{id}/cells
- PATCH, DELETE /boards/{id}/cells/{id}

### Symbols
- GET, POST /symbols/
- GET /symbols/{id}
- POST /symbols/batch (novo — fetch múltiplos por IDs)

### Usage Logs
- POST /usage-logs/, /usage-logs/batch
- GET /usage-logs/

### Sync (LWW-CRDT)
- POST /sync/push
- POST /sync/pull

### Backup
- GET /backup/export
- POST /backup/import

### AI (Claude)
- POST /ai/expand-language
- POST /ai/generate-board
- POST /ai/clinical-insights

### Literacy
- GET, POST /literacy/programs
- GET, PATCH /literacy/programs/{id}
- GET, POST /literacy/activities
- GET, DELETE /literacy/activities/{id}
- POST, GET /literacy/results
- GET /literacy/progress/{program_id}

### Care Relationships
- GET /care/
- GET /care/my-patients
- POST /care/invite
- DELETE /care/{id}

### Research
- GET /research/aggregate/communication
- GET /research/aggregate/vocabulary
- GET /research/aggregate/literacy
- GET /research/cohorts
- GET /research/export/anonymized

### Privacy (LGPD)
- GET /privacy/data-processing
- GET, POST /privacy/consent
- GET, DELETE /privacy/my-data

### Admin
- POST /setup-db (cria tabelas, idempotente)

### Health
- GET /health

---

## Infraestrutura AWS — Custos

| Recurso | Custo/mês (USD) |
|---|---|
| ECS Fargate (1 task) | ~$9 |
| RDS PostgreSQL 16 (t3.micro) | ~$15 |
| ElastiCache Redis (t3.micro) | ~$13 |
| ALB | ~$22 |
| NAT Gateway | ~$33 |
| S3 (3 buckets) | ~$0.10 |
| CloudFront (3 distributions) | ~$3 |
| CloudWatch + Logs | ~$3 |
| **Total** | **~$98/mês** |

**Anthropic API** (fora AWS): ~$15-30/mês dependendo do uso de IA

**Cenários de otimização documentados** em `infrastructure/PROPOSTA_REDUCAO_CUSTOS.md` (até $14/mês minimalista).

---

## Conteúdo em Produção

| Item | Quantidade |
|---|---|
| Símbolos ARASAAC | 953 (core vocabulary + AAC-flagged) |
| Pranchas template públicas | 8 |
| Pranchas do paciente Pedro | 7 (123 células) |
| Atividades de letramento | 15 (4 stages cobertos) |
| Usuários cadastrados | Admin + Terapeuta demo + Pedro profile |

---

## Próximos Passos Sugeridos

### Para Lançamento Público
1. **Apple Developer Account** ($99/ano) → submissão App Store
2. **Google Play Console** ($25) → submissão Google Play
3. **Beta com 5-10 terapeutas reais** → coletar feedback
4. **TestFlight** para iOS beta antes do lançamento público

### Para Qualidade do Produto
5. **Amazon Polly TTS** — vozes neurais premium pt-BR (Camila Neural)
6. **Relatórios PDF clínicos** — exportação para terapeutas
7. **Notificações push** — terapeuta envia atividades, família recebe progresso
8. **Email transacional (SES)** — confirmações e convites
9. **Sistema de lembretes** — sessão de hoje, atividade pendente

### Operacional
10. **CI/CD GitHub Actions** — deploy automático ao push
11. **Sentry/Datadog** — observabilidade de erros em produção
12. **Cost optimization** (Cenário B) → economia ~$60/mês
13. **Testes E2E** — Playwright nos fluxos críticos

---

## Arquivos Relacionados

- [PLANEJAMENTO_PROJETO.md](../PLANEJAMENTO_PROJETO.md) — Documento original do projeto
- [CUSTOS_ATUAL.md](../infrastructure/CUSTOS_ATUAL.md) — Detalhes de custo AWS
- [PROPOSTA_REDUCAO_CUSTOS.md](../infrastructure/PROPOSTA_REDUCAO_CUSTOS.md) — Otimização de custos
- [PROPOSTA_ESCALABILIDADE.md](../infrastructure/PROPOSTA_ESCALABILIDADE.md) — Plano de escala
