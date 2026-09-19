# Flag Admin Web — Documentação Técnica

> **Repositório:** [`flag_admin_web`](https://github.com/cesargranelli/flag_admin_web)  
> **Stack:** Flutter Web, Dart 3.x, Provider / ChangeNotifier (MVVM), Design System Kickster  
> **Consumo:** REST API `/api/v1/`

---

## 1. Visão Geral

O `flag_admin_web` é a aplicação web administrativa da Flag Platform, voltada para gestores de federações, ligas, agremiações e organizadores de campeonatos.

### Diretrizes Arquiteturais Mandatórias:
- **Separação Rígida Create vs Edit:** Arquivos e telas explicitamente separados (`*_create_screen.dart` e `*_edit_screen.dart`). Nunca unificar em componente híbrido.
- **Padrão MVVM Flutter (ADR-011):** Camadas bem definidas: `domain/models/`, `data/services/`, `data/repositories/` (com cache transparente em memória TTL 30s) e `ui/<entidade>/view_models/`.
- **Design System Kickster:** Uso estrito de `KicksterButton`, `KicksterDropdown` (largura ~240px em filtros), `KicksterSearchField`, `KicksterCard`, `KicksterMenuAnchor`, `KicksterCalendar` (nunca `showDatePicker`), `SelectableCard` e `SelectableChip`.

---

## 2. Documentos e Especificações

| Documento | Descrição |
|-----------|-----------|
| [Análise de Otimização de API](analise-otimizacao-api.md) | Diagnóstico de latência, estratégias de payload, cache em memória e redução de overhead de rede. |
| [Harmonização de Formulários](../../design/modelo_visual_harmonizacao_forms.md) | Especificação visual para formulários fluidos em página única agrupados em cards limpos. |
| [Padronização de Agremiações & Organizações](../../design/modelo_visual_organizacao_agremiacao.md) | Alinhamento visual da grade de listagem, busca, filtros e cards. |
| [Design Tokens Oficiais](../../design/tokens.md) | Cores, tipografia DM Sans, elevações e espaçamentos do Kickster. |

---

## 3. Decisões Arquiteturais Relacionadas

- [ADR-001 — Nova Filosofia de Arquitetura](../../adr/ADR-001-nova-filosofia-arquitetura.md)
- [ADR-006 — Team / Roster / Season Refactor](../../adr/ADR-006-team-roster-season-refactor.md)
- [ADR-010 — Autenticação Firebase-First com Custom Claims](../../adr/ADR-010-autenticacao-firebase-custom-claims.md)
- [ADR-011 — Arquitetura Flutter MVVM](../../adr/ADR-011-flutter-mvvm-architecture.md)
