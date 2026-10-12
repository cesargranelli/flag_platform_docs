# REF-03: Priorização e ordenação estrita dos confrontos na Home por esteira operacional

> **Type:** feature
> **Status:** open
> **Effort:** repos/referee_app
> **Repo(s):** flag_referee_app
> **Blocks:** —
> **Blocked by:** —
> **Origin:** relato do usuário — 2026-10-11

## Sintoma

Ao entrar na Home do Referee App, os jogos aparecem agrupados por rodadas e ordenados de forma arbitrária/cronológica, obrigando o árbitro a procurar manualmente a partida que está acontecendo no momento.

## Correção proposta

Em `referee_home_screen.dart`, unificar a exibição dos jogos priorizando a seguinte ordem operacional:
1. **Ao vivo em andamento (`IN_PROGRESS`)**: destaque absoluto no topo com status pulsante e botão de ação direta *"Operar Jogo"*.
2. **Check-in / Iniciar Jogo (`OPEN`)**: partidas abertas para conferência ou prontas para o apito inicial.
3. **Conferência (`CONFERENCE`)**: partidas finalizadas no campo aguardando fechamento e conferência da súmula.
4. **Encerrado (`FINISHED`)**: partidas do dia concluídas.
5. **Últimos 5 jogos passados**: histórico recente, evitando paginação pesada e poluição visual.

## Critérios de aceite

- [ ] A tela Home lista os jogos estritamente na ordem: `IN_PROGRESS` → `OPEN` → `CONFERENCE` → `FINISHED` → passados (máx 5).
- [ ] Partidas em andamento ganham destaque e acesso em 1 toque ao painel de operação.
