# ADR-012: Migrations com Liquibase (YAML) — runtime Oracle ADB

> **Correção (reconciliação de linhagens):** o conteúdo original desta ADR afirmava "Flyway + PostgreSQL + JOOQ DSL em Java".
> O runtime real do `flag_backend` é **Liquibase (changelogs YAML) + Oracle ADB**, e **DDL em código é proibido**
> (ver `AGENTS.md` do `flag_backend` e [ADR-018](ADR-018-espelhamento-firestore-admin-sdk.md)).
> O **nome do arquivo** (`ADR-012-migrations-jooq-java.md`) foi mantido para não quebrar links; a proposta de
> renomeação de título/slug está registrada em
> [architecture/reconciliacao-adr-linhagens.md](../architecture/reconciliacao-adr-linhagens.md).
>
> **Absorção:** o antigo **ADR-007 — Migrações Flyway em Java com JOOQ** (linhagem `main`) tratava deste
> mesmo tema e foi **descartado como duplicata**; seu arquivo standalone foi removido e o conteúdo relevante
> (runtime de migrações) vive nesta ADR-012, corrigido para Liquibase + Oracle ADB.

## Status

Aceito — **corrigido** em 2026-10-03 (o runtime é Liquibase + Oracle ADB, não Flyway/jOOQ).

## Contexto

O `flag_backend` versiona a evolução do schema do **Oracle ADB** exclusivamente por meio de
**changelogs do Liquibase em YAML** (`src/main/resources/db/changelog/`). A regra mandatória do
repositório é: **nunca** escrever DDL/DML de migração em código Java ou SQL solto na aplicação —
toda alteração de schema vive em um `changeSet` versionado do Liquibase.

Versões anteriores da documentação descreviam um pilha diferente (Flyway + PostgreSQL + JOOQ DSL).
Essa pilha **não é** o runtime vigente e não deve ser usada como referência para novas migrações.

## Decisão

Todas as novas migrações são implementadas como **changelogs YAML do Liquibase**, aplicadas ao
**Oracle ADB**.

### Localização

```
src/main/resources/db/changelog/
├── db.changelog-master.yaml        # changelog raiz (includes)
├── 001-baseline.yaml
├── ...
├── 015-remove-athlete-nick-number.yaml
└── 016-rename-athletes-to-participants.yaml
```

O `db.changelog-master.yaml` inclui os arquivos na ordem de execução:

```yaml
databaseChangeLog:
  - include:
      file: db/changelog/001-baseline.yaml
  - include:
      file: db/changelog/015-remove-athlete-nick-number.yaml
```

### Numeração e nomenclatura

- Arquivos: `{NNN}-{descricao-em-kebab-case}.yaml` (ex.: `015-remove-athlete-nick-number.yaml`).
- `changeSet.id`: descritivo e único (ex.: `015-remove-athlete-nick-number`).
- `author`: identificador fixo do time/projeto.

### Template Base

```yaml
databaseChangeLog:
  - changeSet:
      id: 015-remove-athlete-nick-number
      author: flag-platform
      changes:
        - dropColumn:
            schemaName: PLATFORM
            tableName: PARTICIPANTS
            columnName: NICK_NUMBER
      rollback:
        - addColumn:
            schemaName: PLATFORM
            tableName: PARTICIPANTS
            columns:
              - column:
                  name: NICK_NUMBER
                  type: NUMBER(10)
```

### Sintaxes Comuns (Oracle)

#### Criar tabela

```yaml
- changeSet:
    id: 016-create-table
    author: flag-platform
    changes:
      - createTable:
          schemaName: PLATFORM
          tableName: TABELA
          columns:
            - column:
                name: ID
                type: RAW(16)
                constraints:
                  primaryKey: true
                  nullable: false
            - column:
                name: NOME
                type: VARCHAR2(255)
                constraints:
                  nullable: false
```

#### Adicionar / renomear / remover coluna

```yaml
- changeSet:
    id: 017-alter-tabela
    author: flag-platform
    changes:
      - addColumn:
          schemaName: PLATFORM
          tableName: TABELA
          columns:
            - column:
                name: NOVA_COLUNA
                type: VARCHAR2(100)
      - renameColumn:
          schemaName: PLATFORM
          tableName: TABELA
          oldColumnName: ANTIGA
          newColumnName: NOVA
      - dropColumn:
          schemaName: PLATFORM
          tableName: TABELA
          columnName: OBSOLETA
```

#### Criar índice e constraint

```yaml
- changeSet:
    id: 018-indexes
    author: flag-platform
    changes:
      - createIndex:
          schemaName: PLATFORM
          tableName: TABELA
          indexName: IDX_TABELA_COLUNA
          columns:
            - column:
                name: COLUNA
      - addUniqueConstraint:
          schemaName: PLATFORM
          tableName: TABELA
          columnNames: COLUNA_A, COLUNA_B
          constraintName: UK_TABELA_AB
```

### Regras

1. **Sempre Liquibase YAML** — nunca DDL/DML de migração em Java, SQL solto ou anotações de entidade.
2. **Schema qualificado**: `PLATFORM` (Oracle usa identificadores em maiúsculas por padrão).
3. **Tipos Oracle**: `RAW(16)`/`VARCHAR2(n)`/`NUMBER(p,s)`/`TIMESTAMP WITH TIME ZONE`/`CLOB`.
4. **Append-only**: nunca editar um `changeSet` já aplicado (checksums do Liquibase).
5. **Rollback**: declarar `rollback` sempre que viável.
6. **Idempotência/precondições**: usar `preConditions` (`onFail: MARK_RAN`/`HALT`) quando necessário.

### Arquivos Legados (NÃO USAR)

As migrações SQL/Java da pilha antiga (Flyway + PostgreSQL + JOOQ, ex.: `V1__MomentZero.sql`,
`src/main/java/db/migration/`) **não fazem parte do runtime atual** e não devem ser reaproveitadas
nem servir de modelo para novas migrações. O baseline vigente é o changelog do Liquibase.

## Consequências

- **Consistência**: todas as migrações seguem o mesmo padrão Liquibase/YAML.
- **Rastreabilidade**: `changeSet.id`/`author` e checksums garantem histórico auditável.
- **Rollback**: suporte nativo do Liquibase (quando declarado).
- **Portabilidade**: changelogs independentes do dialeto, com `dbms`/`contexts` quando preciso.
- **Legado**: os scripts Flyway/jOOQ permanecem apenas como histórico e não são executados.

## Referências

- [Liquibase — Changelog Formats](https://docs.liquibase.com/concepts/changelogs/yaml-format.html)
- [Liquibase — changeSet & rollback](https://docs.liquibase.com/concepts/changelogs/changeset.html)
- [ADR-018](ADR-018-espelhamento-firestore-admin-sdk.md) — infraestrutura real (Oracle ADB + OCI)
- [ADR-003 — Diagramas de Base de Dados](ADR-003-diagramas-base-de-dados.md)
