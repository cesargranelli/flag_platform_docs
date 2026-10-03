# ADR-016: Gestão de segredos via GitHub Environments

## Status
Aceito

## Data
2026-09-29

## Autor
DevOps Engineer (Flag Platform)

## Contexto

A auditoria de `env`/secrets da plataforma encontrou:

1. **Valores de segredo em texto puro em arquivos rastreados**: fallbacks de
   `key.properties` (`storePassword`/`keyPassword`) no workflow do Hub, senha real de
   datasource como default no `docker-compose.yml` e uma senha de banco em um
   `application-dev.yml` de repositório legado.
2. **Aliases de nome convivendo**: `SPRING_DATASOURCE_*` × `DATASOURCE_*` e
   `CLOUDFLARE_R2_*` × `R2_*`, dificultando saber qual chave está "certa".
3. **Scripts de sincronização** que passavam valores por `--body` (visíveis em `argv`/histórico)
   e/ou embutiam PAT em texto puro.
4. Ausência de um documento canônico de chaves/ambientes e de regras explícitas de
   Secret × Variable.

Em paralelo, o responsável decidiu **não rotacionar** as credenciais expostas nem
**purgar o histórico Git**, considerando que os repositórios/ambientes são controlados
(acesso restrito) e que a prioridade imediata é estancar a exposição do estado atual.

---

## Decisão

1. **GitHub Environments é a fonte única de segredos dos deploys.** Segredos e variáveis
   vivem em *Environment Secrets/Variables* de `staging`/`production` (ou no nível do
   repositório quando comuns a todos os ambientes). Nada de segredo em arquivo versionado
   ou literal em workflow.
2. **Sanitização aprovada e aplicada**: remoção de todos os fallbacks com valores reais
   (workflow, `docker-compose.yml`, `application-dev.yml` legado), substituição de
   identificadores reais por placeholders nos `.env.example`, endurecimento dos
   `.gitignore` e passagem de `gh secret set` para **stdin**.
3. **Abandono dos aliases legados.** Nomes canônicos passam a ser `DATASOURCE_*` e `R2_*`.
   Aliases (`SPRING_DATASOURCE_*`, `CLOUDFLARE_R2_*`, `DB_*`/`DATABASE_*`) ficam apenas como
   compatibilidade **transitória**, documentada, e devem ser migrados ao tocar nos arquivos.
4. **Fail-fast obrigatório**: segredo ausente ⇒ job aborta com `::error::`/`exit 1`.
5. **Rotação de credenciais expostas fica ADIADA** (decisão consciente do responsável), com
   o risco residual registrado neste ADR e no tracker da auditoria.
6. **Purga do histórico Git fica ADIADA** pela mesma razão.

### Concretização

- Documento canônico: [`architecture/env-vars-e-secrets.md`](../architecture/env-vars-e-secrets.md).
- Tracker da auditoria: `_planning/segredos-e-env/`.
- Script de sincronização por stdin: `flag_platform_infra/scripts/sync-github-secrets.ps1`.

---

## Consequências

### Positivas
- Estado atual **livre de segredos rastreados** (varredura `git grep` limpa para os padrões
  sensíveis).
- Segredos com uma única casa e nomes sem ambiguidade.
- Deploys falham cedo e de forma clara quando uma credencial falta (sem "funcionar por acaso").

### Negativas / custos
- O deploy do backend passa a **exigir** o secret `DATASOURCE_PASSWORD` cadastrado; sem ele
  o job falha (intencional). Exige ação única do operador antes do próximo deploy.
- Migração de aliases é incremental e gera um período de convivência.

### Risco residual (aceito conscientemente)
- **Credenciais expostas permanecem válidas** (rotação adiada) até que um incidente ou uma
  janela planejada dispare o runbook de rotação. O histórico Git ainda contém os valores
  antigos (purga adiada).
- Mitigação: repositórios privados, acesso restrito; rotação documentada e pronta para execução.

---

## Alternativas consideradas

| Alternativa | Por que não |
|---|---|
| Rotacionar imediatamente todas as credenciais | Impacto operacional amplo; decisão do responsável foi adiar. |
| Purgar histórico (`git filter-repo`/BFG) e force-push | Reescreve histórico compartilhado; risco alto nos repos ativos; adiado. |
| Manter fallbacks no workflow "para não quebrar" | Viola o princípio fail-fast e mantém segredo em texto puro no repo. |
| Manter dois nomes (alias) indefinidamente | Ambiguidade e dívida; migração canônica é mais segura no médio prazo. |
| Vault externo (ex.: HashiCorp Vault) agora | Overhead para o porte atual (Always Free / custo zero); GitHub Environments já cobre a necessidade. |
