# BACK-06: `POST .../roster` (single) 500 "The given id must not be null" — BLOQUEANTE p/ carga

> **Type:** bug
> **Status:** open
> **Effort:** repos/backend
> **Repo(s):** flag_backend
> **Blocks:** —
> **Blocked by:** —
> **Origin:** `backend-issues.md` (ISSUE-006)

## Sintoma

`POST /api/v1/teams/{teamId}/roster` e `POST /api/v1/teams/{teamId}/competitions/{competitionId}/roster`
respondem **500** com `InvalidDataAccessApiUsageException: The given id must not be null`,
mesmo com `athleteId` válido no corpo (ex.: `{"athleteId":"ee7946f8-7446-4639-96b4-209be714bbcb"}`).

## Evidência (prod, 26/09 14:05–14:07Z)

- 2.367 POSTs, 100% `500`.
- `PersonService.findEntityById(L152) ← findPersonInfoById(L168) ← RosterService.toResponse`.
- Nenhum registro persistido (`GET /api/v1/teams/{id}/roster` = `[]` em todos os 66 times → rollback ok).

## Causa raiz

`roster/mapper/RosterEntryMapper.toEntity(AddRosterEntryRequest)` (MapStruct) mapeia **por nome de
propriedade**: o request tem `athleteId` e a entity tem `personId`
(`RosterEntryEntity.personId`, L32) → `personId` fica `null` e o `toResponse` (após o `save`)
chama `findById(null)`.

O caminho de **batch** não sofre do problema porque faz `entity.setPersonId(item.athleteId())`
explicitamente (`RosterService` L107) — só os adds single (L70 base e L210 por competição) quebram.

## Correção

- `@Mapping(source = "athleteId", target = "personId")` no mapper (ou
  `entity.setPersonId(request.athleteId())` em `addToBaseRoster`/`add`)
  + teste de integração cobrindo add single e batch.

## Impacto na carga

Bloqueia a Etapa 4 (elenco base e elenco por competição) da temporada APFA/CBFA 2026 — 2.367 vínculos
(2.206 atletas + 161 comissão técnica) pendentes.

## Arquivos

- `roster/mapper/RosterEntryMapper.java`
- `roster/service/RosterService.java` (L70, L210)
- `roster/dto/request/AddRosterEntryRequest.java` vs `roster/entity/RosterEntryEntity.java`
