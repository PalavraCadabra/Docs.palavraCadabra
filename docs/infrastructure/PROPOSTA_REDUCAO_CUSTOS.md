# Palavra Cadabra — Proposta de Redução de Custos

> **Data**: 04/04/2026
> **Objetivo**: Reduzir custo do ambiente `dev` de **~$97/mês** para **~$20-30/mês**
> **Redução potencial**: **~70%**

---

## 1. Contexto

O ambiente atual foi provisionado com arquitetura "production-ready" desde o início, o que é **overkill para desenvolvimento**. Três componentes sozinhos respondem por **70% do custo**:

1. **NAT Gateway** — $33/mês (34%)
2. **ALB** — $22/mês (23%)
3. **RDS + Redis** — $25/mês (26%)

Esta proposta apresenta **3 cenários** de otimização, do menos agressivo ao mais radical.

---

## 2. Cenário A — Otimização Conservadora (~$60/mês)

**Redução: ~38%** — Mantém arquitetura mas otimiza componentes caros.

### Mudanças

#### 2.1 Eliminar NAT Gateway (-$33/mês)
Substituir por **VPC Endpoints** para serviços AWS que o ECS precisa:

```hcl
# VPC Endpoints (Gateway type — GRATUITOS)
resource "aws_vpc_endpoint" "s3" {
  vpc_id            = aws_vpc.main.id
  service_name      = "com.amazonaws.us-east-1.s3"
  vpc_endpoint_type = "Gateway"
  route_table_ids   = [aws_route_table.private.id]
}

resource "aws_vpc_endpoint" "dynamodb" {
  vpc_id            = aws_vpc.main.id
  service_name      = "com.amazonaws.us-east-1.dynamodb"
  vpc_endpoint_type = "Gateway"
  route_table_ids   = [aws_route_table.private.id]
}

# VPC Endpoints (Interface type — $7.20/mês cada)
resource "aws_vpc_endpoint" "ecr_api" {
  vpc_id              = aws_vpc.main.id
  service_name        = "com.amazonaws.us-east-1.ecr.api"
  vpc_endpoint_type   = "Interface"
  subnet_ids          = aws_subnet.private[*].id
  security_group_ids  = [aws_security_group.vpc_endpoints.id]
  private_dns_enabled = true
}

resource "aws_vpc_endpoint" "ecr_dkr" {
  # ... similar para ECR Docker
}

resource "aws_vpc_endpoint" "logs" {
  # ... similar para CloudWatch Logs
}

resource "aws_vpc_endpoint" "secretsmanager" {
  # ... para secrets
}
```

**Problema**: Chamadas externas (Anthropic API, ARASAAC sync) ainda precisam de internet. Soluções:
- Mover essas chamadas para Lambda em subnet pública (free tier)
- Ou manter um NAT Instance (EC2 t4g.nano) em vez de NAT Gateway → ~$4/mês

**Economia líquida**: $33 - $21.60 (3 interface endpoints) - $4 (NAT instance) = **-$7.40/mês**

**Não recomendado no Cenário A** — complexidade alta para economia pequena.

#### 2.2 Consolidar CloudFronts (-$1/mês)
Usar um único CloudFront com múltiplos behaviors:
- Path `/*` → S3 web
- Path `/assets/*` → S3 assets

**Economia**: ~$1/mês + simplificação operacional.

#### 2.3 Reduzir Logs CloudWatch (-$1.50/mês)
- Configurar log retention para 7 dias (atualmente infinito)
- Filtrar logs de sucesso, manter apenas warnings/errors

```hcl
resource "aws_cloudwatch_log_group" "api" {
  name              = "/ecs/palavracadabra-dev/api"
  retention_in_days = 7  # Era: nunca expira
}
```

#### 2.4 Reduzir RDS Storage (-$1.15/mês)
- Atual: 20 GB (usando ~1 GB)
- Proposto: 10 GB
- **Nota**: RDS não permite reduzir storage após criação. Requer dump/restore.

#### 2.5 Habilitar RDS Storage Autoscaling
Evita over-provisioning desnecessário no futuro.

### Total Cenário A
| Item | Atual | Otimizado | Economia |
|---|---|---|---|
| CloudFront | $2.05 | $1.05 | -$1.00 |
| CloudWatch Logs | $2.65 | $1.15 | -$1.50 |
| RDS Storage | $2.30 | $1.15 | -$1.15 |
| **Total mensal** | **$97** | **$93** | **-$4** |

