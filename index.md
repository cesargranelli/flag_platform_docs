---
layout: home
---

# Documentação — Flag Platform

Portal unificado de documentação técnica, decisões de arquitetura e especificações de produto da **Flag Platform**.

---

## 1. Decisões de Arquitetura (ADRs)

| ADR | Título | Resumo da Decisão |
|:---:|--------|-------------------|
| **001** | [Nova Filosofia de Arquitetura](adr/ADR-001-nova-filosofia-arquitetura.html) | Hierarquia de 5 níveis, 4 roles, Firebase-First, Modular Monolith e CQRS Light. |
| **002** | [Estratégia de Dados Híbrida](adr/ADR-002-postgres-firestore-cqs.html) | PostgreSQL como fonte da verdade (ACID/Escrita) e Firestore para leitura em tempo real. |
| **003** | [Diagramas de Base de Dados](adr/ADR-003-diagramas-base-de-dados.html) | Diagramas E-R, esquemas das tabelas relacionais e dependências entre módulos. |
| **004** | [Diagramas do Projeto](adr/ADR-004-diagramas-projeto.html) | Diagramas de arquitetura C4, fluxos de sequência e componentes. |
| **005** | [Staging Efêmero E2E](adr/ADR-005-staging-efemero-e2e.html) | Criação de ambientes temporários isolados para execução de testes ponta a ponta. |
| **006** | [Team / Roster / Season Refactor](adr/ADR-006-team-roster-season-refactor.html) | Separação estrita de time, elenco vinculado por temporada e histórico do atleta. |
| **010** | [Autenticação Centralizada Firebase-First](adr/ADR-010-autenticacao-firebase-custom-claims.html) | IdP Firebase Auth em todos os clientes, Custom Claims stateless e Security Rules. |
| **011** | [Padrão Arquitetural Flutter MVVM](adr/ADR-011-flutter-mvvm-architecture.html) | Estrutura obrigatória das aplicações cliente: Domain, Data, Repository (cache) e ViewModel. |
| **012** | [Migrations com jOOQ e Flyway](adr/ADR-012-migrations-jooq-java.html) | Evolução versionada de schema em Java com tipos seguros via jOOQ. |
| **013** | [Gitflow para Gestão de Documentação](adr/ADR-013-gitflow-documentacao.html) | Ciclo de vida de documentação usando branches de feature, develop, release e tags. |
| **014** | [Atualização de Diretivas em ADRs](adr/ADR-014-atualizacao-diretivas.html) | Princípio de documentos vivos: atualizar a ADR preservando histórico em vez de substituí-la. |

---

## 2. Arquitetura da Plataforma

| Documento | Conteúdo |
|-----------|----------|
| [Visão Geral da Solução](architecture/overview.html) | Apresentação da plataforma, arquitetura de camadas e jornada ponta a ponta. |
| [Mapa de Componentes](architecture/components.html) | Catálogo de todos os serviços de backend, clientes web/mobile e infraestrutura. |
| [Fluxos Lógicos e Sequência](architecture/logical-flows.html) | Diagramas Mermaid de autenticação, check-in, scoring em tempo real e chaveamento. |
| [Papéis, Permissões e Hierarquia](architecture/roles-and-permissions.html) | Modelo de autorização: `SUPER_ADMIN`, `ORG_ADMIN`, `MANAGER` e `USER`, com matriz de acesso. |
| [Análise de Custo-Benefício em Nuvem](architecture/cloud-cost-benefit-analysis.html) | Comparativo de custos, dimensionamento de memória, Cloudflare R2/Pages, Cloud Run e PostgreSQL. |
| [Mapeamento de Eventos por Modalidade](architecture/mapeamento-eventos-modalidades.html) | Catálogo e padronização oficial de lances para Flag 5x5, 7x7, 8x8, 9x9 e Full Pads 11x11. |
| [Análise de Enums do Jogo](architecture/analise-completa-enums-backend.html) | Alinhamento detalhado de enums e regras de pontuação entre backend e aplicativos. |
| [Arquitetura do Módulo de Competições](architecture/arquitetura_modulo_competicoes.html) | Modelagem de competições, fases eliminatórias, grupos e geração de partidas. |
| [Arquitetura do Delegado e App Público](architecture/arquitetura_delegado_e_app_publico.html) | Fluxo de validação de súmula pelo delegado e publicação para os torcedores. |
| [Características do Referee App](architecture/flag_referee_app_caracteristicas.html) | Requisitos funcionais, modos de operação offline e regras da mesa. |
| [Filiação e Inscrição de Agremiações](architecture/fluxo_filiacao_e_inscricao_agremiacoes.html) | Ciclo de vida do registro de agremiações e times nos campeonatos. |
| [Guia de Competições e Partidas](architecture/guia-passo-a-passo-competicoes-e-partidas.html) | Manual passo a passo para configuração de novos torneios e tabelas de jogos. |

