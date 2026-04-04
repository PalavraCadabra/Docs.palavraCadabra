# Palavra Cadabra — Custos AWS (Estado Atual)

> **Data**: 04/04/2026
> **Ambiente**: `dev`
> **Região**: `us-east-1` (Virgínia)
> **Conta AWS**: `237494380636`

---

## 1. Visão Geral

O ambiente **dev** foi provisionado via Terraform com **48 recursos AWS**, seguindo a arquitetura planejada no documento `PLANEJAMENTO_PROJETO.md` (seção 5).

**Custo mensal estimado: ~US$ 99/mês (~R$ 540/mês a 5.45)**

---

## 2. Inventário de Recursos

### 2.1 Compute
| Recurso | Configuração |
|---|---|
| **ECS Cluster** | `palavracadabra-dev` (Fargate) |
| **Task Definition** | 256 CPU units (0.25 vCPU), 512 MiB RAM |
| **Service** | 1 task desired, rolling update |
| **ECR Repository** | `palavracadabra/api` (scan on push, lifecycle 10 images) |

### 2.2 Rede
| Recurso | ID / Configuração |
|---|---|
| **VPC** | `vpc-05ad34ced5b27ccff` (10.0.0.0/16) |
| **Subnets Públicas** | 2 (us-east-1a, us-east-1b) |
| **Subnets Privadas** | 2 (us-east-1a, us-east-1b) |
| **Internet Gateway** | 1 (acesso público das subnets públicas) |
| **NAT Gateway** | `nat-0d47bbaade2817290` (única zona) |
| **Elastic IP** | 1 (associado ao NAT) |
| **Route Tables** | 2 (public, private) |
| **Security Groups** | 3 (ALB, ECS, Database) |

### 2.3 Load Balancing
| Recurso | Configuração |
|---|---|
| **Application Load Balancer** | `palavracadabra-dev-alb` (internet-facing) |
| **Target Group** | `palavracadabra-dev-api-tg` (HTTP:8000, health `/health`) |
| **Listener HTTP (80)** | Redirect 301 → HTTPS |
| **Listener HTTPS (443)** | ACM cert + forward to target group, TLS 1.3 |

### 2.4 Banco de Dados
| Recurso | Configuração |
|---|---|
| **RDS PostgreSQL** | `palavracadabra-dev-postgres` |
| **Instance Class** | `db.t3.micro` (2 vCPU burst, 1 GB RAM) |
| **Storage** | 20 GB gp3 |
| **Engine** | PostgreSQL 16 |
| **Multi-AZ** | Desabilitado (single zone) |
| **Backup** | 7 dias de retenção |
| **Parameter Group** | `pg_stat_statements`, slow query logging |

### 2.5 Cache
| Recurso | Configuração |
|---|---|
| **ElastiCache Redis** | `palavracadabra-dev-redis` |
| **Node Type** | `cache.t3.micro` (2 vCPU burst, 0.5 GB RAM) |
| **Engine** | Redis 7.0 |
| **Nós** | 1 (sem réplica) |

### 2.6 Storage
| Recurso | Uso |
|---|---|
| **S3** `palavracadabra-dev-assets` | Símbolos ARASAAC, áudio, media |
| **S3** `palavracadabra-dev-web` | Static hosting do portal Next.js |

### 2.7 CDN
| Recurso | Domain | Uso |
|---|---|---|
| **CloudFront** `E1JD6C88YRK9YO` | `d29juhl7g3u42p.cloudfront.net` | Portal Web (PriceClass_100) |
| **CloudFront** `E1LX2N55E24JV4` | CDN de assets S3 | Imagens/symbols (PriceClass_All) |

### 2.8 Autenticação
| Recurso | ID |
|---|---|
| **Cognito User Pool** | `us-east-1_O1Hj5yDLw` |
| **Cognito App Client (Mobile)** | `4cdcdhqm9sip0ud0vad1m8rgab` |
| **Cognito App Client (Web)** | `3c9c32a6snafg5enhpc5d2sbtr` |

### 2.9 Certificados e DNS
| Recurso | Valor |
|---|---|
| **ACM Certificate** | `*.palavracadabra.com.br` (ISSUED) |
| **DNS** | Hospedado no registro.br (CNAMEs apontando para AWS) |

### 2.10 Observabilidade
| Recurso | Uso |
|---|---|
| **CloudWatch Logs** | `/ecs/palavracadabra-dev/api` |
| **CloudWatch Metrics** | Container Insights habilitado |

### 2.11 IAM
| Recurso | Uso |
|---|---|
| **ECS Task Role** | Acesso a S3 assets, Polly, Bedrock, Cognito |
| **ECS Execution Role** | Pull ECR, escrever CloudWatch Logs |
| **User `palavracadabra-ci`** | GitHub Actions deploy (ECR push, ECS update) |

---

## 3. Breakdown de Custos Mensais

### Cálculo detalhado (us-east-1, preços 2026)

