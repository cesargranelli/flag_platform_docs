# Centralização das Issues de Trabalho e Reorganização do Hub

> **Status:** Proposta (aguardando decisão do dono)
> **Data:** 3 de Outubro de 2026
> **Autor:** Documentação / Governança (agente)
> **Branch:** `docs/reorg-hub` (a partir de `docs/public-app-revisao-ux`, PR #38 — **sem push**)
> **Documento irmão (inventário):** [Inventário do Hub](inventario-hub.md)
> **Decisão-alvo:** todas as **issues de trabalho** centralizadas e sincronizadas em `flag_platform_docs/_planning/`.

---

## 1. Objetivo

Consolidar um **único tracker oficial (local, markdown)** para todo trabalho da Flag Platform — hoje disperso entre `_planning/` do hub, `backend-issues.md` na raiz, `docs/` dos repositórios e GitHub Issues — e definir a **reorganização** mínima do hub para sustentá-lo. Este documento propõe o modelo, os conflitos de governança a resolver e um plano de migração em passos. **Nada é executado além da criação deste documento e do inventário.**

---

## 2. Onde as issues de trabalho vivem hoje

### 2.1 No hub `flag_platform_docs`

| Local | Formato | Conteúdo | Situação |
|---|---|---|---|
| `_planning/<esforco>/` | `map.md` + `issues/NN-*.md` + `spec.md` | 5 esforços (firestore-sync, motor, playoffs, r2, padronização) + segredos | **Modelo embrionário já correto** — falta padronizar campos e completar esforços |
| `_planning/*.md` (soltos) | arquivo único | `estrategia-cqrs-cloud-functions.md`, `plano-migracao-firebase-auth.md` (relocados), `plano-revisao-modulo-auth.md` | Fora do padrão de diretório por esforço |
| `backend-issues.md` (raiz) | arquivo único | `ISSUE-001..006` (pendências do `flag_backend`) | **Issue específica fora do tracker** |
| GitHub Issues | remoto | **#5** (governança/docs) e **#37** (infra: build do hub) abertas | Contraria a regra de tracker local |

> Nota: `_planning/_archive/motor-de-calculo-classificacao/` já é a **referência de estrutura** (um diretório, `map.md` + `spec.md` + `issues/NN-*.md` com `Status:`), embora o campo seja apenas `Status` (sem `Type`).

### 2.2 Nos repositórios (`<repo>/docs/`)

| Repo | Arquivo(s) com trabalho/issues | Formato |
|---|---|---|
| `flag_backend/docs` | `database-portability-assessment.md` (item "Bloqueante"), `deploy-handoff.md`, `versioning.md` | Pendências embutidas em docs de implementação |
| `flag_admin_web/docs` | `backend-integration-pending.md` (11 itens com status), `frontend-backlog.md` (itens ✅), `agents/issue-tracker.md` | Backlogs markdown **+** declaração "Issues live in GitHub Issues" |
| `flag_referee_app/docs` | `plan/feature2-spec.md`, `plan/planejamento-feature2.md`, `plan/plano-execucao-eventos.md`, `plan/evolution-plan.md` | Planos de execução |
| `flag_public_app/docs` | `ux-redesign-tournament-first.md`, `screen-flow.md` | Design/fluxo (não backlog) |
| `flag_platform_infra/docs` | `runbook.md`, `kb/*` | Operação/KB |
| `flag_platform_agent/docs` | `01-avaliacao-modelos.md`, `02-gap-analysis-organizacoes.md` | Análises |
| `flag_platform/docs` | espelho legado do hub (`adr/`, `architecture/`, `design/`, `research/`) | **Duplicata histórica** (monorepo descontinuado) |

### 2.3 GitHub Issues (leitura, via `gh`)

| Repo | Issues abertas |
|---|---|
| `cesargranelli/flag-platform-docs` | **#37**, **#5** |
| `flag_backend`, `flag_admin_web`, `flag_public_app`, `flag_referee_app`, `flag_platform_infra` | **nenhuma** |

**Conclusão:** o tracker é praticamente **local**, mas fragmentado e sem contrato único. O único ponto que vai abertamente contra a regra é o `AGENTS.md`/`docs/agents/issue-tracker.md` do `flag_admin_web` (declara GitHub Issues) e as 2 issues abertas no hub.

---

## 3. Conflitos de governança a resolver

| # | Fonte | Diz | Conflita com |
|---|---|---|---|
| C1 | `AGENTS.md` raiz §7 | "Tracker **local (markdown)** — `flag_platform_docs/_planning/` (transversal) ou `<repo>/docs/` (específico); **não** abrir GitHub Issues por padrão" | `flag_admin_web/AGENTS.md` + `docs/agents/issue-tracker.md` (GitHub Issues) |
| C2 | `flag_admin_web/AGENTS.md` (`Issue tracker`) e `docs/agents/issue-tracker.md` | "Issues live in the repo's GitHub Issues (uses `gh` CLI)" | Regra raiz §7 e a prática real (0 issues abertas no repo; backlogs em markdown) |
| C3 | `architecture/governanca-agentes-skills.md` §1 | Tracker transversal → hub; **específico → `<repo>/docs/`** | Objetivo (4) do dono: **todas** as issues centralizadas no hub |
| C4 | GitHub Issue **#5** de título "registrar diretrizes operacionais de agentes e padrão GitFlow" | Vive no GitHub, não no hub | Governança do próprio hub |
| C5 | `flag_platform/docs` (legado) | Espelha `adr/`, `architecture/`, `design/`, `research/` do hub | "Nunca duplicar documentação entre hub e repos" (`governanca-documentacao.md` §4) |

**Decisão necessária:** o modelo **hub-first** (recomendado, §4) exige mudar a linha do tracker em `governanca-agentes-skills.md` §1 e no `AGENTS.md` raiz §7: o hub passa a ser dono de **todas** as issues; os `docs/` dos repos guardam apenas **documentação de implementação** (specs, contratos, runbooks) — não work items.

---

## 4. Modelo central proposto

### 4.1 Estrutura de `_planning/`

```
_planning/
├── README.md                     # índice de esforços + convenções + registro de IDs
├── <esforco>/                    # transversal (ex.: firestore-sync, r2-imagens)
│   ├── map.md                    # roadmap/wayfinder do esforço (frontier + decisões)
│   ├── spec.md                   # solução consolidada (quando o esforço for wayfinding)
│   └── issues/
│       ├── NN-<slug>.md          # ticket individual (convenção abaixo)
│       └── ...
├── repos/                        # issues de trabalho ESPECÍFICAS de repositório
│   ├── backend/                  # (absorve backend-issues.md)
│   │   ├── map.md
│   │   └── issues/NN-<slug>.md
│   ├── admin-web/
│   ├── referee-app/
│   └── ...
└── _archive/                     # esforços concluídos (histórico preservado)
    └── <esforco>/
```

### 4.2 Contrato do ticket (`issues/NN-<slug>.md`)

Cabeçalho padronizado (campos de máquina, lidos pelo wayfinder e por agentes):

```markdown
# <PREFIXO>-NN: Título curto e imperativo

> **Type:** feature | bug | task | ops | docs | research | design | atividade
> **Status:** open | claimed | resolved | archived
> **Effort:** <esforco>            # diretório-pai
> **Repo(s):** flag_backend, flag_admin_web   # afetados
> **Blocks:** <IDs> | —
> **Blocked by:** <IDs> | —
> **Origin:** <opcional: PR/commit/sessão>

## Contexto / Critérios de aceite / Notas
```

- O `map.md` mantém a **tabela de issues** (ID, Tipo, Título, Critérios, Status) — como já fazem `firestore-sync` e `r2-imagens`.
- Um ticket é **frontier** quando `Status: open`, sem `Blocked by` aberto (mesma semântica já usada no motor de classificação).

### 4.3 Convenção de IDs

- Prefixo derivado do esforço: `FS` (firestore-sync), `R2` (r2-imagens), `MOTOR`, `PLAYOFFS`, `BACK` (backend), `ADMIN` (admin-web), `REF` (referee-app) etc.
- Formato `PREFIXO-NN` (2 dígitos), **estável e nunca reutilizado**; IDs são globais e únicos no hub.
- O `_planning/README.md` mantém o **registro/prefixos** e o próximo número livre por esforço.

### 4.4 Sincronização com o que vive nos repositórios

- **Regra de casas nova (hub-first):** *work item* → hub `_planning/`; *documento de implementação* (spec, contrato, runbook, KB) → `<repo>/docs/`.
- Quando uma issue específica de repo existir, o arquivo canônico fica em `_planning/repos/<repo>/issues/` e o repo pode manter **link** (nunca cópia) no seu `docs/`.
- **Rastreabilidade de execução:** commits/PRs nos repos citam o ID (`Refs: FS-07`) — permite reconciliar o status real com o tracker.
- **Reconciliação (manual/assistida):** revisão periódica dos PRs mesclados para marcar `Status: resolved`; opcionalmente um agente gera um relatório de divergência (doc × PR). Sem CI bloqueante nesta fase.
- **Proibição de duplicata:** nada de manter o mesmo item em `_planning/` **e** em `<repo>/docs/`; um é canônico (hub), o outro vira ponteiro.

### 4.5 O que fazer com `backend-issues.md`

- **Mover** o conteúdo para `_planning/repos/backend/issues/` fatiado por ISSUE-00N → `BACK-01..NN`, com `map.md` de índice; o arquivo da raiz é removido e vira um ponteiro curto **ou** é simplesmente eliminado (o histórico fica no Git).
- Manter as evidências/endpoints/logs como estão (o conteúdo é bom); apenas normalizar o cabeçalho para o contrato de ticket (`Type`/`Status`/`Repo`).
- As **2 GitHub Issues abertas** do hub (#5, #37) devem ser **migradas para `_planning/`** e fechadas no GitHub (ou explicitamente marcadas como exceção).

---

## 5. Atualizações de regras necessárias (descritas — **não editar aqui**)

> Estes arquivos pertencem a outros repos / fora do escopo desta branch. Apenas a **mudança textual** é proposta.

| Arquivo | Mudança proposta |
|---|---|
| `C:\Projetos\America\AGENTS.md` §7 (raiz) | Trocar "tracker local — hub (transversal) **ou** `<repo>/docs/` (específico)" por: "**tracker oficial é o hub** `flag_platform_docs/_planning/` (inclui `repos/<repo>/` para itens específicos); `<repo>/docs/` guarda documentação de implementação, não issues; **não** abrir GitHub Issues por padrão". |
| `flag_admin_web/AGENTS.md` (`### Issue tracker`) | Substituir a frase do GitHub Issues por: "Trabalho/backlog vive em `flag_platform_docs/_planning/` (ver `_planning/repos/admin-web/`); este repo mantém apenas docs de implementação." |
| `flag_admin_web/docs/agents/issue-tracker.md` | Reescrever o conteúdo ("GitHub Issues") para o modelo hub-first. |
| `architecture/governanca-agentes-skills.md` §1 (hub) | Atualizar a linha **Tracker**: hub canônico para **todas** as issues; remover a alternativa `<repo>/docs/` para work items. |
| `architecture/governanca-documentacao.md` | Explicitar que `_planning/` é a fonte única de trabalho e referenciar este documento. |

---

## 6. Plano de migração em passos

| # | Passo | Onde | Depende de |
|---|---|---|---|
| **P0** | Publicar este documento + o [inventário](inventario-hub.md) (feito nesta branch) | hub | — |
| **P1** | Criar `_planning/README.md` com convenções, campos `Type`/`Status` e registro de prefixos/IDs | hub | P0 |
| **P2** | Migrar `backend-issues.md` → `_planning/repos/backend/{map.md, issues/BACK-0N}`; remover o arquivo da raiz | hub | P1 |
| **P3** | Normalizar campos nos esforços existentes (adicionar `Type`, `Repo`, `Blocks`/`Blocked by`); completar `FS-03..FS-05` ou rebaixá-los a linha de tabela | hub | P1 |
| **P4** | Mover os planos soltos para diretórios de esforço (`estrategia-cqrs` → `_archive/`, `plano-*-auth` → `auth-refactor/`) | hub | P1 |
| **P5** | Mover esforços concluídos (motor, segredos, padronização) para `_planning/_archive/` preservando histórico | hub | P1 |
| **P6** | Migrar GitHub Issues #5/#37 para `_planning/` e fechar no GitHub; registrar o modelo no `_planning/README.md` | hub + GitHub | P1 |
| **P7** | Atualizar os `AGENTS.md` e docs de governança (§5) | raiz + `flag_admin_web` + hub | decisão do dono |
| **P8** | Reconciliar numeração de ADR com `main` (fechar lacuna 007–009 / estabilizar 003–004) | hub | decisão do dono |
| **P9** | Enxugar navegação: `README.md` vira ponteiro para `index.md`; corrigir link `market-analysis-restructured`; atualizar §6 com os esforços de `_planning/` | hub | P2–P5 |
| **P10** | Definir rotina de reconciliação tracker × PRs (manual/assistida) | processo | P7 |

Cada passo vira uma fatia independente e pode ser um commit/PR próprio.

---

## 7. Decisões pendentes do dono

1. **Escopo da centralização:** confirmar o modelo **hub-first** (todas as issues no hub, inclusive específicas de repo em `_planning/repos/<repo>/`) — ou manter híbrido (hub transversal + `<repo>/docs/` específico) com link obrigatório?
2. **GitHub Issues:** encerrar definitivamente (fechar #5/#37 e proibir novas) ou manter como canal de entrada com espelhamento para o hub?
3. **`backend-issues.md`:** fatiar em `_planning/repos/backend/` (recomendado) ou apenas mover o arquivo inteiro para `_planning/backend/`?
4. **Numeração de ADR:** qual linhagem prevalece na reunificação com `main` (esta branch renumerou 003/004 e saltou 007–009)?
5. **Esforços concluídos:** arquivar em `_planning/_archive/` (recomendado) ou deletar?
6. **Reconciliação automática:** investir em um agente/script que compara `Status` do tracker com PRs mesclados, ou manter revisão manual?

---

## 8. Referências

- [Inventário do Hub](inventario-hub.md)
- [Governança de Documentação](governanca-documentacao.md) — regra anti-docs-fantasma e Hub × repos
- [Governança de Agentes e Skills](governanca-agentes-skills.md) — casas de artefatos e tracker
- `_planning/_archive/motor-de-calculo-classificacao/map.md` — referência de estrutura de esforço
- PR **#38** (`flag-platform-docs`) — consolidação em aberto para `main`
