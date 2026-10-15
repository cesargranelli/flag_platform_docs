# Mapa — Escopo por Perfil (Organização × Agremiação)

> **Esforço:** `escopo-perfis` · **Prefixo:** `ESC` · **Status:** resolvido (2026-10-15)
> **Repo(s) afetado(s):** flag_backend (vínculo + filtros), flag_admin_web (home, guards, listas)
> **Plano/spec:** [plan.md](plan.md) · **Wireframes:** [wireframes-home.md](wireframes-home.md)

Redesenho da autorização e da visibilidade do admin_web: separar **escopo** (o que enxergo —
`PLATFORM` / `ORGANIZATION` / `INSTITUTION`) de **papel** (o que faço — `ADMIN` / `EDITOR` /
`VIEWER`). O backend vira fonte única do vínculo (`/auth/me`) e do filtro de dados (claim do
token); o app consulta `AppScope` e nunca decide escopo sozinho.

## Problema

O admin_web foi construído sobre a premissa de que todo usuário veria os quadros completos.
Na prática, apenas `ADMIN` enxerga a plataforma inteira. Usuários de organização e de
agremiação veem listagens globais (vazamento de visão) e a home entrega cards sem sentido
(ex.: card "Organizações" para quem só tem uma). Não há guard de rota por perfil: qualquer
autenticado acessa `/users` e `/approvals` por URL.

## Issues

| ID | Tipo | Título | Critérios de aceite | Status |
|----|------|--------|---------------------|--------|
| [ESC-01](issues/01-backend-vinculo-auth-me.md) | feature | Backend: expor vínculo no `/auth/me` + claims no token | `GET /auth/me` retorna `organizationId`/`institutionId`/`scope`; JWT com claim de escopo | resolved |
| [ESC-02](issues/02-backend-filtro-escopo-listagens.md) | feature | Backend: filtro de escopo por claim em todas as listagens | Listagens filtradas por escopo; 403 para escopo sem acesso; escopo `PLATFORM` inalterado | resolved |
| [ESC-03](issues/03-app-modelo-escopo.md) | feature | App: modelo `AppScope` + persistência do vínculo | `AppScope` no domínio; `AuthUser`/`SessionManager` transportam org/agremiação; provider único | resolved |
| [ESC-04](issues/04-app-home-por-escopo.md) | feature | App: home e cabeçalho por escopo | Cards da home por perfil (matriz do plan §3); tarja de escopo no cabeçalho | resolved |
| [ESC-05](issues/05-app-guards-rota.md) | feature | App: guard de rotas por escopo com tela 403 | `redirect` por escopo em `app_router.dart`; tela "Acesso restrito" | resolved |
| [ESC-06](issues/06-app-listas-por-escopo.md) | feature | App: listagens e ações condicionadas ao escopo | ViewModels enviam filtro de escopo; ações criar/editar seguem papel; telas "Minha X" | resolved |
| [ESC-07](issues/07-docs-agents-md.md) | docs | Docs: registrar modelo de escopo × papel | `AGENTS.md` §escopo atualizado; decisões no hub | resolved |

## Fases

| Fase | Escopo | Depende de | Status |
|------|--------|------------|--------|
| F1 — Backend: identidade | ESC-01 (vínculo + claims) | — | ✅ |
| F2 — Backend: dados | ESC-02 (filtros + 403) | F1 | ✅ |
| F3 — App: fundação | ESC-03 (modelo `AppScope`) | F1 | ✅ |
| F4 — App: UI | ESC-04 (home), ESC-05 (guards), ESC-06 (listas) | F2, F3 | ✅ |
| F5 — Docs | ESC-07 | F4 | ✅ |

## Decisões já fechadas (2026-10-15)

1. Mapeamento: `ORGANIZER` → escopo `ORGANIZATION`; `MANAGER` → escopo `INSTITUTION`.
2. Vínculo: backend resolve no `/auth/me` (fonte única de verdade no servidor).
3. Filtro: backend filtra por claim do token; app não envia filtro de escopo por conta própria.
4. Home de organização: sem card "Organizações"; card "Minha Organização" → detalhe.
5. Indicador de escopo: tarja no cabeçalho, abaixo do greeting.
6. Competições da agremiação: um card único (inscritas + janelas abertas).
7. Guard: `redirect` por escopo com tela 403 amigável.

## Referências

- [plan.md](plan.md) — contrato completo (modelo, matriz de visibilidade, fluxo de dados, fases).
- `flag_admin_web/lib/domain/models/auth_user.dart` — vínculo declarado mas nunca populado.
- `flag_admin_web/lib/routing/app_router.dart` — sem guard por perfil hoje.
- [_planning/padronizacao-listagens](../padronizacao-listagens/plan.md) — paginação/filtros server-side, pré-requisito natural do filtro de escopo.
