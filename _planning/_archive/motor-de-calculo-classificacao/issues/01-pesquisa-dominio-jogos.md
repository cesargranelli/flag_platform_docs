# Pesquisa: domínio de jogos (status, resultado, W.O.)

Type: research
Status: resolved

## Question

Como o domínio de jogos do `flag_backend` modela o que o motor de cálculo precisa consumir?

Investigar e registrar:

- Os estados de `GameEntity` / `GameStatus` e a transição que publica `GameResultRegisteredEvent`
  (quem publica, em que momento, e se há alguma condição de idempotência).
- `GameLookup.findFinishedByCompetitionId` — o que exatamente retorna o `FinishedGame`.
- Existe conceito de **W.O. / walkover**? Se não existe, como um W.O. seria representado hoje
  (status dedicado? placar manual?).
- Como `ScoreEventEntity` (placar por lance) se relaciona com o placar final do `GameEntity`.
- Se jogos de playoffs / fase final entram no mesmo cálculo ou só a fase regular
  (ver `Round.phase`: FASE_REGULAR, WILDCARD, SEMIFINAL, BOWL_FINAL).

## Answer

Resolvido por leitura direta do módulo `game` (e vizinhos) no `flag_backend`.

### Estados do jogo
`GameStatus` (`common/enums/GameStatus.java`): SCHEDULED, OPEN, IN_PROGRESS, CONFERENCE, FINISHED,
CANCELLED, POSTPONED. Transições válidas (`GameService.isValidTransition`, `GameService.java:512`):

- SCHEDULED → OPEN | CANCELLED | POSTPONED
- OPEN → IN_PROGRESS | CANCELLED | POSTPONED
- IN_PROGRESS → CONFERENCE
- CONFERENCE → FINISHED
- POSTPONED → SCHEDULED | CANCELLED
- FINISHED, CANCELLED → terminal (retorna `false`)

### Quem publica o evento e quando
`GameService.registerResult` (`GameService.java:311`), endpoint `POST /api/v1/games/{id}/result`
(`ADMIN_OR_COMMISSIONER`):

- Exige status `CONFERENCE` (senão `GameNotInProgressException`).
- Grava `homeScore`/`awayScore` do request e muda para `FINISHED`.
- Publica `GameResultRegisteredEvent(gameId, competitionId)` (`GameService.java:323`).

Consumo: `StandingEventListener.onGameResultRegistered` (`@TransactionalEventListener` AFTER_COMMIT)
→ `StandingService.recalculate(competitionId)` com `REQUIRES_NEW`. Roda **síncrono após o commit**,
na mesma thread, numa transação própria. **Não há guarda explícita de idempotência**, mas como o
status vira FINISHED e FINISHED é terminal, uma segunda chamada falha com `GameNotInProgressException`.
Não existe transição de volta: **corrigir/reabrir um resultado já FINISHED não é possível pela API hoje**.

### O que o motor recebe hoje
`GameLookup.findFinishedByCompetitionId` (`GameService.java:328`) resolve as rodadas da competição e
faz `findAllByRoundIdInAndStatus(roundIds, FINISHED)`, mapeando para
`FinishedGame(homeTeamId, awayTeamId, homeScore, awayScore)` (`FinishedGame.java`). Limitações
relevantes para o motor:

- **Não carrega** `gameId`, rodada nem fase/tipo — não dá para separar fase regular de playoffs, nem
  saber a qual grupo o confronto pertence, apenas com esse record.
- Times vêm por `teamId` (não por `competition_team`), **sem** filtro de `CompetitionTeamStatus`
  (a `StandingService.recalculate` já usa `findAllByCompetitionIdOrderByCreatedAtAsc` sem filtrar status).
- Jogos CANCELLED/POSTPONED ficam de fora (só FINISHED entra).

### Rodadas e fases
`RoundEntity`: competitionId, number, name, `type` (`RoundType`: REGULAR, PLAYOFFS, WILDCARD,
SEMIFINAL, FINAL). O doc `arquitetura_modulo_competicoes.md` fala de `phase` FASE_REGULAR etc., mas o
enum real é `RoundType`. Hoje o cálculo soma **todas** as rodadas, playoffs inclusive.

### Placar e eventos
`GameEntity.homeScore/awayScore` (Integer, nullable até serem setados). Durante IN_PROGRESS o placar é
incrementado por `addScoreEvent`/`handlePlayScoreEvent` (TD=6, FG=3, Safety=2...). No FINISHED vale o
placar do `registerResult`. `ScoreEventEntity` (tabela `score_events`) é timeline de lances; o motor
**não precisa** dela — usa o placar final do jogo.

### W.O. / walkover
**Não existe** em lugar nenhum do backend: nenhum status WALKOVER, nenhum campo de placar padrão,
nenhum tratamento. Hoje um W.O. só seria representado como placar manual em `registerResult` (ou o
jogo fica CANCELLED, fora da tabela). Qualquer regra de W.O. é **modelagem nova**.

### Consequências para os tickets seguintes
- **04 / 05:** para classificação por grupo/fase, `GameLookup`/`FinishedGame` precisam ser estendidos
  (trazer rodada/tipo e/ou o grupo) — ou o motor resolve grupos por outra via.
- **03 / 04:** W.O. é conceito a criar; decidir status dedicado vs placar padrão.
