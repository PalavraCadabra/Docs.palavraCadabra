# palavraCadabra.edu.br — Documento de Planejamento do Projeto

> **Versão**: 1.0
> **Data**: 24/02/2026
> **Projeto**: Software de comunicação para pessoas não verbais + plataforma de alfabetização + pesquisa científica

---

## Índice

1. [Visão Geral do Projeto](#1-visão-geral-do-projeto)
2. [Pesquisa de Mercado — TD-Snap e Ecossistema AAC](#2-pesquisa-de-mercado)
3. [Modelos de Alfabetização para Não Verbais](#3-modelos-de-alfabetização)
4. [Fundamentação Científica — Cognição Não Verbal](#4-fundamentação-científica)
5. [Arquitetura e Stack Tecnológico](#5-arquitetura-e-stack-tecnológico)
6. [Modelagem de Dados](#6-modelagem-de-dados)
7. [Design System](#7-design-system)
8. [Componentes de IA/ML](#8-componentes-de-iaml)
9. [Plano de Negócio e Regras de Entrega](#9-plano-de-negócio)
10. [Roadmap do MVP](#10-roadmap-do-mvp)
11. [Referências Bibliográficas](#11-referências)

---

## 1. Visão Geral do Projeto

### 1.1 Missão

Criar uma plataforma revolucionária de comunicação para pessoas não verbais, que simultaneamente:

1. **Comunica** — Ferramenta de CAA (Comunicação Aumentativa e Alternativa) acessível, em português brasileiro, para iOS e Android
2. **Alfabetiza** — Sistema estruturado de ensino de leitura e escrita para pessoas não verbais, com acompanhamento por tutores e clínicas
3. **Pesquisa** — Plataforma científica para mapear padrões de pensamento e cognição em mentes não verbais, gerando dados para avanço do conhecimento

### 1.2 Problema

- **Custo proibitivo**: O TD-Snap custa ~US$299 (app) ou US$5.000-15.000 (dispositivos dedicados). Com câmbio e importação, torna-se inacessível para a maioria das famílias brasileiras.
- **Sem Android**: As principais soluções (TD-Snap, Proloquo2Go, TouchChat) são exclusivas para iOS. No Brasil, ~85% dos dispositivos são Android.
- **Português limitado**: As soluções existentes tratam o português brasileiro como tradução do inglês, sem profundidade cultural e linguística nativa.
- **Mito dos pré-requisitos**: A crença equivocada de que pessoas não verbais precisam atingir marcos cognitivos antes de aprender a ler e escrever.
- **Desconhecimento científico**: Pouco se sabe sobre como funciona o pensamento em mentes que não conseguem comunicar.

### 1.3 Diferenciais Competitivos

| Diferencial | palavraCadabra | TD-Snap | Proloquo2Go |
|---|---|---|---|
| Plataformas | iOS + Android + Web | iOS + Windows dedicado | iOS apenas |
| Idioma nativo | pt-BR nativo | pt-BR traduzido | pt-BR limitado |
| Custo | Acessível / Freemium | ~US$299 | ~US$249 |
| Alfabetização integrada | Sim | Não | Não |
| IA preditiva | Claude + ML personalizado | Básica | Básica |
| Pesquisa científica | Plataforma integrada | Não | Não |
| Símbolos culturais BR | Sim (ARASAAC + custom) | SymbolStix (americanizado) | SymbolStix |
| Offline | 100% funcional | Parcial | Sim |

---

## 2. Pesquisa de Mercado

### 2.1 TD-Snap — Como Funciona

O TD-Snap (Tobii Dynavox) é a referência mundial em CAA. Funciona como aplicação de geração de fala baseada em símbolos:

- **Modelo de comunicação**: Símbolos pictográficos organizados em grades. Cada símbolo representa uma palavra/frase/conceito. Ao tocar, o sistema converte em fala via TTS.
- **Snap Core First**: Principal conjunto de páginas. Baseado em pesquisa que mostra que ~200-400 palavras-núcleo compõem ~80% da fala cotidiana. Palavras-núcleo ficam na página principal; vocabulário específico em subpáginas por categoria.
- **Planejamento motor consistente**: Palavras-núcleo ficam SEMPRE na mesma posição, permitindo desenvolvimento de memória muscular — princípio fundamental.
- **Grades configuráveis**: De 2x2 (iniciantes) a 8x10+ (avançados), escalando com a evolução do usuário.
- **Acessibilidade**: Suporte a toque, rastreamento ocular (eye gaze), varredura por switch, rastreamento de cabeça.
- **Codificação por cores (Fitzgerald Key)**: Pronomes em amarelo, verbos em verde, adjetivos em azul, substantivos em laranja, frases sociais em rosa.

### 2.2 Fraquezas do TD-Snap (Oportunidades)

1. **Custo elevado** — Proibitivo para países em desenvolvimento
2. **Sem Android** — Exclui a maioria dos usuários brasileiros
3. **Curva de aprendizado íngreme** — Requer fonoaudiólogo para configuração ideal
4. **Suporte limitado ao português** — Cultural e linguisticamente superficial
5. **Símbolos americanizados** — Não representam comida, cultura e cotidiano brasileiro
6. **Construção de mensagem lenta** — Mesmo com predição, compor frases complexas é laborioso
7. **Ecossistema fechado** — Forte dependência do hardware Tobii Dynavox
8. **Sem componente de alfabetização** — Foca apenas na comunicação imediata

### 2.3 Concorrentes

| Solução | Plataforma | Preço | Português | Destaque |
|---|---|---|---|---|
| **TD-Snap** | iOS, Windows dedicado | ~US$299 / US$5K-15K (dispositivo) | Traduzido | Referência mundial, motor planning |
| **Proloquo2Go** | iOS | ~US$249 | Limitado | Popular nos EUA, método Crescendo |
| **LAMP Words for Life** | iOS | ~US$299 | Não | Motor planning por sequência de ícones |
| **TouchChat** | iOS | ~US$299 | Limitado | Múltiplos conjuntos de páginas |
| **CoughDrop** | Web (qualquer dispositivo) | ~US$20/mês | Limitado | Cross-platform, cloud-based |
| **Avaz AAC** | iOS + Android | Mais barato | Não | Modelo para localização |
| **Livox** | iOS + Android | Variável | **Sim** | Brasileiro, premiado |
| **SCALA** | Web | Gratuito | **Sim** | UFRGS, foco em autismo |
| **CBoard** | Web | Gratuito | Parcial | Open-source |
| **LetMeTalk** | iOS + Android | Gratuito | Básico | Funcionalidade básica |

### 2.4 Gaps Críticos no Mercado Brasileiro

1. **Android nativo** com qualidade profissional de CAA
2. **pt-BR nativo** — não traduzido, mas pensado em português
3. **Custo acessível** — freemium ou baixo custo
4. **Símbolos culturalmente brasileiros** — açaí, pão de queijo, feijão, Carnaval, festas juninas
5. **Offline obrigatório** — conectividade inconsistente no Brasil
6. **Sem necessidade de fonoaudiólogo para setup** — onboarding com IA
7. **Integração com SUS** — potencial de adoção pelo sistema público

---

## 3. Modelos de Alfabetização

### 3.1 Princípio Fundamental

> **"Todas as pessoas podem aprender a ler e escrever. Não existem pré-requisitos para o início da instrução de leitura e escrita."**
> — Karen Erickson & David Koppenhaver, Center for Literacy and Disability Studies (UNC)

A ausência de fala **NÃO** indica ausência de capacidade linguística ou cognitiva. Pesquisas de Janice Light (Penn State) demonstram que pessoas com necessidades complexas de comunicação têm o **direito** e a **capacidade** de aprender a ler e escrever.

### 3.2 Metodologias Chave

#### a) Método Fônico Adaptado
- Consciência fonológica interna (sem produção oral)
- Fonética visual — gestos e símbolos escritos representando fonemas
- Fonética mediada por CAA — o dispositivo "produz" os fonemas

#### b) Abordagem Global / Significado
- **Leitura compartilhada**: Adulto lê com o aprendiz, modelando com CAA
- **Abordagem de experiência de linguagem**: Experiências do aprendiz transcritas em texto usando CAA
- **Escrita com gráficos previsíveis**: Frames de frases repetitivas ("Eu gosto de ___")

#### c) Abordagem Visual / Ortográfica
- **Palavras de alta frequência**: Ensino como unidades visuais inteiras
- **Mapeamento ortográfico**: Conexões diretas entre forma escrita e significado
- **Ponte símbolo-texto**: Transição sistemática de CAA baseada em símbolos para texto

#### d) Estimulação de Linguagem Assistida (Aided Language Stimulation)
- Parceiros de comunicação apontam para símbolos no sistema CAA enquanto falam
- Modela uso do sistema E conexão entre palavra falada, escrita e símbolo
- Evidência robusta (Drager et al., 2006; Binger & Light, 2007)

### 3.3 Modelo dos Quatro Blocos (Adaptado para CAA)

Desenvolvido por Cunningham & Hall, adaptado por Erickson & Koppenhaver:

| Bloco | Adaptação para CAA |
|---|---|
| **Leitura Guiada** | Textos acessíveis, sistema CAA para respostas, varredura assistida |
| **Leitura Auto-selecionada** | E-books, texto com suporte de símbolos, discussão via CAA |
| **Escrita** | "Lápis alternativo" (teclado em tela, eye-gaze keyboard, placas de letras) |
| **Trabalho com Palavras** | Consciência fonológica com respostas não-orais, classificação com CAA |

### 3.4 Estágios de Desenvolvimento

```
PRÉ-LETRAMENTO
├── Consciência de impressão (textos têm significado)
├── Compreensão de símbolos (base para CAA e leitura)
├── Consciência fonológica receptiva (rima, aliteração)
├── Conhecimento do alfabeto (reconhecimento de letras)
└── Intenção comunicativa

LETRAMENTO EMERGENTE
├── Princípio alfabético (letras = sons)
├── Escrita emergente com "lápis alternativo"
├── Reconhecimento de palavras de alta frequência
├── Participação em leitura compartilhada via CAA
└── Conceitos sobre impressão (direção, espaçamento)

LETRAMENTO CONVENCIONAL
├── Decodificação de palavras desconhecidas
├── Fluência (velocidade e precisão de reconhecimento)
├── Compreensão leitora (respostas via CAA)
├── Escrita independente (acesso alternativo)
└── Ortografia demonstrada via dispositivo/teclado
```

### 3.5 Contexto Brasileiro

**Pesquisadores e instituições chave:**
- **UERJ** — Leila Nunes e Cátia Walter (CAA em escolas, PECS)
- **UNESP Marília** — Débora Deliberato (CAA e letramento para PC)
- **USP** — Fernando Capovilla (consciência fonológica), Fernanda Dreux (linguagem no autismo)
- **UFSCar** — Gerusa Lourenço (tecnologia assistiva)
- **UFRGS** — Liliana Passerino (SCALA — sistema CAA para autismo)
- **Mackenzie** — LaNeCS (neurociência cognitiva)

**Ferramentas brasileiras existentes:**
- **SCALA** (UFRGS) — CAA para letramento de pessoas com autismo
- **Livox** — App CAA brasileiro premiado (Global Mobile Awards)

**Marco legal:**
- Lei Brasileira de Inclusão (13.146/2015) — garante direito à comunicação e tecnologia assistiva
- Política Nacional de Educação Especial na Perspectiva Inclusiva (2008)
- AEE (Atendimento Educacional Especializado) — deveria incluir CAA

### 3.6 Abordagem do palavraCadabra para Alfabetização

Nosso sistema combinará:

1. **Paulo Freire adaptado** — "Palavras geradoras" do repertório comunicativo CAA do aprendiz
2. **Quatro Blocos adaptado** — Estrutura completa de letramento com CAA integrada
3. **Método fônico mediado por tecnologia** — TTS + visual phonics + feedback multimodal
4. **Gamificação responsável** — Pontos, conquistas, narrativas, dificuldade adaptativa
5. **Ponte símbolo → texto progressiva** — Transição gradual de pictogramas para palavras escritas
6. **Dados de progresso em tempo real** — Para tutores, terapeutas e família

---

## 4. Fundamentação Científica

### 4.1 Cognição Sem Linguagem Verbal

A neurociência moderna é clara: **cognição não requer linguagem verbal**.

- **Fedorenko & Varley (2016)**: Demonstraram em revisão landmark que pensamento e linguagem dependem de sistemas neurais separáveis. Pacientes com redes de linguagem severamente danificadas mantêm capacidade de raciocínio complexo.
- **Temple Grandin**: Documenta pensamento inteiramente em imagens fotorrealistas, como uma "videoteca mental". Propõe continuum: visualizadores de objetos → visualizadores espaciais → pensadores verbais.
- **Varley & Siegal (2000)**: Paciente com afasia agramática severa passou testes de Teoria da Mente, demonstrando cognição social sem gramática.

### 4.2 O Gap Expressivo-Receptivo

Muitos indivíduos não verbais compreendem **muito mais** do que conseguem expressar:

- **Jaswal & Akhtar (2019)**: Desafiaram suposições sobre motivação social no autismo. Aparentar desinteresse ≠ estar desinteressado.
- **DiStefano et al. (2019)**: Usando EEG, demonstraram que muitos autistas não falantes mostram marcadores neurais de processamento semântico (efeito N400), indicando compreensão linguística intacta apesar da ausência de fala.
- **Dificuldades de planejamento motor** (apraxia da fala), não déficits cognitivos, frequentemente são a barreira primária (Shriberg et al., 2011; Chenausky et al., 2019).

### 4.3 Modalidades de Pensamento Não Verbal

| Modalidade | Descrição | Evidência |
|---|---|---|
| **Visual (objetos)** | Imagens fotorrealistas, detalhadas | Grandin (1995, 2022); Kozhevnikov et al. (2005) |
| **Visual (padrões)** | Relações espaciais, configurações abstratas | Kozhevnikov et al. (2005) |
| **Sensorial** | Texturas, sons, sensações físicas, emoções | Relatos de Ido Kedar, Naoki Higashida |
| **Conceitual** | Raciocínio sem linguagem explícita | Fedorenko & Varley (2016) |

### 4.4 Métodos Científicos para Estudar Cognição Não Verbal

1. **Eye-tracking** — Revela processamento de linguagem em tempo real, atenção, compreensão (sem resposta verbal/motora necessária)
2. **EEG/ERPs** — N400 (processamento semântico), P300 (atenção), MMN (processamento auditivo automático)
3. **fMRI/fNIRS** — Ativação cerebral durante estímulos linguísticos e cognitivos
4. **Análise de padrões de uso CAA** — Cada ato comunicativo digitalmente registrado é dado científico

### 4.5 O palavraCadabra como Plataforma de Pesquisa

Dados coletáveis do uso do software:

| Categoria | Métricas | Insight Cognitivo |
|---|---|---|
| **Vocabulário** | Palavras selecionadas, frequência, TTR | Amplitude vocabular, diversidade conceitual |
| **Sintaxe** | MLU via CAA, padrões de ordem | Desenvolvimento gramatical |
| **Temporal** | Taxa comunicativa, latência de resposta | Velocidade de processamento, fadiga |
| **Navegação** | Páginas visitadas, caminho, retrocessos | Modelo mental de organização vocabular |
| **Erros** | Correções, abandonos, seleções fora do alvo | Automonitoramento, padrões semânticos |
| **Pragmática** | Iniciações vs. respostas, manutenção de tópico | Desenvolvimento comunicativo social |
| **Crescimento** | Todas as métricas ao longo do tempo | Trajetória desenvolvimental |

### 4.6 Considerações Éticas

- **LGPD (Lei 13.709/2018)**: Dados de CAA são dados pessoais sensíveis (saúde + deficiência). Requer consentimento explícito, anonimização para pesquisa, DPO designado.
- **Resolução CNS 466/2012**: Pesquisa com seres humanos requer aprovação do CEP (Comitê de Ética em Pesquisa) via Plataforma Brasil.
- **Estatuto da Pessoa com Deficiência (13.146/2015)**: Tomada de decisão apoiada, autonomia.
- **ECA (8.069/1990)**: Proteção especial para dados de crianças e adolescentes.
- **Princípio "Nada Sobre Nós Sem Nós"**: Participação ativa de pessoas não verbais no design e pesquisa.

---

## 5. Arquitetura e Stack Tecnológico

### 5.1 Visão Geral da Arquitetura

```
┌──────────────────────────────────────────────────────────────┐
│                        CLIENTES                               │
│                                                               │
│  ┌─────────────┐  ┌─────────────┐  ┌──────────────────────┐ │
│  │ App Mobile   │  │ App Mobile   │  │ Portal Web           │ │
│  │ iOS (Flutter)│  │Android(Flut.)│  │ Clínica/Escola       │ │
│  │              │  │              │  │ (React + Next.js)    │ │
│  └──────┬───────┘  └──────┬───────┘  └──────────┬──────────┘ │
└─────────┼─────────────────┼─────────────────────┼────────────┘
          │                 │                     │
          └────────────┬────┴─────────────────────┘
                       │
┌──────────────────────▼───────────────────────────────────────┐
│                     AWS CLOUD                                 │
│                                                               │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐ │
│  │  CloudFront   │  │  API Gateway  │  │  Amplify/Vercel    │ │
│  │  (CDN)        │  │  (REST + WS)  │  │  (Web Portal)      │ │
│  └──────┬────────┘  └──────┬────────┘  └────────────────────┘ │
│         │                  │                                   │
│  ┌──────▼────────┐  ┌──────▼────────┐                        │
│  │  S3            │  │  ECS Fargate   │                        │
│  │  (símbolos,    │  │  (FastAPI)      │                        │
│  │  áudio, mídia) │  └──┬───┬───┬────┘                        │
│  └───────────────┘     │   │   │                              │
│              ┌─────────┘   │   └──────────┐                   │
│      ┌───────▼──────┐ ┌───▼────┐  ┌──────▼────────┐         │
│      │ RDS Postgres  │ │ Redis  │  │ SageMaker     │         │
│      │ + TimescaleDB │ │ElastiC.│  │ (predição ML) │         │
│      └──────────────┘ └────────┘  └───────────────┘         │
│                                                               │
│  ┌──────────────┐ ┌────────────┐ ┌────────────────┐         │
│  │ Amazon Polly  │ │ Cognito    │ │ Bedrock        │         │
│  │ (TTS pt-BR)   │ │ (Auth)     │ │ (Claude API)   │         │
│  └──────────────┘ └────────────┘ └────────────────┘         │
└──────────────────────────────────────────────────────────────┘
```

### 5.2 Stack Completo

| Camada | Tecnologia | Justificativa |
|---|---|---|
| **Mobile** | Flutter 3.x | Renderização custom para grids de símbolos, offline-first, codebase única iOS/Android |
| **Portal Web** | React + Next.js 14 | Melhor ecossistema para portal de dados, SSR, mercado de devs BR |
| **Backend** | Python + FastAPI | Nativo para IA/ML, async, documentação automática OpenAPI |
| **Banco Principal** | PostgreSQL 16 + TimescaleDB | Integridade relacional, JSONB, time-series analytics, busca full-text pt |
| **Cache** | Redis (ElastiCache) | Cache de predições, sessões real-time, rate limiting |
| **BD Local (Mobile)** | Isar + Drift (SQLite) | Armazenamento offline no dispositivo |
| **Auth** | Amazon Cognito | Free tier 50K MAU, login social, JWT |
| **Compute** | ECS Fargate | Containers serverless, sem cold start, WebSocket |
| **Storage** | S3 + CloudFront | Biblioteca de símbolos, cache de áudio, assets |
| **TTS** | Amazon Polly (Camila Neural) | Melhor qualidade pt-BR, suporte SSML |
| **IA — Predição** | Modelos custom no SageMaker | Predição personalizada de símbolos |
| **IA — NLP** | BERTimbau + spaCy pt | Compreensão de linguagem portuguesa |
| **IA — Geração** | Claude API (via Bedrock) | Sugestões de prancha, expansão de frases, insights clínicos |
| **IA — Analytics** | Amazon Comprehend | Análise de sentimento, reconhecimento de entidades |
| **Símbolos** | ARASAAC (primário) | Gratuito, excelente pt, 12K+ símbolos, padrão no Brasil |
| **Design System** | Custom sobre Material Design 3 | Cores Fitzgerald Key, componentes AAC-específicos |
| **CI/CD** | GitHub Actions | Build Flutter, build Docker, deploy ECS |

### 5.3 Por Que Flutter para o App

1. **Renderização custom**: Grids de CAA precisam de controle total de pixel — Flutter renderiza via Skia/Impeller
2. **Offline-first**: Bancos locais (Isar, Drift) excelentes no ecossistema Flutter
3. **TTS nativo**: `flutter_tts` funciona com engines nativas iOS/Android + Amazon Polly quando online
4. **Acessibilidade**: Widget `Semantics` mapeia para VoiceOver (iOS) e TalkBack (Android)
5. **Performance**: Sem bridge (diferente do React Native), essencial para latência zero ao tocar símbolo
6. **Codebase única**: iOS + Android com mesma experiência visual

### 5.4 Por Que FastAPI para o Backend

1. **IA/ML nativo**: Python é a linguagem de ML — acesso direto a Hugging Face, spaCy, scikit-learn, BERTimbau
2. **Async nativo**: Critical para I/O (DB, AWS, WebSocket)
3. **OpenAPI automático**: Gera SDK clients para Flutter e Next.js
4. **Claude SDK**: Python-first para Anthropic
5. **Validação Pydantic**: Type safety que se aproxima de TypeScript

### 5.5 Custo Estimado AWS — MVP (Mensal)

| Serviço | Configuração | Custo Est. |
|---|---|---|
| ECS Fargate | 1 task, 0.5 vCPU, 1 GB | ~US$15 |
| RDS PostgreSQL | db.t3.micro, 20 GB | Free tier / ~US$15 |
| ElastiCache Redis | cache.t3.micro | Free tier / ~US$13 |
| S3 | 50 GB storage | ~US$2 |
| CloudFront | 100 GB transfer | Free tier |
| Cognito | < 50K MAU | Free tier |
| Polly | ~500K chars/mês | ~US$8 |
| API Gateway | 1M requests | ~US$3.50 |
| Bedrock (Claude) | ~100K tokens/dia | ~US$15 |
| SageMaker Serverless | Inferência leve | ~US$10 |
| CloudWatch | Logs + métricas | ~US$5 |
| **TOTAL MVP** | | **~US$75-90/mês** |

---

## 6. Modelagem de Dados

### 6.1 Entidades Principais

```
┌─────────────┐     ┌──────────────┐     ┌───────────────┐
│   users      │────▶│ aac_profiles  │────▶│    boards      │
│              │     │              │     │               │
│ - email      │     │ - nome       │     │ - nome        │
│ - role       │     │ - nível comm │     │ - tipo        │
│ - created_at │     │ - motor cap  │     │ - grid_rows   │
└─────────────┘     │ - visual cap │     │ - grid_cols   │
       │             │ - voz pref   │     │ - é template  │
       │             └──────────────┘     └───────┬───────┘
       │                    │                      │
       │                    │              ┌───────▼───────┐
       │                    │              │  board_cells   │
       │                    │              │               │
       │                    │              │ - posição     │
       │                    │              │ - symbol_id   │
       │                    │              │ - ação        │
       │                    │              │ - cor fundo   │
       │                    │              └───────┬───────┘
       │                    │                      │
       │             ┌──────▼──────┐       ┌───────▼───────┐
       │             │ usage_logs   │       │   symbols      │
       │             │ (TimescaleDB)│       │               │
       │             │              │       │ - label_pt    │
       │             │ - timestamp  │       │ - categoria   │
       │             │ - event_type │       │ - imagem S3   │
       │             │ - context    │       │ - classe gram │
       │             └──────────────┘       │ - freq_rank   │
       │                                    └───────────────┘
       │
       │    ┌───────────────────┐    ┌────────────────────┐
       └───▶│ care_relationships │    │ literacy_milestones │
            │                   │    │                    │
            │ - therapist_id    │    │ - profile_id       │
            │ - profile_id      │    │ - milestone_type   │
            │ - tipo relação    │    │ - achieved_at      │
            │ - permissões      │    │ - recorded_by      │
            └───────────────────┘    └────────────────────┘
```

### 6.2 Roles (Papéis)

| Role | Descrição | Permissões |
|---|---|---|
| **patient** | Pessoa não verbal (usuário da CAA) | Usar app, comunicar, acessar pranchas |
| **caregiver** | Pai/mãe/responsável | Gerenciar pranchas, ver logs, configurar |
| **therapist** | Fonoaudiólogo/terapeuta | Tudo do caregiver + relatórios, analytics, editar metas |
| **admin** | Administrador de clínica/escola | Gerenciar usuários, relatórios agregados, configuração institucional |

### 6.3 Modelo de Sync Offline

Utilizaremos **CRDTs (Conflict-free Replicated Data Types)** para sincronização:

- Cada alteração em prancha é uma operação no log CRDT
- Servidor faz merge determinístico de operações
- Vector clocks para rastrear versões
- Funciona 100% offline; sincroniza quando conectado

---

## 7. Design System

### 7.1 Filosofia: "Profissional Acolhedor"

- **Cantos arredondados** (16-20px) — amigável, acessível
- **Sombras suaves** — sensação de segurança
- **Animações sutis** (bounce, scale) na seleção — recompensador
- **Tipografia limpa** (Nunito) — profissional, legível
- **Grid consistente** — previsível, aprendível

### 7.2 Cores — Sistema Fitzgerald Key (Padrão CAA)

```
CORES DE CATEGORIAS AAC (obrigatório seguir o padrão):
├── 🟡 Amarelo  (#FFEB3B) — Pessoas / Pronomes
├── 🟢 Verde    (#4CAF50) — Ações / Verbos
├── 🔵 Azul     (#2196F3) — Adjetivos / Advérbios
├── 🟠 Laranja  (#FF9800) — Substantivos
├── 🩷 Rosa     (#E91E63) — Frases Sociais
├── 🟣 Roxo     (#9C27B0) — Diversos
└── 🩵 Ciano    (#00BCD4) — Perguntas

CORES DA MARCA (UI/chrome, NÃO fundo de símbolos):
├── Primária:   Roxo (#6750A4) — confiança, criatividade
├── Secundária: Verde-azulado (#00BFA5) — crescimento, saúde
├── Fundo:      Branco quente (#FFFBFE)
└── Superfície: Lavanda claro (#F5F0FF)
```

### 7.3 Símbolos — ARASAAC como Padrão Primário

- **12.000+ pictogramas** com traduções pt-BR nativas
- Licença CC BY-NC-SA (gratuito para uso não comercial)
- Padrão já utilizado na prática clínica brasileira
- API disponível para acesso programático
- Estilo artístico consistente e de alta qualidade
- Suporte a customização de cor de pele, cabelo, etc.

### 7.4 Especificações de Acessibilidade

| Aspecto | Especificação |
|---|---|
| **Alvos de toque** | Mínimo 64x64dp (recomendado 80x80dp) |
| **Grade máxima** | 8x10 em tablet, 4x5 em telefone |
| **Espaçamento entre alvos** | Mínimo 8dp |
| **Contraste de texto** | Mínimo 4.5:1 (WCAG AA) |
| **Modo alto contraste** | 7:1 mínimo (WCAG AAA) |
| **Varredura por switch** | 1-switch e 2-switch, velocidade configurável (0.5s-5s) |
| **Feedback tátil** | Impacto médio na seleção (configurável) |
| **Feedback visual** | Scale 95% + sombra + bounce |
| **Feedback auditivo** | Click sutil (opcional, configurável) |

### 7.5 Componentes Custom Necessários

```
Componentes AAC:
├── SymbolCell          — Célula individual (imagem + label + cor)
├── BoardGrid           — Grade de SymbolCells configurável
├── MessageBar          — Barra superior com mensagem composta
├── PredictionStrip     — Faixa horizontal de sugestões IA
├── BoardNavigator      — Breadcrumb/navegação entre pranchas
├── VoiceButton         — Botão grande "falar" com Polly
├── ScanningOverlay     — Indicador visual de varredura
├── SymbolEditor        — Modal de customização de célula
├── BoardEditor         — Interface completa de edição

Componentes de Gestão:
├── ProgressChart       — Visualização analytics para terapeutas
├── SessionTimer        — Timer de sessão com logging
├── MilestoneTracker    — Acompanhamento de marcos de letramento
├── AccessibilityPanel  — Configurações de acessibilidade
└── ReportGenerator     — Geração de relatórios PDF
```

---

## 8. Componentes de IA/ML

### 8.1 Predição de Símbolos (Funcionalidade de Maior Impacto)

**Arquitetura Híbrida Local + Cloud:**

**Local (sempre disponível offline):**
- Modelo de frequência (30% peso) — símbolos mais usados pelo usuário
- Modelo bigrama/trigrama (40% peso) — dado o último símbolo, qual vem depois
- Modelo de horário (15% peso) — "bom dia" de manhã, "boa noite" à noite
- Modelo de recência (15% peso) — boost para símbolos usados recentemente

**Cloud (quando online, aprimoramento):**
- Transformer-based sequence model personalizado
- Filtragem colaborativa (aprender de usuários similares)
- Contexto-aware (localização, horário, parceiro de comunicação)
- Claude para expansão natural de linguagem

### 8.2 Expansão de Linguagem Telegráfica

```
Entrada (símbolos CAA): "eu querer água frio"
Saída (Claude):         "Eu quero água gelada, por favor."
```

Usando Claude via Bedrock para expandir sequências telegráficas de CAA em português gramaticalmente correto, mantendo a intenção original.

### 8.3 Geração Inteligente de Pranchas

Claude gera layouts personalizados baseados no perfil do usuário:
- Idade, interesses, nível comunicativo
- Condição, capacidades motoras e visuais
- Contexto (escola, casa, terapia)

### 8.4 Insights Clínicos para Terapeutas

Geração automática de relatórios com:
- Progresso na comunicação (taxa, diversidade vocabular, MLU)
- Padrões de uso (horários de pico, categorias mais usadas)
- Sugestões para próxima sessão de terapia
- Vocabulário a ser explorado
- Marcos de desenvolvimento atingidos

### 8.5 NLP para Português Brasileiro

- **BERTimbau** (neuralmind/bert-base-portuguese-cased) — embeddings para português
- **spaCy pt_core_news_lg** — análise morfológica e sintática
- **Amazon Comprehend** — análise de sentimento, reconhecimento de entidades

### 8.6 Estratégia de ML por Fase

| Fase | Abordagem | Complexidade |
|---|---|---|
| **MVP** | Frequência + bigrama + Claude API | Baixa |
| **Pós-lançamento (3 meses)** | Filtragem colaborativa + BERTimbau fine-tuned | Média |
| **Escala (6-12 meses)** | Transformer personalizado + federated learning | Alta |

---

## 9. Plano de Negócio

### 9.1 Modelo de Entrega

#### A. App de Comunicação (CAA)
- **Freemium**: Funcionalidades básicas gratuitas (comunicação com pranchas pré-configuradas)
- **Premium**: Predição IA avançada, pranchas customizadas ilimitadas, vozes premium Polly, sync multi-dispositivo
- **Institucional**: Licença para clínicas e escolas com portal de gestão

#### B. Plataforma de Alfabetização
- **Módulo integrado ao app** com programa estruturado de letramento
- **Portal web** para gestão por clínicas e centros de ensino
- **Funcionalidades de coordenação**: acompanhamento de progresso, atribuição de atividades, comunicação tutor-família

#### C. Plataforma de Pesquisa
- **Dados anonimizados** disponíveis para pesquisadores (com consentimento)
- **Dashboard de pesquisa** para análise de padrões agregados
- **Parcerias com universidades** (USP, UERJ, UNESP, UFRGS)

### 9.2 Frontend Dinâmico e Adaptativo

O frontend se adapta automaticamente baseado em:

| Dado | Adaptação do Frontend |
|---|---|
| **Nível comunicativo** | Tamanho da grade, complexidade vocabular, presença de texto |
| **Capacidade motora** | Tamanho dos alvos de toque, método de acesso (toque/switch/eye-gaze) |
| **Capacidade visual** | Contraste, tamanho de fonte, modo alto contraste |
| **Progresso no letramento** | Proporção símbolo/texto, tipos de atividade |
| **Padrões de uso** | Pranchas mais usadas em destaque, predições personalizadas |
| **Contexto** | Horário do dia, ambiente (escola/casa/terapia) |

### 9.3 Processo de Gestão de Alfabetização em Clínica/Escola

```
ESTRUTURA DE GESTÃO

COORDENADOR (Admin da clínica/escola)
├── Cria programas de alfabetização
├── Atribui tutores aos alunos
├── Monitora progresso agregado
├── Gera relatórios institucionais
│
TUTOR (Fonoaudiólogo/Pedagogo)
├── Recebe alunos atribuídos
├── Define metas individualizadas
├── Seleciona atividades do programa
├── Registra observações e marcos
├── Gera relatórios individuais
├── Ajusta dificuldade e conteúdo
│
ALUNO (Pessoa não verbal)
├── Realiza atividades no app
├── Progride pelo programa estruturado
├── Dados de uso coletados automaticamente
│
FAMÍLIA (Caregiver)
├── Visualiza progresso do aluno
├── Recebe orientações do tutor
├── Pode reforçar atividades em casa
└── Comunica-se com tutor via portal
```

### 9.4 Etapas do Programa de Alfabetização

```
ETAPA 1 — FUNDAMENTOS (Pré-letramento)
├── Consciência de que símbolos representam coisas
├── Pareamento símbolo-palavra falada
├── Introdução ao alfabeto (letras como formas)
├── Atividades de consciência fonológica receptiva
└── Duração estimada: 2-4 meses

ETAPA 2 — EMERGENTE (Letramento inicial)
├── Correspondência letra-som (mediada por CAA)
├── Reconhecimento de palavras de alta frequência
├── Escrita emergente com "lápis alternativo"
├── Leitura compartilhada com modelagem CAA
└── Duração estimada: 3-6 meses

ETAPA 3 — DESENVOLVIMENTO (Letramento funcional)
├── Decodificação de palavras simples
├── Composição de frases simples por escrito
├── Compreensão de textos curtos
├── Ponte progressiva símbolo → texto
└── Duração estimada: 6-12 meses

ETAPA 4 — CONVENCIONAL (Letramento independente)
├── Leitura independente de textos adaptados
├── Escrita para comunicação funcional
├── Redução gradual de suporte de símbolos
├── Comunicação por texto como alternativa a símbolos
└── Duração estimada: Ongoing
```

---

## 10. Roadmap do MVP

### Fase 1 — Fundação (Semanas 1-4)
- [ ] Setup infraestrutura AWS (ECS, RDS, S3, Cognito)
- [ ] Setup projeto Flutter + FastAPI
- [ ] CI/CD com GitHub Actions
- [ ] Modelagem de dados e migrations
- [ ] Autenticação (Cognito)
- [ ] Integração ARASAAC (download e indexação de símbolos)

### Fase 2 — CAA Core (Semanas 5-10)
- [ ] Exibição de pranchas de comunicação (BoardGrid)
- [ ] Seleção de símbolos com feedback visual/tátil/auditivo
- [ ] TTS integrado (Polly + fallback nativo)
- [ ] MessageBar (composição de mensagem)
- [ ] Navegação entre pranchas
- [ ] Armazenamento offline (Isar/Drift)
- [ ] Pranchas pré-configuradas baseadas em Core Vocabulary pt-BR

### Fase 3 — Predição v1 (Semanas 11-13)
- [ ] Motor de predição local (frequência + bigrama)
- [ ] PredictionStrip UI
- [ ] Coleta de dados de uso (usage_logs)

### Fase 4 — Sync & Cloud (Semanas 14-16)
- [ ] Engine de sync offline (CRDT)
- [ ] Backup em nuvem
- [ ] Suporte multi-dispositivo

### Fase 5 — Portal Web v1 (Semanas 17-20)
- [ ] Dashboard do terapeuta (Next.js)
- [ ] Gerenciamento de pranchas via web
- [ ] Analytics básico (relatórios de uso)
- [ ] Gestão de relacionamentos terapeuta-paciente

### Fase 6 — IA Enhancement (Semanas 21-24)
- [ ] Integração Claude (expansão de linguagem, geração de pranchas)
- [ ] Predição aprimorada (cloud)
- [ ] Insights clínicos automáticos

### Fase 7 — Polish & Launch (Semanas 25-28)
- [ ] Testes de acessibilidade com usuários reais de CAA
- [ ] Auditoria de segurança e LGPD
- [ ] Beta com 5-10 terapeutas
- [ ] Submissão para App Store e Google Play
- [ ] Landing page palavracadabra.edu.br

---

## 11. Referências

### Pesquisa em CAA e Letramento
- Beukelman, D.R., & Light, J.C. (2020). *Augmentative and Alternative Communication* (5th ed.). Paul H. Brookes.
- Erickson, K.A., & Koppenhaver, D.A. (2020). *Comprehensive Literacy for All*. Paul H. Brookes.
- Light, J., & McNaughton, D. (2012). The changing face of AAC. *AAC*, 28(4), 197-204.
- Light, J., & McNaughton, D. (2013). Putting people first. *AAC*, 29(4), 299-309.
- Binger, C., & Light, J. (2007). Effect of aided AAC modeling. *AAC*, 23(1), 30-43.

### Pesquisa Brasileira
- Nunes, L.R.O.P., & Walter, C.C.F. (2014). *Pesquisa em CAA*. ABPEE.
- Deliberato, D. (2013). Comunicação alternativa na escola. In: *Comunicação alternativa: teoria, prática, tecnologias e pesquisa*. Memnon.
- Passerino, L.M. (2015). SCALA: software de CAA para letramento. *RENOTE*, 13(1).
- Capovilla, F.C. (2000). *Problemas de leitura e escrita*. Memnon.

### Neurociência e Cognição
- Fedorenko, E., & Varley, R. (2016). Language and thought are not the same thing. *Annals of the NYAS*, 1369(1), 132-153.
- Grandin, T. (2006). *Thinking in Pictures* (expanded ed.). Vintage.
- Grandin, T. (2022). *Visual Thinking*. Riverhead Books.
- Jaswal, V.K., & Akhtar, N. (2019). Being vs. appearing socially uninterested. *BBS*, 42, e82.
- DiStefano, C., et al. (2019). Identification of distinct language impairment pattern in minimally verbal autistic individuals.
- Varley, R., & Siegal, M. (2000). Evidence for cognition without grammar. *Current Biology*, 10(12), 723-726.

### Legislação Brasileira
- Lei 13.709/2018 — LGPD (Lei Geral de Proteção de Dados)
- Lei 13.146/2015 — Estatuto da Pessoa com Deficiência
- Lei 8.069/1990 — ECA (Estatuto da Criança e do Adolescente)
- Resolução CNS 466/2012 — Pesquisa com Seres Humanos

---

> **Próximos passos**: Validar este planejamento com stakeholders, montar equipe de desenvolvimento, e iniciar a Fase 1 do Roadmap.
