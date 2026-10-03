# Plano de Reinstituição da Flag Platform - Fase 0

## Objetivos da Fase 0: Estabilização e Consolidação de Dados

Esta fase inicial foca em estabelecer a base técnica e estrutural para o projeto, garantindo a consistência dos dados e a infraestrutura de desenvolvimento para as próximas etapas.

## Diretrizes Estratégicas Desta Fase

- **Reinstituição Total:** Reiniciar a Flag Platform a partir de um "momento zero", implementando uma nova arquitetura e modelo de dados.
- **Preservação de Histórico:** Manter as branches `main_v1` existentes nos repositórios `flag_admin_web` e `flag_backend` para fins de histórico e fallback.
- **Abordagem em Fases:** O desenvolvimento das aplicações ocorrerá sequencialmente, começando pelo `flag_admin_web`.
- **Tecnologias de Persistência e Migrações:** Utilizar **Liquibase (changelogs YAML) + Oracle ADB** para o gerenciamento de migrações. Todas as migrações são `changeSet`s YAML; **DDL em código (Java/SQL solto) é proibido**, conforme a **ADR-012** (nota: a referência histórica a "Flyway/jOOQ" foi corrigida na reconciliação de linhagens).
- **Migração de Dados:** Implementar a migração de dados da hierarquia antiga (`organizations`) para a nova (`institutions`/`clubs`), conforme a **ADR-006** (e ADR-001 v1.1), via `changeSet` Liquibase. Para o "momento zero", todas as tabelas relevantes serão zeradas.

## Passos da Fase 0

### Passo 0.1: Estabilização do Repositório flag-platform-docs
- **Status:** Concluído. Este documento faz parte deste processo.

### Passo 0.2: Ramificação para Rollback
- **Status:** Concluído. Criação e push da branch `main_v1` a partir da `main` atual nos repositórios `flag-platform-docs`, `flag_public_app`, `flag_referee_app` e `flag_tester_e2e`.

### Passo 0.3: Estabilização do Backend e Consolidação de Dados
- **Ação:** Criar a branch `develop` no repositório `flag_backend` a partir da `main`.
- **Ação:** Limpar os scripts de migração legados (Flyway) na branch `develop`.
- **Ação:** Consolidar o DDL de referência (baseline) em `changeSet` do Liquibase (YAML), validando sintaxe para Oracle ADB.
- **Ação:** Configurar o `pom.xml` do `flag_backend` para incluir o **Liquibase**; sem geração de código JOOQ para migrações.
- **Ação:** Implementar a lógica de "zerar dados" e a migração (`organizations` -> `institutions`) nos `changeSet`s do Liquibase do "momento zero".

### Passo 0.4: Desabilitar Pipelines de CI
- **Ação:** Desabilitar globalmente os pipelines de CI/CD para prevenir deploys prematuros durante as fases 1 a 3.
- **Ação:** Atualizar o pipeline de CI do `flag_backend` para incluir a etapa de geração de código do JOOQ.

---
*Este documento é dinâmico e será atualizado conforme o progresso do projeto. Verifique o estado das tarefas no TODO e relate.*
