# Governança e Estrutura da Documentação — Flag Platform

> **Status:** Vigente  
> **Data:** 19 de Setembro de 2026  
> **Autor:** Tech Lead / Arquitetura  
> **Referência Cruzada:** [ADR-013 — Gitflow para Documentação](../adr/ADR-013-gitflow-documentacao.md) • [ADR-014 — Atualização de Diretivas](../adr/ADR-014-atualizacao-diretivas.md)

---

## 1. Regra de Ouro: Proibição de Documentos Fantasma em `.gemini`

> [!CAUTION]
> **REGRA MANDATÓRIA PARA AGENTES DE IA E DESENVOLVEDORES:**  
> **NUNCA salvar documentação, planos arquiteturais, especificações de telas, benchmarks ou decisões técnicas apenas em diretórios temporários ou locais de agentes (`~/.gemini/.../brain/`).**  
> Todo registro de conhecimento do projeto DEVE ser versionado em Git diretamente no repositório correspondente:
> - Decisões e arquiteturas globais: **`flag_platform_docs`**.
> - Especificações locais de componentes: **`docs/` do próprio repositório** (`flag_backend`, `flag_admin_web`, etc.).

Qualquer plano, análise ou especificação gerada durante sessões de pair programming deve ser imediatamente persistida no repositório e comitada.

---

## 2. Divisão de Responsabilidades: Hub Central vs. Repositórios

Adotamos a arquitetura de **Hub Central + Colocalização Específica** para balancear a visibilidade corporativa com a agilidade de desenvolvimento:

```
┌────────────────────────────────────────────────────────────────────────┐
│               flag_platform_docs (HUB CENTRAL GLOBAL)                  │
│                                                                        │
│  • ADRs da Plataforma (001 a 014)                                      │
│  • Arquitetura Transversal (Roles, CQRS, Fluxos de Autenticação)       │
│  • Análise de Custo-Benefício de Nuvem (Cloudflare, Cloud Run, Neon)   │
│  • Catálogo Oficial de Eventos e Regras por Modalidade (5x5 a 11x11)  │
│  • Design System Kickster Global (Tokens, Specs, Formulários)          │
│  • Pesquisas, Benchmarks (FlagStats) e Visão de Produto                │
│  • Planejamento Estratégico (Auth, Migrações, CQRS Projeções)          │
│  • Portal de Integração de Apps (visão de alto nível com links)       │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
       ┌────────────────────────────┼───────────────────────────┐
       ▼                            ▼                           ▼
┌──────────────┐             ┌──────────────┐            ┌──────────────┐
│ flag_backend │             │flag_admin_web│            │flag_referee_ │
│    /docs     │             │    /docs     │            │    /docs     │
│              │             │              │            │              │
│• Modelos DDL │             │• Cache & API │            │• Telas Mesa  │
│• Contratos   │             │• Telas Admin │            │• Súmula      │
│• Enums Java  │             │• ADRs locais │            │• Feature 2   │
└──────────────┘             └──────────────┘            └──────────────┘
```

### O que pertence a `flag_platform_docs`:
1. **ADRs (Architecture Decision Records):** Todas as decisões arquiteturais da plataforma ([ADR-001](../adr/ADR-001-nova-filosofia-arquitetura.md) a [ADR-014](../adr/ADR-014-atualizacao-diretivas.md)).
2. **Arquitetura de Domínio & Plataforma:** Modelo de permissões ([`roles-and-permissions.md`](roles-and-permissions.md)), fluxos lógicos ([`logical-flows.md`](logical-flows.md)) e infraestrutura na nuvem ([`cloud-cost-benefit-analysis.md`](cloud-cost-benefit-analysis.md)).
3. **Regras Esportivas Oficiais:** Catálogo de eventos e pontuações por modalidade ([`mapeamento-eventos-modalidades.md`](mapeamento-eventos-modalidades.md)).
4. **Design System Kickster:** Fonte da verdade visual ([`tokens.md`](../design/tokens.md), [`layout-spec.md`](../design/layout-spec.md)).
5. **Pesquisas e Benchmarks:** Estudos de mercado ([`market-analysis-restructured.md`](../research/market-analysis-restructured.md), [`comparativo-flagstats.md`](../research/comparativo-flagstats.md)).
6. **Planejamento Diretor:** Planos de migração e arquitetura ([`_planning/`](../_planning/)).

### O que pertence ao repositório específico (`<repo>/docs/`):
1. **`flag_backend/docs`:** Modelos relacionais de pessoa/atleta/participante, contratos de API para os consumidores e mapeamentos específicos de banco/jOOQ.
2. **`flag_admin_web/docs`:** Otimizações de rede locais, diretrizes de navegação web, ADRs locais do frontend e telas do painel.
3. **`flag_referee_app/docs`:** Fluxo de telas da mesa, diagramas editáveis draw.io, especificação da Feature 2 (operação ao vivo) e auditoria Kickster do app mobile.
4. **`flag_public_app/docs`:** Fluxo de navegação do torcedor, rotas públicas e diagramas visuais.
5. **`flag_platform_infra/docs`:** Topologia Docker Compose, scripts de inicialização local e runbook operacional.

---

## 3. Matriz de Rastreabilidade

| Repositório | Pasta Local | Papel Documental | Referência Global |
| :--- | :--- | :--- | :--- |
| **`flag_platform_docs`** | `/` (raiz Jekyll) | **Fonte Única da Verdade Global** | [Portal Pages](https://cesargranelli.github.io/flag-platform-docs) |
| **`flag_backend`** | `docs/` | Contratos e modelos da API | [apps/backend/](../apps/backend/README.md) |
| **`flag_admin_web`** | `docs/` | Regras de UI admin e otimizações | [apps/admin-web/](../apps/admin-web/README.md) |
| **`flag_referee_app`** | `docs/` | Operação de mesa e súmula | [apps/referee-app/](../apps/referee-app/README.md) |
| **`flag_public_app`** | `docs/` | Experiência do torcedor | [apps/public-app/](../apps/public-app/README.md) |
| **`flag_platform_infra`** | `docs/` | Operações e Docker Compose | [apps/infra/](../apps/infra/README.md) |
| **`flag_platform`** | `docs/` | Legado / Monorepo original | Redirecionado para `flag_platform_docs` |

---

## 4. Ciclo de Vida e Atualização

1. **Novas Decisões:** Devem seguir o Gitflow da [ADR-013](../adr/ADR-013-gitflow-documentacao.md) em branch `feature/doc-...` com Pull Request.
2. **Mudanças de Diretriz:** Aplicar o princípio de documentos vivos da [ADR-014](../adr/ADR-014-atualizacao-diretivas.md), preservando o histórico da decisão anterior.
3. **Sem Duplicações:** Nunca criar cópias de arquivos entre o hub central e os repositórios individuais. Usar links relativos ou URLs do GitHub.
