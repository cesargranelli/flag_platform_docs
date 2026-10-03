# Spec — Motor de Cálculo de Classificação de Competições

> **Status:** especificação consolidada. Destino do esforço de wayfinding
> `_planning/_archive/motor-de-calculo-classificacao/` (tickets 01–10). Pronta para virar plano de implementação.
> **Nada aqui é código:** é o contrato de decisão para backend (`flag_backend`), admin web
> (`flag_admin_web`) e o contrato de leitura do `flag_public_app`.

---

## 1. Objetivo

Permitir que cada competição da Flag Platform tenha **regras de classificação configuráveis pelo
organizador**, calculadas por um **motor** que gera as tabelas de classificação (por grupo e/ou geral) a
partir dos jogos finalizados, expostas via API, com **disparo manual pelo ADMIN**.

## 2. Escopo

**Dentro:**
- Backend: modelo de dados, motor de cálculo, endpoints de regras/disparo/leitura, W.O.
- Admin web: cadastro/edição das regras (DRAFT) e leitura + card de disparo (ADMIN).
- Contrato de leitura da classificação que o `flag_public_app` consumirá.

**Fora:**
- Tela de visualização de classificação no admin (é do app público).
- UI do `flag_public_app` (aqui só o contrato).
- Motor de regras arbitrárias / plugins — o catálogo é **fechado e computável**.

---

## 3. Modelo de domínio

- **Classificação por campanha W-D-L** (vitórias / empates / derrotas), com modelo configurável:
  - `WINS_DRAWS_LOSSES` (**default**): ordena por **vitórias ↓**, depois **empates ↓**; as **derrotas
    são apenas registradas** (não entram na ordenação primária); empate ⇒ entra o pipeline de desempate.
  - `WIN_PERCENTAGE`: métrica primária **PCT = (V + 0,5·E) / J**.
  - `POINTS_BASED`: métrica primária **pontos = V·pV + E·pE + D·pD** (valores configuráveis, default 3/1/0).
- **Agrupamento:** `GroupingType` = NONE | GROUPS | CONFERENCES | DIVISIONS define os agrupamentos.
- **W.O.:** origem de resultado no jogo (`resultType` = NORMAL | WALKOVER); status do jogo segue FINISHED.

---

## 4. Modelo de dados (`flag_backend` — Oracle/Liquibase)

### 4.1 Nova tabela `competition_standings_rules` (1:1 com a competição)

```yaml
id UUID PK
competition_id UUID NOT NULL UNIQUE  FK -> competitions(id) ON DELETE CASCADE
ranking_system VARCHAR(20) NOT NULL DEFAULT 'WINS_DRAWS_LOSSES'  # WINS_DRAWS_LOSSES | WIN_PERCENTAGE | POINTS_BASED
points_per_win INTEGER NOT NULL DEFAULT 3
points_per_draw INTEGER NOT NULL DEFAULT 1
points_per_loss INTEGER NOT NULL DEFAULT 0
points_per_walkover_win INTEGER NOT NULL DEFAULT 3
walkover_score_pro INTEGER NOT NULL DEFAULT 49
walkover_score_against INTEGER NOT NULL DEFAULT 0
walkover_counts_in_tiebreakers BOOLEAN NOT NULL DEFAULT true
grouping_scope VARCHAR(20) NOT NULL DEFAULT 'BY_GROUP'         # BY_GROUP | OVERALL | BOTH
tiebreaker_criteria_json CLOB NOT NULL                          # lista de objetos (ver §5.4)
created_at TIMESTAMP, updated_at TIMESTAMP
```

- `tiebreaker_criteria_json` é **lista de objetos**: `[{ "criterion": "...", "multiWay"?: "..." }, ...]`.
- Persistida como **CLOB mapeada para `String`** (evita o ORA-17023 já conhecido em `grouping_config`).
- **Sem linha** ⇒ o serviço **sintetiza** as regras default acima.

### 4.2 Expansão de `standings`

- `group_name VARCHAR(50) NOT NULL` com **sentinela `'OVERALL'`** para a tabela geral.
- Nova **unique `(competition_id, group_name, team_id)`** (substitui `(competition_id, team_id)`).
- Colunas derivadas persistidas: `win_percentage DECIMAL(5,3)`, `strength_of_victory DECIMAL(5,3)`,
  `strength_of_schedule DECIMAL(5,3)`, `position INTEGER`.
- Mantêm-se `played, wins, draws, losses, goals_for, goals_against, points` (`goals_*` = pontos de
  placar; `points` usado no modo `POINTS_BASED`).

### 4.3 Migração e retrocompatibilidade

- Novo changeset Liquibase `005-standings-rules.yaml` (criar a tabela de regras; alterar `standings`:
  add colunas + drop/recreate da unique), incluído em `db.changelog-master.yaml`. **Só schema.**
