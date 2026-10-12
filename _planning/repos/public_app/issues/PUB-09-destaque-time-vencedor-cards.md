# PUB-09: Destaque visual do time vencedor nos cards de partidas encerradas

> **Type:** design
> **Status:** resolved
> **Effort:** repos/public_app
> **Repo(s):** flag_public_app
> **Blocks:** —
> **Blocked by:** —
> **Origin:** relato do usuário — 2026-10-11

## Sintoma

Em partidas já finalizadas (`finished` ou `conference`), ambos os times aparecem com a mesma ênfase visual (mesma cor e mesmo peso de fonte), dificultando a identificação imediata de quem ganhou a partida ao bater o olho no card.

## Causa

Nem `FixtureRow`, nem `KicksterMatchCard`, nem `FeaturedMatchCard` calculavam o vencedor ou aplicavam estilo condicional aos nomes e escudos dos participantes.

## Correção proposta

Para jogos com status `finished` ou `conference`:
- Identificar `isHomeWinner = homeScore > awayScore` e `isAwayWinner = awayScore > homeScore`.
- **Time Vencedor**: tipografia reforçada (`FontWeight.w800`), cor com contraste total (`AppColors.textPrimary`), e opacidade plena (`1.0`).
- **Time Perdedor**: tipografia regular (`FontWeight.w500`), cor atenuada (`AppColors.textSecondary`) e opacidade suavizada (`0.65`).
- **Empate**: ambos mantêm o tratamento padrão neutro.

## Critérios de aceite

- [x] Cards de jogos encerrados destacam visualmente a equipe vitoriosa em relação à derrotada.
- [x] Partidas em andamento (`inProgress`) ou agendadas (`scheduled`) mantêm tratamento simétrico.
- [x] Empates permanecem neutros sem destacar nenhum dos lados.

## Resolução

Após refinamento de UX solicitado pelo usuário:
1. **Linha Indicadora de Vitória Larga e Destacada**:
   - A barra verde (`#10B981`, altura 3.5 a 4px) foi alargada para cobrir a largura total do escudo (48px a 52px), garantindo forte contraste visual esportivo sem interferir na geometria dos elementos.
   - Para o time não vencedor, é reservado exatamente o mesmo espaçamento (com cor transparente), assegurando alinhamento milimétrico, simetria e mesma proporção e posição entre ambos os lados.
2. **Escudos com Tamanho Rigorosamente Idêntico**:
   - Ambos os escudos (mandante e visitante) preservam diâmetro e escala milimetricamente iguais (52px em `FixtureRow`, 32/36px em `KicksterMatchCard` e 58px em `FeaturedMatchCard`).
3. **Equilíbrio Visual e Proporção**:
   - Ambos os times mantêm 100% de opacidade e cor plena (`AppColors.textPrimary`), destacando a vitória exclusivamente através da barra superior e da tipografia reforçada (`FontWeight.w800` a `w900`).
   - Aplicado em `FixtureRow`, `KicksterMatchCard` e `FeaturedMatchCard`.
