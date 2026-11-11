# PUB-10: Correção das rotas e navegação para jogos ao vivo (`/live`)

> **Type:** bug
> **Status:** resolved
> **Effort:** repos/public_app
> **Repo(s):** flag_public_app
> **Blocks:** —
> **Blocked by:** —
> **Origin:** relato do usuário — 2026-10-11

## Sintoma

Cliques em "Ver jogos" no carrossel da Home (`TournamentRoundCarousel`) ou em botões de atalho tentavam navegar para `/live`, causando erro de rota 404 ("Página não encontrada") no GoRouter.

## Causa

O GoRouter em `AppRouter` não possuía rota `/live` registrada, pois a navegação de calendário fica em `/schedule`.

## Correção proposta

- Redirecionar `/live` para `/schedule` ou abrir a listagem de partidas com foco adequado.
- Corrigir as chamadas em `TournamentRoundCarousel` e `home_matches_view` para direcionar a `/schedule`.

## Critérios de aceite

- [x] Clicar em "Ver jogos" leva corretamente para a aba Jogos/Calendário (`/schedule`).
- [x] Nenhuma rota inexistente `/live` lança tela de erro 404.

## Resolução

1. Adicionada rota de redirecionamento `/live` -> `/schedule` em `play_by_play_routes.dart`.
2. Corrigidas as navegações diretas em `home_matches_view.dart` e `tournament_home_sections.dart` para navegar diretamente para `/schedule`.
