# ADR-001 — Nova Filosofia de Arquitetura e Simplificação

## Status
Proposto

## Data
2026-09-05

## Autor
Tech Lead (Flag Platform)

---

## Contexto

A Flag Platform está em estágio inicial de desenvolvimento (muitas issues em aberto, nenhuma em produção). A arquitetura atual é um **Modular Monolith** com Spring Boot + PostgreSQL + JWT custom.

Durante análise de mercado comparativa com FlagRoster, Flag50, TeamSnap e GameChanger, identificamos gaps críticos que definem a direção arquitetônica do projeto:

1. **Hierarquia confusa**: Conceitos de Time e Clube estavam mesclados, causando ambiguidade em FKs, permissões e UI
2. **Roles limitados**: Apenas 3 roles (ADMIN, ORGANIZER, MESA) para um ecossistema que precisa de granularidade crescente
3. **Auth não centralizado**: JWT custom separado dificilmente escalável e sem suporte a OAuth/MFA nativo
4. **Apps fragmentados**: Admin Web, Referee App e Public App operavam sem integração clara ou arquitetura compartilhada
5. **Complexidade de dados**: Necessidade de leitura performática (dashboards, leaderboards) coexistir com integridade transacional (scores, check-ins)

Esses gaps definem o escopo da nova filosofia: simplificação, centralização de auth e arquitetura de dados híbrida.

---

## Decisão Arquitetônica Fundamental

Adotar uma **nova filosofia de arquitetura** focada em cinco pilares interdependentes:

### Pilar 1: Hierarquia de Entidades Clara (5 níveis máximos)

Cada conceito de domínio tem sua própria entidade sem conflação:

```
Organization (Federação/Liga)       [1]:[N]
   └── Club (Clube/Universidade)        [1]:[N]
         └── Team (Time de competição)   [1]:[N]
              └── Roster (Elenco por Season) [1]:[N]
                   └── Athlete (Atleta) [1]:[N]
```

**Princípio eliminador**: Time ≠ Clube. Cada conceito é uma entidade distinta com responsabilidades próprias. Esta decisão fundamenta ADR-003 (implementação DB) e ADR-006 (refatoração estrutural).

### Pilar 2: Roles Expandidas (Modelo C - 4 Layers)

Expansão de 3 para 4+ roles, permitindo controle de acesso mais granular:

```
SUPER_ADMIN (1)     → Acesso total + gestão de orgs
    ↓
ORG_ADMIN (N)       → Gestão de clube/liga específica
    ↓
MANAGER (N)         → Cadastros dentro do clube
    ↓
USER (N)            → Atleta/coach/referee com roles específicos
```

**Expansão de 3 para 4 roles** fornece controle mais fino sem adicionar complexidade excessiva sobre a arquitetura existente. Detalhes de role matrix em ADR-010.

### Pilar 3: Autenticação Centralizada (Firebase-First)

Migração do JWT custom para Firebase Auth como Identity Provider, com Custom Claims para roles/permissões stateless no backend:

- **Firebase Auth como IdP**: Todos os clientes (Web e Mobile) realizam auth diretamente pelo Firebase Auth SDK
- **Custom Claims**: Tokens contêm `role + skills` (athlete, coach, referee, manager) — validados statelessmente no backend via chaves públicas do Google em cache
- **Security Rules no Firestore**: Regras de acesso baseadas nas Custom Claims

**Esta decisão substitui e consolida a proposta anterior que colidia com a ADR-004** (ver ADR-010 para detalhes completos de migração e consequências).

### Pilar 4: Apps Especializados (3 + 1 Ferramenta)

Quatro aplicativos com foco distinto, mas compartilhando o mesmo backend unificado:

| App | Foco | Integração |
|-----|------|------------|
| Admin Web | Gestão de org/clube/time | CRUD completo |
| Referee App | Operação de jogo | Scoring em tempo real |
| Public App | Fan Experience | Standings, highlights, live scores |
| Coach Tools | Estratégia | Playbook, stats, escalation |

**Integração**: Todos compartilham a mesma API REST `/api/v1` e backend Modular Monolith, garantindo consistência de dados across apps. Isso é complementado por ADR-011 (MVVM Flutter pattern para cada app).

### Pilar 5: Estratégia de Dados Híbrida (PostgreSQL + Firestore CQRS Light)

Separação de responsabilidades entre gravação e leitura:

| Camada | Responsabilidade | Tecnologia |
|--------|------------------|------------|
| **Command/Write** | Transações, integridade, validações | PostgreSQL (ACID) |
| **Query/Read** | Dashboards, leaderboards, real-time scores | Firestore (escalável) |
| **Sync** | Event-driven via Cloud Functions | PostgreSQL → Firestore |

