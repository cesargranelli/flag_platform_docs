# BACK-01: `POST /api/v1/institutions` retorna 500 (`ORA-17004`) — BLOQUEANTE

> **Type:** bug
> **Status:** open
> **Effort:** repos/backend
> **Repo(s):** flag_backend
> **Blocks:** —
> **Blocked by:** —
> **Origin:** `backend-issues.md` (ISSUE-001)

## Sintoma

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

## Evidência (log do container `flag-backend`)

```text
ERROR b.c.f.c.e.GlobalExceptionHandler : Unhandled exception on /api/v1/institutions:
PreparedStatementCallback; uncategorized SQLException for SQL
[SELECT organization_id FROM platform.institution_organizations WHERE institution_id = ?];
SQL state [99999]; error code [17004]; ORA-17004: Invalid column type
Caused by: java.sql.SQLException: ORA-17004: Invalid column type
```

## Causa raiz

O backend roda em **Oracle**, mas `InstitutionOrganizationRepository` é o **único código do backend com
`JdbcTemplate` cruto** (PostgreSQL-oriented) e passa `java.util.UUID` diretamente como parâmetro JDBC.
O driver Oracle rejeita o tipo (`ORA-17004`); em PostgreSQL o mesmo código funcionava.

Fluxo que falha no `create`:

1. `repository.save(e)` (JPA) grava a instituição;
2. `orgRepo.findOrganizationIds(e.getId())` executa o `JdbcTemplate` → `ORA-17004`;
3. exceção propaga → `@Transactional` **reverte a gravação** → `500`.

Isso também explica por que `GET /institutions` sem filtro responde `200 []`: a tabela está vazia e o
`stream().map(...)` nunca chama o `JdbcTemplate`.

## Arquivos envolvidos

- `institution/repository/InstitutionOrganizationRepository.java:18, 24, 29, 32`
  (`SELECT`/`DELETE`/`INSERT` com literal `platform.` e parâmetro `UUID`)
- `institution/service/InstitutionService.java` (`create`, `list(UUID)`, `setOrganizations`)
- Já mapeado como **Bloqueante** em `flag_backend/docs/database-portability-assessment.md:60`
  (recomendação registrada na linha 167: *“trocar `JdbcTemplate` por JPA”*).

## Correção proposta

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

## Critérios de aceite

- [ ] `POST /api/v1/institutions` → `201` e a linha persiste (visível em `GET /api/v1/institutions`).
- [ ] `GET /api/v1/institutions?organizationId={id}` → `200` (lista ou `[]`).
- [ ] `PUT /api/v1/institutions/{id}/organizations` → `200` e vínculo refletido na leitura.
- [ ] Nenhum `ORA-17004` nos logs após a correção.
- [ ] Deploy na VM OCI e revalidação de `/actuator/health`.

## Verificação rápida (pós-deploy)

```bash
curl -s -X POST "$FLAG_API_BASE_URL/api/v1/institutions" \
  -H "Authorization: Bearer $ID_TOKEN" -H "Content-Type: application/json" \
  -d '{"name":"Smoke Test","type":"CLUB"}'      # esperado: 201
curl -s "$FLAG_API_BASE_URL/api/v1/institutions?organizationId=<ORG_ID>"   # esperado: 200
```

## Desbloqueio associado

Com a BACK-01 corrigida, reexecutar no `america_platform_agent` (comando já pronto e idempotente —
deduplica por nome; nenhum dado parcial foi gravado):

```bash
uv run america-agent push-agremiacoes --org APFA --season 2026 --execute --with-logo
# 80 institutions + 80 afiliações (status PENDING) para a APFA
```

## Referências

- `flag_backend/docs/database-portability-assessment.md`
- `architecture/fluxo_filiacao_e_inscricao_agremiacoes.md`
