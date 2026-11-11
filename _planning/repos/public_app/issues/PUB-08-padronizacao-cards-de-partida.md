# PUB-08: Padronização dos cards de partida com placar em destaque

> **Type:** design
> **Status:** resolved
> **Effort:** repos/public_app
> **Repo(s):** flag_public_app
> **Blocks:** —
> **Blocked by:** —
> **Origin:** relato do usuário — 2026-10-11

## Sintoma

Os cards de partida (`FixtureRow`, `KicksterMatchCard` e `FeaturedMatchCard`) apresentavam diferenças sutis de hierarquia visual e não conferiam o mesmo impacto ao placar observado no cabeçalho de detalhe do jogo (`KicksterGameHeader`).

## Causa

Dispersão de componentes entre `FixtureRow` (usado no Calendário e no time) e `KicksterMatchCard` (usado no carrossel da Home), com tipografia e destaque de container de placar menos imponentes.

## Correção proposta

- Elevar a tipografia e o contraste do bloco central de placar (`homeScore : awayScore`), com fontes consistentes e destaque cromático esportivo (destaque vermelho/vivo quando `inProgress`, contraste forte quando encerrado).
- Harmonizar proporções de escudos e alinhamento dos confrontos.

## Critérios de aceite

- [x] O placar nos cards de jogo ganha leitura imediata e alto contraste em harmonia com a tela de detalhes.
- [x] Jogos ao vivo mantêm indicação clara com status live e cor de destaque.

## Resolução

Após refinamento de UX com o usuário, os cards foram arquitetados com isolamento total do confronto e separação de metadados:
1. **Isolamento de Escudos e Placar**:
   - A linha de confronto contém **exclusivamente** os participantes e o placar central (`[ Mandante ] — [ Placar / VS ] — [ Visitante ]`), sem caixas/containers artificiais.
   - Textos secundários (data, horário e estádio) foram migrados para uma **linha dedicada de metadados**, evitando interferência e poluição visual no bloco do placar.
2. **`FixtureRow` (`fixture_row.dart`)**:
   - Placar em texto limpo central (20px, `FontWeight.w900`), com `VS` quando agendado.
   - Data, horário e estádio dispostos em linha inferior com ícones discretos.
3. **`KicksterMatchCard` (`kickster_match_card.dart`)**:
   - Centro ocupado estritamente pelo placar (15-16px, `FontWeight.w900`).
   - Data e local alinhados em rodapé dedicado.
4. **`FeaturedMatchCard` (`featured_match_card.dart`)**:
   - Placar em 24px `FontWeight.w900`, com linha secundária para data e horário e preservação dos chips de status oficiais.
5. **Equalização dos Escudos (`KicksterTeamShield`)**:
   - Padronização em `52px` para todos os cards de duelo e cabeçalho principal (`FeaturedMatchCard`, `FixtureRow` e `KicksterGameHeader`).
   - Carrossel horizontal (`KicksterMatchCard`) equalizado com escudos de `38px` (compacto) e `42px` (padrão) para equilíbrio com a altura de 144px do card.
   - Restrição estrita de `maxWidth: size` e `maxHeight: size` no `KicksterTeamShield`, garantindo consistência com fallback vetorial.