---

## 3. Aplicações e Módulos do Ecossistema

| Aplicação | Documentação Disponível |
|:---:|---|
| **Backend API (`flag_backend`)** | [Visão Geral](apps/backend/) • [Modelo de Pessoas e Atletas](apps/backend/data-model-persons.html) • [Contratos Admin](apps/backend/flag-admin-web-data-model.html) • [Contratos Referee](apps/backend/flag-referee-app-data-model.html) |
| **Admin Web (`flag_admin_web`)** | [Visão Geral](apps/admin-web/) • [Otimização de API](apps/admin-web/analise-otimizacao-api.html) • [Harmonização Forms](design/modelo_visual_harmonizacao_forms.html) |
| **Referee App (`flag_referee_app`)** | [Visão Geral](apps/referee-app/) • [Fluxo de Telas](apps/referee-app/fluxo-de-telas.html) • [Eventos por Modalidade](architecture/mapeamento-eventos-modalidades.html) |
| **Public App (`flag_public_app`)** | [Visão Geral](apps/public-app/) • [Fluxo de Navegação](apps/public-app/screen-flow.html) |
| **Infraestrutura (`flag_platform_infra`)** | [Visão Geral](apps/infra/) • [Arquitetura de Infra](apps/infra/architecture.html) • [Runbook Operacional](apps/infra/runbook.html) |

---

## 4. Design System Kickster & UX

| Documento | Conteúdo |
|-----------|----------|
| [Design Tokens Oficiais](design/tokens.html) | Paleta de cores Shifty, escala tipográfica DM Sans, bordas e elevações. |
| [Especificação de Layout](design/layout-spec.html) | Grids responsivos, estrutura de páginas em coluna única ou dupla. |
| [Guia Kickster](design/kickster-reference.html) | Padrões de `KicksterButton`, `KicksterDropdown`, `KicksterCard`, `KicksterCalendar`, etc. |
| [Harmonização de Formulários](design/modelo_visual_harmonizacao_forms.html) | Layout fluido em página única com agrupamento por cards limpos. |
| [Padronização Organizações & Agremiações](design/modelo_visual_organizacao_agremiacao.html) | Consistência visual de filtros, paginação e listagem. |
| [Especificação de Telas de Autenticação](design/ux/admin-web-login.html) | Telas de Login, Recuperação de Senha e Cadastro. |

---

## 5. Pesquisas de Mercado e Benchmarks

| Documento | Conteúdo |
|-----------|----------|
| [Mapeamento FlagStats](research/flagstats-mapeamento.html) | Levantamento de recursos do ecossistema FlagStats (FlagStat Go / StatHawk). |
| [Comparativo FlagStats](research/comparativo-flagstats.html) | Análise comparativa das telas de súmula e registro de jogadas ao vivo. |
| [Análise Competitiva de Mercado](research/market-analysis-restructured.html) | Estudo comparativo com GameChanger, TeamSnap, FlagRoster e Flag50. |

---

## 6. Planejamento Estratégico

| Documento | Conteúdo |
|-----------|----------|
| [Visão de Produto](product/vision.html) | Proposta de valor, dores do mercado, personas e fases de lançamento. |
| [Plano Diretor de Autenticação](_planning/plano-revisao-modulo-auth.html) | Roteiro de transição para Firebase Auth e padronização de claims. |
| [Estratégia CQRS Cloud Functions](_planning/estrategia-cqrs-cloud-functions.html) | Arquitetura de projeções para sincronizar PostgreSQL e Firestore. |
| [Plano de Migração Firebase](_planning/plano-migracao-firebase-auth.html) | Cronograma de implementação nos clientes web e mobile. |