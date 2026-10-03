# R2-01: Padronizar 100% das imagens em `pub.america.app.br`

Type: task
Status: open

## Objetivo

Eliminar a divergência de domínio público das imagens: hoje convivem o **custom domain** canônico
`https://pub.america.app.br` (usado pelo upload em produção) e o domínio bruto
`https://pub-87dbed90a1fc49e8b64339efd0e3ed1c.r2.dev` (gravado em dados e em defaults de cliente).
Padronizar tudo em `pub.america.app.br`.

## Contexto

- O upload (`R2StorageClient.upload()`) devolve URL absoluta a partir de `R2_PUBLIC_URL`, que em
  produção é `https://pub.america.app.br`.
- Mas os **seeds do Liquibase** (`flag_backend/src/main/resources/db/changelog/changesets/002-production-data.yaml`,
  linhas ~20 e ~46) gravam `logo_url` com o domínio `pub-87…r2.dev`.
- Os **defaults dos clientes** também apontam para o domínio antigo:
  - `flag_public_app/lib/config/app_config.dart` → `STORAGE_BASE_URL` default `https://pub-87…r2.dev`.
  - `flag_referee_app/lib/config/app_config.dart` → `STORAGE_URL` default `https://pub-87…r2.dev`.
- O `.env.example` da infra ainda usa aliases legados `CLOUDFLARE_R2_*` e o placeholder
  `https://pub-xxx.r2.dev`.
- Efeito: imagens podem carregar por domínio diferente do canônico (cache/CDN inconsistente,
  troca de domínio futura mais dolorosa, risco de exposição do domínio `r2.dev`).

## Critérios de aceitação

- [ ] Seeds do Liquibase com `logo_url`/URLs de imagem sob `https://pub.america.app.br`.
- [ ] Defaults de `STORAGE_BASE_URL` (public app) e `STORAGE_URL` (referee app) apontando para
      `https://pub.america.app.br`.
- [ ] `.env.example` da infra com `R2_PUBLIC_URL=https://pub.america.app.br` (e aliases legados
      marcados como transitórios, conforme ADR-016).
- [ ] Nenhuma referência a `pub-*.r2.dev` em código/config versionado (varredura limpa).
- [ ] Dados existentes no banco migrados/atualizados (migration Liquibase de correção ou re-seed),
      se necessário.
- [ ] Imagem de teste carrega pelo domínio canônico no app público e no admin.

## Arquivos afetados (referência)

- `flag_backend/src/main/resources/db/changelog/changesets/002-production-data.yaml` (seeds).
- `flag_backend/src/main/resources/application*.yml` / `.env.example` (defaults de R2).
- `flag_platform_infra/.env.example` (`R2_PUBLIC_URL`, aliases legados).
- `flag_public_app/lib/config/app_config.dart` (`STORAGE_BASE_URL`).
- `flag_referee_app/lib/config/app_config.dart` (`STORAGE_URL`).
- `flag_admin_web` — verificar se há default de storage e alinhar.
