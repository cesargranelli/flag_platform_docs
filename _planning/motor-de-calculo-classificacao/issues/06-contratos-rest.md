# Contratos REST e versionamento (regras, disparo, leitura)

Type: grilling
Status: resolved
Blocked by: 04, 05

## Question

Definir os contratos REST e o versionamento da API.

Decidir:

- `GET/PUT /api/v1/competitions/{id}/standings-rules` — leitura pelas telas do admin e escrita por
  quem tiver permissão (ver ticket de permissões).
- `POST /api/v1/competitions/{id}/standings/recalculate` — disparo manual; payload de resposta
  (status, `calculatedAt`, contagens de jogos/times/grupos).
- **Evolução do `GET /api/v1/competitions/{id}/standings` existente** para incluir grupos e PCT —
  este é o contrato que o `flag_public_app` vai consumir.
- Versionamento **MINOR** no `flag_backend` e atualização do OpenAPI (`/api-docs`).
- Erros de domínio (RFC-7807 / `ApiException`) para os novos casos (ex.: competição sem regras,
  sem jogos finalizados).

## Answer

Contrato REST confirmado em grilling. Versionamento **MINOR**; OpenAPI (`/api-docs`) atualizado;
erros em **RFC-7807** via `ApiException`.

### 1. `GET /api/v1/competitions/{id}/standings-rules` — **público**
```json
{
  "competitionId": "uuid",
  "rankingSystem": "WIN_PERCENTAGE",
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
Sem regra cadastrada ⇒ devolve o **default sintetizado** (`WIN_PERCENTAGE`, 3/1/0, W.O. 49×0, etc.).

### 2. `PUT /api/v1/competitions/{id}/standings-rules` — **dono ou ADMIN, só em DRAFT**
Body igual ao `GET` (sem `competitionId`/`updatedAt`). Retorna **200** com o recurso salvo. Fora de DRAFT
⇒ **409** (`CompetitionNotEditableException`); não dono ⇒ **403**
(`CompetitionNotOwnedByCreatorException`).

### 3. `POST /api/v1/competitions/{id}/standings/recalculate` — **ADMIN**
Retorna **202 Accepted**:
```json
{ "competitionId": "uuid", "status": "ENQUEUED" }
```

### 4. `GET /api/v1/competitions/{id}/standings` — **público** (evolução do atual)
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

- A tabela geral entra em `groups` como o grupo **`OVERALL`**.
- `lastCalculatedAt` e `recalculationPending` são **derivados** (standings.updatedAt vs. jogos
  finalizados) — sem rota de status dedicada (ticket 05).

### 5. Erros de domínio
- Competição inexistente ⇒ 404 (`CompetitionNotFoundException`).
- Regras fora de DRAFT ⇒ 409 (`CompetitionNotEditableException`).
- Não dono ⇒ 403 (`CompetitionNotOwnedByCreatorException`).
- Payload inválido (ex.: critério desconhecido no JSON) ⇒ 400.
