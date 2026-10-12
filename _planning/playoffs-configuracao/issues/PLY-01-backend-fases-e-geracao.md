# PLY-01: Backend — fases configuráveis e geração fase a fase da chave

> **Type:** feature
> **Status:** open
> **Effort:** playoffs-configuracao
> **Repo(s):** flag_backend
> **Blocks:** PLY-02
> **Blocked by:** —
> **Origin:** [spec-fases-e-geracao.md](../spec-fases-e-geracao.md) · [ADR-021](../../../adr/ADR-021-geracao-fase-a-fase-playoffs.md)

## Objetivo

Implementar no `flag_backend` a configuração de **fases** dos playoffs e o motor de geração
**fase a fase**, conforme [ADR-021](../../../adr/ADR-021-geracao-fase-a-fase-playoffs.md).

## Escopo

### Migration (dba)
- Coluna `phases_json` (texto JSON) em `playoff_configuration` (mesmo padrão de
  `competitions.grouping_config`). Entrar no `db.changelog-runtime.yaml` **e** no master.

### Domínio / DTOs
- `PlayoffConfiguration` + entidade + request/response com a **lista de fases**.
- Fase: `name`, `type` (`WILDCARD`/`SEMIFINAL`/`FINAL`), `matchups[]`.
- Vaga: `{"ref": "STANDING"|"WINNER"|"BEST_OVERALL"|"LOSER", ...}` (ver spec §3).
- Serialização JSON tolerante (defaults quando ausente/corrompido), como no `SerializationHelper` atual.

### Motor de geração (`PlayoffServiceImpl.generate`)
- Gera **a próxima fase pendente**:
  1. sem fases geradas → fase 1 (a partir da classificação);
  2. última fase com **todos** os jogos `FINISHED` → próxima fase (vencedores);
  3. última fase não concluída → **409**.
- **Re-geração**: só se a fase alvo não tem jogos ou **nenhum começou** (`SCHEDULED`/`OPEN`); senão **409**.
- **Fases anteriores intactas** (não apagar rodadas de fases anteriores).
- Resolver `WINNER`/`LOSER` (resultado dos jogos da fase anterior), `STANDING`/`BEST_OVERALL` (classificação).
- **Árvore automática**: `N = qualifiedPerGroup × grupos + wildcardSlots` → chave de **potência de 2
  com byes** (N par ou ímpar).
- **Remover a restrição de DRAFT** (`assertEditable`): permitir a qualquer momento.
- Permissão: **criador** (`assertManagedBy`) ou ADMIN (já é o caso).

## Critérios de aceite

- [ ] `phases_json` persistido e exposto no CRUD de playoffs.
- [ ] Gerar fase 1 a partir da classificação (criador/ADMIN, qualquer status).
- [ ] Gerar fase N só após todos os jogos da fase N-1 terminarem (senão 409).
- [ ] (Re)gerar fase não iniciada substitui os confrontos; com jogo iniciado → 409.
- [ ] Fases anteriores não são alteradas.
- [ ] `mvn -B clean package -DskipTests` verde.

## Notas

- Versionar (PATCH/MINOR conforme a natureza) antes de implementar.
- Atualizar a doc OpenAPI (`PlayoffApi`).
