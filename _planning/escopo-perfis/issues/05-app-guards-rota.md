# ESC-05: App — guard de rotas por escopo com tela 403

> **Type:** feature
> **Status:** resolved
> **Effort:** escopo-perfis
> **Repo(s):** flag_admin_web
> **Blocks:** —
> **Blocked by:** ESC-03
> **Origin:** [_planning/escopo-perfis/plan.md](../plan.md) §2.2

## Contexto

`app_router.dart:22-72` tem gate apenas de autenticação/verificação de e-mail/status. Qualquer
autenticado digita `/users` ou `/approvals` e chega lá — o card escondido é cosmético.

## Escopo

1. `redirect` por escopo no router (após os gates existentes), consultando `AppScopeProvider`:
   - `ORGANIZATION`: nega `/organizations` (lista), `/users`, `/approvals`.
   - `INSTITUTION`: nega `/organizations`*, `/institutions` (lista), `/venues`, `/users`,
     `/approvals`; `/institutions/:id` só se `id == institutionId`.
   - `PLATFORM`: livre.
2. Tela "Acesso restrito" (403 amigável) com retorno à home.
3. IDs alheios: `/institutions/:id`, `/organizations/:id`, `/teams/:id` para escopo sem
   acesso redirecionam para a 403 (não 404 silencioso — evitar enumeração).

> *`/organizations/:id` para `ORGANIZATION` só se `id == organizationId` (rota "Minha Organização").

## Critérios de aceite

- [ ] URL direta a rota negada → 403 amigável (testado por cada escopo).
- [ ] ID alheio em detalhe → 403 (não vaza existência).
- [ ] Admin continua acessando tudo.
- [ ] `flutter analyze lib` sem issues novas; testes de redirect por escopo.

## Notas

- Guard do app é **defesa em profundidade**; o gate real é o 403 do backend (ESC-02).
