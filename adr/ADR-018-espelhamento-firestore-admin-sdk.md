# ADR-018: Espelhamento Oracle ADB → Firestore pelo backend (Firebase Admin SDK)

## Status
Aceito — a implementar

> O código descrito neste ADR **não existe no `flag_backend` atual**: não há dependência
> `firebase-admin`/Firestore no `pom.xml`, não há módulo `realtime` e `flag_backend/functions/`
> está vazio. O rascunho original afirmava "implementado para `games` + `scoreEvents`" e adotava
> PostgreSQL — ambas as premissas não correspondem ao estado do repositório.

## Data
2026-10-03

## Autor
Tech Lead (Flag Platform)

## Contexto

O [ADR-002](ADR-002-postgres-firestore-cqs.md) define a arquitetura **CQRS Light**: o banco
relacional é a *source of truth* (write path) e o Firestore é o **espelho de leitura** (read path)
para dashboards, leaderboards e **placar/lances em tempo real**.

Faltava definir **como** a sincronização acontece. O documento
[estrategia-cqrs-cloud-functions.md](../_planning/estrategia-cqrs-cloud-functions.md) propunha
**Cloud Functions** em três camadas (HTTP trigger + `onWrite` + Cloud Scheduler). Esse documento
permaneceu com status *Proposto* e nunca foi implementado — e fica **superado por este ADR**.

Restrições reais do projeto:
- Volume baixo (~10-20 TPS) — não justifica *event broker* dedicado.
- O **backend Spring Boot (Modular Monolith) é o único produtor de escritas** no banco relacional:
  o Referee App e o Admin Web escrevem pelo REST, não direto no banco.
- O espelho é consumido pelos apps (Public App, dashboards) via SDKs nativos.

### Contexto de infraestrutura real (verificado em 2026-10-03)

As premissas de infraestrutura do rascunho estavam desatualizadas. A verdade consolidada:

- **Banco primário:** **Oracle Autonomous Database 23ai (Always Free)**, schema/usuário `platform`,
  conexão **mTLS via wallet**, URL JDBC `jdbc:oracle:thin:@devdb_high`. Migrations de schema via
  **Liquibase** (YAML). **Não** há PostgreSQL, Cloud SQL, jOOQ nem Flyway no runtime atual.
- **Runtime:** VM **OCI Always Free** + **Docker** + **Caddy** (reverse proxy/TLS). O **Cloud Run
  foi desativado** em `d961e04` (24/09). O container do backend roda com `mem_limit 560m`.
- **Firebase:** projeto **`flag-platform`** (Auth/FCM ativos). O backend usa `FIREBASE_CREDENTIALS`
  (default `classpath:firebase-service-account.json`) e valida JWTs via JWKS. O service account do
  **Admin SDK** fica **gitignored** em `flag_backend/src/main/resources/firebase-service-account.json`
  (`.gitignore` linhas 26–27).
- **Estado do código — ponto crítico:** no `flag_backend` atual **não existe** a dependência
  `firebase-admin` no `pom.xml`, **não existe** o módulo `realtime` e `flag_backend/functions/` está
  vazio. Portanto a sincronização **nunca foi implementada**.
- **Public App:** já lê `games/{id}` em tempo real via `FirestoreLiveGameRepository`
  (`flag_public_app/lib/data/repositories/live_game_repository.dart`, commit `dae1814`). Ou seja,
  **hoje o app lê uma coleção que ninguém escreve**.
- **Regras/índices divergentes:** existem `firestore.rules`/`firestore.indexes.json` no
  `flag_admin_web` e no `flag_backend`, e eles **não são coerentes entre si**:
  - **Admin Web:** leitura pública + `allow write: if false` — coerente com o desenho.
  - **Backend:** chega a permitir **escrita pelo cliente** — `games` com `update` por
    `isOrganizer() || isMesa()` e `scoreEvents` com `create` por `isMesa()`. Isso **contradiz** o
    desenho de escrita exclusiva pelo backend, e ainda usa a claim `organizationId` e a role `MESA`
    que divergem do restante da plataforma.
  - Os `indexes.json` também divergem: o do admin web está vazio; o do backend declara índices de
    `games` (5 compostos) e não declara os demais que os apps podem consultar.

