# Permissões: quem configura regras e quem dispara o recálculo

Type: grilling
Status: resolved
Blocked by: 02

## Question

Definir quem pode o quê, alinhado ao RBAC real apurado na pesquisa.

Decidir:

- Quem **configura** as regras de classificação: o organizador dono da competição? o ADMIN?
- Quem **dispara** o recálculo: o card ADMIN-only no admin, ou o organizador dono?
- Como isso se expressa em `@PreAuthorize` / `SecurityExpressions` no backend e na visibilidade do
  `flag_admin_web` (o requisito original fala em card "apenas para o perfil admin").

## Answer

Decidido em grilling, apoiado na pesquisa *RBAC — organizador dono vs ADMIN*.

| Ação | Regra | Expressão |
|------|-------|-----------|
| Ler regras (`GET .../standings-rules`) | **Público** | sem `@PreAuthorize` (convenção `GET /competitions/**`) |
| Cadastrar/editar regras (`PUT .../standings-rules`) | **Criador (ORGANIZER dono) ou ADMIN**, **só em DRAFT** | `@PreAuthorize(SecurityExpressions.ADMIN_OR_ORGANIZER)` + `assertManagedBy(competitionId, email)` + guarda de status `DRAFT` |
| Disparar recálculo (`POST .../standings/recalculate`) | **Somente ADMIN** | `@PreAuthorize(SecurityExpressions.ADMIN)` |
| Recálculo automático (evento de resultado) | Sistema | sem permissão de usuário |

Detalhes:

- A edição das regras segue o mesmo modelo da edição da competição: **DRAFT-only**
  (`CompetitionNotEditableException`); ao publicar, as regras **congelam**.
- `assertManagedBy` = ADMIN **ou** `createdBy == userId` (legado sem `createdBy` → só ADMIN), conforme a
  pesquisa 02.
- No `flag_admin_web`: o card de disparo usa `isAdminUser(user)`; a edição de regras usa
  `canEditCompetition(user, competition)` — ambos só UX; o backend é a fonte da verdade.
- O disparo é exclusivo do ADMIN **independentemente de ser o criador** — o organizador dono configura
  e lê, mas **não** dispara.
