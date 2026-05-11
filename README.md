# Docs.palavraCadabra

Documentação central do projeto **Palavra Cadabra** — plataforma de Comunicação Aumentativa e Alternativa (CAA) para pessoas não verbais, com alfabetização integrada e pesquisa científica.

## Estrutura

```
docs/
├── PLANEJAMENTO_PROJETO.md       — Documento mestre (visão, stack, fases)
├── desenvolvimento/
│   ├── HISTORICO_EVOLUCAO.md     — Linha do tempo das evoluções
│   ├── GUIA_OPERACIONAL.md       — Como rodar, deployar, debugar
│   └── CHANGELOG.md              — Registro de mudanças
└── infrastructure/
    ├── CUSTOS_ATUAL.md           — Custos AWS detalhados
    ├── PROPOSTA_REDUCAO_CUSTOS.md — Cenários de otimização
    └── PROPOSTA_ESCALABILIDADE.md — Plano de escala (beta → 100K MAU)
```

## Links Rápidos

- 🌐 [Produção](https://www.palavracadabra.com.br)
- 📡 [API Swagger](https://api.palavracadabra.com.br/docs)
- 📱 [Baixar app](https://www.palavracadabra.com.br/baixar)
- 🔬 [Privacidade (LGPD)](https://www.palavracadabra.com.br/privacidade)

## Outras Refs

- [GitHub org](https://github.com/PalavraCadabra)
- Stack: Flutter + FastAPI + Next.js + PostgreSQL + AWS
- Símbolos: ARASAAC (13.733 pictogramas pt-BR)
- IA: Anthropic Claude API

## Status

MVP completo + Beta-ready. 43 endpoints API, 953 símbolos em produção, 15 atividades de letramento, 7 pranchas demo, autenticação JWT + LGPD-compliant.
