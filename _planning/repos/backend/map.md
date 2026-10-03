# Mapa — Pendências do `flag_backend`

> **Esforço:** `repos/backend` · **Prefixo:** `BACK` · **Status:** aberto
> **Origem:** migração de `backend-issues.md` (raiz do hub) — fatiado por ticket em 2026-10-03.
> **Repo afetado:** [`cesargranelli/flag_backend`](https://github.com/cesargranelli/flag_backend)

Pendências críticas observadas em produção (VM OCI `163.176.195.27:8080` → container `flag-backend`
→ Oracle ADB, schema `platform`) durante a carga APFA/CBFA 2026.

## Bloqueantes

| ID | Título | Repo(s) | Status |
|----|--------|---------|--------|
| [BACK-01](issues/BACK-01-institutions-ora-17004.md) | `POST /api/v1/institutions` retorna 500 (`ORA-17004`) | flag_backend | open |
| [BACK-05](issues/BACK-05-competitions-ora-17023.md) | Leitura de `competitions` quebra com `ORA-17023` (`getBytes`) | flag_backend | open |
| [BACK-06](issues/BACK-06-roster-single-null-id.md) | `POST .../roster` (single) 500 "The given id must not be null" | flag_backend | open |

## Demais pendências

| ID | Tipo | Título | Repo(s) | Status |
|----|------|--------|---------|--------|
| [BACK-02](issues/BACK-02-404-mascarado-500.md) | bug | `404` mascarado como `500` (handlers ausentes) | flag_backend | open |
| [BACK-03](issues/BACK-03-upload-sem-multipart.md) | bug | `POST /api/v1/upload` sem multipart → `500` | flag_backend | open |
| [BACK-04](issues/BACK-04-regra-negocio-500.md) | bug | Regra de negócio/estado inválido responde 500 em vez de 409/404 | flag_backend | open |

## Notas do esforço

- `flag_backend/docs/database-portability-assessment.md` — item 3 (`JdbcTemplate` + literal `platform.`)
  classificado como **Bloqueante**; recomendação na linha 167: *“trocar `JdbcTemplate` por JPA”*.
- `architecture/fluxo_filiacao_e_inscricao_agremiacoes.md` — fluxo de filiação que depende dos vínculos
  instituição↔organização (BACK-01).
- Logs da VM: `docker logs flag-backend` (host `163.176.195.27`, usuário `opc`).

## Fora de escopo

- Carga de dados APFA/CBFA 2026 (pertence ao `flag_platform_agent`; usa este tracker apenas como
  desbloqueio).
