# W.O.: modelagem no domínio de jogos (estado, transições, placar padrão)

Type: grilling
Status: resolved

## Question

Definir **como o W.O. (vitória por ausência / desistência) vira um conceito no módulo `game`**, já que
hoje ele não existe (ver *Pesquisa: domínio de jogos*). Decidido no ticket de regras que o v1 terá W.O.
dedicado com placar padrão configurável; falta a modelagem.

Decidir:

- **Representação:** novo valor em `GameStatus` (ex.: `WALKOVER`) vs flag booleana no jogo vs registro
  de resultado com origem/motivo. Impacto nas transições válidas (`GameService.isValidTransition`).
- **Quem registra** o W.O. e em que ponto do fluxo (o `registerResult` hoje exige `CONFERENCE`).
- **Placar padrão:** de onde vem (regra da competição), default 49×0; como fica no `GameEntity`
  (`homeScore`/`awayScore`).
- **Efeito na classificação:** como o motor conta `played`, `V/E/D`, `PF/PA` e pontos de W.O.
- **Permissão** para registrar/corrigir um W.O.

## Answer

Decidido em grilling.

1. **Representação:** novo campo de **origem do resultado** no jogo — `resultType` ∈ {`NORMAL`,
   `WALKOVER`} (enum de domínio). O `GameStatus` **não** muda; um W.O. termina em `FINISHED` com
   `resultType = WALKOVER`. (Separa "tipo de resultado" do ciclo de status.)
2. **Registro:** endpoint dedicado `POST /api/v1/games/{id}/walkover`, recebendo o **lado que
   desistiu** (`forfeitTeamId`); o backend grava o **placar padrão da competição** (default 49×0, vindo
   das regras — ticket 04), `resultType = WALKOVER` e `status = FINISHED`. Permissão
   `@PreAuthorize(SecurityExpressions.ADMIN_OR_COMMISSIONER)`.
3. **Transição:** o W.O. pode ser lançado de **qualquer estado não-terminal** — SCHEDULED, OPEN,
   IN_PROGRESS, CONFERENCE, POSTPONED → FINISHED/WALKOVER (não de FINISHED/CANCELLED).
4. **Efeito na classificação (revisado na reabertura):** **configurável por competição** — parâmetro
   "jogos de W.O. contam nos critérios de desempate?" (**default: ligado**).
   - Ligado: o placar padrão entra em PF/PA, saldo, pontos pró/sofrido e PCT.
   - Desligado: V-E-D e pontos **continuam** contando, mas PF/PA, saldo e SOS/SOV **ignoram** o jogo
     (segue a CBFA — Superliga FA D1).
   Em ambos os casos, a campanha V-E-D registra a vitória/derrota do W.O.

Premissas:

- O placar padrão e os pontos de W.O. vivem nas **regras da competição** (ticket 04), não no jogo.
- Correção/reversão de um W.O. cai na mesma limitação de FINISHED terminal (ver fog no mapa).

Consequência: `FinishedGame`/`GameLookup` (ticket 05) podem opcionalmente carregar `resultType` para
exibição/auditoria; o cálculo em si não depende disso.

## Reabertura

Reaberto para reavaliar o **item 4** (efeito do W.O. na classificação), após o achado da pesquisa
*Pesquisa: fórmulas de SOS e SOV*: a **CBFA (Superliga FA D1 2025)** determina que *"jogos com W.O.
não serão considerados jogos válidos para critérios de desempate"* — o oposto da decisão original.

**Questão em aberto:** jogos de W.O. devem contar nos critérios de desempate (saldo, PF/PA, SOS/SOV)?

**Resolução:** **configurável por competição** (default: ligado) — ver item 4 revisado acima. Os itens
1–3 (representação, endpoint, transição) permanecem válidos.
