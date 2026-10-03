# Inventário do Hub `flag_platform_docs`

> **Status:** Levantamento concluído
> **Data:** 3 de Outubro de 2026
> **Autor:** Documentação / Governança (agente)
> **Branch do levantamento:** `docs/reorg-hub` (a partir de `docs/public-app-revisao-ux`, PR #38)
> **Escopo:** inventário das pastas/arquivos do repositório, problemas de organização e divergências com `main`.
> **Documento irmão:** [Centralização de Issues e Reorganização](centralizacao-issues-e-reorganizacao.md)

---

## 1. Sumário executivo

| Área | Arquivos rastreados | Observação |
|------|:---:|------------|
| Raiz (`README.md`, `index.md`, `backend-issues.md`, `_config.yml`) | 4 | `README` e `index` duplicam a navegação; `backend-issues.md` está solto |
| `.github/workflows/` | 1 | Deploy do GitHub Pages (Jekyll) |
| `_includes/` | 1 | `head-custom.html` |
| `_planning/` | 30 | **Tracker local** — 5 esforços + 3 planos soltos |
| `adr/` | 15 | Lacunas 007–009 (existem só no `main`) |
| `architecture/` | 15 | Inclui 1 `.puml` em `diagrams/` |
| `design/` | 21 | 8 `.md` + 6 `.json` (Figma) + 5 imagens + 2 `.md` em `ux/` |
| `apps/` | 5 | 1 README de visão geral por repositório |
| `product/` | 1 | `vision.md` |
| `research/` | 3 | Benchmarks |
| **Total** | **96** | |

**Natureza do repositório:** site Jekyll publicado no GitHub Pages (workflow só roda em `main`). A navegação oficial é `index.md` (links `.html`) e `README.md` (links `.md`) — mantidos manualmente e **em paralelo**.

---

## 2. Árvore de diretórios (1º/2º nível) e finalidade

```
flag_platform_docs/
├── .github/workflows/        # CI do portal (pages.yml → deploy Jekyll em main)
├── _includes/                # Overrides de tema Jekyll (head-custom.html)
├── _planning/                # TRACKER LOCAL (markdown) — esforços e planos
│   ├── firestore-sync/       #   esforço: espelhamento Oracle → Firestore
│   ├── motor-de-calculo-classificacao/  # esforço: motor de classificação (concluído → spec)
│   ├── playoffs-configuracao/ #   esforço: configuração de playoffs
│   ├── r2-imagens/           #   esforço: imagens no Cloudflare R2
│   ├── padronizacao-listagens/ #  esforço: paginação/filtros do admin web
│   ├── segredos-e-env/       #   esforço: auditoria de secrets (concluído)
│   └── (planos soltos na raiz de _planning)  # auth, CQRS
├── adr/                      # Architecture Decision Records
├── architecture/             # Documentação transversal da plataforma
│   └── diagrams/             #   PlantUML (modules.puml)
├── apps/                     # Porta de entrada por aplicação (1 README cada)
├── design/                   # Design System Kickster (tokens, specs, UX)
│   ├── figma-reference/      #   tokens/estilos exportados do Figma (JSON)
│   └── ux/                   #   especificações de UX + figma-ref/ (PNG)
├── product/                  # Visão de produto
├── research/                 # Pesquisas de mercado e benchmarks
├── _config.yml               # Config do Jekyll
├── index.md                  # Home do portal (links .html)
├── README.md                 # Espelho do index (links .md)
└── backend-issues.md         # Pendências do flag_backend (SOLTO — fora de _planning)
```

---

## 3. Inventário por arquivo

### 3.1 Raiz e infraestrutura do portal

| Caminho | Tipo | Status | Propósito |
|---|---|---|---|
| `README.md` | Navegação | — | Índice do repo com links relativos `.md`; duplica `index.md` |
| `index.md` | Navegação | — | Home do Jekyll com links `.html`; duplica `README.md` (com divergências) |
| `backend-issues.md` | Issues | Aberta | Pendências críticas do `flag_backend` (ISSUE-001..006) — **deveria estar em `_planning/`** |
| `_config.yml` | Config | — | Configuração do Jekyll |
| `_includes/head-custom.html` | Config | — | Override de `<head>` do tema |
| `.github/workflows/pages.yml` | CI | — | Build + deploy do GitHub Pages (só `main`) |

### 3.2 ADRs (`adr/`)

| Caminho | Status | Propósito |
|---|---|---|
| `ADR-001-nova-filosofia-arquitetura.md` | Proposto | Hierarquia, roles, Firebase-First, modular monolith, CQRS Light |
| `ADR-002-postgres-firestore-cqs.md` | Atualizado por ADR-018 | Oracle ADB primário + Firestore espelho CQRS |
| `ADR-003-diagramas-base-de-dados.md` | Proposto | Diagramas E-R e esquemas relacionais |
| `ADR-004-diagramas-projeto.md` | Proposto | Diagramas de fluxo/C4 do projeto |
| `ADR-005-staging-efemero-e2e.md` | Aceito | Staging efêmero + E2E como quality gate |
| `ADR-006-team-roster-season-refactor.md` | Proposto | Separação Team/Roster/Season |
| `ADR-010-autenticacao-firebase-custom-claims.md` | Aceito | Auth Firebase-First + custom claims stateless |
| `ADR-011-flutter-mvvm-architecture.md` | Aceito | Padrão MVVM Flutter (Domain/Data/Repository/VM) |
| `ADR-012-migrations-jooq-java.md` | Accepted | Migrations Flyway + jOOQ DSL (premissa Oracle/Liquibase no ADR-018) |
| `ADR-013-gitflow-documentacao.md` | Proposto | Gitflow para documentação |
| `ADR-014-atualizacao-diretivas.md` | Proposto | Princípio de documentos vivos |
| `ADR-015-importacao-rascunho-organizacoes.md` | Proposto | Modo `importDraft` de organizações |
| `ADR-016-gestao-segredos-github-environments.md` | Aceito | Gestão de segredos via GitHub Environments |
| `ADR-017-configuracao-playoffs.md` | aceito | Modelo de dados/estratégia de playoffs |
| `ADR-018-espelhamento-firestore-admin-sdk.md` | Aceito — a implementar | Espelhamento pelo backend com Firebase Admin SDK |

### 3.3 Arquitetura transversal (`architecture/`)

| Caminho | Status | Propósito |
|---|---|---|
| `overview.md` | — | Visão geral da solução e jornada ponta a ponta |
| `components.md` | — | Mapa de componentes do ecossistema multi-repo |
| `logical-flows.md` | — | Fluxos lógicos/sequência (auth, check-in, live, chaveamento) |
| `roles-and-permissions.md` | **desatualizado** (ver §5) | Modelo RBAC — cita `SUPER_ADMIN`/`ORG_ADMIN` divergentes do código |
| `cloud-cost-benefit-analysis.md` | Concluída / Revisada | Comparativo de custo e provedores de nuvem |
| `mapeamento-eventos-modalidades.md` | — | Catálogo oficial de lances por modalidade |
| `arquitetura_modulo_competicoes.md` | — | Modelagem de competições, fases, grupos e partidas |
| `arquitetura_delegado_e_app_publico.md` | — | Mesa do delegado + app público |
| `flag_referee_app_caracteristicas.md` | — | Requisitos e operação do Referee App |
| `fluxo_filiacao_e_inscricao_agremiacoes.md` | — | Ciclo de filiação/inscrição de agremiações |
| `guia-passo-a-passo-competicoes-e-partidas.md` | — | Roadmap operacional de competições e partidas |
| `env-vars-e-secrets.md` | Ativo | Catálogo de variáveis de ambiente e segredos |
| `governanca-documentacao.md` | Vigente | Regra anti-docs-fantasma; Hub vs. repos |
| `governanca-agentes-skills.md` | — | Casas de skills/agentes/trackers (anti-duplicação) |
| `diagrams/modules.puml` | — | Diagrama PlantUML de módulos |

### 3.4 Design System (`design/`)

| Caminho | Propósito |
|---|---|
| `tokens.md` | Design tokens oficiais (cores, tipografia, bordas) |
| `layout-spec.md` | Grids e estrutura de páginas do admin web |
| `kickster-reference.md` | Referência dos componentes `Kickster*` (Figma) |
| `modelo_visual_harmonizacao_forms.md` | Harmonização visual dos formulários |
| `modelo_visual_organizacao_agremiacao.md` | Padrão visual de organizações/agremiações |
| `modelo_visual_padronizacao_agremiacao_filiacao.md` | Padronização de cadastro/edição + filiação |
| `padroes_assets_publicacao.md` | Padrões de assets para publicação (mobile/web) |
| `public-app-revisao-ux.md` | Revisão de UX/UI do Public App |
| `harmonizacao_forms_admin_1788822988849.jpg` | Imagem de referência |
| `padronizacao_agremiacao_filiacao_1788821314553.jpg` | Imagem de referência |
| `figma-reference/kickster_{auth,components_home_live,dropdown,dropdown_list,element,styleguide}.json` | 6 exports de estilo/token do Figma |
| `ux/admin-web-login.md` | Especificação das telas de login/signup |
| `ux/referencias.md` | Referências de design (login/signup + timeline) |
| `ux/figma-ref/{login,signup,forgot-password}.png` | Capturas de referência |

### 3.5 Produto, pesquisa e apps

| Caminho | Propósito |
|---|---|
| `product/vision.md` | Proposta de valor, personas e fases de lançamento |
| `research/market-analysis.md` | Análise competitiva de mercado |
| `research/comparativo-flagstats.md` | Comparativo de telas/súmula vs. FlagStats |
| `research/flagstats-mapeamento.md` | Mapeamento de recursos do ecossistema FlagStats |
| `apps/backend/README.md` | Visão geral e integração do `flag_backend` |
| `apps/admin-web/README.md` | Visão geral do `flag_admin_web` |
| `apps/referee-app/README.md` | Visão geral do `flag_referee_app` |
| `apps/public-app/README.md` | Visão geral do `flag_public_app` |
| `apps/infra/README.md` | Visão geral do `flag_platform_infra` |

### 3.6 Tracker local (`_planning/`) — 30 arquivos

| Esforço | Arquivos | Status | Propósito |
|---|---|---|---|
| `firestore-sync/` | `map.md` + `issues/01,02,06,07,08,09` | **aberto** (issues `open`) | Espelhamento Oracle→Firestore (FS-01..FS-09; mapa lista FS-03..FS-05 sem arquivo) |
| `motor-de-calculo-classificacao/` | `map.md` + `spec.md` + `issues/01..11` + `assets/08` | **concluído** (tickets `resolved`) | Wayfinding até a spec do motor de classificação |
| `playoffs-configuracao/` | `map.md` + `spec.md` | **rascunho** | Configuração de playoffs (spec em rascunho) |
| `r2-imagens/` | `map.md` + `issues/01` | **aberto** | Endurecimento de imagens no Cloudflare R2 (R2-01..R2-05) |
| `padronizacao-listagens/` | `plan.md` | **em execução** | Paginação + filtros no admin web (F1/F2 feitos; F3 = docs pendente) |
| `segredos-e-env/` | `README.md` | **concluído (rot/purga adiadas)** | Auditoria e sanitização de env/secrets |
| (solto) `estrategia-cqrs-cloud-functions.md` | 1 | **Proposto (superado)** | ADR-007 relocado; superseded pelo ADR-018 |
| (solto) `plano-migracao-firebase-auth.md` | 1 | — | Plano de migração Firebase Auth (relocado de `adr/`) |
| (solto) `plano-revisao-modulo-auth.md` | 1 | Proposto / Em Planejamento | Plano diretor do módulo `auth` |

---

## 4. Contagem e sinais de histórico/obsoletos

- **Concluídos/arquiváveis (com valor histórico):** `_planning/motor-de-calculo-classificacao/` (11 tickets `resolved` + spec), `_planning/segredos-e-env/` (sanitização concluída), `_planning/padronizacao-listagens/plan.md` (F1/F2 feitos).
- **Superados:** `_planning/estrategia-cqrs-cloud-functions.md` (ADR-007 → superseded pelo ADR-018), `_planning/plano-migracao-firebase-auth.md` (relocado de `adr/`).
- **Possível inconsistência de ADR:** `ADR-012` (título "Flyway/jOOQ") vs. runtime real **Liquibase/Oracle**; o ADR-002 e o ADR-018 já corrigem a premissa, mas o texto do ADR-012 não.
- **Sem placeholders `TODO/TBD/FIXME`** nos documentos (apenas usos legítimos de "placeholder" no sentido de template/env).

---

## 5. Problemas de organização (ranqueados)

1. **`README.md` × `index.md` duplicados e divergentes.** Os dois mantêm a mesma navegação com estilos de link diferentes (`.md` vs `.html`). Já divergem: `README` §4 lista "Referências Visuais Figma"; `index` §4 não. Nenhum dos dois cita os esforços novos de `_planning/` (firestore-sync, r2-imagens, motor, playoffs).
2. **Lacuna na numeração de ADRs e divergência de títulos com `main`.** Nesta branch existem `001–006` e `010–018`; **faltam `007, 008, 009`**. No `main`, `ADR-003/004` têm **outros títulos** (`modular-monolith`, `firebase-auth-migration`) e `007–009` existem. Ou seja, a numeração não é estável entre as duas linhagens (ver §6).
3. **Documentos relocados criam duplicatas entre branches.** `estrategia-cqrs-cloud-functions.md` está em `_planning/` (aqui) e em `adr/` (no `main`); `plano-migracao-firebase-auth.md` idem. `fluxo-de-telas.*` do referee app saiu do hub (estava em `apps/referee-app/` no `main`).
4. **Issues de trabalho fora de `_planning/`.** `backend-issues.md` está solto na raiz do hub, com IDs próprios (`ISSUE-001..006`) que não seguem a convenção dos esforços (`FS-01`, `R2-01`, …). Há ainda **2 GitHub Issues abertas** no próprio hub (#5, #37), contrariando a regra de tracker local.
5. **Referências quebradas a arquivo inexistente.** `architecture/governanca-documentacao.md:58` e `adr/ADR-002-postgres-firestore-cqs.md:110` apontam para **`market-analysis-restructured.md`**, que não existe (o arquivo atual é `research/market-analysis.md`).
6. **Docs desatualizados vs. código.** `architecture/roles-and-permissions.md` cita `SUPER_ADMIN`/`ORG_ADMIN` e `category_id` em `standings`, divergindo do RBAC real (`ADMIN/ORGANIZER/COMMISSIONER/REFEREE/MANAGER/FAN`) já registrado no mapa do motor de classificação.
7. **READMEs de app com premissas antigas.** `apps/backend/README.md` descreve "PostgreSQL 16, jOOQ, Flyway"; `apps/infra/README.md` cita "Terraform, PostgreSQL 16, RabbitMQ" — divergem do runtime Oracle ADB/Liquibase/OCI já consolidado nos ADRs 002/012/018.
8. **Esforço aberto com issues sem arquivo.** O `firestore-sync/map.md` lista `FS-03/04/05` só na tabela ("ficam só na tabela até virarem fatia"), enquanto `01/02/06/07/08/09` têm arquivo — mistura de granularidades dentro do mesmo esforço.

---

## 6. Divergências com `main` (branch `docs/reorg-hub`)

- Relação com `origin/main`: **58 commits atrás / 26 à frente** (merge-base `832dcced`). O PR **#38** (para `main`) ainda está aberto.
- **Presentes só no `main`** (17 caminhos): `.ai/project-context.md`; `adr/ADR-003-modular-monolith.md`, `adr/ADR-004-firebase-auth-migration.md`, `adr/ADR-007..009`; `architecture/ajustes-autenticacao.md`, `architecture/autenticacao-hibrida.md`; `apps/referee-app/fluxo-de-telas.{md,drawio}`; `plans/plano_reestruturacao_fase_0..5.md`; `research/market-analysis-restructured.md`.
- **Apenas nesta branch** (26 commits, ~63 arquivos adicionados): todo o `_planning/`, os ADRs 010–018, `architecture/` expandida, `design/` expandida, `backend-issues.md`.
- **Consequência:** o `main` é uma linhagem paralela (PRs de tokens + `plans/` + `.ai/`) que **não contém** o trabalho deste PR #38. Reunificar exige decisão do dono sobre qual numeração de ADR prevalece.

---

## 7. Recomendações de reorganização (proposta)

1. **Fonte única de navegação:** manter `index.md` como home Jekyll gerada a partir de um único sumário; converter `README.md` em ponteiro curto para `index.md` (elimina a duplicação).
2. **Fechar a lacuna de ADRs:** reconciliar com `main` (decidir se 003/004 desta branch viram novos números ou se os do `main` prevalecem) e **nunca reutilizar número**.
3. **Centralizar issues:** mover `backend-issues.md` para `_planning/` (ver [centralizacao](centralizacao-issues-e-reorganizacao.md)); converter GitHub Issues do hub em arquivos locais ou marcá-las explicitamente como exceção.
4. **Corrigir links quebrados** apontando para `research/market-analysis.md`.
5. **Atualizar `roles-and-permissions.md` e os READMEs de app** para o runtime Oracle/Liquibase/OCI.
6. **Arquivar esforços concluídos** (motor, segredos, padronização) sob `_planning/_archive/` preservando histórico.

---

## 8. Referências

- [Governança de Documentação](governanca-documentacao.md)
- [Governança de Agentes e Skills](governanca-agentes-skills.md) — define o tracker local
- [Centralização de Issues e Reorganização](centralizacao-issues-e-reorganizacao.md) — documento irmão
