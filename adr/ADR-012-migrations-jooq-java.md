# ADR-012: Migrations Flyway com JOOQ DSL (Java)

## Status

Accepted

## Contexto

O projeto utiliza Flyway para gerenciamento de migrations do PostgreSQL. Historicamente, existem migrations em SQL puro (`src/main/resources/db/migration/`) e migrations em Java com JOOQ DSL (`src/main/java/db/migration/`). As migrations SQL legadas (V1-V3) nao devem ser utilizadas para novas alteracoes.

## Decisao

Todas as novas migrations Flyway devem ser escritas em **Java usando JOOQ DSL**, seguindo o padrao estabelecido nas migrations V4-V14.

### Localizacao

```
src/main/java/db/migration/
├── V15__RemoveAthleteNickNumber.java
├── V16__RenameAthletesToParticipants.java
├── V17__SuaProximaMigracao.java
└── ...
```

### Numeracao

- Ultima migration existente: V14 (CreateCompetitionEnrollmentWindows)
- Proximas migrations: V15, V16, V17, ...
- Formato: `V{numero}__{DescricaoCamelCase}.java`

### Template Base

```java
package db.migration;

import java.sql.Connection;
import org.flywaydb.core.api.migration.BaseJavaMigration;
import org.flywaydb.core.api.migration.Context;
import org.jooq.DSLContext;
import org.jooq.SQLDialect;
import org.jooq.impl.DSL;
import org.jooq.impl.SQLDataType;

/**
 * Migration V{N}: Descricao da migracao.
 *
 * <p>Detalhes sobre o que esta migracao faz.
 */
public class V{N}__Descricao extends BaseJavaMigration {

    @Override
    public void migrate(Context context) throws Exception {
        Connection connection = context.getConnection();
        DSLContext dsl = DSL.using(connection, SQLDialect.POSTGRES);

        // DDL usando JOOQ DSL
        dsl.createTableIfNotExists(DSL.name("schema", "table"))
                .column(DSL.field(DSL.name("id"), SQLDataType.UUID.nullable(false)
                        .defaultValue(DSL.function("gen_random_uuid", SQLDataType.UUID))))
                // ... mais colunas
                .constraint(DSL.constraint(DSL.name("pk_table")).primaryKey(DSL.field(DSL.name("id"))))
                .execute();
    }
}
```

### Sintaxes Comuns

#### Criar tabela

```java
dsl.createTableIfNotExists(DSL.name("platform", "tabela"))
    .column(DSL.field(DSL.name("coluna"), SQLDataType.VARCHAR(100).nullable(false)))
    .constraint(DSL.constraint(DSL.name("pk_tabela")).primaryKey(DSL.field(DSL.name("id"))))
    .execute();
```

#### Adicionar coluna

```java
dsl.alterTable(DSL.name("platform", "tabela"))
    .add(
        DSL.field(DSL.name("nova_coluna"), SQLDataType.VARCHAR(100)),
        DSL.field(DSL.name("outra_coluna"), SQLDataType.INTEGER)
    )
    .execute();
```

#### Renomear tabela

```java
dsl.alterTable(DSL.name("platform", "tabela_antiga"))
    .renameTo(DSL.name("platform", "tabela_nova"))
    .execute();
```

#### Criar indice

```java
dsl.createIndexIfNotExists(DSL.name("idx_tabela_coluna"))
    .on(DSL.table(DSL.name("platform", "tabela")),
        DSL.field(DSL.name("coluna")))
    .execute();
```

#### Operacoes complexas (FK, constraints renomeadas)

Para operacoes que o JOOQ DSL nao suporta nativamente (como renomear constraints ou criar FKs com WHERE), usar SQL direto:

```java
dsl.execute("ALTER TABLE platform.tabela DROP CONSTRAINT IF EXISTS constraint_antiga");
dsl.execute("ALTER TABLE platform.tabela ADD CONSTRAINT constraint_nova FOREIGN KEY (col) REFERENCES outra_tabela(id)");
```

### Regras

1. **Sempre usar JOOQ DSL** quando possible (CREATE TABLE, ALTER TABLE ADD/DROP column, CREATE INDEX)
2. **SQL direto** apenas para operacoes nao suportadas pelo JOOQ (rename constraint, FK complexas)
3. **Schema**: sempre qualificado com `platform.` (via `DSL.name("platform", "table")`)
4. **Nomes de constraints**: usar prefixo descritivo (ex: `pk_`, `fk_`, `uk_`, `idx_`)
5. **Comentarios Javadoc**: documentar o proposito da migration
6. **Rollback**: nao suportado pelo Flyway; migration deve ser idempotente quando possivel

### Arquivos Legados (NAO USAR)

```
src/main/resources/db/migration/
├── V1__MomentZero.sql      # Legado - schema inicial
├── V2__*.sql                # Legado - nao usar
└── V3__*.sql                # Legado - nao usar
```

Esses arquivos SQL sao do inicio do projeto e nao devem ser modificados ou utilizados para novas migrations.

## Consequencias

- **Consistencia**: todas as migrations seguem o mesmo padrao Java/JOOQ
- **Type-safety**: JOOQ DSL oferece verificacao em tempo de compilacao
- **Manutencao**: migrations mais faveis de ler e modificar
- **Flexibilidade**: SQL direto disponivel para operacoes complexas
- **Legado**: migrations SQL antigas permanecem para compatibilidade com ambientes existentes

## Referencias

- [jOOQ Manual - DDL Statements](https://www.jooq.org/doc/latest/manual/sql-building/ddl-statements/)
- [Flyway Java Migrations](https://flywaydb.org/documentation/concepts/java-based-migrations)
- Existing migrations: V4-V14 em `src/main/java/db/migration/`
