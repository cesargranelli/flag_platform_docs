# Mapa: Configuracao de Playoffs

## Ordem de execucao

1. **Spec** (`spec.md`) — ja criada
2. **ADR** — decidir modelo de dados e estrategia de geracao
3. **Backend**
   - 3.1 Entidade `PlayoffConfiguration` + migration Liquibase
   - 3.2 DTOs e endpoint CRUD
   - 3.3 Motor de geracao de jogos de playoffs
   - 3.4 Critérios de desempate para playoffs
4. **Admin Web** — tela de configuracao
5. **App Publico** — exibicao
6. **Testes** (tester)
7. **Docs** (`flag_backend/docs/playoffs.md`)

## Responsaveis

| Etapa | Agente |
|-------|--------|
| ADR | tech-lead |
| Backend (entidade, migration, DTOs, endpoint) | backend + dba |
| Backend (motor de geracao) | backend |
| Admin Web | frontend |
| App Publico | app |
| Testes | tester |