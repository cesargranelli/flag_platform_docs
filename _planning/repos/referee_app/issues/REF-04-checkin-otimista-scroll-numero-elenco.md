# REF-04: Check-in otimista imediato, resolução de loop de scroll, número cadastrado e filtro de elenco

> **Type:** bug / perf
> **Status:** done
> **Effort:** repos/referee_app
> **Repo(s):** flag_referee_app, flag_backend
> **Blocks:** —
> **Blocked by:** —
> **Origin:** relato do usuário — 2026-10-11

## Sintoma

1. O toggle de presença Presente/Ausente aguarda a resposta da requisição HTTP à API para depois refletir na tela; com rede lenta ou intermitente no campo, o botão não reage e gera confusão.
2. A lista de atletas entra em looping de rebuild/scroll ao rolar a tela de check-in.
3. O número oficial do atleta não está em destaque no card (só aparecia se houvesse substituição/override).
4. A lista de atletas traz inscrições que não correspondem estritamente ao elenco da equipe para a modalidade da partida (ex.: x8, x5).

## Correção proposta

1. **Mutação Otimista Imediata**: o toggle de presença atualiza visualmente no estado local em 0ms ao toque. A requisição de rede roda em segundo plano com tratamento gracioso de falha e retry offline-first.
2. **Correção do Loop de Scroll**: substituir `SingleChildScrollView` com `for` loop aninhado em `AppRefreshable` por `ListView.builder` com chaves estáveis `ValueKey(checkIn.athleteId)`.
3. **Número Cadastrado**: exibir com clareza o número oficial `#${athleteNumber}` no cabeçalho do card de cada atleta.
4. **Filtro de Elenco por Modalidade**: garantir que apenas atletas que compõem o elenco do time para a categoria e modalidade da partida apareçam na lista.

## Critérios de aceite

- [x] Toque no botão ON/OFF muda imediatamente na UI sem travamento.
- [x] Scroll da lista de check-in é fluido e sem loops.
- [x] O número cadastrado do atleta fica visível em destaque para conferência da mesa.
- [x] Apenas atletas elegíveis para a modalidade da partida aparecem no roster.
