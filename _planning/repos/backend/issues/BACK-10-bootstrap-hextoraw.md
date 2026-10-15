# BACK-10: Bootstrap local quebra em PostgreSQL (changeset 002 usa HEXTORAW)

> **Type:** bug
> **Status:** open
> **Effort:** repos/backend
> **Repo(s):** flag_backend
> **Blocks:** —
> **Blocked by:** —
> **Origin:** descoberta durante smoke do esforço `escopo-perfis` (ESC-01) em 2026-10-15

## Contexto

O changeset `002-production-data.yaml` usa a função Oracle `HEXTORAW` para popular dados de
produção. Em PostgreSQL local o mesmo script **falha**, impedindo o bootstrap do banco do zero
(`mvn liquibase:update` a partir de `001`). O desenvolvedor contornou aplicando `001` + `003` +
`004` manualmente; `005+` rodou no boot normalmente.

## Critérios de aceite

- [ ] Bootstrap local completo (`mvn liquibase:update` do zero) funciona em PostgreSQL sem
      erros.
- [ ] `002` adaptado de forma portável (não `HEXTORAW` nativo) ou condicionado a perfil.
- [ ] `mvn -B clean verify` verde; smoke de startup local.
- [ ] Produção (Oracle ADB) inalterada — validação de que a changeset ainda é aplicável lá.

## Notas

- Relacionado ao assessment de portabilidade do repo
  (`docs/database-portability-assessment.md`, item 3: `JdbcTemplate` + literal `platform.`).
- Não bloqueia `escopo-perfis`, mas impede reprodução local do smoke de F1/F2.