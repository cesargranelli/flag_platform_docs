# Flag Backend — Documentação Técnica

> **Repositório:** [`flag_backend`](https://github.com/cesargranelli/flag_backend)  
> **Stack:** Java 21, Spring Boot 4.1.0, PostgreSQL, JOOQ, Flyway, Maven, Firebase Admin SDK  
> **API:** RESTful sob `/api/v1/`

---

## 1. Visão Geral

O `flag_backend` é o núcleo transacional da Flag Platform (Modular Monolith), responsável pela integridade dos dados, regras de negócio desportivas, segurança e validação stateless de autenticação.

### Principais Responsabilidades:
- **Domínio Esportivo:** Gestão de Organizações, Agremiações/Instituições, Equipes, Temporadas, Elencos e Atletas.
- **Competições:** Gestão de Torneios, Fases, Grupos, Partidas e Classificações (Standings).
- **Operação de Jogo & Súmula:** Registro de pontuações, infrações, check-in e participantes de jogo (árbitros/delegados).
- **Autenticação:** Validação de ID Tokens do Firebase Auth e Custom Claims (`SUPER_ADMIN`, `ORG_ADMIN`, `MANAGER`, `USER`).

---

## 2. Modelos de Dados e Contratos

| Documento | Descrição |
|-----------|-----------|
| [Modelo de Dados — Persons, Athletes e Participantes](data-model-persons.md) | Estrutura unificada de pessoas físicas, vínculo com equipes (`team_roster`) e jogos (`game_participants`). |
| [Contratos para Flag Admin Web](flag-admin-web-data-model.md) | Especificação de endpoints consumidos pelo painel administrativo (atribuição de árbitros, delegações). |
| [Contratos para Flag Referee App](flag-referee-app-data-model.md) | Especificação de endpoints de partidas, check-in e eventos consumidos pelo app da mesa. |
| [Análise de Enums do Jogo](../../architecture/analise-completa-enums-backend.md) | Alinhamento de enums entre o backend Spring Boot e os aplicativos clientes. |

---

## 3. Decisões Arquiteturais Relacionadas

- [ADR-001 — Nova Filosofia de Arquitetura](../../adr/ADR-001-nova-filosofia-arquitetura.md)
- [ADR-002 — CQRS Light com PostgreSQL e Firestore](../../adr/ADR-002-postgres-firestore-cqs.md)
- [ADR-003 — Diagramas de Base de Dados](../../adr/ADR-003-diagramas-base-de-dados.md)
- [ADR-010 — Autenticação Firebase-First](../../adr/ADR-010-autenticacao-firebase-custom-claims.md)
- [ADR-012 — Migrations com jOOQ e Flyway](../../adr/ADR-012-migrations-jooq-java.md)
