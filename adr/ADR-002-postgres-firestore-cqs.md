# ADR-002: Manter Oracle ADB como banco de dados primário + Firestore como espelho CQRS (Light)

> **Atualizado por [ADR-018](ADR-018-espelhamento-firestore-admin-sdk.md).** A decisão CQRS Light
> permanece; a premissa de banco foi corrigida de PostgreSQL para **Oracle Autonomous Database 23ai**
> e o **"como"** do espelhamento passa a ser o **Firebase Admin SDK no backend** (ADR-018), e não
> Cloud Functions. O texto abaixo já reflete essa correção.

## Contexto

A Flag Platform possui um domínio de gestão esportiva (flag football) com:
- Estrutura hierárquica bem definida: Organization → Institution → Team → Roster → Athlete (Person)
- Necessidade de integridade referencial, transações e consultas complexas (leaderboards, estatísticas)
- Volume de dados moderado (10-20 TPS de escrita em picos de rodadas), com consultas públicas massivas
- Schema atual no **Oracle Autonomous Database 23ai (Always Free)** — schema/usuário `platform`,
  conexão mTLS via wallet, `jdbc:oracle:thin:@devdb_high` — evoluído via migrations **Liquibase** (YAML)

## Decisão

**Manter o Oracle ADB como banco de dados primário** (fonte da verdade transacional ACID) e **implementar Firestore como espelho CQRS leve** para consultas de leitura e casos de uso de tempo real.

O **como** do espelhamento está definido no [ADR-018](ADR-018-espelhamento-firestore-admin-sdk.md):
**Firebase Admin SDK no próprio backend**, best-effort pós-commit — e **não** Cloud Functions.

### Por quê?

| Critério | Oracle ADB | Firestore |
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
│  Oracle ADB │     │  Firestore   │     │  Firestore  │
│  (Primário) │     │  (CQS Mirror)│     │  (CQS Mirror)│
│  - Todos os │     │  - Leituras  │     │  - Dashboards│
│    dados    │     │  - Statistics│     │  - Real-time│
│  - Transações│    │  - Notificações│    │  - Live scores│
└─────────────┘     └──────────────┘     └─────────────┘
```

### Campos de Espelhamento (CQS Light)

| Entidade | Tabela (Oracle ADB) | Firestore Collection | Observações |
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

1. **Preservação do domínio atual** – Nenhuma reestruturação do schema Oracle ADB
2. **Performance de escrita** – Onde importa (inserções de scores, check-ins) permanece no Oracle ADB
3. **Consultas rápidas de leitura** – Dashboards, leaderboards, estatísticas em Firestore
4. **Escalabilidade gradual** – Firestore escala bem para leituras massivas sem impacto na escrita
5. **Custo controlado** – Oracle ADB Always Free + Firestore Free Tier (monitorar operações de leitura — ver nota de custo no ADR-018)

### Limitações

- **Soft deletes** devem ser implementados em ambas as fontes (PK + `deleted_at`)
- **Transações** que envolvem múltiplas tabelas devem permanecer no Oracle ADB
- **Auditoria** completa (logs de alterações) mantida no Oracle ADB
- **Integração com autenticação** (Firebase Auth) permanece no Oracle ADB (tabela `users`)

## Execução

O **"como"** do espelhamento está definido no [ADR-018](ADR-018-espelhamento-firestore-admin-sdk.md)
(Firebase Admin SDK no backend, best-effort pós-commit). Em resumo:

1. **Espelhar** via `GameChangedEvent` + `@TransactionalEventListener(AFTER_COMMIT)` no módulo `realtime`
   do `flag_backend` (primeiro corte: `games`; depois `scoreEvents` e catálogo).
2. **Implementar soft deletes** nas entidades espelhadas (coluna `deleted_at`).
3. **Migrar dados iniciais** (seed) para Firestore via backfill opt-in.
4. **Reconciliar** `firestore.rules`/`indexes.json` (escrita só pelo backend; leitura pública).
5. **Monitorar** operações/drift do espelho.

## Próximos Passos

- [ ] Implementar o writer de `games` no backend (dependência `firebase-admin` + módulo `realtime`)
- [ ] Implementar soft deletes nas entidades espelhadas
- [ ] Backfill inicial para Firestore
- [ ] Criar/validar índices e a coleção de leitura em Firestore (dashboards, leaderboards)
- [ ] Validar performance e custo de consultas em Firestore

## Referências

- [ADR-001](ADR-001-nova-filosofia-arquitetura.md) – Filosofia inicial
- [ADR-018](ADR-018-espelhamento-firestore-admin-sdk.md) – **Como** o espelhamento acontece (Admin SDK)
- [Market Analysis](../research/market-analysis.md) – Comparativo de soluções
- [FlagStats Mapping](../research/flagstats-mapeamento.md) – Mapeamento de métricas

---

**Status**: Approved — atualizado por ADR-018
**Created**: 2026-09-06
**Version**: 1.0