- **Default `WINS_DRAWS_LOSSES` para todas as competições, existentes inclusive** (V-E-D por contagem). **Sem
  backfill** de linhas nem de 3-1-0 — o default é sintetizado quando não há regra.
- A tabela já publicada reflete PCT **no próximo resultado** ou no **disparo manual do ADMIN**.
- **Versionamento MINOR** no `pom.xml`; tag de deploy igual à versão.

---

## 5. Motor de cálculo

### 5.1 Agrupamentos de classificação
Chave pelo `groupingType`:
- `NONE` → só a tabela geral (`OVERALL`).
- `GROUPS` → um agrupamento por `CompetitionTeam.groupName`.
- `CONFERENCES` → um agrupamento por `conferenceName`.
- `DIVISIONS` → um agrupamento por `divisionName`.

### 5.2 `grouping_scope`
- `BY_GROUP` (**default**): só as tabelas por agrupamento.
- `OVERALL`: só a tabela geral (todos os times).
- `BOTH`: balde + geral.
- Em `groupingType=NONE`, só `OVERALL` faz sentido.

### 5.3 Rodadas consideradas
Somente `RoundType.REGULAR`. Playoffs (PLAYOFFS/WILDCARD/SEMIFINAL/FINAL) ficam fora.
Consequência: `GameLookup`/`FinishedGame` restringem a jogos de rodadas REGULAR (hoje soma todas) e
carregam `resultType` para aplicar `walkover_counts_in_tiebreakers`.

### 5.4 Pipeline de desempate
- Cadeia montada de `tiebreaker_criteria_json` como `Comparator.thenComparing` encadeado.
- Cada critério desempata apenas o subgrupo ainda empatado; ao separar parte, a cadeia **reinicia**
  para o remanescente (confirmado por regulamentos CBFA/BFA).
- `HEAD_TO_HEAD.multiWay`: `DISCARD` (**default**, descarta o critério com 3+ empatados) ou `MINI_TABLE`
  (só jogos entre os empatados).
- `RANDOM_DRAW`: usa as `position` anteriores (sorteia uma vez e persiste).

**Catálogo fechado do v1 (8):** `HEAD_TO_HEAD`, `POINT_DIFFERENTIAL` (saldo PF−PA), `POINTS_FOR`,
`POINTS_AGAINST` (menor sofrido), `GROUP_WIN_PERCENTAGE` (aproveitamento no grupo),
`STRENGTH_OF_VICTORY` (SOV), `STRENGTH_OF_SCHEDULE` (SOS), `RANDOM_DRAW`. O organizador escolhe o
subconjunto e a ordem. "Regra nova" = evoluir o catálogo no backend (o JSON já a acomoda).

### 5.5 SOS e SOV (definição NFL)
Com `PCT = (W + 0,5·T) / (W+L+T)` por equipe:
- **SOV** = PCT combinado dos adversários **vencidos**.
- **SOS** = PCT combinado de **todos** os adversários.
- Agregados **por confronto** (um rival enfrentado 2× conta 2×); "in all games" (toda a fase regular).
- **Duas passagens** (não iterativo): (1) PCT de cada equipe; (2) SOS/SOV a partir dos PCTs.
- A "força de tabela" da CBFA/BFA é equivalente a SOS.

### 5.6 W.O. na classificação
- `walkover_counts_in_tiebreakers` ligado (**default**): o placar padrão entra em PF/PA, saldo e PCT.
- Desligado: V-E-D e pontos **continuam** contando, mas PF/PA, saldo e SOS/SOV **ignoram** o jogo
  (segue a CBFA).

### 5.7 Execução
- **Assíncrona** via `@Async` (pool interno) — evento de resultado e disparo manual enfileiram; **sem
  tabela de job**.
- **Status** derivado por timestamps: `standings.updatedAt` vs. jogos finalizados mais recentes.
- **Idempotência:** o recálculo apaga e recria as linhas da competição; reexecutar é seguro.

---

## 6. Contratos REST (`flag_backend`)

Versionamento **MINOR**; OpenAPI (`/api-docs`) atualizado; erros em **RFC-7807** (`ApiException`).

### 6.1 `GET /api/v1/competitions/{id}/standings-rules` — **público**
```json
{
  "competitionId": "uuid",
  "rankingSystem": "WINS_DRAWS_LOSSES",
  "pointsPerWin": 3, "pointsPerDraw": 1, "pointsPerLoss": 0, "pointsPerWalkoverWin": 3,
  "walkoverScorePro": 49, "walkoverScoreAgainst": 0,
  "walkoverCountsInTiebreakers": true,
  "groupingScope": "BY_GROUP",
  "tiebreakerCriteria": [
    { "criterion": "HEAD_TO_HEAD", "multiWay": "DISCARD" },
    { "criterion": "POINT_DIFFERENTIAL" },
    { "criterion": "RANDOM_DRAW" }
  ],
  "updatedAt": "2026-..."
}
```
Sem regra cadastrada ⇒ devolve o **default sintetizado**.

