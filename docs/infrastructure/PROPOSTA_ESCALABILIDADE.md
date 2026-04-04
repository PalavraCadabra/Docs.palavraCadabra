# Palavra Cadabra — Proposta de Escalabilidade

> **Data**: 04/04/2026
> **Objetivo**: Planejar escalabilidade do ambiente de produção conforme o crescimento da base de usuários
> **Faixas**: Beta (até 100 users) → Launch (1K) → Growth (10K) → Scale (100K+)

---

## 1. Contexto

O Palavra Cadabra é um app de CAA (Comunicação Aumentativa e Alternativa) para pessoas não verbais, com componentes de alfabetização e pesquisa científica. A arquitetura deve escalar considerando:

### Características do tráfego
- **Uso intensivo por sessão**: ~50-200 eventos/minuto durante comunicação ativa
- **Concentração em horários**: Escola (8h-17h), terapia (horários fixos)
- **Dados sensíveis**: LGPD, dados de saúde de pessoas com deficiência
- **Offline-first**: 80% do uso pode ser offline; sync bursts quando online
- **Alta importância de SLA**: é uma ferramenta de acessibilidade; downtime impacta comunicação

### Métricas-chave a monitorar
- **MAU** (Monthly Active Users)
- **Sessões simultâneas** (peak concurrency)
- **Eventos/segundo** (usage_logs)
- **Chamadas à API de IA** (Claude expansion + board gen + insights)
- **Storage de symbols/boards** (crescimento linear com users)

---

## 2. Faixas de Escalabilidade

### Faixa 1 — Beta Fechado (até 100 usuários ativos)
**Timing**: Primeiros 3 meses após lançamento

| Componente | Configuração | Justificativa |
|---|---|---|
| **ECS Fargate** | 1 task (0.5 vCPU, 1 GB) | Suporta ~50 usuários simultâneos |
| **RDS** | `db.t3.small` + 50 GB gp3 | 2 GB RAM permite buffer pool adequado |
| **ElastiCache** | `cache.t3.micro` × 1 | Cache de símbolos e sessões |
| **ALB** | Standard | — |
| **CloudFront** | PriceClass_100 (US/EU) | Brasil é atendido bem pelas edges US |
| **NAT Gateway** | 1 zona | HA não crítico em beta |
| **Bedrock/Claude** | ~200K tokens/dia | Moderação pela Anthropic |

**Custo estimado**: ~**$135/mês**

---

### Faixa 2 — Launch Público (até 1.000 MAU)
**Timing**: 3-12 meses após lançamento

#### Mudanças obrigatórias
| Componente | Mudança | Por quê |
|---|---|---|
| **ECS** | 2 tasks (HA) + auto-scaling (2-5) | Failover + picos |
| **RDS** | `db.t3.small` → `db.t4g.medium` | 4 GB RAM, suporta mais conexões |
| **RDS Multi-AZ** | **SIM** | SLA 99.95% obrigatório para produção |
| **RDS Read Replica** | 1 replica | Ler analytics sem sobrecarregar primário |
| **ElastiCache** | `cache.t4g.small` + réplica | HA do cache |
| **WAF** | AWS WAF com regras managed | Proteção contra DDoS/SQLi/XSS |
| **Shield Standard** | Incluído no WAF | Proteção L3/L4 DDoS |
| **CloudFront** | Mesmo + regras WAF | — |
| **Backup** | Automated + snapshot manual semanal | Compliance LGPD |
| **Secrets Manager** | Credenciais do RDS | Rotação automática de senhas |

#### Observabilidade
| Serviço | Uso |
|---|---|
| **CloudWatch Metrics** | Custom metrics da API (latência, error rate) |
| **CloudWatch Alarms** | CPU > 80%, erros > 1%, latência > 500ms |
| **X-Ray** | Distributed tracing (API → RDS → Redis → Bedrock) |
| **SNS** | Notificações de alarms para Slack/email |

