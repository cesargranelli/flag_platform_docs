# Mapa — Pendências do `flag_public_app`

> **Esforço:** `repos/public_app` · **Prefixo:** `PUB` · **Status:** aberto
> **Repo afetado:** [`cesargranelli/flag_public_app`](https://github.com/cesargranelli/flag_public_app)
> **Origem:** achados no release público **v0.7.1** (staging → Play `internal`) em 2026-10-04.

Pendências técnicas observadas durante o primeiro release público do app
([release v0.7.1](https://github.com/cesargranelli/flag_public_app/releases/tag/v0.7.1)),
que orquestra o deploy via hub [`flag_platform_infra`](https://github.com/cesargranelli/flag_platform_infra).

**Runs de referência:**

- Satélite (app): [run `37167838570`](https://github.com/cesargranelli/flag_public_app/actions/runs/37167838570)
  — workflow `Flag Public App — CI & Publicação Google Play`, evento `release` (tag `v0.7.1`).
- Hub (infra): [run `37167944372`](https://github.com/cesargranelli/flag_platform_infra/actions/runs/37167944372)
  — workflow `Flag Platform — Hub Central de Deploys`, evento `repository_dispatch`
  (`deploy_public_app` / `staging`).

## Issues

| ID | Tipo | Título | Repo(s) | Status |
|----|------|--------|---------|--------|
| [PUB-01](issues/PUB-01-release-ps1-primeira-release.md) | bug | `tool/release.ps1` falha na primeira release (sem tags `v*`) | flag_public_app | open |
| [PUB-02](issues/PUB-02-ci-flutter-version-pin-ignorado.md) | bug | `ci.yml` usa `version:` (input inexistente) no `subosito/flutter-action` — pin do Flutter ignorado | flag_public_app | open |
| [PUB-03](issues/PUB-03-sha1-play-signing-firebase.md) | atividade | Confirmar SHA-1 da chave de assinatura do Play no Firebase após o 1º upload | flag_public_app | open |
| [PUB-04](issues/PUB-04-calendario-scheduledat-nulo.md) | bug | Calendário não carrega quando algum jogo tem `scheduledAt` nulo | flag_public_app, flag_backend | resolved |
| [PUB-05](issues/PUB-05-lance-a-lance-dados-incompletos.md) | bug | Lance a lance com dados incompletos (atletas, recebedores, tempo e pontuações) | flag_public_app, flag_referee_app | resolved |
| [PUB-06](issues/PUB-06-placar-ao-vivo-sem-refresh-tempo-real.md) | bug | Tela ao vivo e placares sem atualização em tempo real (fallback de polling e Firestore) | flag_public_app | resolved |
| [PUB-07](issues/PUB-07-pull-to-refresh-telas-public-app.md) | task | Revisão de telas sem atualização / adição de pull-to-refresh global | flag_public_app | resolved |
| [PUB-08](issues/PUB-08-padronizacao-cards-de-partida.md) | design | Padronização dos cards de partida com placar em destaque | flag_public_app | resolved |
| [PUB-09](issues/PUB-09-destaque-time-vencedor-cards.md) | design | Destaque visual do time vencedor nos cards de partidas encerradas | flag_public_app | resolved |
| [PUB-10](issues/PUB-10-rota-e-navegacao-ao-vivo.md) | bug | Correção das rotas e navegação para jogos ao vivo (`/live`) | flag_public_app | resolved |

## Notas do esforço

- Prefixo `PUB` inaugurado por este esforço (registrado no [README do tracker](../../README.md#52-issues-por-repositório-repos)).
- O `RELEASE.md` (`§5 — Checklist do Play Console`) já descreve a pendência coberta por PUB-03;
  o playbook de release é `flag_public_app/RELEASE.md`.
- Dívida correlata de versão no hub tem ticket próprio: [`INFRA-01`](../infra/issues/INFRA-01-app-version-build-hub.md)
  (build do hub não passa `APP_VERSION`).

## Fora de escopo

- Runbooks e KB de release do app vivem em `flag_public_app/RELEASE.md` e `flag_public_app/docs/`.
- Orquestração/deploy pelo hub pertence a `repos/infra/` (ver `INFRA-`).
