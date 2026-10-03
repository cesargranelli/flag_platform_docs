# LIST-01: Documentar paginação + toolbar única no `AGENTS.md`

> **Type:** docs
> **Status:** open
> **Effort:** padronizacao-listagens
> **Repo(s):** flag_admin_web
> **Blocks:** —
> **Blocked by:** —
> **Origin:** F3 de [_planning/padronizacao-listagens/plan.md](../plan.md)

## Contexto

F1 (backend) e F2 (admin web) foram concluídas: os 14 endpoints de lista passaram a paginar no
servidor com filtros, e as 14 telas do admin web adotaram `PagedListController` +
`KicksterFilterBar` + `KicksterPagination` com toolbar única abaixo do breadcrumb.

Falta apenas **registrar o padrão**.

## Critérios de aceite

- [ ] `AGENTS.md` §3 (Design System Kickster) estendido com **paginação** e o modelo de
      **toolbar única** (busca + filtros + refresh + ação primária).
- [ ] Decisões registradas no `flag_platform_docs` (este esforço).

## Referências

- [plan.md](../plan.md) — contrato e status das fases.
- `flag_admin_web/AGENTS.md` §3.
