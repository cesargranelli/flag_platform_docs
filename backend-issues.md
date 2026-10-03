# Backend Issues — Pendências Críticas do `flag_backend`

> **Status:** Aberta
> **Data:** 25 de Setembro de 2026
> **Repositório afetado:** [`cesargranelli/flag_backend`](https://github.com/cesargranelli/flag_backend)
> **Origem:** agente `america_platform_agent` (push de 80 agremiações APFA/2026)
> **Ambiente de produção:** VM OCI `163.176.195.27:8080` → container `flag-backend` → Oracle ADB (`jdbc:oracle:thin:@devdb_high`, schema/usuário `platform`)

---

## ISSUE-001 — `POST /api/v1/institutions` retorna 500 (`ORA-17004`) — **BLOQUEANTE**

### Sintoma

Toda tentativa de **criar uma instituição** (clube/universidade) falha com `500 Internal Server Error`,
independente do payload (testado com payload mínimo `{"name":"X","type":"CLUB"}` e com payload completo).
A gravação é revertida: nenhuma linha parcial fica no banco.

O mesmo erro derruba o **filtro por organização** e a **definição de vínculos**:

| Endpoint | Resultado esperado | Resultado atual |
| :--- | :--- | :--- |
| `POST /api/v1/institutions` | `201` | **500** |
| `GET /api/v1/institutions?organizationId={id}` | `200 []` | **500** |
| `PUT /api/v1/institutions/{id}/organizations` | `200` | **500** (mesmo código) |
| `GET /api/v1/institutions` (sem filtro) | `200 []` | `200` ✅ (não passa pelo JDBC) |
| `GET /api/v1/organizations/{id}/clubs` | `200 []` | `200` ✅ (JPA puro) |

> **Impacto colateral:** o painel administrativo (`flag_admin_web`) também não consegue criar/editar
> vínculo instituição↔organização em produção — não é um problema exclusivo do agente de importação.

### Evidência (log do container `flag-backend`)

```text
ERROR b.c.f.c.e.GlobalExceptionHandler : Unhandled exception on /api/v1/institutions:
PreparedStatementCallback; uncategorized SQLException for SQL
[SELECT organization_id FROM platform.institution_organizations WHERE institution_id = ?];
SQL state [99999]; error code [17004]; ORA-17004: Invalid column type
Caused by: java.sql.SQLException: ORA-17004: Invalid column type
```

### Causa raiz

O backend roda em **Oracle**, mas `InstitutionOrganizationRepository` é o **único código do backend com
`JdbcTemplate` cruto** (PostgreSQL-oriented) e passa `java.util.UUID` diretamente como parâmetro JDBC.
O driver Oracle rejeita o tipo (`ORA-17004`); em PostgreSQL o mesmo código funcionava.

Fluxo que falha no `create`:

1. `repository.save(e)` (JPA) grava a instituição;
2. `orgRepo.findOrganizationIds(e.getId())` executa o `JdbcTemplate` → `ORA-17004`;
3. exceção propaga → `@Transactional` **reverte a gravação** → `500`.

Isso também explica por que `GET /institutions` sem filtro responde `200 []`: a tabela está vazia e o
`stream().map(...)` nunca chama o `JdbcTemplate`.

### Arquivos envolvidos

- `institution/repository/InstitutionOrganizationRepository.java:18, 24, 29, 32`
  (`SELECT`/`DELETE`/`INSERT` com literal `platform.` e parâmetro `UUID`)
- `institution/service/InstitutionService.java` (`create`, `list(UUID)`, `setOrganizations`)
- Já mapeado como **Bloqueante** em `flag_backend/docs/database-portability-assessment.md:60`
  (recomendação registrada na linha 167: *“trocar `JdbcTemplate` por JPA”*).

### Correção proposta

**Opção A — recomendada (alinha com o portability assessment):** substituir o `JdbcTemplate` por JPA.

- Criar `InstitutionOrganizationEntity` (PK composta `institution_id` + `organization_id`) com
  `@Table(name = "institution_organizations", schema = "platform")` e repositório Spring Data;
- Reescrever `findOrganizationIds`, `findInstitutionIds` e `setOrganizations` como `@Query`/derived queries;
- Remover o `JdbcTemplate` (único uso no código inteiro do backend).

**Opção B — corretivo mínimo:** manter o `JdbcTemplate` e converter o binding para o tipo aceito pelo Oracle.

- Consultar o tipo real da coluna (`ALL_TAB_COLUMNS`) antes de escolher:
  - `RAW(16)` → bindar `uuid.toString()` convertido para `byte[]` / `oracle.jdbc.OracleTypes.RAW`;
  - `VARCHAR2(36)` → bindar `uuid.toString()`.
- Cobrir os 4 pontos (2 `SELECT`, 1 `DELETE`, 1 `INSERT`) e adicionar teste de integração cobrindo
  `create` + `list?organizationId=` + `setOrganizations`.

### Critérios de aceite

- [ ] `POST /api/v1/institutions` → `201` e a linha persiste (visível em `GET /api/v1/institutions`).
- [ ] `GET /api/v1/institutions?organizationId={id}` → `200` (lista ou `[]`).
- [ ] `PUT /api/v1/institutions/{id}/organizations` → `200` e vínculo refletido na leitura.
- [ ] Nenhum `ORA-17004` nos logs após a correção.
- [ ] Deploy na VM OCI e revalidação de `/actuator/health`.

### Verificação rápida (pós-deploy)

```bash
curl -s -X POST "$FLAG_API_BASE_URL/api/v1/institutions" \
  -H "Authorization: Bearer $ID_TOKEN" -H "Content-Type: application/json" \
  -d '{"name":"Smoke Test","type":"CLUB"}'      # esperado: 201
curl -s "$FLAG_API_BASE_URL/api/v1/institutions?organizationId=<ORG_ID>"   # esperado: 200
```

### Desbloqueio associado

Com a ISSUE-001 corrigida, reexecutar no `america_platform_agent` (comando já pronto e idempotente —
deduplica por nome; nenhum dado parcial foi gravado):

```bash
uv run america-agent push-agremiacoes --org APFA --season 2026 --execute --with-logo
# 80 institutions + 80 afiliações (status PENDING) para a APFA
```

---

## ISSUE-002 — `404` mascarado como `500` (handlers ausentes)

Entidades não encontradas geram `500 Internal Server Error` em vez de `404`, porque
`GlobalExceptionHandler` não possui handler para `EntityNotFoundException` / exceptions de domínio
(caindo no `@ExceptionHandler(Exception.class)`):

```text
Unhandled exception on /api/v1/institutions/00000000-.../affiliations: Agremiação não encontrada
Unhandled exception on /api/v1/institutions/00000000-...: Institution not found
```

- **Esperado:** `404` com corpo de erro.
- **Correção sugerida:** adicionar `@ExceptionHandler(EntityNotFoundException.class)` (e as exceptions
  de domínio equivalentes) devolvendo `HttpStatus.NOT_FOUND`.

---

## ISSUE-003 — `POST /api/v1/upload` sem multipart → `500`

`Current request is not a multipart request` é respondido como `500`; o correto é `400 Bad Request`
(`MissingServletRequestPartException` não está no `GlobalExceptionHandler`). Baixa prioridade.

---

## ISSUE-004 — Regra de negócio/estado inválido responde 500 em vez de 409/404 (alta)

- **Sintoma:** `POST /api/v1/institutions/{id}/affiliations` sem janela aberta (ou com a janela
  fora das datas) retorna **500 Internal Server Error** em vez de `400`/`409` com a mensagem de negócio.
- **Evidência (prod, 25/09 16:48Z):** `GlobalExceptionHandler: Unhandled exception on
  /api/v1/institutions/{id}/affiliations: O período de inscrições de filiação para a temporada 2026
  está encerrado ou ainda não foi aberto por esta organização.`
- **Causa:** `InstitutionAffiliationService.requestAffiliation` lança `IllegalStateException`;
  `GlobalExceptionHandler` não a mapeia.
- **Mesma família da ISSUE-002:** `EntityNotFoundException` parcialmente mapeada;
  `IllegalStateException` e `IllegalArgumentException` não são.
- **Aceite:** `409` para janela fechada/solicitação pendente duplicada; `404` para entidade
  inexistente; nunca `500` por regra de negócio.
- **Arquivos:** `affiliation/service/InstitutionAffiliationService.java` (`requestAffiliation`, ~L45),
  `config/e/GlobalExceptionHandler.java`.

---

## ISSUE-005 — Leitura de `competitions` quebra com ORA-17023 (`getBytes`) — **BLOQUEANTE**

- **Sintoma:** `GET /api/v1/competitions`, `GET /api/v1/competitions/{id}` e
  `GET /api/v1/organizations/{id}/competitions` respondem **500**. Com a tabela vazia a listagem
  respondia `200 []` — a falha só aparece quando existe ao menos 1 linha.
- **Evidência (prod, 25/09 23:07–23:08Z):**
  `JpaSystemException: Could not extract column [8] from JDBC ResultSet [ORA-17023: Unsupported feature: getBytes]`
  em `CompetitionService.listAllPublic` (L104), `findEntityById` (L224) e `findByOrganizationId` (L91).
- **Causa raiz:** DDL `competitions.grouping_config = CLOB`
  (`db/changelog/changesets/001-initial-schema.yaml` L334) + mapeamento
  `@JdbcTypeCode(SqlTypes.JSON)` em `CompetitionEntity.groupingConfig` (L65–67).
  O Hibernate 6 extrai a coluna via `ResultSet.getBytes()`, recurso que o Oracle JDBC não suporta
  para `CLOB`.
- **Impacto:** escrita ok (`201` em 12/12), **leitura quebrada** — o painel não lista campeonatos e
  as 12 competições criadas ficam invisíveis via API (e na UI).
- **Correções possíveis:** (a) changeset trocando a coluna para `VARCHAR2(2000)`/`JSON`;
  (b) mapear o campo como `String` (remover `@JdbcTypeCode(JSON)` do `Map`) e converter na camada
  de serviço/DTO; (c) manter `CLOB` mapeado como `String` com `@JdbcTypeCode(SqlTypes.CLOB)` e
  adaptar o response.
- **Aceite:** com ≥1 linha, `GET /api/v1/competitions` → `200` e `groupingConfig` serializado.
- **Contexto (criadas em 25/09 23:06Z):**
  - **APFA (4):** `c200ca8a-8a61-4b1f-ab56-0d26a139e0da`, `58ad836b-82dc-4107-a920-c7e9e511509b`,
    `fca54f6e-fbde-467f-b69d-57d43844e889`, `25af8e6d-ca0a-49df-a6b9-4ee78643d24e`
  - **CBFA (8):** `e5f975c6-1728-4dd6-9985-1e02da6bfc75`, `0b410bd8-2cd3-4e5f-a147-69f84e3ac05e`,
    `c0071e10-c5c0-4010-aeed-c51506d12287`, `f35e204d-5675-4e6d-b74a-6ec88a21effd`,
    `ea972d91-7c26-44eb-9d7b-7e4835884fef`, `1973fcfa-de72-4b4f-867a-b2feddd7c368`,
    `0ff763aa-bee3-44ad-ab59-d4787a062a57`, `aead40fb-0e94-47c0-95e0-34e84a751d1b`

---


---

## ISSUE-006 — `POST .../roster` (single) 500 "The given id must not be null" — **BLOQUEANTE p/ carga**

- **Sintoma:** `POST /api/v1/teams/{teamId}/roster` e `POST /api/v1/teams/{teamId}/competitions/{competitionId}/roster`
  respondem **500** com `InvalidDataAccessApiUsageException: The given id must not be null`,
  mesmo com `athleteId` válido no corpo (ex.: `{"athleteId":"ee7946f8-7446-4639-96b4-209be714bbcb"}`).
- **Evidência (prod, 26/09 14:05–14:07Z):** 2.367 POSTs, 100% `500`;
  `PersonService.findEntityById(L152) ← findPersonInfoById(L168) ← RosterService.toResponse`;
  nenhum registro persistido (`GET /api/v1/teams/{id}/roster` = `[]` em todos os 66 times → rollback ok).
- **Causa raiz:** `roster/mapper/RosterEntryMapper.toEntity(AddRosterEntryRequest)` (MapStruct)
  mapeia **por nome de propriedade**: o request tem `athleteId` e a entity tem `personId`
  (`RosterEntryEntity.personId`, L32) → `personId` fica `null` e o `toResponse` (após o `save`)
  chama `findById(null)`.
  O caminho de **batch** não sofre do problema porque faz `entity.setPersonId(item.athleteId())`
  explicitamente (`RosterService` L107) — só os adds single (L70 base e L210 por competição) quebram.
- **Correção:** `@Mapping(source = "athleteId", target = "personId")` no mapper
  (ou `entity.setPersonId(request.athleteId())` em `addToBaseRoster`/`add`) + teste de integração
  cobrindo add single e batch.
- **Impacto na carga:** bloqueia a Etapa 4 (elenco base e elenco por competição) da temporada
  APFA/CBFA 2026 — 2.367 vínculos (2.206 atletas + 161 comissão técnica) pendentes.
- **Arquivos:** `roster/mapper/RosterEntryMapper.java`,
  `roster/service/RosterService.java` (L70, L210),
  `roster/dto/request/AddRosterEntryRequest.java` vs `roster/entity/RosterEntryEntity.java`.

---
## Referências

- `flag_backend/docs/database-portability-assessment.md` — item 3 (`JdbcTemplate` + `platform.` literal) classificado como **Bloqueante**.
- `flag_platform_docs/architecture/fluxo_filiacao_e_inscricao_agremiacoes.md` — fluxo de filiação que depende deste vínculo.
- Logs da VM: `docker logs flag-backend` (host `163.176.195.27`, usuário `opc`).
