# ADR-002: Manter PostgreSQL como banco de dados primário + Firestore como espelho CQRS (Light)

## Contexto

A Flag Platform possui um domínio de gestão esportiva (flag football) com:
- Estrutura hierárquica bem definida: Organization → Institution → Team → Roster → Athlete (Person)
- Necessidade de integridade referencial, transações e consultas complexas (leaderboards, estatísticas)
- Volume de dados moderado (10-20 TPS de escrita em picos de rodadas), com consultas públicas massivas
- Existing PostgreSQL schema evoluído via migrations Java Flyway

## Decisão

**Manter PostgreSQL como banco de dados primário** (fonte da verdade transacional ACID) e **implementar Firestore como espelho CQRS leve** para consultas de leitura e casos de uso de tempo real.

### Por quê?

| Critério | PostgreSQL | Firestore |
|----------|------------|------------|
| Integridade referencial | ✅ Garantida | ❌ Manual |
| Transações (scores, limites) | ✅ Suportado | ❌ Não |
| Consultas complexas (join, agregações) | ✅ Nativo | ⚠️ Lenta/Não escalável |
| Auditoria e histórico | ✅ Trátil | ⚠️ Limitado |
| Custo operacional | ✅ Baixo | ✅ Baixo |
| Tempo de implementação | ✅ Já feito | ⚠️ Reestruturação necessária |

### Arquitetura Proposta

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│   Public App │◄────►│  Firestore   │◄────►│  Analytics  │
│  (Web/Flutter)│     │  (Espelho)   │     │  (Leaderboards)│
└─────────────┘     └──────────────┘     └─────────────┘
       │                    │                        │
       ▼                    ▼                        ▼
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│  PostgreSQL │     │  Firestore   │     │  Firestore  │
│  (Primário) │     │  (CQS Mirror)│     │  (CQS Mirror)│
│  - Todos os │     │  - Leituras  │     │  - Dashboards│
│    dados    │     │  - Statistics│     │  - Real-time│
│  - Transações│    │  - Notificações│    │  - Live scores│
└─────────────┘     └──────────────┘     └─────────────┘
```

### Campos de Espelhamento (CQS Light)

| Entidade | Tabela PostgreSQL | Firestore Collection | Observações |
|----------|-------------------|----------------------|-------------|
| Organization | `organizations` | `organizations` | Federações e Ligas organizadoras |
| Institution | `institutions` | `institutions` | 1:N com Teams |
| Affiliation | `institution_affiliations` | `affiliations` | Filiação formal da agremiação à organização |
| Competition | `competitions` | `competitions` | 1:N com Rounds e Categorias |
| Category | `categories` | `categories` | Modalidade, gênero e faixa etária |
| Team | `teams` | `teams` | Equipe vinculada à Instituição |
| Person | `persons` | `persons` | Pessoa física unificada (CPF único) |
| RosterEntry | `team_roster` | `roster_entries` | Inscrição de atleta em elenco por temporada |
| Game | `games` | `games` | Partida oficial com placar agregado |
| GameParticipant | `game_participants` | `game_participants` | Árbitros e delegados na mesa |
| ScoreEvent | `score_events` | `score_events` | Lances e pontuações da partida |
| CheckIn | `checkins` | `checkins` | Presença confirmada de atletas em campo |
| Standing | `standings` | `standings` | Tabela de classificação projetada |

### Benefícios

1. **Preservação do domínio atual** – Nenhuma reestruturação do schema PostgreSQL
2. **Performance de escrita** – Onde importa (inserções de scores, check-ins) permanece em PostgreSQL
3. **Consultas rápidas de leitura** – Dashboards, leaderboards, estatísticas em Firestore
4. **Escalabilidade gradual** – Firestore escala bem para leituras massivas sem impacto na escrita
5. **Custo controlado** – Ambos são gratuitos ou de baixo custo (PostgreSQL gerenciado + Firestore Free Tier)

### Limitações

- **Soft deletes** devem ser implementados em ambas as fontes (PK + `deleted_at`)
- **Transações** que envolvem múltiplas tabelas devem permanecer em PostgreSQL
- **Auditoria** completa (logs de alterações) mantida em PostgreSQL
- **Integração com autenticação** (Firebase Auth) permanece no PostgreSQL (user table)

## Execução

1. **Configurar Cloud Functions** para sincronização bidirecional (PostgreSQL → Firestore)
2. **Implementar soft deletes** em todas as entidades (adicionar `deleted_at` column)
3. **Migrar dados iniciais** (seed) para Firestore
4. **Criar APIs de leitura** em Firestore para dashboards e notificações
5. **Monitorar performance** e ajustar estratégias de indexação

## Próximos Passos

- [ ] Criar Cloud Functions para sincronização CQRS
- [ ] Implementar soft deletes em PostgreSQL
- [ ] Migrar dados iniciais para Firestore
- [ ] Criar endpoints de leitura em Firestore (dashboards, leaderboards)
- [ ] Validar performance de consultas em Firestore

## Referências

- [ADR-001](ADR-001-nova-filosofia-arquitetura.md) – Filosofia inicial
- [Market Analysis](market-analysis-restructured.md) – Comparativo de soluções
- [FlagStats Mapping](flagstats-mapeamento.md) – Mapeamento de métricas

---

**Status**: Approved
**Created**: 2026-09-06
**Version**: 1.0
