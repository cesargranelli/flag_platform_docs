# BACK-09: `recalculationPending` acusa falso-positivo em edições que não mudam o resultado

> **Type:** bug
> **Status:** resolved
> **Effort:** repos/backend
> **Repo(s):** flag_backend
> **Blocks:** —
> **Blocked by:** —
> **Origin:** follow-up de BACK-07/BACK-08 — verificação em prod (`api.america.app.br`) — 2026-10-08

## Sintoma

O indicador `recalculationPending` da classificação ficava `true` após **qualquer** edição de um
jogo finalizado (ex.: troca de campo/horário), mesmo sem alteração de resultado — sugerindo um
recálculo pendente que não existia.

## Causa

`GameService.findLastFinishedGameUpdatedAt` usava `games.updated_at`, que sobe em **qualquer**
alteração (`BaseEntity.@PreUpdate`) — inclusive campos irrelevantes para a classificação.

## Correção (v1.8.8)

- Migration `013`: coluna `games.result_updated_at` (+ backfill com `updated_at` nos jogos `FINISHED`).
- `result_updated_at` só é atualizado quando o resultado/estado relevante muda:
  - finalizar o jogo (`registerResult`, `registerWalkover`, `updateStatus` → `FINISHED`);
  - alterar **time/rodada** de um jogo já finalizado (`update`).
- `GameRepository.findMaxResultUpdatedAtByRoundIdInAndStatus` passa a usar `resultUpdatedAt`.
- `update()` de jogo finalizado que muda time/rodada também publica `GameResultRegisteredEvent`
  (dispara o recálculo, pois a classificação muda).

## Critérios de aceite

- [x] Editar campo/horário de jogo finalizado **não** marca `recalculationPending`.
- [x] Finalizar jogo marca pendente até o recálculo consolidar.
- [x] Alterar time/rodada de jogo finalizado marca pendente **e** dispara o recálculo.
- [x] Migration aplicada em staging/produção (backfill de 209 jogos `FINISHED`).

## Arquivos

- `db/changelog/changesets/013-games-result-updated-at.yaml` (+ master/runtime)
- `game/entity/GameEntity.java` (`resultUpdatedAt`)
- `game/repository/GameRepository.java` (`findMaxResultUpdatedAtByRoundIdInAndStatus`)
- `game/service/GameService.java` (`update`, `registerResult`, `registerWalkover`, `updateStatus`, `findLastFinishedGameUpdatedAt`)
