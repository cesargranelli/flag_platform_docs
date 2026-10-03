# FS-08: Backfill sem paginação (`findAll()` + N+1)

Type: Issue
Status: open

## Objetivo

Registrar que o `GameMirrorBackfill` carrega todos os jogos com `findAll()` e faz *lookups* por item
(padrão N+1). Funciona no volume atual, mas precisa ser paginado antes que a base cresça.

## Contexto

- Revelado pela implementação do [ADR-018](../../../adr/ADR-018-espelhamento-firestore-admin-sdk.md)
  no PR #73 do `flag_backend` (commits `3489548`, `99c411c`).
- O backfill é opt-in (`app.firestore.backfill-on-startup=true`) e não roda a cada request, mas um
  `findAll()` sem paginação + *lookups* N+1 escala mal: pico de memória/CPU no container
  (`mem_limit 560m`) e rajada de leituras no ADB durante a subida.
- Precedente de volume baixo (`~10-20 TPS`) torna isso aceitável **hoje**; é dívida técnica a pagar
  quando a coleção `games` crescer.

## Critérios de aceitação

- [ ] Backfill processa em lotes paginados (ex.: *keyset* por `id`/`updatedAt`) em vez de `findAll()`.
- [ ] Eliminar o N+1 (carregar dependências em lote / *join* de projeção) ou justificar por medição.
- [ ] Backfill continua **idempotente** e opt-in; sem credencial Firebase segue **no-op**.
- [ ] `mvn -B clean verify` verde.

## Arquivos afetados (referência)

- `flag_backend/src/main/java/.../realtime/**` — `GameMirrorBackfill` (paginação + carga em lote).
- `flag_backend/src/main/java/.../game/**` — projeções/lookups usados pelo backfill.
- `flag_backend/src/main/resources/application*.yml` — flag de backfill.
