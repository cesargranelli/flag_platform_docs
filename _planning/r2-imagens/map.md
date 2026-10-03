# Mapa — Imagens servidas direto do Cloudflare R2

Effort de endurecimento e padronização do acesso público às imagens (logos, fotos de atletas etc.)
no Cloudflare R2.

> **Regra:** as issues deste mapa são trabalhadas **nesta ordem**. A detalhada tem arquivo em
> `issues/`; as demais ficam só na tabela até virarem fatia.

## Contexto verificado (2026-10-03)

- **Bucket de produção:** `america-platform` (`R2_BUCKET` no `.env` da infra).
- **Domínio público canônico:** `R2_PUBLIC_URL=https://pub.america.app.br` (custom domain sobre o bucket).
- **Upload:** `R2StorageClient.upload(byte[], key, contentType)` envia o objeto **sem `Cache-Control`**
  e devolve a **URL absoluta** montada a partir de `R2_PUBLIC_URL` (`{public-url}/{key}`).
- **Configuração de bucket:** **sem CORS** habilitado, **sem lifecycle/retenção** e **sem política de
  cache** no nível do bucket.
- **Dados e defaults ainda em `pub-87…r2.dev` (divergência):**
  - Seeds do Liquibase (`flag_backend/src/main/resources/db/changelog/changesets/002-production-data.yaml`,
    linhas ~20 e ~46) gravam `logo_url` apontando para `https://pub-87dbed90a1fc49e8b64339efd0e3ed1c.r2.dev/...`.
  - Default do app cliente (`flag_public_app/lib/config/app_config.dart`) usa o mesmo domínio
    `pub-87…r2.dev` como `R2_PUBLIC_URL` de fallback.
  - O `.env.example` da infra ainda usa os aliases legados `CLOUDFLARE_R2_*` e o placeholder
    `https://pub-xxx.r2.dev`.

**Estado desejado:** 100% das imagens resolvidas por `https://pub.america.app.br`, com CORS, cache e
retenção explícitos, e um contrato de URL único entre backend e os 3 clientes.

## Ordem de execução

1. **R2-01 — Padronizar 100% em `pub.america.app.br`** (dados + defaults). Detalhada em
   [`issues/01-padronizar-dominio.md`](issues/01-padronizar-dominio.md).
2. **R2-02 — CORS do bucket** (origens dos apps web).
3. **R2-03 — `Cache-Control` nos uploads** (política de cache por tipo de asset).
4. **R2-04 — Lifecycle/retenção + LGPD** (fotos de atletas).
5. **R2-05 — Contrato de URL** (guardar chave e resolver nos clientes vs URL absoluta).

## Issues

| ID | Tipo | Título | Critérios de aceitação | Status |
|----|------|--------|------------------------|--------|
| **R2-01** | task | Padronizar 100% das imagens em `pub.america.app.br` | Ver [issue detalhada](issues/01-padronizar-dominio.md) | open |
| **R2-02** | task | Habilitar CORS no bucket `america-platform` | CORS permite apenas as origens dos apps web (admin/public) e métodos `GET`/`HEAD`; assets carregam em canvas/build sem erro; sem wildcard aberto a `*` em produção | open |
| **R2-03** | task | Definir `Cache-Control` nos uploads | `R2StorageClient.upload()` envia `Cache-Control` por tipo (ex.: `public, max-age=…, immutable` para assets versionados); conteúdo mutável com TTL curto; documentado o porquê por classe de asset | open |
| **R2-04** | ops | Lifecycle/retenção e LGPD (fotos de atletas) | Regra de lifecycle no bucket (expiração/limpeza de órfãos); fotos de atletas tratadas como dado pessoal (retenção, remoção sob solicitação, sem indexação pública indevida); documentado no contrato de dados | open |
| **R2-05** | design | Definir contrato de URL (chave vs URL absoluta) | Decidido se o backend grava a **chave** e cada cliente resolve a URL (facilita trocar domínio/CDN) ou se grava a URL absoluta (como hoje); portado para backend + 3 clientes; migração dos dados existentes conforme a decisão | open |

## Fora de escopo

- Migração para outro provedor de storage (Cloudflare R2 é decisão vigente).
- Processamento de imagem (redimensionamento/otimização) — pode virar issue futura, não aqui.
- Mudança do modo de upload (presigned URL pelo cliente) — hoje o upload passa pelo backend.
