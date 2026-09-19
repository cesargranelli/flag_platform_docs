# ADR-010: Espelhamento PostgreSQL → Firestore pelo backend (Firebase Admin SDK)

## Status
Aceito — implementado para `games` + `scoreEvents` (Bloco 6.1). Catálogo (competitions, teams, venues, standings) fica para depois.

## Contexto

O [ADR-002](ADR-002-postgres-firestore-cqs.md) define a arquitetura **CQRS Light**: PostgreSQL é a *source of truth* (write path) e Firestore é o **espelho de leitura** (read path) para dashboards, leaderboards e **placar/lances em tempo real**.

Faltava definir **como** a sincronização acontece. O documento [estrategia-cqrs-cloud-functions.md](estrategia-cqrs-cloud-functions.md) propunha **Cloud Functions** em três camadas (HTTP trigger + `onWrite` + Cloud Scheduler). Esse documento permaneceu com status *Proposto* e nunca foi implementado.

Restrições reais do projeto:
- Volume baixo (~10-20 TPS) — não justifica *event broker* dedicado.
- O **backend Spring Boot (Modular Monolith) é o único produtor de escritas** em PostgreSQL: o Referee App e o Admin Web escrevem pelo REST, não direto no banco.
- O **Firebase Admin SDK já é dependência** do backend e já está inicializado de forma resiliente (`FirebaseConfig` — vira no-op quando não há credencial, ex.: dev local).
- `firestore.rules` e `firestore.indexes.json` já estavam desenhados, com **leitura pública** em `games` e `scoreEvents`.

## Decisão

O espelhamento é feito **pelo próprio backend**, com o **Firebase Admin SDK**, e **não** por Cloud Functions.

- **Gatilho:** o módulo `game` publica `GameChangedEvent(gameId)` em todos os caminhos de escrita (`create`, `createBatch`, `update`, `updateStatus`, `registerResult`, `registerScoreEvent`, `handlePlayScoreEvent`, `correctScore`).
- **Execução:** o módulo `realtime` escuta com `@TransactionalEventListener(AFTER_COMMIT)` — nunca espelha estado que sofreu rollback.
- **Tolerância a falha:** escrita **best-effort** (try/catch + `warn`); falha no Firestore jamais quebra a request principal. Sem `FirebaseApp` configurado, o espelho é **no-op**.
- **Contrato de leitura:** o módulo `game` expõe `GameMirrorSnapshot`/`ScoreEventSnapshot` via `GameLookup` (API pública, sem vazar tipos de subpacote — disciplina do Spring Modulith).
- **Recuperação:** backfill opt-in (`app.firestore.backfill-on-startup=true`) para sincronizar o que já existe.
- **Datas:** campos de *wall clock* (`scheduledAt`, `createdAt`, `updatedAt`) são gravados como **ISO-8601 sem zona**, casando com o contrato REST (`LocalDateTime`). Gravar `Timestamp` (instante) deslocaria o horário exibido no app quando o servidor roda em fuso diferente do cliente. `syncedAt` é `serverTimestamp()` (metadado de infraestrutura — aí o instante é o correto).

### Por que não Cloud Functions (por ora)

| Critério | Admin SDK no backend | Cloud Functions |
|---|---|---|
| Nova stack/runtime | Não | Sim (Node.js + deploy próprio) |
| Escritas já passam pelo backend | Sim | Reintroduz acoplamento HTTP backend → Function |
| Reuso de credenciais/beans | Sim | Não |
| Custo operacional (projeto solo) | Baixo | Maior |
| Cobertura de escritas | 100% (backend é o único escritor) | 100% via trigger |

A camada `onWrite` das Functions só se torna necessária se **passar a existir escrita direta no Firestore pelo cliente** (otimização offline/mobile-first), o que hoje não existe.

## Consequências

**Positivas**
- Sem stack nova; uma única peça de infraestrutura e um único lugar para evoluir o espelhamento.
- Espelhamento consistente com o commit da transação de origem.
- Degradação graciosa: sem Firebase, nada quebra (dev/offline).
- Sem vazamento da API interna do módulo `game` (usa projeção pública).

**Negativas / riscos**
- O espelhamento roda **dentro do processo da API** (CPU/rede do request seguinte); aceitável no volume atual.
- Sem a camada `onWrite`, o espelho **não cobre** escrita direta no Firestore — inexistente hoje.
- Sem retry automático: drift é corrigido por backfill manual.

## Revisitar quando
- Surgir escrita direta no Firestore pelo cliente (aí, adotar a camada `onWrite`).
- O volume crescer a ponto de justificar desacoplamento (aí, `@Async` ou broker).
- O espelho passar a incluir o catálogo completo e consultas complexas no lado de leitura.
