# ESC-04: App — home e cabeçalho por escopo

> **Type:** feature
> **Status:** resolved
> **Effort:** escopo-perfis
> **Repo(s):** flag_admin_web
> **Blocks:** —
> **Blocked by:** ESC-03
> **Origin:** [_planning/escopo-perfis/plan.md](../plan.md) §3 (wireframes aprovados)

## Contexto

A home (`home_view_model.dart:26-65`) monta os mesmos cards para todos, escondendo apenas os
dois de admin. Para organização e agremiação os cards não têm sentido (ex.: "Organizações"
para quem só tem uma) ou vazam visão.

## Escopo

Home por escopo (ler `AppScopeProvider`):

| Escopo | Cards |
|---|---|
| `PLATFORM` | Organizações · Agremiações · Competições · Sedes · Aprovações · Usuários (como hoje) |
| `ORGANIZATION` | **Minha Organização** (detalhe, não lista) · Agremiações · Competições · Sedes — **sem** card Organizações/Aprovações/Usuários |
| `INSTITUTION` | **Minha Agremiação** (detalhe) · Equipes · Competições (card único: inscritas + janelas abertas) — sem Organizações/Sedes/Aprovações/Usuários |

Detalhes de UI (aprovados):

- **Tarja de escopo** no cabeçalho, abaixo do greeting: `▸ <nome da org/agremiação>`.
- Cards de lista mostram cota "**18/48**" (meus/total) para tornar o filtro visível.

## Critérios de aceite

- [ ] 3 homes distintas conforme a matriz acima; admin inalterado.
- [ ] Tarja presente nos escopos `ORGANIZATION` e `INSTITUTION`; ausente em `PLATFORM`.
- [ ] Cards levam às telas corretas (org → detalhe da org em `/organizations/:id`).
- [ ] `flutter analyze lib` sem issues novas; widget test da home por escopo.

## Notas

- A home não decide o filtro dos dados — só a navegação. O dado filtrado vem da F2 (backend).
