# Motor: escopo de cálculo (grupo/geral) e pipeline de desempate

Type: grilling
Status: resolved
Blocked by: 01, 03, 09, 10

## Question

Definir o comportamento do motor de cálculo: escopo de agrupamento e o pipeline de comparadores.

Decidir:

- **Escopo de agrupamento** (`grouping_scope`): por grupo, geral, ou ambos. Como
  `CompetitionEntity.groupingType` (NONE/GROUPS/CONFERENCES/DIVISIONS) combinado com
  `CompetitionTeam.groupName` / `conference` / `division` produce os baldes de classificação.
- Como o **pipeline de comparadores** é montado a partir do `tiebreaker_criteria_json`.
- Como o **confronto direto** é calculado quando 3+ equipes empatam (mini-tabela entre elas).
- **Onde o motor roda** na evolução do `StandingService` / `StandingEventListener`: síncrono no
  afterCommit atual vs assíncrono; e o custo do pré-cálculo iterativo de SOS/SOV, se adotados.
- **Idempotência** do disparo manual e comportamento ao recalcular a competição inteira.

## Answer

Motor definido em grilling (complementa os tickets 03, 04, 09 e 10).

### 1. Baldes de classificação
A chave do balde vem do `groupingType` da competição:
- `NONE` → só a tabela geral (sentinela `OVERALL`).
- `GROUPS` → um balde por `CompetitionTeam.groupName`.
- `CONFERENCES` → um balde por `conferenceName`.
- `DIVISIONS` → um balde por `divisionName`.

### 2. grouping_scope
- `BY_GROUP` (**default**): gera só as tabelas por balde.
- `OVERALL`: gera só a tabela geral (todos os times).
- `BOTH`: balde + geral.
- Em `groupingType=NONE`, só `OVERALL` faz sentido.

### 3. Rodadas consideradas
Só rodadas `RoundType.REGULAR` (fase regular); playoffs (PLAYOFFS/WILDCARD/SEMIFINAL/FINAL) ficam fora.
Consequência: `GameLookup`/`FinishedGame` precisam restringir a jogos de rodadas REGULAR (hoje
`findFinishedByCompetitionId` soma todas) — e carregar `resultType` para aplicar
`walkover_counts_in_tiebreakers` (ticket 09).

### 4. Pipeline de desempate
- Cadeia montada a partir de `tiebreaker_criteria_json` (lista de objetos) como um
  `Comparator.thenComparing` encadeado.
- Cada critério desempata apenas o subgrupo ainda empatado; ao separar parte, a cadeia **reinicia** para
  o remanescente (premissa do ticket 03, confirmada pelos regulamentos CBFA/BFA).
- `HEAD_TO_HEAD.multiWay`: `DISCARD` (default) ou `MINI_TABLE` (ticket 03).
- `RANDOM_DRAW`: usa as `position` anteriores (ticket 04).

### 5. Execução
- **Assíncrona** via `@Async` (pool interno): tanto o evento de resultado quanto o disparo manual
  enfileiram o recálculo; **sem nova tabela**.
- **Status** derivado por timestamps: `standings.updatedAt` (último cálculo) vs. jogos finalizados mais
  recentes (pendência).
- **Disparo manual** (`POST .../recalculate`) responde **202 "enfileirado"** — não retorna contagens
  imediatas (ajusta o contrato do ticket 06).
- Idempotência: o recálculo apaga e recria as linhas da competição; reexecutar é seguro.

### Consequências para outros tickets
- **06 (contratos):** recálculo vira **202/assíncrono**; o card precisa de estado de status (ou derivar
  dos timestamps); o `GET standings` expõe grupos + PCT/SOS/SOV.
- **08 (UX):** o card deve refletir "processando" e atualizar a tabela após o término.
