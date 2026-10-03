# Reconciliação de ADRs entre Linhagens (`main` × `docs/public-app-revisao-ux`)

> **Status:** **Implementado** (merge de `origin/main` em `docs/public-app-revisao-ux`, PR #38).
> **Data:** 3 de Outubro de 2026
> **Autor:** Documentação / Governança (agente)
> **Base:** decisão do dono para tornar o PR #38 mergeável.

---

## 1. Objetivo

Reconciliar a numeração de ADRs das duas linhagens na reunificação do PR #38, eliminando colisões de
número e **sem deixar links quebrados**, aplicando integralmente a decisão do dono:

1. Manter os nossos **ADR-003** (Diagramas de Base de Dados) e **ADR-004** (Diagramas de Fluxo do Projeto)
   e renumerar os do `main` que colidiam.
2. Absorver os ADRs **008** e **009** do `main` nos documentos vivos correspondentes e remover os arquivos
   standalone.
3. Corrigir o conteúdo do **ADR-012** (runtime real).
4. Registrar aqui o resultado efetivo.

---

## 2. Mapa final da numeração

| Nº | Arquivo | Origem / Situação |
|:---:|---|---|
| 001 | `adr/ADR-001-nova-filosofia-arquitetura.md` | Nossa (linhagem vigente) |
| 002 | `adr/ADR-002-postgres-firestore-cqs.md` | Nossa (Oracle ADB + Firestore; título do arquivo é legado) |
| 003 | `adr/ADR-003-diagramas-base-de-dados.md` | **Nossa — mantida** |
| 004 | `adr/ADR-004-diagramas-projeto.md` | **Nossa — mantida** |
| 005 | `adr/ADR-005-staging-efemero-e2e.md` | Alinhada |
| 006 | `adr/ADR-006-team-roster-season-refactor.md` | Alinhada |
| 010 | `adr/ADR-010-autenticacao-firebase-custom-claims.md` | Nossa; **absorveu o ADR-008** do `main` (§6) |
| 011 | `adr/ADR-011-flutter-mvvm-architecture.md` | Nossa |
| 012 | `adr/ADR-012-migrations-jooq-java.md` | Nossa; **conteúdo corrigido** para Liquibase + Oracle ADB |
| 013 | `adr/ADR-013-gitflow-documentacao.md` | Nossa |
| 014 | `adr/ADR-014-atualizacao-diretivas.md` | Nossa |
| 015 | `adr/ADR-015-importacao-rascunho-organizacoes.md` | Nossa |
| 016 | `adr/ADR-016-gestao-segredos-github-environments.md` | Nossa |
| 017 | `adr/ADR-017-configuracao-playoffs.md` | Nossa |
| 018 | `adr/ADR-018-espelhamento-firestore-admin-sdk.md` | Nossa |
| 019 | `adr/ADR-019-modular-monolith.md` | **Renumerado** do antigo `ADR-003-modular-monolith` (`main`) |
| 020 | `adr/ADR-020-firebase-auth-migration.md` | **Renumerado** do antigo `ADR-004-firebase-auth-migration` (`main`) |

**Não há números duplicados.** Os antigos `007`, `008` e `009` do `main` deixaram de existir como
arquivos standalone:

| Antigo (`main`) | Destino |
|---|---|
| `ADR-003-modular-monolith.md` | Renumerado → **ADR-019** |
| `ADR-004-firebase-auth-migration.md` | Renumerado → **ADR-020** |
| `ADR-007-migracoes-flyway-em-java-com-jooq.md` | **Absorvido/descartado como duplicata do ADR-012** (tema idêntico; ver §4) |
| `ADR-008-ajustes-autenticacao.md` | **Absorvido no ADR-010 §6** (removido) |
| `ADR-009-organizacao-e-agremiacao.md` | **Absorvido em `architecture/fluxo_filiacao_e_inscricao_agremiacoes.md` §5** (removido) |

---

## 3. Renumeração 003/004 do `main` e ajuste de links

- `adr/ADR-003-modular-monolith.md` → `adr/ADR-019-modular-monolith.md` (título interno e nota de rastreabilidade atualizados).
- `adr/ADR-004-firebase-auth-migration.md` → `adr/ADR-020-firebase-auth-migration.md` (título interno e nota de rastreabilidade atualizados).
- Referências atualizadas (as que apontam para os **nossos** diagramas permanecem 003/004):
  - `adr/ADR-010-...md` — links e menções (ADR-003 → 019; link quebrado `ADR-004-api-first` → ADR-004 diagramas; ajuste das menções históricas de colisão → ADR-020).
  - `adr/ADR-014-...md` — "ADR-003 (Modular Monolith)" → **ADR-019**.
  - `adr/ADR-001-...md` — tabela "Integração com Outros ADRs" alinhada (003/004 = diagramas; incluído ADR-019).
  - `_planning/estrategia-cqrs-cloud-functions.md` — referências (`../adr/...`) com 003→019 e 004 API First → ADR-004 diagramas.
  - `architecture/ajustes-autenticacao.md` — ADR-004 → **ADR-020** e caminho do plano corrigido.
  - `.ai/project-context.md` e `plans/plano_reestruturacao_fase_0.md` — ADR-007 → **ADR-012**; ADR-003 (hierarquia) → **ADR-006**.

---

## 4. Correção do ADR-012 (runtime real)

O arquivo `adr/ADR-012-migrations-jooq-java.md` foi **reescrito** para refletir o runtime real:
**Liquibase (changelogs YAML) + Oracle ADB**, com DDL em código proibido (AGENTS.md do `flag_backend`).
O **nome do arquivo foi mantido** para não quebrar links.

> **Proposta registrada (não executada):** renomear o arquivo para algo como
> `ADR-012-migrations-liquibase.md` e o H1 para refletir Liquibase/Oracle ADB. Como renomear quebra links
> relativos existentes (`README.md`, `index.md`, `architecture/overview.md`, `architecture/components.md`,
> `apps/backend/README.md`), a renomeação fica **pendente de uma fatia própria** aprovada pelo dono.

Absorção: o antigo **ADR-007** (`main`, Flyway/jOOQ) tratava do mesmo tema e foi descartado como duplicata.

---

## 5. Rastreabilidade das absorções

- **ADR-008 → ADR-010 §6** ("Ajustes de Implementação — Firebase Auth exclusivo + PENDING read-only"),
  com nota de rastreabilidade e código de implementação (`flag_backend@4482c5e`, `flag_admin_web@96b2c0d`).
- **ADR-009 → `architecture/fluxo_filiacao_e_inscricao_agremiacoes.md` §5** ("Separação de Domínios e Perfis"),
  com nota de rastreabilidade; a afiliação por temporada (`institution_affiliations`) já era modelada no §2 do doc.
- **ADR-007 → ADR-012** (mesmo tema; descartado como duplicata).

---

## 6. Referências

- `adr/ADR-001-nova-filosofia-arquitetura.md`, `adr/ADR-002-postgres-firestore-cqs.md`,
  `adr/ADR-003-diagramas-base-de-dados.md`, `adr/ADR-004-diagramas-projeto.md`
- `adr/ADR-010-autenticacao-firebase-custom-claims.md` (§6), `adr/ADR-012-migrations-jooq-java.md`,
  `adr/ADR-019-modular-monolith.md`, `adr/ADR-020-firebase-auth-migration.md`
- `architecture/fluxo_filiacao_e_inscricao_agremiacoes.md` (§5)
- `_planning/firestore-sync/map.md` (FS-05 — correção de runtime nos docs)
