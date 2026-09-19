# Flag Referee App — Documentação Técnica

> **Repositório:** [`flag_referee_app`](https://github.com/cesargranelli/flag_referee_app)  
> **Stack:** Flutter (Mobile/Tablet), Dart 3.x, Provider / ChangeNotifier (MVVM), Design System Kickster  
> **Consumo:** REST API `/api/v1/` + Local Cache / Offline-tolerant

---

## 1. Visão Geral

O `flag_referee_app` é a ferramenta operacional utilizada pela equipe de arbitragem e mesa durante as partidas. Permite o check-in biométrico/visual de atletas, escalação inicial e controle de cronômetro, faltas, descidas e pontuação lance a lance.

---

## 2. Documentos e Especificações

| Documento | Descrição |
|-----------|-----------|
| [Fluxo de Telas (Documento)](fluxo-de-telas.md) | Mapeamento textual completo da jornada da mesa: autenticação, seleção de partida, check-in e operação ao vivo. |
| [Fluxo de Telas (Diagrama Draw.io)](fluxo-de-telas.drawio) | Diagrama de arquitetura de navegação para abertura no Draw.io / VS Code. |
| [Características e Requisitos Funcionais](../../architecture/flag_referee_app_caracteristicas.md) | 10 características essenciais, papéis de mesa, regras de check-in e validações. |
| [Mapeamento de Eventos por Modalidade](../../architecture/mapeamento-eventos-modalidades.md) | Catálogo oficial de jogadas e eventos de Flag 5x5, 7x7, 8x8, 9x9 e Full Pads 11x11. |
| [Comparativo FlagStats](../../research/comparativo-flagstats.md) | Benchmarking com FlagStats para otimização da interface de súmula digital. |

---

## 3. Decisões Arquiteturais Relacionadas

- [ADR-001 — Nova Filosofia de Arquitetura](../../adr/ADR-001-nova-filosofia-arquitetura.md)
- [ADR-010 — Autenticação Firebase-First com Custom Claims](../../adr/ADR-010-autenticacao-firebase-custom-claims.md)
- [ADR-011 — Arquitetura Flutter MVVM](../../adr/ADR-011-flutter-mvvm-architecture.md)