| # | Serviço | Configuração | Fórmula | Custo/mês |
|---|---|---|---|---|
| 1 | **ECS Fargate** | 0.25 vCPU + 0.5 GB × 730h | 0.25 × $0.04048 × 730 + 0.5 × $0.004445 × 730 | **$9.01** |
| 2 | **RDS t3.micro** | db.t3.micro | $0.017 × 730 | **$12.41** |
| 3 | **RDS Storage gp3** | 20 GB | 20 × $0.115 | **$2.30** |
| 4 | **ElastiCache t3.micro** | cache.t3.micro | $0.017 × 730 | **$12.41** |
| 5 | **ALB** | Fixo + 1 LCU | $0.0225 × 730 + $0.008 × 730 | **$22.27** |
| 6 | **NAT Gateway** | Fixo + ~10 GB | $0.045 × 730 + 10 × $0.045 | **$33.30** |
| 7 | **Elastic IP** | 1 associado | Grátis (em uso) | **$0.00** |
| 8 | **S3 Storage** | ~1 GB total | 1 × $0.023 | **$0.02** |
| 9 | **S3 Requests** | ~100k requests | 100 × $0.0004 | **$0.04** |
| 10 | **CloudFront Web** (PC_100) | ~10 GB | 10 × $0.085 | **$0.85** |
| 11 | **CloudFront Assets** (PC_All) | ~10 GB | 10 × $0.120 | **$1.20** |
| 12 | **ECR Storage** | ~500 MB | 0.5 × $0.10 | **$0.05** |
| 13 | **CloudWatch Logs** | ~5 GB ingestion + storage | 5 × $0.50 + 5 × $0.03 | **$2.65** |
| 14 | **Data Transfer Out (ALB)** | ~5 GB | 5 × $0.09 | **$0.45** |
| 15 | **ACM Certificate** | Wildcard | Grátis | **$0.00** |
| 16 | **Route53** | Não usado | - | **$0.00** |
| 17 | **Cognito** | < 50K MAU | Free Tier | **$0.00** |
| 18 | **Bedrock/SageMaker** | Não usado | - | **$0.00** |
| 19 | **Polly** | Não usado (TTS nativo do device) | - | **$0.00** |

### **TOTAL: ~US$ 96.96/mês (~R$ 528/mês)**

---

## 4. Distribuição Percentual do Custo

```
NAT Gateway       ████████████████████      34.3%  ($33.30)
ALB               ██████████████             23.0%  ($22.27)
ElastiCache       ████████                   12.8%  ($12.41)
RDS PostgreSQL    ████████                   12.8%  ($12.41)
ECS Fargate       ██████                      9.3%  ($9.01)
CloudWatch        ██                          2.7%  ($2.65)
RDS Storage       ██                          2.4%  ($2.30)
CloudFront        ██                          2.1%  ($2.05)
Outros            ▌                           0.6%  ($0.56)
```

---

## 5. Observações Importantes

### 5.1 Custos fora da AWS
- **Anthropic API (Claude)**: Pago diretamente à Anthropic, cobrado por token. Estimativa:
  - ~100K tokens/dia para expansão de linguagem + geração de pranchas + insights
  - Custo estimado: **~US$ 15-30/mês** dependendo do uso
- **Domínio palavracadabra.com.br**: Pago no registro.br (~R$ 40/ano)

### 5.2 Custos que podem escalar
- **Data Transfer**: Cresce linearmente com o tráfego
- **NAT Gateway**: O processamento de dados ($0.045/GB) pode aumentar se a API fizer muitas chamadas externas
- **RDS Storage**: Cresce com uso — cada GB adicional = $0.115/mês
- **CloudWatch Logs**: Ingestion é o mais caro ($0.50/GB)

### 5.3 Recursos free-tier ativos
- **Cognito**: até 50.000 MAU grátis
- **ACM Certificates**: grátis para uso com AWS services
- **S3**: primeiros 5 GB grátis no primeiro ano (já vencido)
- **CloudWatch**: 10 metrics + 5 GB logs grátis

---

## 6. Comparação com Estimativa Original

O planejamento original (seção 5.5 do PLANEJAMENTO_PROJETO.md) previa **US$ 75-90/mês** para o MVP.

**Diferença**: ~US$ 7-22/mês acima do estimado, principalmente porque:
1. ALB custa mais que estimado ($22 vs ~$15 previsto)
2. NAT Gateway não estava claramente detalhado no estimate original
3. Não contabilizamos CloudFront adicional para assets

---

## 7. Histórico de Gastos (a monitorar)

Configurar **AWS Cost Explorer** e **Budgets Alerts** para:
- Alerta em US$ 100/mês
- Alerta em US$ 150/mês (50% acima do esperado)
- Alerta em US$ 200/mês (crítico)

```bash
aws budgets create-budget --account-id 237494380636 --budget '{
  "BudgetName": "palavracadabra-monthly",
  "BudgetLimit": {"Amount": "150", "Unit": "USD"},
  "TimeUnit": "MONTHLY",
  "BudgetType": "COST"
}'
```

---

## 8. Arquivos Relacionados
- [PROPOSTA_REDUCAO_CUSTOS.md](PROPOSTA_REDUCAO_CUSTOS.md) — Como reduzir para ~$30/mês
- [PROPOSTA_ESCALABILIDADE.md](PROPOSTA_ESCALABILIDADE.md) — Como escalar para produção
