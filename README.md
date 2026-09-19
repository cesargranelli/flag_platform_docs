# Documentação Técnica — Flag Platform

Repositório central de documentação e especificações técnicas de todo o ecossistema da **Flag Platform**.

> **Publicação Online:** Acesse o portal renderizado no [GitHub Pages](https://cesargranelli.github.io/flag-platform-docs).  
> **Processo de Contribuição:** Seguimos o fluxo Gitflow documentado na [ADR-013](adr/ADR-013-gitflow-documentacao.md).

---

## 1. Decisões de Arquitetura (ADRs)

Os Architecture Decision Records (ADRs) registram as decisões técnicas mandatórias do projeto, operando sob o princípio de **documentos vivos** ([ADR-014](adr/ADR-014-atualizacao-diretivas.md)):

| ADR | Título | Resumo da Decisão |
|:---:|--------|-------------------|
| **001** | [Nova Filosofia de Arquitetura e Simplificação](adr/ADR-001-nova-filosofia-arquitetura.md) | Hierarquia de 5 níveis, 4 roles, Firebase-First, Modular Monolith e CQRS Light. |
| **002** | [Estratégia de Dados Híbrida (PostgreSQL + Firestore)](adr/ADR-002-postgres-firestore-cqs.md) | PostgreSQL como fonte da verdade (ACID/Escrita) e Firestore como espelho para leitura em tempo real. |
| **003** | [Diagramas de Base de Dados](adr/ADR-003-diagramas-base-de-dados.md) | Diagramas E-R, esquemas das tabelas relacionais e dependências entre módulos. |
| **004** | [Diagramas do Projeto](adr/ADR-004-diagramas-projeto.md) | Diagramas de arquitetura C4, fluxos de sequência e componentes. |
| **005** | [Staging Efêmero E2E](adr/ADR-005-staging-efemero-e2e.md) | Criação de ambientes temporários isolados para execução de testes ponta a ponta. |
| **006** | [Team / Roster / Season Refactor](adr/ADR-006-team-roster-season-refactor.md) | Separação estrita de time, elenco vinculado por temporada e histórico do atleta. |
| **010** | [Autenticação Centralizada Firebase-First](adr/ADR-010-autenticacao-firebase-custom-claims.md) | IdP Firebase Auth em todos os clientes, Custom Claims stateless e Security Rules. |
| **011** | [Padrão Arquitetural Flutter MVVM](adr/ADR-011-flutter-mvvm-architecture.md) | Estrutura obrigatória das aplicações cliente: Domain, Data, Repository (cache) e ViewModel. |
| **012** | [Migrations com jOOQ e Flyway](adr/ADR-012-migrations-jooq-java.md) | Evolução versionada de schema em Java com tipos seguros via jOOQ. |
| **013** | [Gitflow para Gestão de Documentação](adr/ADR-013-gitflow-documentacao.md) | Ciclo de vida de documentação usando branches de feature, develop, release e tags. |
| **014** | [Atualização de Diretivas em ADRs](adr/ADR-014-atualizacao-diretivas.md) | Princípio de documentos vivos: atualizar a ADR preservando histórico em vez de substituí-la. |

---

## 2. Arquitetura da Plataforma

Documentação transversal que rege todo o ecossistema e suas integrações:

| Documento | Conteúdo |
|-----------|----------|
| [Visão Geral da Solução](architecture/overview.md) | Apresentação da plataforma, arquitetura de camadas e jornada ponta a ponta. |
| [Mapa de Componentes](architecture/components.md) | Catálogo de todos os serviços de backend, clientes web/mobile e infraestrutura. |
| [Fluxos Lógicos e Sequência](architecture/logical-flows.md) | Diagramas Mermaid de autenticação, check-in, scoring em tempo real e chaveamento. |
| [Papéis, Permissões e Hierarquia](architecture/roles-and-permissions.md) | Modelo de autorização: `SUPER_ADMIN`, `ORG_ADMIN`, `MANAGER` e `USER`, com matriz de acesso. |
| [Análise de Custo-Benefício em Nuvem](architecture/cloud-cost-benefit-analysis.md) | Comparativo de custos, dimensionamento de memória, Cloudflare R2/Pages, Cloud Run e PostgreSQL. |
| [Mapeamento de Eventos por Modalidade](architecture/mapeamento-eventos-modalidades.md) | Catálogo e padronização oficial de lances para Flag 5x5, 7x7, 8x8, 9x9 e Full Pads 11x11. |
| [Análise de Enums do Jogo](architecture/analise-completa-enums-backend.md) | Alinhamento detalhado de enums e regras de pontuação entre backend e aplicativos. |
| [Arquitetura do Módulo de Competições](architecture/arquitetura_modulo_competicoes.md) | Modelagem de competições, fases eliminatórias, grupos e geração de partidas. |
| [Arquitetura do Delegado e App Público](architecture/arquitetura_delegado_e_app_publico.md) | Fluxo de validação de súmula pelo delegado e publicação para os torcedores. |
| [Características do Referee App](architecture/flag_referee_app_caracteristicas.md) | Requisitos funcionais, modos de operação offline e regras da mesa. |
| [Filiação e Inscrição de Agremiações](architecture/fluxo_filiacao_e_inscricao_agremiacoes.md) | Ciclo de vida do registro de agremiações e times nos campeonatos. |
| [Guia de Competições e Partidas](architecture/guia-passo-a-passo-competicoes-e-partidas.md) | Manual passo a passo para configuração de novos torneios e tabelas de jogos. |

---

## 3. Aplicações e Módulos do Ecossistema

Documentações especializadas por projeto/repositório:

| Aplicação | Repositório | Documentação |
|:---:|:---:|---|
| **Backend API** | [`flag_backend`](https://github.com/cesargranelli/flag_backend) | [Visão Geral](apps/backend/README.md) • [Modelo de Pessoas e Atletas](apps/backend/data-model-persons.md) • [Contratos Admin](apps/backend/flag-admin-web-data-model.md) • [Contratos Referee](apps/backend/flag-referee-app-data-model.md) |
| **Admin Web** | [`flag_admin_web`](https://github.com/cesargranelli/flag_admin_web) | [Visão Geral](apps/admin-web/README.md) • [Otimização de API](apps/admin-web/analise-otimizacao-api.md) • [Harmonização Forms](design/modelo_visual_harmonizacao_forms.md) |
| **Referee App** | [`flag_referee_app`](https://github.com/cesargranelli/flag_referee_app) | [Visão Geral](apps/referee-app/README.md) • [Fluxo de Telas](apps/referee-app/fluxo-de-telas.md) • [Diagrama Draw.io](apps/referee-app/fluxo-de-telas.drawio) • [Eventos](architecture/mapeamento-eventos-modalidades.md) |
| **Public App** | [`flag_public_app`](https://github.com/cesargranelli/flag_public_app) | [Visão Geral](apps/public-app/README.md) • [Fluxo de Navegação](apps/public-app/screen-flow.md) • [Diagrama Draw.io](apps/public-app/assets/flag-public-app-flow.drawio) |
| **Infraestrutura** | [`flag_platform_infra`](https://github.com/cesargranelli/flag_platform_infra) | [Visão Geral](apps/infra/README.md) • [Arquitetura de Infra](apps/infra/architecture.md) • [Runbook Operacional](apps/infra/runbook.md) |

---

## 4. Design System Kickster & UX

Diretrizes visuais obrigatórias para todas as interfaces da Flag Platform:

| Documento | Conteúdo |
|-----------|----------|
| [Design Tokens Oficiais](design/tokens.md) | Paleta de cores Shifty, escala tipográfica DM Sans, bordas e elevações. |
| [Especificação de Layout](design/layout-spec.md) | Grids responsivos, estrutura de páginas em coluna única ou dupla. |
| [Guia Kickster](design/kickster-reference.md) | Padrões de `KicksterButton`, `KicksterDropdown`, `KicksterCard`, `KicksterCalendar`, etc. |
| [Harmonização de Formulários](design/modelo_visual_harmonizacao_forms.md) | Layout fluido em página única com agrupamento por cards limpos. |
| [Padronização Organizações & Agremiações](design/modelo_visual_organizacao_agremiacao.md) | Consistência visual de filtros, paginação e listagem. |
| [Especificação de Telas de Autenticação](design/ux/admin-web-login.md) | Telas de Login, Recuperação de Senha e Cadastro. |
| [Referências Visuais Figma](design/ux/referencias.md) | Capturas de tela e fluxos de referência. |

---

## 5. Pesquisas de Mercado e Benchmarks

| Documento | Conteúdo |
|-----------|----------|
| [Mapeamento FlagStats](research/flagstats-mapeamento.md) | Levantamento de recursos do ecossistema FlagStats (FlagStat Go / StatHawk). |
| [Comparativo FlagStats](research/comparativo-flagstats.md) | Análise comparativa das telas de súmula e registro de jogadas ao vivo. |
| [Análise Competitiva de Mercado](research/market-analysis-restructured.md) | Estudo comparativo com GameChanger, TeamSnap, FlagRoster e Flag50. |

---

## 6. Planejamento Estratégico

| Documento | Conteúdo |
|-----------|----------|
| [Visão de Produto](product/vision.md) | Proposta de valor, dores do mercado, personas e fases de lançamento. |
| [Plano Diretor de Autenticação](_planning/plano-revisao-modulo-auth.md) | Roteiro de transição para Firebase Auth e padronização de claims. |
| [Estratégia CQRS Cloud Functions](_planning/estrategia-cqrs-cloud-functions.md) | Arquitetura de projeções para sincronizar PostgreSQL e Firestore. |
| [Plano de Migração Firebase](_planning/plano-migracao-firebase-auth.md) | Cronograma de implementação nos clientes web e mobile. |
