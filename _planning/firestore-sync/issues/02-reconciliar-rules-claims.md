# FS-02: Reconciliar `firestore.rules`/`firestore.indexes.json` e claims

> **Type:** task
> **Status:** open
> **Effort:** firestore-sync
> **Repo(s):** flag_backend, flag_admin_web
> **Blocks:** —
> **Blocked by:** —
> **Origin:** ADR-018 / PR #73

## Objetivo

Consolidar **um único conjunto canônico** de `firestore.rules` e `firestore.indexes.json` com
**leitura pública** (ou autenticada, a decidir) e **escrita exclusiva do backend** (Admin SDK).

> Esta issue **já existe no mapa** como FS-02. Este arquivo **detalha / complementa** o item —
> **não** é uma nova pendência. O achado abaixo (rules divergentes) foi revelado pela implementação
> do ADR-018 e deve ser incorporado à mesma FS-02.

## Contexto

- Complemento vindo da implementação do
  [ADR-018](../../../adr/ADR-018-espelhamento-firestore-admin-sdk.md) no PR #73 do `flag_backend`
  (commits `3489548`, `99c411c`).
- **Rules divergentes** entre repositórios (fonte: ADR-018, "Contexto de infraestrutura real"):
  - `flag_admin_web/firestore.rules`: leitura pública + `allow write: if false` — **coerente** com o
    desenho.
  - `flag_backend/firestore.rules`: chega a permitir **escrita pelo cliente** — `games` com `update`
    por `isOrganizer() || isMesa()` e `scoreEvents` com `create` por `isMesa()`. Isso **contradiz** o
    desenho (escrita exclusiva pelo backend) e ainda usa a claim `organizationId` e a role `MESA`
    que divergem do restante da plataforma.
  - **Claims divergentes:** a plataforma emite `organization_id` (snake_case) em alguns pontos e o
    rules do backend referencia `organizationId` (camelCase) — precisam ser alinhadas ao que o
    backend **realmente** emite.
- **`indexes.json` divergentes:** o do admin web está **vazio**; o do backend declara **5 índices
  compostos de `games`** e não declara os demais que os apps podem consultar. Regra prática do
  ADR-018: índices refletem **apenas queries reais**; o app escuta **documento único**, não varre a
  coleção.

## Critérios de aceitação

- [ ] Um único conjunto canônico de `firestore.rules` (decidido leitura pública **ou** autenticada).
- [ ] **Escrita só pelo backend**: remover `write` por `isOrganizer()`/`isMesa()` do rules do backend.
- [ ] Claims alinhadas ao que o backend realmente emite (`organization_id` vs `organizationId`;
      role `MESA` vs o vocabulário vigente).
- [ ] `firestore.indexes.json` reflete **apenas queries reais** (remover índices não usados;
      adicionar os que faltam), com os dois repositórios usando o mesmo conjunto.
- [ ] Nenhum caminho de escrita direta pelo cliente sobrevive nas rules.

## Arquivos afetados (referência)

- `flag_backend/firestore.rules` e `flag_backend/firestore.indexes.json`.
- `flag_admin_web/firestore.rules` e `flag_admin_web/firestore.indexes.json`.
- Emissão de custom claims no backend (`organization_id`/role) — apenas referência para alinhamento.
