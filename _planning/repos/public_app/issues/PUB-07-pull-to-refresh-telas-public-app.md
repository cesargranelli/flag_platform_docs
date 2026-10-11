# PUB-07: Revisão de telas sem atualização / adição de pull-to-refresh global

> **Type:** task
> **Status:** resolved
> **Effort:** repos/public_app
> **Repo(s):** flag_public_app
> **Blocks:** —
> **Blocked by:** —
> **Origin:** relato do usuário — 2026-10-11

## Sintoma

Usuário não consegue forçar a atualização (pull-to-refresh) na maioria das telas do aplicativo público:
- `GameDetailScreen` (abas Estatísticas e Lances);
- `CompetitionStatsScreen` / `TournamentStatsScreen`;
- `CompetitionDetailScreen`;
- `TeamDetailScreen` (`TeamRosterView`, `TeamGamesView`).

Apenas `HomeScreen`, `ScheduleScreen` e `CompetitionStandingsScreen` possuíam `RefreshIndicator`.

## Causa

Ausência de envolver os `ListView` das referidas telas em `RefreshIndicator` conectado à invalidação dos providers correspondentes do Riverpod.

## Correção proposta

Adicionar `RefreshIndicator` com cor padrão Kickster (`AppColors.primary`) e método `onRefresh` que invalida os providers de dados específicos de cada tela.

## Critérios de aceite

- [x] Todas as abas da tela de Jogo (Estatísticas e Lances) respondem ao gesto de pull-to-refresh.
- [x] Telas de Estatísticas da Competição e Detalhes da Equipe respondem ao gesto de pull-to-refresh.
- [x] A experiência de rolagem continua fluida com `AlwaysScrollableScrollPhysics`.

## Resolução

1. **Pull-to-Refresh**:
   Implementado `RefreshIndicator` com `AlwaysScrollableScrollPhysics` e recarregamento/invalidação transparente nos seguintes componentes:
   - `GamePlaysView` (`game_plays_view.dart`): invoca `vm.refresh()` no ViewModel do jogo.
   - `GameStatsView` (`game_stats_view.dart`): invoca `vm.refresh()` no ViewModel do jogo.
   - `CompetitionStatsScreen` (`competition_stats_screen.dart`): invalida standings, games e teams da competição com `Future.wait`.
   - `CompetitionDetailScreen` (`competition_detail_screen.dart`): invalida e revalida a competição corrente.
   - `TeamDetailScreen` (`team_detail_screen.dart`): recarrega dados do time e elenco.
   - `TeamGamesView` (`team_games_view.dart`): recarrega lista de jogos da equipe.

2. **Botão de Atualização na TopBar (`PublicAppBar`)**:
   Adicionado botão de atualização (`AppBarIconButton(icon: Icons.refresh, tooltip: 'Atualizar')`) na barra superior das telas:
   - `GameDetailScreen`: aciona `_viewModel.refresh()`.
   - `CompetitionDetailScreen`: invalida competições e standings/jogos da competição focada.
   - `TournamentStatsScreen`: invalida standings, jogos e times da competição.
   - `TeamDetailScreen`: invalida detalhes do time, elenco e jogos.
   - `ScheduleScreen`: invalida os jogos do calendário.
   - `HomeScreen`: revalida todas as seções e hero da Home.
