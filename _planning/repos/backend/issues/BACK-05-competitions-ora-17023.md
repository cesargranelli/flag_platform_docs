# BACK-05: Leitura de `competitions` quebra com `ORA-17023` (`getBytes`) — BLOQUEANTE

> **Type:** bug
> **Status:** open
> **Effort:** repos/backend
> **Repo(s):** flag_backend
> **Blocks:** —
> **Blocked by:** —
> **Origin:** `backend-issues.md` (ISSUE-005)

## Sintoma

`GET /api/v1/competitions`, `GET /api/v1/competitions/{id}` e
`GET /api/v1/organizations/{id}/competitions` respondem **500**. Com a tabela vazia a listagem
respondia `200 []` — a falha só aparece quando existe ao menos 1 linha.

## Evidência (prod, 25/09 23:07–23:08Z)

```text
JpaSystemException: Could not extract column [8] from JDBC ResultSet [ORA-17023: Unsupported feature: getBytes]
```

em `CompetitionService.listAllPublic` (L104), `findEntityById` (L224) e `findByOrganizationId` (L91).

## Causa raiz

- DDL `competitions.grouping_config = CLOB` (`db/changelog/changesets/001-initial-schema.yaml` L334)
  + mapeamento `@JdbcTypeCode(SqlTypes.JSON)` em `CompetitionEntity.groupingConfig` (L65–67).
- O Hibernate 6 extrai a coluna via `ResultSet.getBytes()`, recurso que o Oracle JDBC não suporta
  para `CLOB`.

## Impacto

Escrita ok (`201` em 12/12), **leitura quebrada** — o painel não lista campeonatos e as 12 competições
criadas ficam invisíveis via API (e na UI).

## Correções possíveis

- (a) changeset trocando a coluna para `VARCHAR2(2000)`/`JSON`;
- (b) mapear o campo como `String` (remover `@JdbcTypeCode(JSON)` do `Map`) e converter na camada
  de serviço/DTO;
- (c) manter `CLOB` mapeado como `String` com `@JdbcTypeCode(SqlTypes.CLOB)` e adaptar o response.

## Critérios de aceite

- [ ] Com ≥1 linha, `GET /api/v1/competitions` → `200` e `groupingConfig` serializado.

## Contexto (competições criadas em 25/09 23:06Z)

- **APFA (4):** `c200ca8a-8a61-4b1f-ab56-0d26a139e0da`, `58ad836b-82dc-4107-a920-c7e9e511509b`,
  `fca54f6e-fbde-467f-b69d-57d43844e889`, `25af8e6d-ca0a-49df-a6b9-4ee78643d24e`
- **CBFA (8):** `e5f975c6-1728-4dd6-9985-1e02da6bfc75`, `0b410bd8-2cd3-4e5f-a147-69f84e3ac05e`,
  `c0071e10-c5c0-4010-aeed-c51506d12287`, `f35e204d-5675-4e6d-b74a-6ec88a21effd`,
  `ea972d91-7c26-44eb-9d7b-7e4835884fef`, `1973fcfa-de72-4b4f-867a-b2feddd7c368`,
  `0ff763aa-bee3-44ad-ab59-d4787a062a57`, `aead40fb-0e94-47c0-95e0-34e84a751d1b`
