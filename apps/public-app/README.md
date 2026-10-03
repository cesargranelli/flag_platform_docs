# Flag Public App — Visão Geral & Integração

> **Repositório:** [`cesargranelli/flag_public_app`](https://github.com/cesargranelli/flag_public_app)  
> **Escopo:** Aplicativo do Torcedor e Fan Experience (Flutter Mobile/Web, Riverpod / MVVM, Kickster)  
> **Documentação Específica do Repositório:** [`flag_public_app/docs/`](https://github.com/cesargranelli/flag_public_app/tree/blocos-estabilizacao/docs)

---

## 1. Papel no Ecossistema

O `flag_public_app` atende o público final: torcedores, atletas, pais, imprensa e fãs do esporte. Proporciona acompanhamento das competições em tempo real, tabelas de classificação, chaveamentos eliminatórios, estatísticas de jogadores e transmissão ao vivo lance a lance.

---

## 2. Documentações Específicas (No Repositório `flag_public_app`)

Os fluxos de telas públicas e especificações de navegação residem no repositório do app:

| Documento Específico | Descrição |
|----------------------|-----------|
| [Fluxo de Navegação (Screen Flow)](https://github.com/cesargranelli/flag_public_app/blob/blocos-estabilizacao/docs/screen-flow.md) | Mapeamento completo das telas: Home, Torneios, Jogo ao Vivo, Classificação e Perfil do Atleta. |
| [Diagrama de Telas Mermaid](https://github.com/cesargranelli/flag_public_app/blob/blocos-estabilizacao/docs/diagrama.md) | Versão interativa do fluxo em diagramas Mermaid. |
| [Diagrama Draw.io](https://github.com/cesargranelli/flag_public_app/blob/blocos-estabilizacao/docs/assets/flag-public-app-flow.drawio) | Diagrama fonte editável para abertura no Draw.io. |

---

## 3. Diretrizes e Decisões Globais Aplicáveis

- [ADR-001 — Nova Filosofia de Arquitetura](../../adr/ADR-001-nova-filosofia-arquitetura.md)
- [ADR-002 — Estratégia de Dados Híbrida (PostgreSQL + Firestore CQRS)](../../adr/ADR-002-postgres-firestore-cqs.md)
- [ADR-011 — Padrão Arquitetural Flutter MVVM](../../adr/ADR-011-flutter-mvvm-architecture.md)
- [Arquitetura Delegado e App Público](../../architecture/arquitetura_delegado_e_app_publico.md)
