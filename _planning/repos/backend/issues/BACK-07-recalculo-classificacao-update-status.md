# BACK-07: Recálculo da classificação não dispara ao encerrar jogo pelo status (updateStatus)

> **Type:** bug
> **Status:** resolved
> **Effort:** repos/backend
> **Repo(s):** flag_backend
> **Blocks:** —
> **Blocked by:** — (BACK-08 resolvido em 2026-10-08)
> **Origin:** verificação em prod (`api.america.app.br`) — 2026-10-07

## Sintoma

`GET /api/v1/competitions/{id}/standings` permanece com `recalculationPending: true` e
`lastCalculatedAt` estagnado, mesmo após jogos serem encerrados. A classificação fica
desatualizada (valores não batem com os jogos).

## Evidência (prod, 2026-10-07)

- `lastCalculatedAt = 2026-09-30T02:36` e `recalculationPending = true` há mais de uma semana.
- 3 jogos FINISHED atualizados em **2026-10-02** sem recálculo:
  - `bd5a147c` (Silver Knights 6×8 UFABC) — upd 2026-10-02 14:01
  - `2ccb4077` (Avaré 35×0 Black Panthers) — upd 2026-10-02 01:40
  - `17dea1e9` (Mustangs 22×8 Lizards) — upd 2026-10-02 01:40
- Prova numérica: Black Panthers `pointsAgainst=90`, mas a soma dos 5 jogos atuais = 125
  (diferença de 35 = o jogo `2ccb4077`), confirmando que o resultado não entrou na classificação.

## Causa

O recálculo é disparado por `GameResultRegisteredEvent`, publicado apenas em
`GameService.registerResult` e `registerWalkover` (usados pelo **app do árbitro**).
O **painel administrativo** finaliza jogos via `PATCH /api/v1/games/{id}/status`
(`CONFERENCE→FINISHED`), que chamava somente `publishChanged` — **sem publicar o evento** —
então `StandingEventListener` → `StandingsCalculationService.recalculateAsync` nunca era acionado.

## Correção (v1.8.7)

`GameService.updateStatus` passa a publicar `GameResultRegisteredEvent` ao entrar em `FINISHED`,
espelhando o `registerResult`. Assim qualquer caminho até FINISHED dispara o recálculo
(o motor já filtra apenas rodadas REGULAR).

## Critérios de aceite

- [x] Encerrar jogo via `PATCH /games/{id}/status` (CONFERENCE→FINISHED) dispara o recálculo.
- [x] `registerResult`/`registerWalkover` continuam disparando (sem regressão).
- [x] Build verde (`mvn clean package -DskipTests`).

## Arquivos

- `game/service/GameService.java` (`updateStatus`, ~L316)

## Notas

- Observação secundária (não tratada neste ticket): `recalculationPending` compara apenas o
  `updatedAt` do jogo, então edições que não alteram resultado (ex.: troca de campo) marcam
  "pendente" sem mudança de placar.
- Deploy: v1.8.7 (tag/Release em `staging`).