## Decisão

O espelhamento é feito **pelo próprio backend**, com o **Firebase Admin SDK**, e **não** por Cloud
Functions.

- **Gatilho:** o módulo `game` publica `GameChangedEvent(gameId)` em todos os caminhos de escrita
  (`create`, `createBatch`, `update`, `updateStatus`, `registerResult`, `registerScoreEvent`,
  `handlePlayScoreEvent`, `correctScore`).
- **Execução:** o módulo `realtime` escuta com `@TransactionalEventListener(AFTER_COMMIT)` — nunca
  espelha estado que sofreu rollback.
- **Tolerância a falha:** escrita **best-effort** (try/catch + `warn`); falha no Firestore jamais
  quebra a request principal. Sem `FirebaseApp` configurado, o espelho é **no-op**.
- **Contrato de leitura:** o módulo `game` expõe `GameMirrorSnapshot`/`ScoreEventSnapshot` via
  `GameLookup` (API pública, sem vazar tipos de subpacote — disciplina do Spring Modulith).
- **Recuperação:** backfill opt-in (`app.firestore.backfill-on-startup=true`) para sincronizar o que
  já existe.
- **Datas:** campos de *wall clock* (`scheduledAt`, `createdAt`, `updatedAt`) são gravados como
  **ISO-8601 sem zona**, casando com o contrato REST (`LocalDateTime`). Gravar `Timestamp` (instante)
  deslocaria o horário exibido no app quando o servidor roda em fuso diferente do cliente.
  `syncedAt` é `serverTimestamp()` (metadado de infraestrutura — aí o instante é o correto).

### Escopo do primeiro corte

1. **`games`** — espelhar `games/{id}` (este corte; ver
   [`_planning/firestore-sync/issues/01-writer-games.md`](../_planning/firestore-sync/issues/01-writer-games.md)).
2. **`scoreEvents`** — próximo item.
3. **Catálogo** (competitions/teams/venues/standings) — depois.

### Por que não Cloud Functions

| Critério | Admin SDK no backend | Cloud Functions |
|---|---|---|
| Nova stack/runtime | Não | Sim (Node.js + deploy próprio) |
| Escritas já passam pelo backend | Sim | Reintroduz acoplamento HTTP backend → Function |
| Reuso de credenciais/beans | Sim | Não |
| Custo operacional (projeto solo) | Baixo | Maior |
| Cobertura de escritas | 100% (backend é o único escritor) | 100% via trigger |

A camada `onWrite` das Functions só se torna necessária se **passar a existir escrita direta no
Firestore pelo cliente** (otimização offline/mobile-first), o que hoje não existe.

### Nota de custo (conciliação com a análise de custo)

A análise [cloud-cost-benefit-analysis.md](../architecture/cloud-cost-benefit-analysis.md)
(revisada em `0ea4284`) classifica **Firestore/Realtime DB como "não recomendado"** por cobrar por
operação de leitura/escrita. Esta decisão **não contradiz** a análise; concilia-se assim:

- O Firestore é um **espelho de leitura**, não a persistência primária — o custo por operação incide
  sobre o read path público, deliberadamente estreito (primeiro corte: só `games`).
- No **plano gratuito**, o risco real é estourar o teto diário de operações (indisponibilidade),
  não a fatura. Mitigações: **leitura pública apenas**, `listen` em **documento único** (`games/{id}`)
  em vez de varrer a coleção, e monitoramento de operações (issue de monitoramento/drift no mapa).
- Se o read path crescer, a saída é cache/CDN no cliente ou reavaliar o espelho — mantendo o ADB como
  *source of truth* (ADR-002), a porta continua aberta.
- **Regra prática:** nenhuma query de listagem sem índice; o app escuta o documento, não a coleção.

## Alternativas consideradas

