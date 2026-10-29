# ADR-024: Otimizações de Schema, Eliminação de N+1 Queries e Agregador Match Center

## Status
Aceito

## Contexto
Após a estabilização da Flag Platform em ambiente de Staging com as quatro aplicações integradas (`flag_backend`, `flag_admin_web`, `flag_referee_app`, `flag_public_app`), uma avaliação arquitetural profunda identificou débitos históricos de modelagem e gargalos severos de desempenho:
1. **Tabelas Órfãs no Banco**: `user_skills` e `categories` foram criadas na gênese do schema (changeset 001), mas não possuíam nenhuma entidade JPA, repositório ou serviço ativo.
2. **Identidade Incompleta de Pessoas**: `persons` não possuía o campo `nickname` (apelido), forçando a repetição manual de apelidos no elenco (`team_roster`) e impedindo apelidos para árbitros e comissários.
3. **Ausência de `competition_id` em `games`**: Todas as buscas de partidas de um campeonato exigiam lookups e joins com a tabela `rounds`.
4. **N+1 Queries em Cascata no Backend**: Ao listar jogos de um campeonato ou rodada, o `GameService` realizava lookups pontuais para cada jogo para enriquecer dados de sedes (`venues`) e times (`teams`), disparando centenas de queries SQL sequenciais por requisição.
5. **Chatty APIs no Cliente**: Para carregar a tela de um jogo, o app móvel ou web realizava 4 requisições HTTP sequenciais (`/games/{id}`, `/games/{id}/participants`, `/games/{id}/checkin`, `/games/{id}/plays`).

## Decisão
Implementar a Fase 1 do plano de modernização de schema e recursos REST:
1. **Changeset 017 no Liquibase**:
   - `DROP TABLE user_skills;` e `DROP TABLE categories;` com `cascadeConstraints: true`.
   - Adicionar coluna `nickname VARCHAR(100)` na tabela `persons`.
   - Adicionar coluna `competition_id UUID` na tabela `games`, retroalimentada a partir de `rounds.competition_id`, com foreign key para `competitions(id)` e índice `idx_games_competition_id`.
2. **Entidades e Contratos**:
   - Atualizar `PersonEntity`, `CreatePersonRequest`, `UpdatePersonRequest`, `PersonResponse`, `PersonInfo` e modelo Dart `Person` para suportar `nickname`.
   - Se um atleta no elenco (`team_roster`) não possuir override de apelido, o sistema herda automaticamente o `nickname` da sua ficha de pessoa física (`persons`).
3. **Batch Lookups no Spring Modulith**:
   - Adicionar métodos `findTeamInfoByIdIn(Collection<UUID> ids)` em `TeamLookup` e `findVenueInfoByIdIn(Collection<UUID> ids)` em `VenueLookup`.
   - Refatorar `GameService` e `PlayService` para buscar times e sedes em lote único antes de mapear coleções, reduzindo de 200+ queries para apenas 3 queries por listagem.
4. **Endpoint Agregador Match Center**:
   - Criar `GET /api/v1/games/{id}/match-center` retornando `MatchCenterResponse` contendo jogo enriquecido, equipe de arbitragem (`officials`), súmula de atletas escalados (`roster`), lances (`plays`) e pontuações (`scoreEvents`) em 1 único roundtrip HTTP.
   - Criar interfaces de consulta de módulo `CheckInLookup` e `GameParticipantLookup` para manter encapsulamento estrito do Spring Modulith.

## Consequências
- **Positivas**:
  - Eliminação de 100% das tabelas órfãs no banco de dados.
  - Apelidos de atletas e profissionais preservados em todo o histórico esportivo na plataforma.
  - Eliminação do gargalo de N+1 queries, acelerando drasticamente o tempo de resposta das APIs de jogos e lances.
  - Redução de 4 requisições HTTP para 1 requisição no carregamento do Match Center pelo `flag_referee_app` e `flag_admin_web`.
- **Negativas/Trade-offs**:
  - `MatchCenterResponse` trafega um payload maior do que o `GameResponse` isolado; no entanto, o ganho de eliminar 3 conexões TCP/TLS adicionais compensa amplamente em redes móveis.