**Observação**: Cenário A sozinho tem pouco impacto. O maior ganho vem do Cenário B.

---

## 3. Cenário B — Otimização Moderada (~$30/mês)

**Redução: ~69%** — Substitui componentes caros por alternativas serverless/instância pequena.

### 3.1 Substituir ALB por CloudFront + Lambda URL (-$22/mês)

Em vez de ALB → ECS Fargate, usar:
- **CloudFront** com origem direto no container
- Ou migrar a API para **Lambda + API Gateway** (HTTP API: $1/1M requests)

**Arquitetura proposta**:
```
Cliente → CloudFront → API Gateway HTTP → Lambda (FastAPI com Mangum)
                                         → RDS/Redis (via VPC)
```

**Custo Lambda (1M requests/mês)**:
- Requests: $1.00
- Compute (512 MB × 200ms): ~$2.50
- **Total**: ~$3.50/mês (vs $22 do ALB)

**Economia**: **-$18.50/mês**

**Trade-off**: Cold starts (~500ms primeira chamada). Para AAC isso é aceitável em dev.

### 3.2 Substituir ElastiCache por Redis em EC2 (-$8.50/mês)

- **Atual**: cache.t3.micro = $12.41/mês
- **Proposto**: EC2 t4g.nano + Redis = ~$3.90/mês
- Ou: **eliminar Redis em dev** (usar cache em memória) = $0

**Economia**: **-$12.41/mês** (eliminar) ou **-$8.50/mês** (EC2)

### 3.3 NAT Gateway → NAT Instance (-$29/mês)

- **NAT Gateway**: $33.30/mês
- **NAT Instance** (EC2 t4g.nano com Amazon Linux): ~$4/mês
- Throughput: t4g.nano suporta até ~250 Mbps — suficiente para dev

**Economia**: **-$29/mês**

**Trade-off**: Gerenciamento manual (updates, failover). Em dev isso é OK.

### 3.4 Aurora Serverless v2 em vez de RDS (opcional)
- Aurora Serverless v2 escala de 0.5 ACU a N
- Em dev com pouco uso: ~$43/mês (mínimo 0.5 ACU × 730h × $0.12)
- **Pior que RDS t3.micro** em dev

**NÃO recomendado em dev.**

### Total Cenário B
| Item | Atual | Otimizado | Economia |
|---|---|---|---|
| ALB | $22.27 | $3.50 (Lambda) | -$18.77 |
| NAT Gateway | $33.30 | $4.00 (NAT instance) | -$29.30 |
| ElastiCache | $12.41 | $0 (in-memory) | -$12.41 |
| CloudFront | $2.05 | $1.05 | -$1.00 |
| CloudWatch Logs | $2.65 | $1.15 | -$1.50 |
| **Total mensal** | **$97** | **$34** | **-$63** |

---

## 4. Cenário C — Dev Minimalista (~$15/mês)

**Redução: ~85%** — Ideal para 1-2 devs em fase de desenvolvimento ativo.

### Arquitetura proposta

```
┌──────────────────────────────────────┐
│ EC2 t4g.small (2 vCPU, 2 GB RAM)     │
│ - Docker Compose rodando:            │
│   ├── FastAPI                         │
│   ├── PostgreSQL 16                   │
│   └── Redis 7                         │
│ - Caddy como reverse proxy + SSL     │
└──────────────────────────────────────┘
        ↓
    CloudFront (free tier)
        ↓
    api.palavracadabra.com.br
```

### Recursos
| Item | Custo |
|---|---|
| **EC2 t4g.small** (spot ou reserved) | $8/mês |
| **EBS 30 GB gp3** | $2.40/mês |
| **Elastic IP** | $0 (em uso) |
| **CloudFront** | $1/mês |
| **S3** (assets + backups) | $0.50/mês |
| **Route53** (opcional para failover) | $0.50/mês |
| **CloudWatch Logs** básico | $1/mês |
| **Data Transfer** | $0.50/mês |
| **Total** | **~$14/mês** |

### Vantagens
- **Custo mínimo**: 85% de economia
- **Simplicidade operacional**: tudo em um servidor
- **Fácil debug**: SSH direto no servidor
- **Ideal para ambientes de feature branches**

