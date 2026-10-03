# BACK-02: `404` mascarado como `500` (handlers ausentes)

> **Type:** bug
> **Status:** open
> **Effort:** repos/backend
> **Repo(s):** flag_backend
> **Blocks:** —
> **Blocked by:** —
> **Origin:** `backend-issues.md` (ISSUE-002)

## Sintoma

Entidades não encontradas geram `500 Internal Server Error` em vez de `404`, porque
`GlobalExceptionHandler` não possui handler para `EntityNotFoundException` / exceptions de domínio
(caindo no `@ExceptionHandler(Exception.class)`):

```text
Unhandled exception on /api/v1/institutions/00000000-.../affiliations: Agremiação não encontrada
Unhandled exception on /api/v1/institutions/00000000-...: Institution not found
```

## Correção sugerida

- Adicionar `@ExceptionHandler(EntityNotFoundException.class)` (e as exceptions de domínio
  equivalentes) devolvendo `HttpStatus.NOT_FOUND`.

## Critérios de aceite

- [ ] Entidade inexistente responde `404` com corpo de erro.
- [ ] Nenhum `500` originado de entidade não encontrada.

## Referências

- `config/e/GlobalExceptionHandler.java`
- Mesma família da [BACK-04](BACK-04-regra-negocio-500.md).
