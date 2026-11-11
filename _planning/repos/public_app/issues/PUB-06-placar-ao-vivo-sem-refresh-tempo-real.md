# PUB-06: Tela ao vivo e placares sem atualização em tempo real (fallback de polling e Firestore)

> **Type:** bug
> **Status:** resolved
> **Effort:** repos/public_app
> **Repo(s):** flag_public_app
> **Blocks:** —
> **Blocked by:** —
> **Origin:** relato do usuário — 2026-10-11

## Sintoma

Ao acompanhar um jogo ao vivo ou visualizar as partidas em andamento:
1. O placar e os eventos na tela de detalhe do jogo (`/game/:id`) não atualizam sozinhos.
2. O Hero card da Home com a partida ao vivo não reflete as alterações do placar sem ação manual do usuário.

## Causa

1. Em `GameDetailViewModel.startLive()`, se o Firebase estiver disponível (`isFirebaseAvailable == true`), o ViewModel desativa o polling de fallback (`pauseAutoRefresh()`). Caso o documento `games/{id}` no Firestore não exista, não seja espelhado ou o backend esteja sem o espelho ativo, a stream fica silenciada e nenhum dado é atualizado.
2. Além disso, eventos de lance sem pontuação não alteram o placar no PostgreSQL nem propagam atualização no Firestore.
3. Na Home, `liveGamesProvider` é um `FutureProvider` estático sem política periódica de revalidação para jogos com status `inProgress`.

## Correção proposta

- **Hibridismo Resiliente no Detalhe do Jogo**: Manter um heartbeat de polling (ex.: a cada 10-12s) como rede de segurança ativa mesmo com o stream do Firestore ligado, garantindo atualização de placar e novos lances.
- **Auto-Refresh na Home**: Implementar revalidação periódica controlada para partidas ao vivo na tela inicial enquanto houver jogos `inProgress`.

## Critérios de aceite

- [x] A tela de detalhe do jogo atualiza placar e lances automaticamente sem intervenção do usuário enquanto a partida estiver em andamento.
- [x] Partida ao vivo na Home atualiza placar periodicamente sem necessidade de reiniciar o app ou puxar a tela manualmente.

## Resolução

1. **`GameDetailViewModel`**:
   - `startLive()` não desativa mais o safety heartbeat. Foi mantido um timer periódico de 10s ativo para `inProgress`, `conference` e `open`, garantindo sincronização mesmo em cenários sem stream do Firestore ou com lances que não alteram placar.
   - Os timers e subscrições são devidamente limpos no `dispose()`.
2. **`game_providers.dart`**:
   - `liveGamesProvider` transformado em provider auto-refresh com polling a cada 15 segundos e descarte automático ao sair da tela (`ref.onDispose`).
   - `scheduleProvider` monitora partidas ativas e agenda auto-revalidação periódica (a cada 20 segundos) se houver jogos `inProgress` ou `conference`.