### 6.2 `PUT /api/v1/competitions/{id}/standings-rules` — **dono ou ADMIN, só em DRAFT**
Body igual ao `GET` (sem `competitionId`/`updatedAt`). **200** com o recurso salvo.
Fora de DRAFT ⇒ **409**; não dono ⇒ **403**.

### 6.3 `POST /api/v1/competitions/{id}/standings/recalculate` — **ADMIN**
**202 Accepted** → `{ "competitionId": "uuid", "status": "ENQUEUED" }`.

### 6.4 `GET /api/v1/competitions/{id}/standings` — **público** (evolução do atual)
```json
{
  "competitionId": "uuid",
  "lastCalculatedAt": "2026-...",
  "recalculationPending": false,
  "groups": [
    { "name": "Grupo A", "entries": [
      { "position": 1, "teamId": "uuid", "teamName": "Indaiatuba Alpacas",
        "played": 5, "wins": 5, "draws": 0, "losses": 0,
        "pointsFor": 120, "pointsAgainst": 30, "pointsDifference": 90,
        "winPercentage": 100.0, "strengthOfVictory": 0.42, "strengthOfSchedule": 0.44 }
    ]}
  ]
}
```
A tabela geral entra em `groups` como o grupo **`OVERALL`**. `lastCalculatedAt` e
`recalculationPending` são **derivados** (sem rota de status dedicada).

### 6.5 Erros de domínio
Competição inexistente ⇒ 404; regras fora de DRAFT ⇒ 409; não dono ⇒ 403; payload inválido (critério
desconhecido) ⇒ 400.

---

## 7. Permissões

| Ação | Regra | Expressão |
|------|-------|-----------|
| Ler regras | **Público** | sem `@PreAuthorize` |
| Editar regras | Criador (ORGANIZER dono) ou ADMIN, **só em DRAFT** | `ADMIN_OR_ORGANIZER` + `assertManagedBy` + guarda DRAFT |
| Disparar recálculo | **Somente ADMIN** | `@PreAuthorize(SecurityExpressions.ADMIN)` |
| Registrar W.O. | `ADMIN_OR_COMMISSIONER` | idem resultado |

No admin web: card de disparo por `isAdminUser`; edição por `canEditCompetition` (só UX).

---

## 8. UI admin web (`flag_admin_web`)

Protótipo: **[assets/08-ux-regras-admin.md](assets/08-ux-regras-admin.md)**.

- Seção **"Regras de Classificação"** no fluxo **create/edit** (DRAFT), no padrão
  `competition_grouping_section.dart`; **somente leitura** no detail.
- Componentes: `SelectableCard` (modelo/escopo), `KicksterInput` (pontuação/W.O.),
  **reordenação dos critérios por drag & drop**, **parâmetros por critério em `KicksterMenuAnchor`**.
- Card de disparo (detail) só para `isAdminUser`: `KicksterStatusBadge`, `KicksterButton` primário,
  `showKicksterConfirmDialog`, estado "Processando" após o 202.
- O admin **não** exibe a tabela de classificação.

---

## 9. Fora do v1 / futuro (fog)

- Critérios avançados fora do catálogo (ex.: saldo de touchdowns).
- Auditoria/trace de quem disparou o cálculo.
- Correção/reabertura de resultado FINISHED (hoje sem transição de volta).
- Reconciliação do doc `architecture/roles-and-permissions.md` (desatualizado).

---

## 10. Rastreabilidade

| Ticket | Assunto |
|--------|---------|
| [01](issues/01-pesquisa-dominio-jogos.md) | Domínio de jogos (status, resultado, W.O.) |
| [02](issues/02-pesquisa-rbac.md) | RBAC — organizador dono vs ADMIN |
| [03](issues/03-regras-classificacao-v1.md) | Regras de classificação e critérios do v1 |
| [04](issues/04-modelo-dados-migracao.md) | Modelo de dados e migração |
| [05](issues/05-motor-escopo-grupos.md) | Escopo de cálculo e pipeline |
| [06](issues/06-contratos-rest.md) | Contratos REST e versionamento |
| [07](issues/07-permissoes-disparo.md) | Permissões |
| [08](issues/08-ux-telas-regras-admin.md) | UX no admin |
| [09](issues/09-wo-dominio-jogos.md) | W.O. no domínio de jogos |
| [10](issues/10-pesquisa-sos-sov.md) | Fórmulas de SOS e SOV |
