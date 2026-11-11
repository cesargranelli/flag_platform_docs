# BACK-08: Recálculo da classificação falha com ORA-12860 (ADB `_high` + DML paralelo)

> **Type:** bug
> **Status:** resolved
> **Effort:** repos/backend
> **Repo(s):** flag_backend, flag_platform_infra
> **Blocks:** BACK-07 (a via de recálculo nunca conclui)
> **Blocked by:** —
> **Origin:** logs do container `flag-backend` (VM OCI `163.176.195.27`) — 2026-10-08

## Sintoma

`POST /api/v1/competitions/{id}/standings/recalculate` responde **202 ENQUEUED**, o motor é
invocado, mas a classificação **não muda** (`lastCalculatedAt` permanece o mesmo) e
`recalculationPending` continua `true`.

## Evidência (container `flag-backend`, 2026-10-08T01:15)

```text
POST .../standings/recalculate → 202 ENQUEUED
WARN  [task-2] HHH000247: ErrorCode: 12860, SQLState: 72000
WARN  [task-2] ORA-12860: deadlock detected while waiting for a sibling row lock
ERROR [task-2] SimpleAsyncUncaughtExceptionHandler:
  Unexpected exception ... StandingsCalculationService.recalculateAsync ...
  insert into platform.standings (...) values (...)
Caused by: java.sql.SQLException: ORA-12860 ...
```

A falha ocorre no **commit** (`JpaTransactionManager.doCommit` → `InsertCoordinatorStandard.insert`),
após o `DELETE` + `INSERT` do recálculo — rollback total, classificação intacta.

## Causa

O datasource usa o service **`devdb_high`** (`DATASOURCE_URL=jdbc:oracle:thin:@devdb_high?TNS_ADMIN=/app/wallet`).
No Autonomous Database, o `_high` habilita **paralelismo automático de DML (PDML)**. O `DELETE` em
massa do recálculo (`StandingRepository.deleteAllByCompetitionId`) dispara *parallel slaves*, e o
`INSERT` subsequente das mesmas linhas deadlocka — o Oracle define o ORA-12860 como *"Two Parallel
Data Manipulation Language (PDML) sibling transactions deadlocked one another while waiting for a row lock"*.

Referências: `docs.oracle.com/error-help/db/ora-12860/`; relato de `_high` resolvido com `_low`
(blog.termian.dev/posts/ora-12860-oci-high/).

## Correção aplicada

Trocado o service do ADB de `devdb_high` para **`devdb_low`** (desabilita o paralelismo de DML), sem
alterar código:
- VM: `DATASOURCE_URL=jdbc:oracle:thin:@devdb_low?TNS_ADMIN=/app/wallet` no `.env` + container `flag-backend` recriado.
- Persistência: `flag_platform_infra/docker/backend/docker-compose.yml` (default → `devdb_low`, commit `afc34a4` em `main`).

### Verificação (2026-10-08T01:55)

`POST /standings/recalculate` → `lastCalculatedAt=2026-10-08T01:55:01`, `recalculationPending=false`,
sem `ORA-12860`. Os 3 jogos de 2026-10-02 foram consolidados (ex.: Silver Knights PF 81→116;
Black Panthers PA 90→160).

Alternativas (não preferidas):
- Desabilitar PDML por sessão via Hikari `connection-init-sql` — acopla o código ao Oracle (fere a
  portabilidade multi-vendor do projeto).
- Reestruturar o recálculo para evitar `DELETE` em massa — mudança maior, sem garantia.

## Critérios de aceite

- [ ] `POST /standings/recalculate` atualiza `lastCalculatedAt` e zera `recalculationPending`.
- [ ] Sem `ORA-12860` nos logs.
- [ ] Encerrar jogo via `PATCH /games/{id}/status` (BACK-07) recalcula a classificação.

## Arquivos / referências

- `flag_platform_infra/docker/backend/docker-compose.yml` (`DATASOURCE_URL`)
- `flag_platform_infra/.env` (`DB_NAME=devdb_high`, `SPRING_DATASOURCE_URL`)
- `flag_backend/src/main/java/br/com/flagplatform/standing/service/StandingService.java` (`recalculate`)
