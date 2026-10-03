# Mapa — Padronização das Listagens do Admin Web

> **Esforço:** `padronizacao-listagens` · **Prefixo:** `LIST` · **Status:** em execução
> **Repo afetado:** flag_admin_web (+ flag_backend para paginação/filtros server-side)
> **Plano/spec:** [plan.md](plan.md)

Padronização de todas as telas de listagem do `flag_admin_web`: **paginação no servidor**,
**filtros server-side** e **toolbar única** (busca + filtros + refresh + ação primária) abaixo do
breadcrumb, no Design System Kickster.

## Issues

| ID | Tipo | Título | Critérios de aceite | Status |
|----|------|--------|---------------------|--------|
| [LIST-01](issues/01-docs-agents-md.md) | docs | F3 — documentar paginação + toolbar única no `AGENTS.md` | `AGENTS.md` §3 com paginação e o modelo de toolbar/filtros; decisões registradas no hub | open |

## Fases

| Fase | Escopo | Status |
|------|--------|--------|
| F1 — Backend | 14 endpoints com `page`/`size` + `X-Total-Count` + filtros (v1.6.0) | ✅ |
| F2 — Admin web | `PagedListController` + `KicksterFilterBar` + `KicksterPagination`; 14 telas migradas | ✅ |
| F3 — Docs | Estender `AGENTS.md` §3 (paginação + toolbar) | ⏳ ([LIST-01](issues/01-docs-agents-md.md)) |

## Referências

- [plan.md](plan.md) — contrato completo (backend, admin web, inventário de endpoints) e status por fase.
