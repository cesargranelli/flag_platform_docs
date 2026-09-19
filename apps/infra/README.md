# Flag Platform Infra — Visão Geral & Operações

> **Repositório:** [`cesargranelli/flag_platform_infra`](https://github.com/cesargranelli/flag_platform_infra)  
> **Escopo:** Infraestrutura como Código (Docker Compose, Terraform, PostgreSQL 16, RabbitMQ, Cloudflare)  
> **Documentação Específica do Repositório:** [`flag_platform_infra/docs/`](https://github.com/cesargranelli/flag_platform_infra/tree/main/docs)

---

## 1. Papel no Ecossistema

O repositório `flag_platform_infra` gerencia a automação e provisionamento da infraestrutura necessária para suportar todos os serviços da Flag Platform em ambientes locais (desenvolvimento), ambientes efêmeros de teste (staging E2E) e produção gerenciada.

---

## 2. Documentações Específicas (No Repositório `flag_platform_infra`)

A topologia dos containers locais e as instruções operacionais residem no repositório de infra:

| Documento Específico | Descrição |
|----------------------|-----------|
| [Arquitetura de Infraestrutura](https://github.com/cesargranelli/flag_platform_infra/blob/main/docs/architecture.md) | Topologia de rede, serviços em container, persistência e conectividade entre módulos. |
| [Runbook Operacional](https://github.com/cesargranelli/flag_platform_infra/blob/main/docs/runbook.md) | Procedimentos para subir o ambiente, aplicar migrações de banco, backups e restaurações. |

---

## 3. Diretrizes e Decisões Globais Aplicáveis

- [ADR-002 — Estratégia de Dados Híbrida (PostgreSQL + Firestore)](../../adr/ADR-002-postgres-firestore-cqs.md)
- [ADR-005 — Staging Efêmero para Testes E2E](../../adr/ADR-005-staging-efemero-e2e.md)
- [Análise de Custo-Benefício em Nuvem](../../architecture/cloud-cost-benefit-analysis.md)