### Desvantagens
- **Sem alta disponibilidade**: ponto único de falha
- **Backup manual**: não tem snapshot automático do RDS
- **Manutenção manual**: atualizações do OS, Docker, etc.
- **Não escala**: se usuário dobrar, precisa fazer upgrade manual

### Quando usar
- Dev ativo com 1-3 pessoas
- Staging para QA
- Ambientes temporários de feature branches

### Quando NÃO usar
- Produção
- Se SLA for crítico
- Se não quiser gerenciar servidor manualmente

---

## 5. Comparação dos Cenários

| Cenário | Custo/mês | Economia | Complexidade | HA | Uso Recomendado |
|---|---|---|---|---|---|
| **Atual** | $97 | — | Baixa | Parcial | Dev com padrão enterprise |
| **Cenário A** | $93 | 4% | Baixa | Parcial | Ajustes mínimos |
| **Cenário B** | $34 | 65% | Média | Não | Dev otimizado |
| **Cenário C** | $14 | 86% | Alta | Não | Dev minimalista / feature branches |

---

## 6. Recomendação

### Para o Palavra Cadabra no momento atual:

**Recomendo Cenário B (~$34/mês)** com os seguintes ajustes:

1. **Manter RDS t3.micro** (backups automáticos são importantes mesmo em dev)
2. **Eliminar Redis** em dev (usar cache em memória no FastAPI)
3. **NAT Gateway → NAT Instance** t4g.nano (economia $29/mês)
4. **Manter ALB** por enquanto (migração para Lambda é complexa)
5. **Reduzir logs retention** para 7 dias
6. **Consolidar CloudFronts**

### Custo projetado do Cenário B ajustado:
| Item | Valor |
|---|---|
| ECS Fargate | $9.01 |
| ALB | $22.27 |
| RDS t3.micro + 20 GB | $14.71 |
| NAT Instance (t4g.nano) | $4.00 |
| S3 + CloudFront consolidado | $1.50 |
| CloudWatch | $1.15 |
| Outros | $1.00 |
| **Total** | **~$53/mês** |

**Economia: $44/mês (~45%)**

---

## 7. Script de Migração para Cenário B

### Passo 1: Desabilitar Redis na API

Editar `app/config.py` para permitir modo sem Redis:
```python
REDIS_ENABLED: bool = False  # Em dev, usar cache em memória
```

Depois, remover o recurso do Terraform.

### Passo 2: Substituir NAT Gateway por NAT Instance

```hcl
# terraform/modules/networking/nat_instance.tf
resource "aws_instance" "nat" {
  ami                    = "ami-0c02fb55956c7d316"  # Amazon Linux 2 ARM
  instance_type          = "t4g.nano"
  subnet_id              = aws_subnet.public[0].id
  vpc_security_group_ids = [aws_security_group.nat_instance.id]
  source_dest_check      = false

  user_data = <<-EOF
    #!/bin/bash
    echo 1 > /proc/sys/net/ipv4/ip_forward
    iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
  EOF
}

# Remover aws_nat_gateway.main
# Atualizar route table privada para usar a instance
```

### Passo 3: Configurar Log Retention

```hcl
resource "aws_cloudwatch_log_group" "api" {
  name              = "/ecs/palavracadabra-dev/api"
  retention_in_days = 7
}
```

### Passo 4: Aplicar mudanças

```bash
cd terraform/environments/dev
terraform plan -var="db_password=..."
terraform apply -var="db_password=..."
```

---

## 8. Observações Finais

### 8.1 Custos fixos inevitáveis (mesmo em dev minimalista)
- **Domínio**: R$ 40/ano (registro.br)
- **ACM SSL**: Grátis
- **Anthropic API**: ~$15-30/mês (pago à parte)
- **Apple Developer**: $99/ano
- **Google Play Console**: $25 (única vez)

### 8.2 Free Tier que ajuda
- Cognito (50K MAU)
- CloudFront (1 TB/mês no primeiro ano)
- Lambda (1M requests/mês)
- S3 (5 GB no primeiro ano — já expirou)

### 8.3 Arquivos Relacionados
- [CUSTOS_ATUAL.md](CUSTOS_ATUAL.md) — Estado atual detalhado
- [PROPOSTA_ESCALABILIDADE.md](PROPOSTA_ESCALABILIDADE.md) — Escalando para produção real
