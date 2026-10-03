# Plano — Padronização das listagens do Admin Web (paginação + filtros)

> **Status:** em execução. Decisão do produto: paginação **no servidor** (opção A),
> manter os filtros existentes + acrescentar os óbvios, e padronizar a barra de
> busca/filtros/ações em **uma única linha abaixo do breadcrumb**.

---

## 1. Objetivo

Todas as telas do `flag_admin_web` que exibem listas passam a ter:

1. **Paginação no servidor** (`page`/`size` + `X-Total-Count`) — hoje a maioria faz
   "carregar tudo" (`getAllPages`), anulando o `default 20` do backend.
2. **Filtros** no modelo da tela de **Organizações** (busca + dropdowns), aplicados
   **no servidor** (obrigatório: filtro no cliente com paginação só enxergaria a página atual).
3. **Toolbar única** (busca + filtros + refresh + ação primária) na **mesma linha
   abaixo do breadcrumb**, maximizando o espaço dos cards.

---

## 2. Contrato (padrão mandatório)

### 2.1 Backend (`flag_backend`)

- **Todo** endpoint de lista aceita:
  - `page` (>= 0, default `0`) e `size` (1..200, default `Pagination.DEFAULT_SIZE`);
  - `q` — busca textual (opcional), aplicada aos campos de nome da entidade;
  - filtros de domínio opcionais (ver inventário §3).
- Retorno: `List<Dto>` + header **`X-Total-Count`**.
- Serviço: `PagedResponse<XResponse> findAll(page, size, q, ...filtros)`, usando
  `Pagination.of(page, size)` e `Page<Entity>` (`PagedResponse` em
  `common/pagination/PagedResponse.java`).
- Controller: `response.setHeader("X-Total-Count", String.valueOf(result.total()))`.
- Ordenação padrão determinística (nome/criação) para paginação estável.

### 2.2 Admin web (`flag_admin_web`)

- **`PagedListController<T>`** (base reutilizável, `ChangeNotifier`):
  `page`, `size`, `total`, `isLoading`, `isRevalidating`, `errorMessage`, filtros,
  e comandos `load()`, `setPage()`, `setSize()`, `setQuery()`, `setFilter(k,v)`.
- **`KicksterFilterBar`**: `Row` única com `KicksterSearchField` (expande) +
  N `KicksterDropdown` (largura fixa) + `IconButton(Icons.refresh)` + ação primária (ex.: "Novo").
- **`KicksterPagination`**: anterior/próxima + "X–Y de N" + seletor de itens por página.
- Camada de dados: `ApiClient.getListWithTotal(path, {page, size, q, ...})` → `PagedResult<T>`.

### 2.3 UI

- Toolbar em **uma linha**, imediatamente **abaixo do breadcrumb**.
- Grid/lista ocupam todo o restante da altura.

---

## 3. Inventário de endpoints (14)

| # | Endpoint | Módulo | Pagina hoje? | Filtros |
|---|----------|--------|--------------|---------|
| 1 | `GET /api/v1/organizations` | organization | ✅ | `q`, `organizationType`, `includeDisabled` |
| 2 | `GET /api/v1/institutions` | institution | ❌ | `q`, `type` |
| 3 | `GET /api/v1/persons` | person | ✅ | `q`, `role` |
| 4 | `GET /api/v1/competitions` | competition | ✅ | `q`, `status`, `season`, `modality`, `organizationId`, `includeDisabled` |
| 5 | `GET /api/v1/venues` | venue | ✅ | `q`, `organizationId`, `state`, `city` |
| 6 | `GET /api/v1/auth/users` | user | ✅ | `q`, `role`, `status` |
| 7 | `GET /api/v1/auth/users/pending` | approval | ❌ | `q`, `role` |
| 8 | `GET /api/v1/competitions/{id}/games` | game | ❌ | `q`, `status`, `roundId`, `venueId` |
| 9 | `GET /api/v1/rounds/{id}/games` | game | ❌ | `q`, `status` |
| 10 | `GET /api/v1/competitions/{id}/rounds` | round | ❌ | `q`, `type` |
| 11 | `GET /api/v1/competitions/{id}/teams` | team | ✅ | `q`, `status`, `groupName` |
| 12 | `GET /api/v1/teams/{teamId}/roster` | roster | ❌ | `q`, `status` |
| 13 | `GET /api/v1/games/{gameId}/participants` | game | ❌ | `q`, `function` |
| 14 | `GET /api/v1/organizations/{orgId}/affiliations` | organization | ❌ | `q`, `season`, `status`, `type` |