**Custo estimado**: ~**$320/mês**

### Breakdown Faixa 2

| Item | Valor |
|---|---|
| ECS Fargate (2-5 tasks média 3) | $54 |
| RDS t4g.medium Multi-AZ | $105 |
| RDS Read Replica (t4g.small) | $36 |
| RDS Storage 100 GB gp3 | $11.50 |
| ElastiCache t4g.small + replica | $40 |
| ALB + LCUs (moderado) | $28 |
| NAT Gateway | $35 |
| WAF (managed rules) | $12 |
| CloudWatch + X-Ray | $10 |
| Secrets Manager | $0.80 |
| Backup storage | $5 |
| CloudFront + Data Transfer | $15 |
| ECR + S3 + outros | $5 |
| **Total** | **~$357/mês** |

---

### Faixa 3 — Growth (1K → 10K MAU)
**Timing**: 12-24 meses

#### Mudanças arquiteturais

##### 3.1 Database scaling
```
RDS t4g.medium → db.r6g.large (PostgreSQL)
+ 2 Read Replicas
+ Connection pooling (RDS Proxy)
```

**Por quê**: PostgreSQL tem limite de conexões. Com 10K MAU, picos podem chegar a 2K conexões simultâneas. RDS Proxy gerencia pool e reduz ~90% das conexões diretas.

##### 3.2 Cache strategy
```
ElastiCache t4g.small → cache.r6g.large (cluster mode)
+ Cache warming para símbolos populares
+ CDN para respostas de API imutáveis
```

##### 3.3 ECS Fargate → mix com Spot
- 50% On-Demand (SLA crítico)
- 50% Fargate Spot (economia ~70%)

##### 3.4 TimescaleDB para usage_logs
O PostgreSQL já tem extensão TimescaleDB habilitada. Aproveitar:
- Hypertables para usage_logs
- Compressão automática de dados > 30 dias
- Retention policies (manter agregados, comprimir raw)

##### 3.5 Separar workloads
| Workload | Serviço |
|---|---|
| **API sync (requests rápidos)** | ECS Fargate |
| **AI (Claude) calls** | Lambda (async) |
| **Sync offline batch** | Lambda + SQS |
| **Analytics pesado** | Athena + S3 (Parquet) |
| **Background jobs** | ECS scheduled tasks |

##### 3.6 Queue para eventos de uso
```
App → API → SQS → Lambda → TimescaleDB
```
Evita gargalo no RDS com milhões de eventos/dia.

#### Custo estimado: ~**$850/mês**

### Breakdown Faixa 3

| Item | Valor |
|---|---|
| ECS Fargate (mix on-demand + spot) | $180 |
| RDS r6g.large Multi-AZ | $340 |
| RDS 2 Read Replicas (r6g.medium) | $180 |
| RDS Proxy | $28 |
| RDS Storage 300 GB + backups | $50 |
| ElastiCache cluster r6g.large | $120 |
| ALB + alto LCU | $60 |
| NAT Gateway (2 AZs) | $70 |
| Lambda (AI calls async) | $25 |
| SQS + event processing | $10 |
| S3 + Athena (analytics) | $15 |
| WAF + Shield | $20 |
| CloudWatch + X-Ray (intensive) | $40 |
| CloudFront (mais tráfego) | $40 |
| Secrets Manager | $2 |
| **Total** | **~$1.180/mês** |

**Custo por MAU**: ~$0.12/usuário

---

### Faixa 4 — Scale (10K → 100K+ MAU)
**Timing**: 24+ meses

#### 4.1 Multi-region
```
Região primária: us-east-1 (N. Virginia)
Região secundária: sa-east-1 (São Paulo) — latência menor no BR
```

**Estratégia**:
- **Active-Active** usando Aurora Global Database
- **Route53 failover** baseado em health checks
- **CloudFront** com origins em ambas regiões

