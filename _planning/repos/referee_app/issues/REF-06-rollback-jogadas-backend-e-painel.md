# REF-06: Rollback confiável de lances no backend (estorno de pontos e score events) e botão no painel

> **Type:** feature
> **Status:** done
> **Effort:** repos/referee_app
> **Repo(s):** flag_referee_app, flag_backend
> **Blocks:** —
> **Blocked by:** —
> **Origin:** relato do usuário — 2026-10-11

## Sintoma

1. Jogadas registradas por engano não são descartadas da base quando a mesa tenta reverter.
2. O árbitro não possui forma rápida de anular o último lance diretamente no painel de operação.
3. Não há endpoint no backend para exclusão de lances (`PlayApi` possui apenas GET, POST e PATCH). Se o lance computou pontos (`pointsToAdd > 0`), o placar do jogo e os eventos da súmula ficam corrompidos.

## Correção proposta

1. **Backend (`flag_backend`)**:
   - Implementar endpoint `DELETE /api/v1/games/{gameId}/plays/{playId}` (e opcionalmente `DELETE /api/v1/games/{gameId}/plays/last`).
   - Se o lance continha pontos (`pointsToAdd > 0`), subtrair os pontos do placar do jogo (`homeScore` ou `awayScore`).
   - Excluir o `ScoreEvent` correspondente.
   - Disparar evento para sincronização com Firestore / Public App.
2. **Referee App (`flag_referee_app`)**:
   - Adicionar botão *"Desfazer último lance"* com confirmação de 1 toque no topo da timeline de lances no painel de operação.
   - Atualizar imediatamente a timeline e o placar local na memória.

## Critérios de aceite

- [x] Chamada de rollback remove o lance da lista de jogadas de forma permanente.
- [x] Pontos atrelados ao lance revertido são subtraídos automaticamente do placar do jogo.
- [x] Botão de desfazer no painel do árbitro permite corrigir erros de lançamento em 1 toque.
