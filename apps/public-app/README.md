# Flag Public App — Documentação Técnica

> **Repositório:** [`flag_public_app`](https://github.com/cesargranelli/flag_public_app)  
> **Stack:** Flutter (Mobile/Web), Dart 3.x, MVVM, Design System Kickster  
> **Consumo:** REST API `/api/v1/` + Firestore CQRS Light (Real-time)

---

## 1. Visão Geral

O `flag_public_app` é a experiência do torcedor, atleta e fã do esporte. Proporciona acompanhamento em tempo real dos jogos, tabelas de classificação, estatísticas de atletas, chaveamentos e notícias dos campeonatos.

---

## 2. Documentos e Especificações

| Documento | Descrição |
|-----------|-----------|
| [Fluxo de Telas (Screen Flow)](screen-flow.md) | Mapeamento das telas públicas: Home, Competições, Partida ao Vivo, Classificação e Perfil do Atleta. |
| [Diagramas de Navegação](diagrama.md) | Diagramas conceituais da experiência do usuário público. |
| [Diagrama Draw.io (Fluxo)](assets/flag-public-app-flow.drawio) | Arquivo fonte do fluxo visual de navegação. |
| [Arquitetura Delegado e App Público](../../architecture/arquitetura_delegado_e_app_publico.md) | Integração da publicação de dados da mesa e delegado com o consumo no app público. |

---

## 3. Decisões Arquiteturais Relacionadas

- [ADR-001 — Nova Filosofia de Arquitetura](../../adr/ADR-001-nova-filosofia-arquitetura.md)
- [ADR-002 — CQRS Light com PostgreSQL e Firestore](../../adr/ADR-002-postgres-firestore-cqs.md)
- [ADR-011 — Arquitetura Flutter MVVM](../../adr/ADR-011-flutter-mvvm-architecture.md)
