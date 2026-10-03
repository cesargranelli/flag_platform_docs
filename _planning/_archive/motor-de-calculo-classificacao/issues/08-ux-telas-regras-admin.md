# UX das telas de regras no admin (Design System Kickster)

Type: prototype
Status: resolved
Blocked by: 03, 04

## Question

Desenhar o fluxo e a UX das telas de regras no `flag_admin_web`, no Design System Kickster.

Produzir (protótipo / wireframe):

- Onde vive a configuração: formulário fluido no create/edit da competição — respeitando a separação
  **create vs edit** (ADR do projeto) — e leitura/exibição no detail da competição.
- Componentes Kickster: `KicksterDropdown` para o modelo de classificação; **reordenação dos
  critérios** (drag & drop vs lista numerada — definir); `SelectableCard`/`SelectableChip` se houver
  escolha de escopo; `KicksterCalendar` não se aplica aqui.
- **Card ADMIN-only de disparo** no `competition_detail_screen.dart`, com
  `showKicksterConfirmDialog` e estado `isLoading` durante a chamada.
- Confirmar explicitamente: o admin **não** mostra a tabela de classificação — apenas as regras
  configuradas (a tabela é do app público).

## Answer

Protótipo (wireframe + mapeamento de componentes): **[assets/08-ux-regras-admin.md](../assets/08-ux-regras-admin.md)**.

Decisões de UX confirmadas na reação ao protótipo:

1. **Local:** seção "Regras de Classificação" no fluxo **create/edit** (DRAFT, congelada ao publicar),
   espelhando o padrão de `competition_grouping_section.dart` (`KicksterSectionTitle` + `Card`
   `surface`/borda `line`). No **detail**, a mesma seção em modo **somente leitura**.
2. **Componentes Kickster:**
   - Modelo de classificação e escopo: `SelectableCard` (mutuamente exclusivos).
   - Pontuação e placar de W.O.: `KicksterInput`.
   - "W.O. conta nos desempates": controle binário (switch/`SelectableChip`).
   - **Reordenação dos critérios: drag & drop** (`ReorderableListView` com handle).
   - **Parâmetros por critério em `KicksterMenuAnchor`** (⋮) — ex.: `multiWay` do Confronto Direto.
3. **Card de disparo (detail):** só para `isAdminUser`; `KicksterStatusBadge` (EM DIA / PENDENTE),
   `KicksterButton` primário, `showKicksterConfirmDialog`, e estado "Processando" após o `202`.
4. **Confirmado:** o admin **não** exibe a tabela de classificação — só as regras e o card de disparo;
   a tabela é do `flag_public_app`.