**Princípios chave**:
- **Escrita consistente**: PostgreSQL garante ACID para scores, check-ins, limits
- **Leitura performática**: Firestore escalável para dashboards e analytics
- **Custo controlado**: Ambos gratuitos/low-cost no tier inicial
- **Zero refatoração de schema**: Dados de domínio continuam em PostgreSQL

Esta estratégia é definida detalhadamente em **ADR-002** (PostgreSQL + Firestore CQRS Light).

---

## Consequências

### Positivas
- ✅ **Hierarquia clara**: Organization→Club→Team→Roster→Atleta, sem conflação de conceitos
- ✅ **Mais controle**: 4 roles granulares em vez de 3, com matrix detalhada em ADR-010
- ✅ **Auth simplificado**: Firebase Auth com Custom Claims, stateless validation
- ✅ **Apps integrados**: Todos compartilham o mesmo backend, consistência de dados
- ✅ **Escalável**: Pode adicionar novos roles sem refatoração major (ADR-010)
- ✅ **Leitura otimizada**: Firestore para dashboards e real-time (ADR-002)
- ✅ **Zero refatoração de schema**: Dados de domínio continuam em PostgreSQL

### Negativas
- ⚠️ **Migração necessária**: Reverter builds anteriores do Firestore, atualizar clientes
- ⚠️ **Risco de quebra**: Mudança de auth afeta testes existentes e requires client updates
- ⚠️ **Complexidade inicial**: Implementar Firebase Auth + Custom Claims + CQRS sync
- ⚠️ **Tempo**: Requer revisão de issues existentes e coordenação across teams
- ⚠️ **Dual-write**: Sincronização eventual entre PostgreSQL ↔ Firestore (aceitável para leitura, crítico para escrita)

---

## Alternativas Consideradas

### A. Manter status quo (Modular Monolith + JWT custom)
- **Prós**: Sem migração, menos risco inicial
- **Contras**: Hierarquia confusa, roles limitados, auth não escalável, apps fragmentados — **descartado**

### B. Microsserviços
- **Prós**: Escalabilidade total por domínio
- **Contras**: Complexidade excessiva, viola a filosofia de monolito modular, over-engineering para volume atual (10-20 TPS) — **descartado**

### C. Firebase somente para Auth (mantém PostgreSQL)
- **Prós**: Auth centralizado, dados permanecem em PostgreSQL
- **Contras**: Dois sistemas de autenticação (JWT custom + Firebase), complexidade de sincronização de states, não aproveita Firestore para leitura — **descartado**

### D. Firebase completo (Auth + Firestore)
- **Prós**: Tudo em um lugar, escalável nativamente
- **Contras**: Lock-in no Firebase, requer reverter builds anteriores, perde benefícios de PostgreSQL (joins, FK, integridade transacional), custo de migration — **descartado**

---

## Decisão Final

**Adotar a Nova Filosofia com:**

1. **Hierarquia org→clube→time→elenco→atleta** — cada entidade distinta, sem conflation (fundamenta ADR-003 e ADR-006)
2. **4 roles**: SUPER_ADMIN, ORG_ADMIN, MANAGER, USER — matrix detalhada em ADR-010
3. **Firebase Auth com Custom Claims** — auth centralizada, validação stateless (substitui JWT custom)
4. **Manter PostgreSQL para dados de domínio** — source of truth para escrita, transações, integridade
5. **Firestore como espelho CQRS Light para leitura** — dashboards, leaderboards, real-time scores (definido em ADR-002)
6. **Apps especializados integrados** — Admin Web, Referee App, Public App, Coach Tools compartilhando API unificada

---

## Integração com Outros ADRs

Esta ADR serve como o **documento mestre de filosofia** que direciona e dá contexto para os ADRs especializados abaixo:

| ADR | Relação com ADR-001 |
|-----|---------------------|
| **ADR-002** | Define a estratégia CQRS (PostgreSQL ↔ Firestore) mencionada no Pilar 5 |
| **ADR-003** | Diagramas de Base de Dados — schema relacional (organizations, institutions, teams, persons, jogos) do Pilar 1 |
| **ADR-004** | Diagramas de Fluxo do Projeto — fluxos Firebase-First e ciclo de jogo, integrados à nova auth (ver ADR-010) |
| **ADR-019** | Modular Monolith (Spring Boot) — arquitetura de backend vigente |
| **ADR-010** | Implementa a autenticação Firebase-First e matrix de roles, substituindo a decisão de auth de ADR-001 |
| **ADR-006** | Refatoração estrutural de Team/Roster/Season, implementando o pilar 1 (hierarquia) de ADR-001 |
| **ADR-011** | Padrão MVVM Flutter para implementação dos apps especializados (Pilar 4) |
| **ADR-013** | Gitflow para gestão de documentação — fluxo recomendado para mudanças nestas decisões |
| **ADR-014** | Princípio de ADRs como documentos vivos — como atualizar esta filosofia quando diretrizes mudam |

