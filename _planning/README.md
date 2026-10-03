# Tracker Oficial de Trabalho — `_planning/`

> **Status:** vigente
> **Data:** 3 de Outubro de 2026
> **Modelo:** **hub-first** — todas as *issues de trabalho* da Flag Platform vivem aqui.
> **Base:** [Centralização das Issues e Reorganização](../architecture/centralizacao-issues-e-reorganizacao.md) ·
> [Inventário do Hub](../architecture/inventario-hub.md)

Esta pasta é a **fonte única da verdade** para trabalho (features, bugs, tasks, ops, research,
design). Os repositórios (`<repo>/docs/`) guardam apenas **documentação de implementação**
(specs, contratos, runbooks, KB) e, no máximo, um **link** para o ticket — **nunca uma cópia**.

---

## 1. Como o tracker funciona

- Cada **esforço** é um diretório com `map.md` (ponto de entrada), `spec.md` quando existir e
  `issues/NN-<slug>.md` com os tickets.
- Issues **específicas de repositório** ficam em `repos/<repo>/issues/` (mesmo formato).
- Esforços **concluídos** são movidos para `_archive/<esforco>/` — nunca apagados.
- Reconciliação entre `Status` e PRs é **manual** (sem automação nesta fase).

### Estados de um ticket

| Status | Significado |
|---|---|
| `open` | Aberto, ainda não reclamado. |
| `claimed` | Em execução por um agente/autor. |
| `resolved` | Entregue/decidido (ver `Origin`/`Blocks` para a evidência). |
| `archived` | Arquivado junto ao esforço em `_archive/`. |

Um ticket é **frontier** (pronto para ser trabalhado) quando está `open`, sem `Blocked by` aberto
e sem `Status: claimed` — mesma semântica usada no esforço do motor de classificação.

---

## 2. Contrato do ticket

Todo arquivo `issues/NN-<slug>.md` começa com o cabeçalho de máquina abaixo (campos lidos por
agentes e pelo wayfinder):

```markdown
# PREFIXO-NN: Título curto e imperativo

> **Type:** feature | bug | task | ops | docs | research | design | atividade
> **Status:** open | claimed | resolved | archived
> **Effort:** <esforco>                      # diretório-pai (ex.: firestore-sync)
> **Repo(s):** flag_backend, flag_admin_web # repos afetados (ou — se transversal)
> **Blocks:** <IDs> | —
> **Blocked by:** <IDs> | —
> **Origin:** <opcional: PR/commit/issue/arquivo de origem>

## Contexto / Critérios de aceite / Notas
```

O `map.md` de cada esforço mantém a **tabela de issues** (ID, Tipo, Título, Critérios de aceite,
Status) e a ordem de execução.

---

## 3. Convenção de IDs

- Formato **`PREFIXO-NN`** (NN com 2 dígitos), **estável e nunca reutilizado**.
- IDs são **globais e únicos** dentro do hub.
- O prefixo deriva do esforço/repositório (tabela abaixo).
- Ao criar um ticket, use o **próximo número livre** do prefixo e **atualize o registro** (§5).

---

## 4. Estrutura de pastas

```
_planning/
├── README.md                     # este arquivo (convenções + registro)
├── <esforco>/                    # transversal (ex.: firestore-sync, r2-imagens)
│   ├── map.md                    # roadmap/wayfinder do esforço
│   ├── spec.md                   # solução consolidada (quando houver)
│   └── issues/NN-<slug>.md
├── repos/                        # issues específicas de repositório
│   └── <repo>/
│       ├── map.md
│       └── issues/PREFIXO-NN-<slug>.md
└── _archive/
    └── <esforco>/                # esforços concluídos (histórico preservado)
```

---

## 5. Registro de esforços e IDs

### 5.1 Esforços ativos

| Esforço | Prefixo | Local | Status | Último ID | Próximo livre |
|---|---|---|---|:---:|:---:|
| firestore-sync | `FS` | `firestore-sync/` | aberto | FS-09 | `FS-10` |
| r2-imagens | `R2` | `r2-imagens/` | aberto | R2-05 | `R2-06` |
| playoffs-configuracao | `PLAYOFFS` | `playoffs-configuracao/` | rascunho | — | `PLAYOFFS-01` |
| padronizacao-listagens | `LIST` | `padronizacao-listagens/` | em execução | LIST-01 | `LIST-02` |
| governanca-operacional-agentes | `GOV` | `governanca-operacional-agentes/` | resolvido | GOV-01 | `GOV-02` |

### 5.2 Issues por repositório (`repos/`)

| Repositório | Prefixo | Local | Último ID | Próximo livre |
|---|---|---|:---:|:---:|
| flag_backend | `BACK` | `repos/backend/` | BACK-06 | `BACK-07` |
| flag_platform_infra | `INFRA` | `repos/infra/` | INFRA-01 | `INFRA-02` |
| flag_admin_web | `ADMIN` | `repos/admin-web/` (a criar) | — | `ADMIN-01` |
| flag_referee_app | `REF` | `repos/referee-app/` (a criar) | — | `REF-01` |
| flag_public_app | `PUB` | `repos/public-app/` (a criar) | — | `PUB-01` |
| flag_platform_agent | `AGENT` | `repos/agent/` (a criar) | — | `AGENT-01` |

### 5.3 Esforços arquivados (`_archive/`)

| Esforço | Prefixo | Concluído em | Nota |
|---|---|:---:|---|
| motor-de-calculo-classificacao | `MOTOR` | 2026-10-03 | Wayfinding encerrado; destino = `spec.md` |
| segredos-e-env | `SEC` | 2026-10-03 | Sanitização concluída (rotação/purga adiadas por decisão do dono) |

### 5.4 Planos ainda soltos na raiz de `_planning/` (pendente — P4)

| Arquivo | Situação | Destino proposto |
|---|---|---|
| `plano-revisao-modulo-auth.md` | Proposto / Em planejamento | esforço `auth-refactor/` |
| `plano-migracao-firebase-auth.md` | Relocado de `adr/` | esforço `auth-refactor/` |
| `estrategia-cqrs-cloud-functions.md` | **Superado** pelo ADR-018 | `_archive/estrategia-cqrs-cloud-functions.md` |

---

## 6. Regras de higiene

1. **Anti-duplicata:** um item é canônico **aqui**; no repo, só um link.
2. **Rastreabilidade:** commits/PRs citam o ID (`Refs: FS-07`).
3. **Links relativos** corretos; ao mover um esforço para `_archive/`, ajustar quem o referencia.
4. **GitHub Issues não é o tracker** — exceções precisam ser migradas para cá e fechadas.
5. Sem valores de segredo em tickets (ver [env-vars-e-secrets](../architecture/env-vars-e-secrets.md)).

---

## 7. Referências

- [Centralização das Issues e Reorganização](../architecture/centralizacao-issues-e-reorganizacao.md)
- [Inventário do Hub](../architecture/inventario-hub.md)
- [Governança de Documentação](../architecture/governanca-documentacao.md)
- [Governança de Agentes e Skills](../architecture/governanca-agentes-skills.md)
