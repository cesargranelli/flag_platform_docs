# Mapa — Pendências do `flag_platform_infra`

> **Esforço:** `repos/infra` · **Prefixo:** `INFRA` · **Status:** aberto
> **Repo afetado:** [`cesargranelli/flag_platform_infra`](https://github.com/cesargranelli/flag_platform_infra)
> (hub de deploy)

## Issues

| ID | Tipo | Título | Repo(s) | Status |
|----|------|--------|---------|--------|
| [INFRA-01](issues/INFRA-01-app-version-build-hub.md) | bug | Public App: build do hub não passa `APP_VERSION` — versão exibida fica "dev" | flag_platform_infra, flag_public_app | open |
| [INFRA-02](issues/INFRA-02-upload-google-play-track-deprecado.md) | bug | Hub usa `track` (deprecado) no `upload-google-play` — migrar para `tracks` | flag_platform_infra | open |

## Notas do esforço

- Migrado da GitHub Issue **#37** do hub (`flag-platform-docs`) em 2026-10-03.
- INFRA-02 registrado a partir do release público **v0.7.1** (2026-10-04); run do hub:
  https://github.com/cesargranelli/flag_platform_infra/actions/runs/37167944372.
- Issues de operação/KB que **não** forem work items permanecem em `flag_platform_infra/docs/`
  (`runbook.md`, `kb/*`).

## Fora de escopo

- Runbooks e KB operacional — vivem em `flag_platform_infra/docs/`.
