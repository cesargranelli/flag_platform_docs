# FS-10: App público não exibe a competição 8x8 — verificar espelhamento Firestore

> **Type:** bug
> **Status:** resolved
> **Effort:** firestore-sync
> **Repo(s):** flag_public_app, flag_backend, flag_platform_infra
> **Blocks:** —
> **Blocked by:** —
> **Origin:** relato do usuário — 2026-10-08

## Diagnóstico (2 achados)

### 1. Causa do sintoma — a competição está em `DRAFT`
`GET /api/v1/competitions/{id}` → `status: DRAFT`. O app público lista via **API**
(`GET /api/v1/competitions`) e filtra **`status == published`** (`ongoingCompetitionsProvider`) →
a 8x8 não aparece. **Não é o espelhamento.** Ação: **publicar** a competição (ou decidir se o app
deve exibir DRAFT).

### 2. Bug real de espelhamento (corrigido)
O espelho Firestore estava **desabilitado (no-op)**:
`WARN FirestoreGateway: app.firestore habilitado, mas nenhuma credencial Firebase foi encontrada`.
Causa: `/home/opc/flag_platform/firebase/service-account.json` era `-rw------- opc:opc`, ilegível
pelo container (uid **999**/`flagplatform`).
- **Correção**: `chown 999:999` + `chmod 600` na VM; persistido no deploy
  (`platform-deploy-hub.yml`) e exposto `FIRESTORE_ENABLED`/`FIRESTORE_BACKFILL_ON_STARTUP` no compose.
- **Backfill**: `Backfill Firestore games concluído: 217 ok, 0 falhas`.

> Observação: o app usa Firestore **apenas para os jogos ao vivo**; a lista de competições vem da API.

## Critérios de aceite

- [x] Causa identificada (status DRAFT + espelho em no-op).
- [x] Espelho habilitado e backfill dos jogos executado (217 docs).
- [ ] App público exibe a 8x8 após **publicá-la** (ação do usuário).
