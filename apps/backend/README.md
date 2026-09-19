# Flag Backend — Visão Geral & Integração

> **Repositório:** [`cesargranelli/flag_backend`](https://github.com/cesargranelli/flag_backend)  
> **Escopo:** Backend Modular Monolith (Spring Boot 4.1.0, Java 21, PostgreSQL 16, jOOQ, Flyway)  
> **Documentação Específica do Repositório:** [`flag_backend/docs/`](https://github.com/cesargranelli/flag_backend/tree/develop/docs)

---

## 1. Papel no Ecossistema

O `flag_backend` é o núcleo transacional e guardião das regras de negócio da Flag Platform. Ele é a única fonte de verdade para escritas (ACID) e expõe a API RESTful `/api/v1/` consumida pelos clientes administrativos e operacionais.

### Responsabilidades Centrais:
- **Gestão de Domínio:** Organizações, Agremiações/Instituições, Equipes, Temporadas e Atletas.
- **Competições:** Torneios, Fases, Grupos, Chaveamentos, Partidas e Classificações.
- **Operação de Partida:** Registro oficial de súmula, check-in e validação de participantes.
- **Segurança & Claims:** Validação stateless de ID Tokens emitidos pelo Firebase Auth.

---

## 2. Documentações Específicas (No Repositório `flag_backend`)

As especificações detalhadas de esquemas e contratos residem diretamente no repositório do backend:

| Documento Específico | Descrição |
|----------------------|-----------|
| [Modelo de Dados — Persons & Athletes](https://github.com/cesargranelli/flag_backend/blob/develop/docs/data-model-persons.md) | Estrutura unificada de pessoas físicas, vínculo com equipes (`team_roster`) e partidas (`game_participants`). |
| [Contratos para Flag Admin Web](https://github.com/cesargranelli/flag_backend/blob/develop/docs/flag-admin-web-data-model.md) | Especificação de endpoints de gestão de participantes, delegações e funções. |
| [Contratos para Flag Referee App](https://github.com/cesargranelli/flag_backend/blob/develop/docs/flag-referee-app-data-model.md) | Endpoints operacionais de partidas, validação de check-in e eventos de lance. |
| [Análise & Proposta de Enums](https://github.com/cesargranelli/flag_backend/blob/develop/docs/analise-completa-enums-backend.md) | Mapeamento e adequação dos enums Java com as regras de arbitragem. |

---

## 3. Diretrizes e Decisões Globais Aplicáveis

- [ADR-001 — Nova Filosofia de Arquitetura](../../adr/ADR-001-nova-filosofia-arquitetura.md)
- [ADR-002 — Estratégia de Dados Híbrida (PostgreSQL + Firestore)](../../adr/ADR-002-postgres-firestore-cqs.md)
- [ADR-003 — Diagramas de Base de Dados](../../adr/ADR-003-diagramas-base-de-dados.md)
- [ADR-010 — Autenticação Centralizada Firebase-First](../../adr/ADR-010-autenticacao-firebase-custom-claims.md)
- [ADR-012 — Migrations com jOOQ e Flyway](../../adr/ADR-012-migrations-jooq-java.md)
- [Catálogo Oficial de Eventos por Modalidade](../../architecture/mapeamento-eventos-modalidades.md)
