# Flag Platform Infra — Documentação Técnica

> **Repositório:** [`flag_platform_infra`](https://github.com/cesargranelli/flag_platform_infra)  
> **Stack:** Docker Compose, Terraform, PostgreSQL 16, RabbitMQ, Cloudflare (DNS, Pages, R2), GitHub Actions  
> **Ambientes:** Local Dev, Staging Efêmero (E2E), Produção

---

## 1. Visão Geral

O repositório `flag_platform_infra` gerencia a infraestrutura como código (IaC), orquestração de containers locais e na nuvem, pipelines de CI/CD e automações operacionais para todo o ecossistema Flag Platform.

---

## 2. Documentos e Especificações

| Documento | Descrição |
|-----------|-----------|
| [Arquitetura de Infraestrutura](architecture.md) | Topologia de rede, serviços em container, persistência e conectividade entre módulos. |
| [Runbook Operacional](runbook.md) | Guia passo a passo para inicialização de ambientes, rotinas de backup, restauração e troubleshooting. |
| [Análise de Custo-Benefício em Nuvem](../../architecture/cloud-cost-benefit-analysis.md) | Comparativo de custos entre arquiteturas (Cloudflare R2/Pages + Cloud Run vs Multi-Cloud vs Monolito). |

---

## 3. Decisões Arquiteturais Relacionadas

- [ADR-002 — CQRS Light com PostgreSQL e Firestore](../../adr/ADR-002-postgres-firestore-cqs.md)
- [ADR-005 — Staging Efêmero para Testes E2E](../../adr/ADR-005-staging-efemero-e2e.md)