---

## 4. Fases

### F1 — Backend (MINOR)
1. **Tracer bullet:** `organizations` — adicionar `q` + `organizationType` (já paginado).
2. Replicar o padrão nos 13 restantes (paginação onde falta + `q` + filtros do §3).
3. `mvn -B clean verify` + validar via `liquibase`/startup local.

### F2 — Admin web (MINOR)
1. **Tracer bullet:** tela de **Organizações** com `PagedListController` + `KicksterFilterBar`
   + `KicksterPagination` + toolbar única.
2. Migrar as demais telas por similaridade (Institution → Venue → Person → Competition →
   User/Approval → Rounds/Games → Roster/Participants/Affiliations/CompetitionTeams).
3. `flutter analyze` em 0 issues novos.

### F3 — Docs
1. Estender o `AGENTS.md` §3 com **paginação** + o modelo de toolbar/filtros.
2. Registrar decisões em `flag_platform_docs` (este diretório).

---

## 5. Fora de escopo

- Endpoints de dropdown (busca de `?fields=id,name`) — ver `docs/analise-otimizacao-api.md`, Fase 3.
- Refatoração dos `FutureProvider` duplicados (Game/Round) — tratado junto na F2, se trivial.

---

## 6. Status

### F1 — Backend ✅ (versão `1.6.0`)

Todos os **14 endpoints** passaram a aceitar `page`/`size` (default 20, máx 200) + header
`X-Total-Count` + filtros opcionais, implementados com `*Specifications` (Criteria) —
**sem SQL nativo** e retrocompatível (param `includeDisabled` etc. preservados).

Validado: `mvn -B clean compile` (BUILD SUCCESS) + smoke local em `8099`
(`organizations`, `institutions`, `competitions`, `venues` → `200` + `X-Total-Count`).

**Gaps de busca textual** (a entidade não guarda o nome; ele vive em outro módulo):

| Endpoint | `q` implementado | Gap |
|----------|------------------|-----|
| `games` | nome do time mandante/visitante via `TeamLookup.findTeamIdsByNameContaining` → `home/away IN (ids)` | — |
| `roster` | apenas campos locais (`nickname`, `positions`) | **nome da pessoa** (sem busca por nome no `PersonLookup`) |
| `participants` | — | sem `q` (entidade sem campo textual; nome vem de `person`) |
| `affiliations` | `season`, `requestedBy`, `reviewedBy`, `rejectionReason` | **nome da agremiação** (sem busca no `InstitutionLookup`); sem filtro `type` |

> **Decisão (produto):** ✅ **adicionar** `findIdsByNameContaining` aos Lookups de
> `person`/`institution` para habilitar esses `q` — **feito** (`PersonLookup`/`InstitutionLookup`),
> combinando `OR` (campos locais OU nome). Commit `978529d`.

### F2 — Admin web ✅

Pacote criado: `PagedListController<T>` (`lib/ui/core/controllers/`) +
`KicksterFilterBar` + `KicksterPagination` (`lib/ui/core/ui/`), exportados no `core_imports`.

As **14 telas** foram migradas para paginação no servidor + **toolbar única** abaixo do
breadcrumb; busca textual com *debounce* (`q`) e filtros de domínio server-side.
`Game`/`Round` deixaram de renderizar de `FutureProvider` e passaram a usar o ViewModel paginado.
Métodos antigos de service/repository (`getAll`/`list`) foram preservados para os dropdowns.

`flutter analyze lib` → **No issues found!** (os 21 `info` pré-existentes também foram zerados).

### F3 — Docs ⏳ (a fazer)
Atualizar o `AGENTS.md` §3 com paginação + a toolbar única.