#### 4.2 Aurora Global
```
RDS r6g.large → Aurora PostgreSQL Serverless v2
- Escala automaticamente 1-16 ACUs
- Replicação cross-region < 1s
- 5-15 read replicas conforme demanda
```

#### 4.3 ECS Fargate → EKS (Kubernetes)
Quando o número de serviços e complexidade justificar:
- API principal
- AI service (Lambda ou pod dedicado)
- Background workers
- Analytics service
- Research service
- Notification service

#### 4.4 Caching multi-camada
```
CloudFront → API Gateway cache → Redis → RDS
```

#### 4.5 Data lake para pesquisa científica
```
S3 data lake (Parquet + Delta Lake)
+ Glue Catalog
+ Athena para queries
+ QuickSight para dashboards de pesquisa
```

#### 4.6 ML Pipeline
Para Fase 2 da predição (roadmap):
- **SageMaker** para treinar modelos de predição personalizados
- **Feature Store** com dados históricos dos usuários
- **Endpoints** serverless para inferência
- **Pipelines** automatizados (retrain semanal)

#### Custo estimado Faixa 4: ~**$3.500-5.000/mês**

---

## 3. Comparação Consolidada

| Faixa | Usuários | Custo/mês | Custo/MAU |
|---|---|---|---|
| **Beta** | 100 | $135 | $1.35 |
| **Launch** | 1K | $357 | $0.36 |
| **Growth** | 10K | $1.180 | $0.12 |
| **Scale** | 100K+ | $4.000 | $0.04 |

**Insight**: Custo por usuário cai dramaticamente com escala — de $1.35/user no beta para $0.04/user em escala.

---

## 4. Triggers para Upgrade

### Quando subir de faixa:

#### Beta → Launch
- [ ] Mais de 50 MAU consistentes por 2 semanas
- [ ] Picos de CPU no RDS > 70%
- [ ] Algum evento de downtime não aceito pelos usuários
- [ ] Feedback de SLA de clínicas/escolas parceiras

#### Launch → Growth
- [ ] 500+ MAU crescendo 20% ao mês
- [ ] Mais de 10 parceiros institucionais (clínicas/escolas)
- [ ] Analytics pesados começando a impactar API
- [ ] Crescimento de usage_logs > 1M eventos/dia
- [ ] Primeiros sinais de gargalos em RDS connections

#### Growth → Scale
- [ ] 5K+ MAU
- [ ] Latência > 500ms em picos
- [ ] Necessidade de compliance adicional (ANVISA, INMETRO)
- [ ] Expansão internacional (Mercosul, Portugal)
- [ ] Projetos de pesquisa ativos com universidades

---

## 5. Padrões de Escalabilidade

### 5.1 Auto-scaling de ECS
```hcl
resource "aws_appautoscaling_target" "api" {
  max_capacity       = 10
  min_capacity       = 2
  resource_id        = "service/${var.cluster}/${var.service}"
  scalable_dimension = "ecs:service:DesiredCount"
  service_namespace  = "ecs"
}

resource "aws_appautoscaling_policy" "api_cpu" {
  name               = "cpu-scaling"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_appautoscaling_target.api.resource_id
  scalable_dimension = aws_appautoscaling_target.api.scalable_dimension
  service_namespace  = aws_appautoscaling_target.api.service_namespace

  target_tracking_scaling_policy_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ECSServiceAverageCPUUtilization"
    }
    target_value = 70.0
  }
}
```

### 5.2 RDS Performance Insights
Habilitar para identificar queries lentas e dimensionar corretamente.

### 5.3 Database connection pooling
Usar **RDS Proxy** quando MAU > 5K:
- Reduz conexões ao RDS em ~90%
- Failover < 10s em vez de ~60s
- Custo: $0.015/vCPU/hora (~$11/mês em t4g.medium)

