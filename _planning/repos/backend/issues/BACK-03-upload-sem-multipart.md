# BACK-03: `POST /api/v1/upload` sem multipart → `500`

> **Type:** bug
> **Status:** open
> **Effort:** repos/backend
> **Repo(s):** flag_backend
> **Blocks:** —
> **Blocked by:** —
> **Origin:** `backend-issues.md` (ISSUE-003)

## Sintoma

`Current request is not a multipart request` é respondido como `500`; o correto é `400 Bad Request`
(`MissingServletRequestPartException` não está no `GlobalExceptionHandler`).

## Prioridade

Baixa.

## Critérios de aceite

- [ ] Requisição sem multipart responde `400 Bad Request` com corpo de erro.

## Referências

- `config/e/GlobalExceptionHandler.java`
