# REF-02: Catálogo e revisão semântica de ícones para todos os registros de jogadas

> **Type:** design
> **Status:** open
> **Effort:** repos/referee_app
> **Repo(s):** flag_referee_app
> **Blocks:** —
> **Blocked by:** —
> **Origin:** relato do usuário — 2026-10-11

## Sintoma

Os ícones utilizados nos registros de jogadas estão genéricos e repetidos para categorias inteiras (ex.: todas as pontuações usam o mesmo troféu `Icons.emoji_events`, todas as defesas usam o escudo `Icons.security`, todos os chutes usam setas verticais `Icons.unfold_more` e todas as jogadas de ataque usam a mesma bola genérica).

## Correção proposta

Atribuir ícones semânticos esportivos específicos para cada tipo de jogada:
- **Touchdown**: `Icons.sports_score` / `Icons.stars`
- **Extra Point (+1 / +2)**: `Icons.plus_one` / `Icons.exposure_plus_2`
- **Safety / Pick Two**: `Icons.shield_outlined` / `Icons.swap_horiz`
- **Field Goal**: `Icons.expand_less` (trave)
- **Passe Completo**: `Icons.near_me` (trajetória aérea)
- **Passe Incompleto**: `Icons.block` / `Icons.close`
- **Corrida**: `Icons.directions_run`
- **First Down**: `Icons.double_arrow` / `Icons.flag`
- **Retirada de Fita**: `Icons.content_cut`
- **Sack**: `Icons.back_hand` / `Icons.gavel`
- **Passe Desviado**: `Icons.pan_tool_outlined`
- **Interceptação**: `Icons.change_circle` / `Icons.sync_alt`
- **Punt / Kickoff**: `Icons.arrow_upward` / `Icons.sports_martial_arts`
- **Falta**: `Icons.flag_circle`

## Critérios de aceite

- [ ] Cada tipo de lance possui ícone visualmente coerente e intuitivo.
- [ ] Ícones de pontuação, passe, corrida, defesa e chutes claramente diferenciáveis à primeira vista.
