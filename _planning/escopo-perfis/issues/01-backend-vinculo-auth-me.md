# ESC-01: Backend — expor vínculo no `/auth/me` + claims no token

> **Type:** feature
> **Status:** resolved
> **Effort:** escopo-perfis
> **Repo(s):** flag_backend
> **Blocks:** ESC-02, ESC-03
> **Blocked by:** —
> **Origin:** [_planning/escopo-perfis/plan.md](../plan.md) §2.1

## Contexto

O app não sabe a quem o usuário pertence: `AuthUser.organizationId`/`institutionId` existem no
modelo Flutter mas nunca são populados, porque o `GET /api/v1/auth/me` não os traz. Sem esse
vínculo, nenhum escopo é possível.

## Escopo

1. `GET /api/v1/auth/me` passa a retornar `organizationId?`, `institutionId?` e `scope`
   (`PLATFORM` | `ORGANIZATION` | `INSTITUTION`).
2. JWT emitido no login ganha claims de escopo (`scope`, `organizationId?`, `institutionId?`).
3. Regra de derivação: `ADMIN` → `PLATFORM`; possui `organizationId` → `ORGANIZATION`;
   possui `institutionId` → `INSTITUTION`; sem vínculo e não-admin → estado "pendente de
   vínculo" (comportamento claro, alinhado ao fluxo `/pending-approval` existente).
4. Aprovação/cadastro de usuário passa a gravar o vínculo (quem é dono de org/agremiação ao
   se registrar ou ser aprovado).

## Critérios de aceite

- [ ] `/auth/me` devolve `organizationId`/`institutionId`/`scope` para usuário vinculado.
- [ ] Token JWT contém as claims de escopo.
- [ ] Usuário `ADMIN` → `scope=PLATFORM` mesmo sem vínculo.
- [ ] Usuário sem vínculo não-admin recebe tratamento explícito (não vaza como `null` silencioso).
- [ ] `mvn -B clean verify` verde; smoke local do `/auth/me` com os 3 escopos.
- [ ] Retrocompatível: campos novos são aditivos (nenhum client existente quebra).

## Notas

- O modelo de usuário já tem os campos; não deve haver migração Liquibase (validar).
