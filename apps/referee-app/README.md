# Flag Referee App — Visão Geral & Integração

> **Repositório:** [`cesargranelli/flag_referee_app`](https://github.com/cesargranelli/flag_referee_app)  
> **Escopo:** Aplicativo Operacional de Mesa e Arbitragem (Flutter Mobile/Tablet, MVVM, Kickster)  
> **Documentação Específica do Repositório:** [`flag_referee_app/docs/`](https://github.com/cesargranelli/flag_referee_app/tree/develop/docs)

---

## 1. Papel no Ecossistema

O `flag_referee_app` é a ferramenta de campo utilizada pelos árbitros e mesários durante os jogos. Ele gerencia o fluxo em tempo real da partida: autenticação dos oficiais, check-in e conferência dos elencos, cronômetro, descidas (downs), faltas e pontuação lance a lance com suporte a tolerância a falhas de rede.

---

## 2. Documentações Específicas (No Repositório `flag_referee_app`)

O detalhamento da navegação da mesa e o planejamento de features operacionais residem no repositório do app:

| Documento Específico | Descrição |
|----------------------|-----------|
| [Fluxo de Telas (Navegação)](https://github.com/cesargranelli/flag_referee_app/blob/develop/docs/fluxo-de-telas.md) | Mapeamento detalhado das rotas, estados de tela e transições da mesa. |
| [Diagrama de Navegação Draw.io](https://github.com/cesargranelli/flag_referee_app/blob/develop/docs/fluxo-de-telas.drawio) | Arquivo editável com o fluxo visual completo da aplicação. |
| [Feature 2 — Operação ao Vivo](https://github.com/cesargranelli/flag_referee_app/blob/develop/docs/plan/feature2-spec.md) | Especificação técnica de registro de lances e modelo de dados de eventos. |
| [Planejamento de Entrega Feature 2](https://github.com/cesargranelli/flag_referee_app/blob/develop/docs/plan/planejamento-feature2.md) | Escopo incremental e requisitos de UI para a súmula digital. |
| [Plano de Execução de Eventos](https://github.com/cesargranelli/flag_referee_app/blob/develop/docs/plan/plano-execucao-eventos.md) | Roteiro de implementação das jogadas de Flag Football no Flutter. |
| [Auditoria Kickster](https://github.com/cesargranelli/flag_referee_app/blob/develop/docs/design/kickster-audit.md) | Avaliação e adequação visual aos componentes do design system. |

---

## 3. Diretrizes e Decisões Globais Aplicáveis

- [ADR-001 — Nova Filosofia de Arquitetura](../../adr/ADR-001-nova-filosofia-arquitetura.md)
- [ADR-010 — Autenticação Firebase-First com Custom Claims](../../adr/ADR-010-autenticacao-firebase-custom-claims.md)
- [ADR-011 — Padrão Arquitetural Flutter MVVM](../../adr/ADR-011-flutter-mvvm-architecture.md)
- [Catálogo Oficial de Eventos por Modalidade (5x5 a 11x11)](../../architecture/mapeamento-eventos-modalidades.md)
- [Características Funcionais do Referee App](../../architecture/flag_referee_app_caracteristicas.md)
- [Benchmark Comparativo FlagStats](../../research/comparativo-flagstats.md)
