# REF-08: Registro de tempo, períodos (Início/Fim), revisão e estabilização da ordenação

> **Type:** bug / feature
> **Status:** open
> **Effort:** repos/referee_app
> **Repo(s):** flag_referee_app, flag_backend
> **Blocks:** —
> **Blocked by:** —
> **Origin:** relato do usuário — 2026-10-11

## Sintoma

1. Não há registro explícito de tempo de jogo/cronômetro ao registrar os lances.
2. Não há registros de Início e Término de Período (1Q, 2Q, Halftime, 3Q, 4Q, Overtime) e Revisão de Jogada.
3. A sequência de registros na timeline desordena e flutua ao sair e voltar da tela devido à dependência exclusiva de ordenação por timestamp aproximado sem desempate estável.
4. Ao abrir um registro existente, as jardas registradas não são exibidas na visualização de detalhes.

## Correção proposta

1. **Tempo de Jogo**: garantir campo de tempo `MM:SS` (ou timestamp do relógio) enviado no payload do lance.
2. **Eventos Administrativos de Período**:
   - `PERIOD_START` e `PERIOD_END` com seleção do quarto (1Q, 2Q, Halftime, 3Q, 4Q, OT).
   - `PLAY_REVIEW`: registro de conferência/revisão de jogada pela arbitragem.
3. **Exibição de Jardas**: mostrar as jardas registradas no detalhe do lance e no card de resumo da timeline.
4. **Estabilidade de Ordenação**:
   - Garantir ordenação por `createdAt DESC, id DESC` no backend (`PlayRepository`) e no frontend, ou vincular `sequenceNumber` autoincremental por jogo, evitando embaralhamento dos lances.

## Critérios de aceite

- [ ] Campo de tempo é persistido e exibido nos lances da timeline.
- [ ] Início e término de períodos podem ser registrados formalmente.
- [ ] Lista de lances mantém rigorosamente a mesma ordem antes e depois de recarregar a tela.
- [ ] Detalhe do registro exibe as jardas anotadas na jogada.
