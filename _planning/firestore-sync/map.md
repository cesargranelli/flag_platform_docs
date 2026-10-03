# Mapa — Espelhamento Oracle ADB → Firestore (Admin SDK)

Effort de implementação do espelho de leitura do Firestore. Decisão consolidada no
[ADR-018](../../adr/ADR-018-espelhamento-firestore-admin-sdk.md).

> **Regra:** as issues deste mapa são trabalhadas **nesta ordem**. As detalhadas têm arquivo em
> `issues/`; as demais ficam só na tabela até virarem fatia.

## Contexto

- **Banco primário:** Oracle Autonomous Database 23ai (Always Free), schema `platform`, mTLS via
  wallet. Migrations **Liquibase**. Sem PostgreSQL/Flyway/jOOQ no runtime.
- **Runtime:** VM OCI Always Free + Docker + Caddy; container backend `mem_limit 560m`. Cloud Run
  desativado (`d961e04`).
- **Firebase:** projeto `flag-platform` (Auth/FCM). Service account do Admin SDK **gitignored** em
  `flag_backend/src/main/resources/firebase-service-account.json`.
- **Estado atual (o problema):**
  - O `flag_backend` **não tem** `firebase-admin` no `pom.xml`, **não tem** módulo `realtime` e
    `flag_backend/functions/` está vazio.
  - O `flag_public_app` **já lê** `games/{id}` em tempo real (`FirestoreLiveGameRepository`, commit
    `dae1814`) — ou seja, **hoje o app lê uma coleção que ninguém escreve**.
  - `firestore.rules`/`indexes.json` existem em `flag_admin_web` e `flag_backend`, mas **divergem**:
    o do admin web tem leitura pública + `write: false`; o do backend permite escrita por
    organizer/mesa — **contradiz** o desenho.

## Decisão (resumo do ADR-018)

Espelhar **pelo próprio backend** com o **Firebase Admin SDK** (não Cloud Functions):
`GameChangedEvent` publicado pelo módulo `game` → módulo `realtime` escuta com
`@TransactionalEventListener(AFTER_COMMIT)` → escrita **best-effort** no Firestore, **no-op sem
credencial**. Datas de *wall clock* em ISO-8601 **sem zona**; `syncedAt` como `serverTimestamp()`.
Backfill opt-in (`app.firestore.backfill-on-startup=true`).

Primeiro corte: **`games`**. Depois `scoreEvents`. Catálogo (competitions/teams/venues/standings)
por último.

## Ordem de execução

1. **FS-01 — Writer de `games`** (Admin SDK + dep no `pom.xml` + módulo `realtime`). Detalhada em
   [`issues/01-writer-games.md`](issues/01-writer-games.md).
2. **FS-02 — Reconciliar `firestore.rules`/`indexes.json` e claims** (um conjunto canônico).
3. **FS-03 — Backfill + testes** de ponta a ponta.
4. **FS-04 — Monitoramento/drift** do espelho.
5. **FS-05 — Consolidar premissas de infra nos docs** (Oracle/Liquibase/OCI vs PostgreSQL/Flyway/Cloud Run).

## Issues

| ID | Tipo | Título | Critérios de aceitação | Status |
|----|------|--------|------------------------|--------|
| **FS-01** | feature | Implementar writer de `games` no backend (Admin SDK) | Ver [issue detalhada](issues/01-writer-games.md) | open |
| **FS-02** | task | Reconciliar `firestore.rules`/`firestore.indexes.json` e nomes de claims | Um único conjunto canônico; leitura pública (ou autenticada, decidida) e **escrita só pelo backend**; remover `write` por `isOrganizer()`/`isMesa()` do backend; alinhar claims (`role`, `organizationId`) ao que o backend realmente emite; índices refletem apenas queries reais (o admin web hoje está vazio; o backend tem índices de `games`) | open |
| **FS-03** | task | Backfill inicial `games` + testes E2E (write→mirror→read) | Backfill idempotente e opt-in popula `games/{id}`; teste E2E cobre registrar resultado e o app/leitor enxergar o doc; no-op sem credencial não quebra o build | open |
| **FS-04** | ops | Monitoramento e detecção de drift do espelho | Contadores de sync OK/falha por coleção; alerta de coleção vazia/desatualizada; runbook de backfill manual; nenhum dado sensível no espelho | open |
| **FS-05** | docs | Consolidar premissas de infra nos docs | `flag_platform_docs` (e `flag_backend/docs`) sem afirmar PostgreSQL/jOOQ/Flyway/Cloud Run como runtime; Oracle ADB + Liquibase + OCI/Docker/Caddy como verdade; ajustar ADR-002, ADR-012 e análise de custo onde couber | open |

## Fora de escopo

- Espelhamento do catálogo completo (competitions/teams/venues/standings) — fase posterior.
- Camada `onWrite` (Cloud Functions) — só se surgir escrita direta pelo cliente.
- Escrita direta no Firestore pelo cliente — não existe e não existirá neste desenho.
