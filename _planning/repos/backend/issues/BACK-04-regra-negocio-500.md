# BACK-04: Regra de negócio/estado inválido responde 500 em vez de 409/404

> **Type:** bug
> **Status:** open
> **Effort:** repos/backend
> **Repo(s):** flag_backend
> **Blocks:** —
> **Blocked by:** —
> **Origin:** `backend-issues.md` (ISSUE-004)

## Sintoma

`POST /api/v1/institutions/{id}/affiliations` sem janela aberta (ou com a janela fora das datas)
retorna **500 Internal Server Error** em vez de `400`/`409` com a mensagem de negócio.

## Evidência (prod, 25/09 16:48Z)

```text
GlobalExceptionHandler: Unhandled exception on
/api/v1/institutions/{id}/affiliations: O período de inscrições de filiação para a temporada 2026
está encerrado ou ainda não foi aberto por esta organização.
```

## Causa

`InstitutionAffiliationService.requestAffiliation` lança `IllegalStateException`;
`GlobalExceptionHandler` não a mapeia.

## Contexto

- Mesma família da [BACK-02](BACK-02-404-mascarado-500.md): `EntityNotFoundException` parcialmente
  mapeada; `IllegalStateException` e `IllegalArgumentException` não são.

## Critérios de aceite

- [ ] `409` para janela fechada/solicitação pendente duplicada.
- [ ] `404` para entidade inexistente.
- [ ] Nunca `500` por regra de negócio.

## Arquivos

- `affiliation/service/InstitutionAffiliationService.java` (`requestAffiliation`, ~L45)
- `config/e/GlobalExceptionHandler.java`
