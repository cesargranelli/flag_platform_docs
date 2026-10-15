# ESC-06: App — listagens e ações condicionadas ao escopo

> **Type:** feature
> **Status:** resolved
> **Effort:** escopo-perfis
> **Repo(s):** flag_admin_web
> **Blocks:** —
> **Blocked by:** ESC-02, ESC-03
> **Origin:** [_planning/escopo-perfis/plan.md](../plan.md) §2.2, §3

## Contexto

Os ViewModels de instituição/venue/competição já recebem o param `organizationId` no service
mas nunca o enviam; competições têm filtro manual por dropdown. A escrita usa 3 padrões
ad-hoc (`isAdmin`, `canWrite = admin|organizer|manager`, `canWrite = user != null`).

## Escopo

1. **Listas**: ViewModels deixam de buscar lista global — para `ORGANIZATION` a listagem de
   agremiações/competições/sedes vem já filtrada do backend (ESC-02); remover o dropdown
   manual de filtro de organização em competições quando o escopo já define.
2. **Telas de detalhe próprio**: "Minha Organização" e "Minha Agremiação" apontam para o
   detalhe existente (`/organizations/:id`, `/institutions/:id`) com o id do escopo.
3. **Ações de escrita**: substituir os 3 padrões por um único `canWrite(papel)` =
   `ADMIN | EDITOR`; `VIEWER` só lê (botões ocultos/desabilitados + `EditRestrictionNote`
   quando fizer sentido). Aplicar em Organizations, Institutions, Competitions, Rounds, Games,
   Venues.
4. **Equipes**: `INSTITUTION` vê só as suas (`/institutions/:id/teams`); `ORGANIZATION` só as
   das suas agremiações; eliminando o uso de `GET /api/v1/teams` global para esses escopos.
5. **Cache**: chavear o cache em memória dos repositórios por escopo (ou reduzir TTL), para
   não servir dado de um escopo a outro na mesma sessão.
6. **Pessoas/Elenco**: definir escopo mínimo por organização/agremiação (ver plan §3; ajustar
   `person_service` que hoje é global).

## Critérios de aceite

- [ ] Nenhuma tela renderiza item fora do escopo do usuário (inspeção por escopo).
- [ ] Padrão de escrita único: `canWrite(papel)`; os 3 padrões ad-hoc removidos.
- [ ] Usuário `VIEWER` não tem ação de escrita exposta em nenhuma listagem.
- [ ] Cache não sirva dado entre escopos.
- [ ] `flutter analyze lib` sem issues novas.

## Notas

- `GET /api/v1/auth/users` e `/pending` são exclusivos de `PLATFORM` (guard da ESC-05 cobre).
