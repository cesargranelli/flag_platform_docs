# FS-09: Pré-requisito de deploy — credencial Firebase no container

> **Type:** atividade
> **Status:** open
> **Effort:** firestore-sync
> **Repo(s):** flag_backend, flag_platform_infra
> **Blocks:** —
> **Blocked by:** —
> **Origin:** ADR-018 / PR #73

## Objetivo

Garantir que o container do backend em produção tenha a **credencial do Firebase Admin SDK**
disponível. Sem ela o espelho é **no-op** e o app público segue lendo a coleção `games` vazia.

## Contexto

- Revelado pela implementação do [ADR-018](../../../adr/ADR-018-espelhamento-firestore-admin-sdk.md)
  no PR #73 do `flag_backend` (commits `3489548`, `99c411c`).
- O desenho é **no-op sem credencial** (degradação graciosa para dev/CI). Isso é correto para o
  build, mas cria um **pré-requisito operacional de deploy**: a VM OCI precisa receber
  `FIREBASE_CREDENTIALS` (default `classpath:firebase-service-account.json`) apontando para o
  service account do Admin SDK.
- O service account é **gitignored** em
  `flag_backend/src/main/resources/firebase-service-account.json` (`.gitignore` linhas 26–27), então
  não viaja no artefato — precisa ser **montado/injetado** no container (volume/secret) em produção.
- Sintoma de falha silenciosa: build verde e API saudável, mas `games/{id}` continua vazio e o
  `FirestoreLiveGameRepository` do app não recebe documento.

## Critérios de aceitação

- [ ] Documentar o pré-requisito de deploy (variável/segredo + montagem do service account no
      container) onde vive a infra (`flag_platform_infra`).
- [ ] Container de produção sobe com a credencial e o espelho **não** é no-op (log/handshake do
      Admin SDK confirmando `FirebaseApp` inicializado).
- [ ] Backfill opt-in popula `games/{id}` após o deploy e o app público enxerga o documento.
- [ ] Rotação da credencial coberta pelo runbook de segredos (ver esforço `_archive/segredos-e-env`).

## Arquivos afetados (referência)

- `flag_backend/src/main/resources/application*.yml` — `FIREBASE_CREDENTIALS` / caminho padrão.
- `flag_backend/.gitignore` — linha do service account (referência, não alterar).
- `flag_platform_infra/**` — compose/Caddy/deploy da VM OCI (montagem do segredo).
- `flag_platform_docs/_planning/_archive/segredos-e-env` — governança de segredos.
