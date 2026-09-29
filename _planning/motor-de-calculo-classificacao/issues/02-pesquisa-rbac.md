# Pesquisa: RBAC — organizador dono vs ADMIN (backend + admin_web)

Type: research
Status: resolved

## Question

Quais são os papéis reais e o mecanismo de autorização (RBAC) que governam *"configurar regras de uma
competição"* e *"disparar o recálculo"*?

Investigar e registrar:

- O `UserRole` **real** no backend (`common/enums`) e as expressões em
  `SecurityExpressions` / `SecurityConfig`. O doc `architecture/roles-and-permissions.md` parece
  desatualizado (cita `SUPER_ADMIN`/`ORG_ADMIN` enquanto o enum tem `ADMIN`/`ORGANIZER`/`MANAGER`/
  `REFEREE`/`COMMISSIONER`/`FAN`).
- Como "organizador dono da competição" é verificado hoje (ex.: `CompetitionNotOwnedByCreatorException`,
  campos de dono/organização na entidade).
- Como o `flag_admin_web` decide visibilidade por papel (`AuthViewModel` / `isAdmin`) — o card
  ADMIN-only do disparo depende disso.
- Se o disparo deve ser ADMIN global ou o ORGANIZER dono — trazer as evidências, não a decisão.

## Answer

Apurado no `flag_backend` (segurança) e no `flag_admin_web` (espelho de UX).

### Papéis reais (backend)
`UserRole` (`common/enums/UserRole.java`): **ADMIN, ORGANIZER, COMMISSIONER, REFEREE, MANAGER, FAN**.
O doc `architecture/roles-and-permissions.md` está **desatualizado**: cita `SUPER_ADMIN`/`ORG_ADMIN`/
`USER` e dá a `standings` uma FK `category_id` — nada disso existe no código (a tabela usa
`competition_id` e não há SUPER_ADMIN).

### Autorização em duas camadas

**1) Filtro (`config/SecurityConfig.java`):** `GET` em `/api/v1/competitions/**`, `/api/v1/standings/**`,
`/api/v1/games/**` etc. é `permitAll` → **leitura pública**. Todo o resto exige autenticação
(`.anyRequest().authenticated()`). Ou seja, um `GET .../standings-rules` cairia em rota
**pública por padrão**, salvo se receber `@PreAuthorize`.

**2) Método (`@PreAuthorize` + `common/security/SecurityExpressions.java`):**

| Expressão | Regra | Uso |
|---|---|---|
| `ADMIN_OR_ORGANIZER` | `hasAuthority('STATUS_ACTIVE') and hasAnyRole('ADMIN','ORGANIZER')` | escrita de gestão: criar/editar/desativar/encerrar competição, criação de jogos, conferências/divisões |
| `ADMIN_OR_COMMISSIONER` | `...hasAnyRole('ADMIN','COMMISSIONER','REFEREE')` | operação de jogo: status, resultado, score events |
| `ADMIN` | `...hasRole('ADMIN')` | exclusivo: reativar competição (`CompetitionApi.java:78`), gestão de usuários |
| `MANAGER`, `INSTITUTION_WRITE`, `ACTIVE` | — | outros contextos |

Todas exigem `STATUS_ACTIVE`.

### Ownership ("organizador dono")
Regra de **serviço**, não de role: `CompetitionService.assertManagedBy(competitionId, email)`
(`CompetitionService.java:179`):

- Se `userLookup.isAdminByEmail(email)` (`AuthService.java:232`, checa `role == ADMIN`) → passa.
- Senão exige `entity.getCreatedBy() == userLookup.findUserIdByEmail(email)`; legado sem `created_by`
  → só ADMIN. Falha = `CompetitionNotOwnedByCreatorException` (403).

`createdBy` é o **UUID do usuário criador** (setado no `create`, `CompetitionService.java:70`), **não**
a organização. Aplicado no `update`/`deactivate`/`finish` da competição e nos módulos filhos
(game, conference, division) via `competitionLookup.assertManagedBy`.

**Ponto importante:** o `registerResult` (`POST /games/{id}/result`) usa `ADMIN_OR_COMMISSIONER`
**sem** `assertManagedBy` — o comissário/árbitro opera o jogo sem ser o dono da competição.

### Espelho no admin web
- `lib/domain/enums/user_role.dart` espelha exatamente o enum do backend.
- `lib/domain/competition_permissions.dart`: `isAdminUser(user) => user?.role == UserRole.admin`;
  `canEditCompetition(user, competition)`: ADMIN sempre; senão `competition.createdBy == user.id`;
  legado sem `createdBy` → ADMIN. É só UX — "o backend é a fonte da verdade".
- `AuthUser`/`AuthController` carregam `role`, `status`, `organizationId`; **não existe** ainda um
  helper tipo `canRecalculateStandings` — o card do disparo usaria `isAdminUser`.

### Evidências para a decisão (não a decisão)
- **Configurar regras** = escrita de gestão da competição. O padrão vigente é
  `ADMIN_OR_ORGANIZER` + `assertManagedBy` (criador **ou** ADMIN) — o ORGANIZER **dono** já é quem
  gerencia a competição.
- **Disparar o recálculo** = ação sobre a competição. O requisito pede card "apenas ADMIN", mas o
  modelo real oferece "criador (ORGANIZER) ou ADMIN"; o recálculo automático hoje roda independente
  de permissão.
- **ADMIN** aqui é "Administrador da plataforma"; não há SUPER_ADMIN.
