# FS-01: Implementar writer de `games` no backend (Admin SDK)

Type: feature
Status: open

## Objetivo

Espelhar a coleção **`games`** no Firestore a partir do backend, de forma best-effort pós-commit,
conforme o [ADR-018](../../../adr/ADR-018-espelhamento-firestore-admin-sdk.md). É a primeira fatia
da sincronização: hoje o `flag_public_app` já escuta `games/{id}` e a coleção está vazia porque
ninguém escreve nela.

## Contexto

- `flag_backend` **não possui** a dependência `firebase-admin` no `pom.xml` nem o módulo `realtime`.
- O módulo `game` é o único produtor de escritas; precisa publicar `GameChangedEvent(gameId)` nos
  caminhos `create`, `createBatch`, `update`, `updateStatus`, `registerResult`, `registerScoreEvent`,
  `handlePlayScoreEvent`, `correctScore`.
- O reader do app (`live_game_repository.dart`) espera os campos `roundId`, `competitionId`,
  `homeTeamId`, `awayTeamId`, `roundNumber`, `homeTeamName`, `awayTeamName`, `venueId`, `venueName`,
  `venueAddress`, `venueMapsUrl`, `scheduledAt`, `status`, `homeScore`, `awayScore` — e datas como
  **ISO-8601 sem zona** (comentário no próprio reader referenciando o ADR de espelhamento).

## Critérios de aceitação

- [ ] Dependência `firebase-admin` adicionada ao `pom.xml` (versão fixada; sem quebrar o build).
- [ ] Módulo `realtime` criado com `@TransactionalEventListener(AFTER_COMMIT)` consumindo
      `GameChangedEvent`.
- [ ] `GameChangedEvent` publicado em **todos** os caminhos de escrita do módulo `game`.
- [ ] Projeção pública (`GameMirrorSnapshot`) exposta via `GameLookup`, sem vazar tipos de subpacote.
- [ ] Escrita no Firestore em `games/{id}` **best-effort** (try/catch + `warn`), sem quebrar a request.
- [ ] **No-op** quando não há `FirebaseApp`/credencial (dev/CI) — build e subida não quebram.
- [ ] Datas de *wall clock* gravadas como ISO-8601 **sem zona**; `syncedAt` como `serverTimestamp()`.
- [ ] O documento resultante é lido corretamente pelo `FirestoreLiveGameRepository` do app.
- [ ] `mvn -B clean verify` verde; sem SQL nativo; versionamento do `pom.xml` incrementado (MINOR).

## Arquivos afetados (referência)

- `flag_backend/pom.xml` — dependência `firebase-admin`.
- `flag_backend/src/main/java/.../realtime/**` — novo módulo (listener + mapper + cliente Firestore).
- `flag_backend/src/main/java/.../game/**` — publicar `GameChangedEvent`; expor `GameMirrorSnapshot`
  via `GameLookup`.
- `flag_backend/src/main/resources/application*.yml` — flag de backfill/credencial (opcional neste corte).
- `flag_public_app/lib/data/repositories/live_game_repository.dart` — apenas leitura de contrato
  (não alterar sem necessidade; valida o formato dos campos).
