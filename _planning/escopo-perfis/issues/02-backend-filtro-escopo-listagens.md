# ESC-02: Backend — filtro de escopo por claim em todas as listagens

> **Type:** feature
> **Status:** resolved
> **Effort:** escopo-perfis
> **Repo(s):** flag_backend
> **Blocks:** ESC-06
> **Blocked by:** ESC-01
> **Origin:** [_planning/escopo-perfis/plan.md](../plan.md) §2.1

## Contexto

Todas as listagens são globais hoje. Com o vínculo disponível (ESC-01), o backend deve filtrar
**no servidor, antes de paginar**, a partir do usuário autenticado — nunca confiando em query
param enviado pelo cliente.

## Escopo

Filtro por escopo em `organizations`, `institutions`, `competitions`, `venues`, `teams`
(`/api/v1/teams` global), `persons`:

| Escopo | Comportamento |
|---|---|
| `PLATFORM` | inalterado (vê tudo) |
| `ORGANIZATION` | org: só a sua; institutions/competitions/venues: `organizationId` da sua; teams: das agremiações da sua org |
| `INSTITUTION` | institutions/teams: só os seus; organizations/venues: `403` |

- Implementar nas `*Specifications`/camada de serviço, integrando à paginação já existente
  (esforço `padronizacao-listagens`), **sem SQL nativo** e **sem migração Liquibase**.
- `403` com corpo claro em endpoints fora do escopo.

## Critérios de aceite

- [ ] Usuário `ORGANIZATION` não enxerga dados de outra organização em nenhuma listagem.
- [ ] Usuário `INSTITUTION` recebe `403` em `organizations`/`venues`; vê só as próprias
      institutions/teams.
- [ ] `PLATFORM` byte-compatível (regressão zero para admin; checklist de smoke).
- [ ] Filtro aplicado **antes** da paginação (`X-Total-Count` coerente com o escopo).
- [ ] `mvn -B clean verify` verde; smoke local com os 3 escopos nos endpoints da tabela.

## Notas

- Ponto de atenção: `GET /api/v1/enrollment-windows/open` (janelas de inscrição) é usado pela
  visão de agremiação — definir se é global-legítimo (competições abertas) ou filtrado.
