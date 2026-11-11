# REF-07: Novos tipos de jogada: Kickoff/Retorno, FG Errados/Bloqueados, Toss, Faltas Duplas, Ejeção

> **Type:** regra / api
> **Status:** done
> **Effort:** repos/referee_app
> **Repo(s):** flag_referee_app, flag_backend
> **Blocks:** —
> **Blocked by:** —
> **Origin:** relato do usuário — 2026-10-11

## Sintoma

O catálogo atual de jogadas no `PlayType` e backend não suporta lances fundamentais da dinâmica de jogo:
1. Kickoff e retorno de kickoff (ou touchback), sem registro de onde a jogada começa.
2. Não há opção de registrar Passe Toss (lateral/pitch).
3. Não há opção de marcar erro em tentativa de Field Goal ou bloqueio do chute.
4. Faltas não contemplam faltas duplas (offsetting fouls), falta de conduta antidesportiva e marcação se a falta acarreta perda de descida ou repetição.
5. Não há registro formal de Ejeção/Expulsão de atleta ou comissão.
6. Não há marcação de Snap Ruim (`Bad Snap`).
7. Não há registro oficial do Aviso dos 2 Minutos (`Two-minute warning`).

## Correção proposta

Expandir o enum `PlayType` e modelos de request com os seguintes tipos:
- **`KICKOFF` & Retorno / Touchback**: suporte ao ponto de chute e retorno (ou início padrão na linha de 20 jardas em touchback).
- **`PASS_TOSS`**: passe curto lateral / toss.
- **`FIELD_GOAL_MISSED`**: erro de field goal (posse no ponto do chute ou linha de 20).
- **`FIELD_GOAL_BLOCKED`**: field goal bloqueado pela defesa.
- **`DOUBLE_PENALTY`**: faltas simultâneas de ambas as equipes (anula lance e repete descida).
- **`UNSPORTSMANLIKE_CONDUCT`**: conduta antidesportiva.
- **`EJECTION`**: desqualificação/expulsão de participante.
- **`BAD_SNAP`**: perda de jardas por snap falho.
- **`TWO_MINUTE_WARNING`**: parada regulamentar dos dois minutos finais.
- **Opção de Perda de Descida**: toggle no diálogo de falta indicando se consome a descida (loss of down) ou repete.

## Critérios de aceite

- [x] Todos os novos tipos de jogadas estão disponíveis para seleção na modalidade compatível.
- [x] Regras de posse e descida reagem adequadamente a cada novo evento.
