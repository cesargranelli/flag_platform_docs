# Flag Admin Web — Visão Geral & Integração

> **Repositório:** [`cesargranelli/flag_admin_web`](https://github.com/cesargranelli/flag_admin_web)  
> **Escopo:** Aplicação Web Administrativa (Flutter Web, Provider / ChangeNotifier MVVM, Kickster Design System)  
> **Documentação Específica do Repositório:** [`flag_admin_web/docs/`](https://github.com/cesargranelli/flag_admin_web/tree/develop/docs)

---

## 1. Papel no Ecossistema

O `flag_admin_web` é o painel de controle operacional e administrativo da plataforma. É utilizado por diretores de federações, gestores de ligas e administradores de agremiações para cadastros, gerenciamento de inscrições, agendamento de confrontos e auditoria.

### Regras Mandatórias de Arquitetura (AGENTS.md):
- **Separação Rígida Create vs Edit:** NUNCA unificar criação e edição no mesmo arquivo ou widget. Toda entidade possui telas e ViewModels explicitamente segregados (`*_create_screen.dart` e `*_edit_screen.dart`).
- **Padrão MVVM Flutter (ADR-011):** `domain/models/` → `data/services/` → `data/repositories/` (com cache de 30s) → `ui/<entidade>/view_models/` → `ui/<entidade>/widgets/`.
- **Design System Kickster:** Uso mandatório de componentes Kickster (`KicksterButton`, `KicksterDropdown`, `KicksterCard`, `KicksterCalendar`, `KicksterMenuAnchor`).

---

## 2. Documentações Específicas (No Repositório `flag_admin_web`)

As especificações de telas e decisões técnicas locais residem no repositório web:

| Documento Específico | Descrição |
|----------------------|-----------|
| [Análise de Otimização de API](https://github.com/cesargranelli/flag_admin_web/blob/develop/docs/analise-otimizacao-api.md) | Otimizações de payload, eliminação de consultas N+1 e política de cache em memória. |
| [Harmonização Visual de Formulários](https://github.com/cesargranelli/flag_admin_web/blob/develop/docs/modelo_visual_harmonizacao_forms.md) | Diretrizes para formulários de página única fluidos agrupados em cards limpos. |
| [Padronização Organizações & Agremiações](https://github.com/cesargranelli/flag_admin_web/blob/develop/docs/modelo_visual_organizacao_agremiacao.md) | Grade de 2 colunas, filtros com `KicksterDropdown` e listagem consistente. |
| [ADR Local: Atleta MVVM](https://github.com/cesargranelli/flag_admin_web/blob/develop/docs/adr/011-athlete-module-mvvm-migration.md) | Migração arquitetural específica do módulo de atletas. |

---

## 3. Diretrizes e Decisões Globais Aplicáveis

- [ADR-001 — Nova Filosofia de Arquitetura](../../adr/ADR-001-nova-filosofia-arquitetura.md)
- [ADR-006 — Team / Roster / Season Refactor](../../adr/ADR-006-team-roster-season-refactor.md)
- [ADR-010 — Autenticação Centralizada Firebase-First](../../adr/ADR-010-autenticacao-firebase-custom-claims.md)
- [ADR-011 — Padrão Arquitetural Flutter MVVM](../../adr/ADR-011-flutter-mvvm-architecture.md)
- [Design Tokens Oficiais (Kickster)](../../design/tokens.md)
- [Especificações Globais de Layout](../../design/layout-spec.md)
