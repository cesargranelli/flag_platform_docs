# Modelo de dados: competition_standings_rules + expansão de standings + migração

Type: grilling
Status: resolved
Blocked by: 01, 03, 09

## Question

Definir o modelo de persistência das regras de classificação e as migrações de schema.

Decidir:

- **Nova tabela `competition_standings_rules`** (1:1 com a competição, FK `ON DELETE CASCADE`):
  colunas de pontuação, W.O. (placar padrão), `ranking_system`, `grouping_scope`,
  `tiebreaker_criteria_json` (CLOB persistida como `String`, evitando o problema de CLOB/Oracle já
  conhecido em `grouping_config`), timestamps.
- **Expansão de `standings`** para suportar grupo e PCT: `group_name`, `win_percentage` e,
  possivelmente, colunas avançadas (SOS/SOV). Impacto na *unique constraint* atual
  `(competition_id, team_id)` — passa a incluir `group_name`?
- **Migrações Liquibase** (`src/main/resources/db/changelog`) e **versionamento MINOR** no `pom.xml`.
  (Nota: `adr/ADR-012` cita Flyway/jOOQ, mas o `AGENTS.md` do `flag_backend` manda usar Liquibase —
  confirmar qual é o vigente.)
- **Retrocompatibilidade:** qual regra default para competições já existentes (3-1-0 vs PCT) e se é
  preciso recalcular os dados existentes.

## Answer

Modelo de dados decidido em grilling. Vale para o `flag_backend` (Oracle + Liquibase).

### 1. Nova tabela `competition_standings_rules` (1:1 com a competição)

```yaml
id UUID PK
competition_id UUID NOT NULL UNIQUE  FK -> competitions(id) ON DELETE CASCADE
ranking_system VARCHAR(20) NOT NULL DEFAULT 'WIN_PERCENTAGE'   # WIN_PERCENTAGE | POINTS_BASED
points_per_win INTEGER NOT NULL DEFAULT 3
points_per_draw INTEGER NOT NULL DEFAULT 1
points_per_loss INTEGER NOT NULL DEFAULT 0
points_per_walkover_win INTEGER NOT NULL DEFAULT 3
walkover_score_pro INTEGER NOT NULL DEFAULT 49
walkover_score_against INTEGER NOT NULL DEFAULT 0
walkover_counts_in_tiebreakers BOOLEAN NOT NULL DEFAULT true
grouping_scope VARCHAR(20) NOT NULL DEFAULT 'BY_GROUP'         # BY_GROUP | OVERALL | BOTH
tiebreaker_criteria_json CLOB NOT NULL
created_at TIMESTAMP, updated_at TIMESTAMP
```

- `tiebreaker_criteria_json` é **lista de objetos** (não strings): cada item
  `{ "criterion": "...", "multiWay"?: "...", ... }`.
- Persistida como **CLOB mapeada para `String`** (evita o ORA-17023 já conhecido em `grouping_config`).
- **Ausência de linha** ⇒ o serviço **sintetiza** as regras default.

### 2. Expansão de `standings`

- `group_name VARCHAR(50) NOT NULL` com **sentinela `'OVERALL'`** para a tabela geral.
- Nova **unique `(competition_id, group_name, team_id)`** (substitui `(competition_id, team_id)`).
- Colunas derivadas persistidas: `win_percentage DECIMAL(5,3)`, `strength_of_victory DECIMAL(5,3)`,
  `strength_of_schedule DECIMAL(5,3)`, `position INTEGER`.
- Mantêm-se `played, wins, draws, losses, goals_for, goals_against, points` (`points` usado no modo
  `POINTS_BASED`; `goals_*` representam pontos de placar).
- **Sorteio:** no recálculo o motor lê as `position` anteriores antes de apagar as linhas e mantém a
  ordem relativa das equipes ainda empatadas em `RANDOM_DRAW` (sorteia uma vez e persiste).

### 3. Retrocompatibilidade

- Default **`WIN_PERCENTAGE`** para **todas** as competições, existentes inclusive (**W-D-L/PCT**).
  Sem backfill em 3-1-0.
- **Sem backfill de linhas**: quando não há regra, o serviço sintetiza o default; a linha é criada ao
  salvar. Migração apenas de schema.
- A classificação já publicada passa a refletir PCT **no próximo resultado** ou no **disparo manual do
  ADMIN**.

### 4. Migração

- Novo changeset Liquibase `005-standings-rules.yaml` (criar `competition_standings_rules`; alterar
  `standings`: add colunas + drop/recreate da unique), incluído em `db.changelog-master.yaml`.
  **Só schema**, sem lógica de negócio.
- **Versionamento MINOR** no `pom.xml` do `flag_backend`; tag de deploy igual à versão.