| Alternativa | Por que não |
|---|---|
| **Cloud Functions** (HTTP + `onWrite` + Scheduler) | Superado por este ADR. Reintroduz stack Node, deploy e acoplamento. Ver [estrategia-cqrs-cloud-functions.md](../_planning/estrategia-cqrs-cloud-functions.md). |
| **CDC do banco** (Debezium → Pub/Sub) | Infraestrutura extra (broker, conector, deploy) desproporcional ao volume de ~10-20 TPS. |
| **Dual-write no cliente** | Sem retry nem reconciliação; qualquer falha gera drift imediato e permanente. |
| **Event broker dedicado** (Kafka/Pub/Sub) | *Over-engineering* para o volume atual; reservado para TPS > 100. |
| **Apenas Cloud Scheduler** | Latência de até 15 min para qualquer leitura — inaceitável para placar ao vivo. |

## Consequências

**Positivas**
- Sem stack nova; uma única peça de infraestrutura e um único lugar para evoluir o espelhamento.
- Espelhamento consistente com o commit da transação de origem.
- Degradação graciosa: sem Firebase, nada quebra (dev/CI/offline) — o espelho vira no-op.
- Sem vazamento da API interna do módulo `game` (usa projeção pública).
- O Public App passa a ler dados reais (hoje a coleção `games` está vazia).

**Negativas / riscos**
- O espelhamento roda **dentro do processo da API** (CPU/rede), consumindo parte dos `560m` do
  container; aceitável no volume atual.
- Sem a camada `onWrite`, o espelho **não cobre** escrita direta no Firestore — inexistente hoje,
  desde que as *rules* sejam reconciliadas (issue FS-02).
- Sem retry automático: drift é corrigido por backfill manual (issue FS-03) e monitorado (FS-04).
- Dependência operacional do Firebase para o read path em tempo real; o REST continua como fallback
  (`NoopLiveGameRepository`).

## Critérios de aceitação

- [ ] `pom.xml` do `flag_backend` com a dependência `firebase-admin` e o módulo `realtime` criado.
- [ ] `GameChangedEvent` publicado nos caminhos de escrita do módulo `game` e consumido com
      `@TransactionalEventListener(AFTER_COMMIT)`.
- [ ] `games/{id}` espelhado após cada escrita, best-effort, sem quebrar a request em falha.
- [ ] Espelho **no-op sem credencial** (dev/CI sem Firebase) — build e testes não quebram.
- [ ] Datas de *wall clock* em ISO-8601 **sem zona** no documento; `syncedAt` como `serverTimestamp()`.
- [ ] Backfill opt-in funcional (`app.firestore.backfill-on-startup=true`).
- [ ] `firestore.rules`/`firestore.indexes.json` reconciliados: **escrita só pelo backend (Admin SDK)**,
      leitura pública, claims alinhadas; um único conjunto canônico (issue FS-02).
- [ ] O Public App encontra dados reais em `games/{id}`.
- [ ] Monitoramento/drift mínimo: contadores de sync OK/falha e alerta de coleção vazia (FS-04).

## Referências

- [ADR-002](ADR-002-postgres-firestore-cqs.md) – CQRS Light (atualizado por este ADR)
- [ADR-010](ADR-010-autenticacao-firebase-custom-claims.md) – Autenticação Firebase-First com Custom Claims
- [ADR-011](ADR-011-flutter-mvvm-architecture.md) – Arquitetura Flutter MVVM
- [estrategia-cqrs-cloud-functions.md](../_planning/estrategia-cqrs-cloud-functions.md) – **superado** por este ADR
- [cloud-cost-benefit-analysis.md](../architecture/cloud-cost-benefit-analysis.md) – análise de custo (Firestore "não recomendado")
- [Mapa do esforço de sincronização](../_planning/firestore-sync/map.md)
- `flag_public_app/lib/data/repositories/live_game_repository.dart` (commit `dae1814`)
- `flag_platform_infra/docs/kb/oci-always-free-backend.md` (commit `d961e04`)