---

## Plano de Implementação (Resumo)

As fases de implementação já estão documentadas nos ADRs especializados:

- **Fases 1-2** (Roles e Hierarquia): ADR-003, ADR-006
- **Fase 2** (Firebase Auth): ADR-010
- **Fase 3** (CQRS Sync): ADR-002
- **Fase 4** (Apps): ADR-011
- **Fase 5** (Monitoramento): ADR-002

---

## Critérios de Aceitação (Filosóficos)

- [ ] Hierarquia Organization→Club→Team→Roster→Atleta clara e documentada
- [ ] 4 roles definidas com matrix de permissões (ADR-010)
- [ ] Firebase Auth integrado como IdP em todos os clientes
- [ ] PostgreSQL permanece como source of truth para escritas
- [ ] Firestore configurado como espelho CQRS para leitura (ADR-002)
- [ ] Arquitetura substitui JWT custom por auth centralizada

---

## Riscos

| Risco | Mitigação |
|-------|-----------|
| Quebra de builds anteriores | Reverter commits de Firebase/Firestore, manter PostgreSQL como base |
| Complexidade de migração | Fases graduais, testes em cada sprint, conforme plano de cada ADR |
| Resistência da equipe | Comunicação clara dos benefícios da nova filosofia |
| Lock-in Firebase | Usar apenas Auth + Custom Claims; dados em PostgreSQL permanecem portáveis |
| **Eventual consistency** | **Aceitável para leitura (dashboards), crítico para escrita (PostgreSQL)** — validado por ADR-002 |

---

## Histórico de Atualizações (ADR-014)

| Data | Versão | Mudança | Motivo | Autor |
|:---:|:---:|---|---|---|
| 2026-09-19 | 1.1 | Evolução de Club → Institution e Athlete → Person | Separação estrita de federações e agremiações, vínculo de times diretamente a instituições e unificação cadastral com CPF | Tech Lead |
| 2026-09-05 | 1.0 | Versão original da Nova Filosofia | 5 pilares: hierarquia, 4 roles, Firebase-First, Modular Monolith e CQRS Light | Tech Lead |

---

## Decisão Atualizada (Update v1.1)

**Anterior (v1.0):** `Organization → Club → Team → Roster → Athlete`.  
**Atual (v1.1):** 
```
Organization (Federação / Liga Esportiva)
    └── Competition (Campeonato)
          └── Category (Modalidade + Gênero + Faixa)

Institution (Agremiação / Clube / Associação / Universidade)
    ├── InstitutionAffiliation (Filiação formal com Organization)
    └── Team (Equipe Esportiva)
          └── TeamRoster (Elenco da Temporada)
                └── Person (Pessoa Física com CPF único)
```

**Motivo da Atualização:**
1. **Institutions vs Organizations:** Clubes e agremiações participam de torneios de diferentes ligas e federações. A relação não é de posse hierárquica fixa (`1:N` estrito), mas de **filiação formal por temporada** (`InstitutionAffiliation`).
2. **Pessoas Físicas Unificadas (`Person`):** Uma pessoa física pode ser atleta em um time, técnico em outro ou atuar como árbitro/delegado. A tabela `persons` com CPF único evita duplicidade cadastral e preserva o histórico desportivo.

**Impacto:** Refletido em ADR-002, ADR-003, ADR-006 e nas migrations Flyway V5/V7/V12/V17/V24/V26.

---

## Referências Cruzadas

- **ADR-002** — Estratégia de dados PostgreSQL + Firestore CQRS Light
- **ADR-003** — Diagramas de Base de Dados (schema com institutions e persons)
- **ADR-004** — Diagramas do Projeto (fluxos Firebase-First e ciclo de jogo)
- **ADR-010** — Autenticação Firebase-First com Custom Claims
- **ADR-006** — Refatoração estrutural Team/Roster/Season
- **ADR-011** — MVVM Flutter para todos os clientes
- **ADR-013** — Gitflow para gestão de documentação
- **ADR-014** — Atualização de diretivas em ADRs (documentos vivos)

---

*Esta ADR serve como documento mestre de filosofia arquitetônica, atualizado sob as diretrizes de documentos vivos da ADR-014.*