### 5.4 Caching agressivo
```
Cache layers:
1. CloudFront (assets estáticos) — TTL 1 dia
2. API Gateway cache (respostas públicas) — TTL 5 min
3. Redis (sessions, user data) — TTL 1 hora
4. PostgreSQL query cache (automatic)
```

### 5.5 Read Replicas
```
Leituras pesadas → Read Replicas:
- Analytics endpoints
- Relatórios de pesquisa
- Portal do terapeuta (dados do paciente)

Escritas → Primary:
- CAA usage events
- Login/auth
- Board updates
```

---

## 6. Recursos Futuros e Custos

### 6.1 Amazon Polly (TTS premium)
Quando ativarmos TTS via Polly em vez de nativo do device:
- **Neural TTS**: $16/1M chars
- **Estimativa**: 500K chars/mês × 1K users = 500M chars = **$8.000/mês**
- **Mitigação**: Cache de áudio em S3 + CloudFront (TTL infinito por phrase)
- **Com cache**: ~$500/mês para 10K users

### 6.2 SageMaker para predição personalizada
- **Endpoint serverless**: $0.20/mês + $0.20 por inferência/hora
- **Estimativa**: ~$150/mês para 10K users

### 6.3 Bedrock para IA
Migrar de Anthropic API direto para Bedrock:
- **Vantagem**: Tudo na AWS, facilita compliance
- **Custo**: Similar (~$15-30/mês por 1K users)
- **Desvantagem**: Latência um pouco maior

---

## 7. Custo Total Projetado (com features completas)

| Faixa | AWS base | +Polly | +ML | +Multi-region | **Total** |
|---|---|---|---|---|---|
| Beta (100) | $135 | $50 | — | — | **$185** |
| Launch (1K) | $357 | $150 | — | — | **$507** |
| Growth (10K) | $1.180 | $500 | $150 | — | **$1.830** |
| Scale (100K) | $4.000 | $3.000 | $800 | $1.500 | **$9.300** |

---

## 8. Comparação com Concorrentes

| Software | Modelo | Custo mensal equivalente |
|---|---|---|
| **Palavra Cadabra** | Freemium + Institucional | $0-30/usuário |
| **TD-Snap** | Licença + dispositivo | $299 + $5-15K hardware |
| **Proloquo2Go** | Compra única | $249 + iOS obrigatório |
| **Avaz AAC** | Assinatura | ~$10/mês |

**Vantagem competitiva**: Mesmo na Faixa 4 (100K MAU), nosso custo por usuário é **$0.09/mês**, permitindo oferecer **gratuito freemium** com monetização apenas institucional.

---

## 9. Observações Finais

### 9.1 Princípios de escalabilidade do projeto
1. **Offline-first**: reduz dependência do backend, suaviza picos
2. **CRDT sync**: permite múltiplos clients sem conflitos
3. **Prediction local primeiro**: só chama IA cloud quando necessário
4. **Cache agressivo**: símbolos ARASAAC raramente mudam
5. **Batch operations**: sync em lotes, não individual

### 9.2 O que NÃO escalar cedo
- Multi-region (só com mais de 10K MAU)
- Kubernetes (Fargate serve até bem longe)
- SageMaker (Claude API é mais barato e melhor até ~10K MAU)
- Aurora (RDS t4g/r6g até ~50K MAU é melhor custo-benefício)

### 9.3 O que escalar cedo
- **Observabilidade**: desde o dia 1 (CloudWatch + X-Ray)
- **Backups**: Multi-AZ desde launch
- **WAF**: desde launch (proteção básica)
- **Monitoring de custos**: alerts + Cost Explorer
- **Secrets Manager**: desde launch (não hardcode credentials)

---

## 10. Arquivos Relacionados
- [CUSTOS_ATUAL.md](CUSTOS_ATUAL.md) — Estado atual do ambiente dev
- [PROPOSTA_REDUCAO_CUSTOS.md](PROPOSTA_REDUCAO_CUSTOS.md) — Como reduzir custos de dev
