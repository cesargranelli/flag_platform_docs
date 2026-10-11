# PUB-05: Lance a lance com dados incompletos (atletas, recebedores, tempo e pontuações)

> **Type:** bug
> **Status:** resolved
> **Effort:** repos/public_app
> **Repo(s):** flag_public_app, flag_referee_app
> **Blocks:** —
> **Blocked by:** —
> **Origin:** relato do usuário — 2026-10-11

## Sintoma

O lance a lance (play-by-play) apresenta informações faltando tanto na gravação pelo `flag_referee_app` quanto na exibição pelo `flag_public_app`:
1. No detalhe do jogo (`game_plays_view.dart`), nomes de atletas aparecem apenas como números ("12") ou vazios.
2. Não há indicação dos pontos somados (+6 no touchdown, +1/+2 no extra point) nos cards da timeline.
3. Não há indicação clara da conexão passador -> recebedor em jogadas de passe.
4. O horário (`time`) fica sempre fixo em `00:00` ou ausente, pois o referee app não enviava o relógio.

## Causa

- **Referee App (`play_dialog.dart`)**:
  - `_buildNumberGrid` atribui `_selectedPlayer = number` (somente a string do número) e `_selectedPlayerId = entry?.athleteId`, descartando `entry?.athleteName`.
  - Para recebedores, apenas `_selectedReceiver = number` é atribuído.
  - Ao invocar `vm.registerPlay(...)` em `game_operation_panel.dart`, o parâmetro `time` é omitido (`null`).
  - Jogadas sem atleta selecionado enviavam string vazia, arriscando falha de validação ou exibição anônima.
- **Public App (`play_response.dart` / `game_plays_view.dart`)**:
  - `PlayResponse` e `Play` não suportavam `pointsToAdd`.
  - `KicksterTimelineEvent` exibia apenas título e tipo genérico, sem formatar jardas, pontuação ou passador/recebedor.

## Correção proposta

- **Referee App**:
  - Gravar `playerName` formatado com nome do atleta quando presente (ex.: `#12 Nome`), ou fallback `#12`.
  - Gravar `receiverName` formatado.
  - Enviar timestamp/tempo de jogo no registro do lance.
- **Public App**:
  - Mapear `pointsToAdd` em `PlayResponse` e `Play`.
  - Exibir pontuação (`+6`, `+1`), passador + recebedor e jardas no item da timeline de `GamePlaysView`.
  - Tratar fallback de `time` elegantemente.

## Critérios de aceite

- [x] Lances gravados contêm nome legível do atleta e do recebedor quando selecionados.
- [x] Timeline do app público destaca pontuação e detalhes de jardas e conexão de passe.
- [x] Não há quebra em lances de equipe sem atleta individual.

## Resolução

1. **`flag_referee_app`**:
   - Em `play_dialog.dart`: adicionado método de formatação `#<número> <nome>` a partir do `RosterEntry`. Para recebedor, agora o número e o nome são preenchidos corretamente. Se não houver atleta individual, usa o fallback de nome do time.
   - Em `play_dialog.dart`: calcula e retorna `timeStr` no formato `HH:mm`.
   - Em `game_operation_panel.dart`: repassado `time: result.time` na chamada `vm.registerPlay(...)`.
2. **`flag_public_app`**:
   - Em `play_response.dart` e `live_game.dart`: adicionado mapeamento do campo `pointsToAdd`.
   - Em `game_providers.dart`: propagado `pointsToAdd` e `time` ao mapear os lances para a entidade de domínio `Play`.
   - Em `game_plays_view.dart`: timeline aprimorada com badge de pontos (`+6 pts`, `+1 pt`), detalhes de jardas, conexão de passe (`para #88 Recebedor`) e modal de detalhe completo com pontuação.
