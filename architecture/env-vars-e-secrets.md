# Variáveis de Ambiente e Gestão de Segredos — Flag Platform

> **Status:** Ativo
> **Casa oficial:** `flag_platform_docs/architecture/` (documento transversal canônico)
> **Última atualização:** 2026-09-29
> **Escopo:** convenção de nomes, matriz de chaves, decisão Secret × Variable, ambientes,
> runbooks de rotação/incidente e higiene local para todos os repositórios da plataforma.

Este documento é a **fonte única de verdade** sobre nomes de variáveis e onde cada
credencial vive. Detalhes específicos de runtime do backend continuam em
[`flag_backend/docs/environment-variables.md`](https://github.com/cesargranelli/flag_backend/blob/main/docs/environment-variables.md);
o catálogo de credenciais por provedor vive no KB da infra (`flag_platform_infra/docs/kb/credentials-and-secrets.md`).

---

## 1. Princípios

1. **GitHub Environments é a fonte única de segredos dos deploys.** Segredos/variáveis usados
   por CI/CD vivem nos *Environment Secrets/Variables* (ou no nível do repositório quando
   valem para todos os ambientes). Nunca em arquivo versionado.
2. **Nada de segredo em repo ou workflow.** Workflows referenciam `${{ secrets.* }}` /
   `${{ vars.* }}`; nunca contêm valores literais. Arquivos `.env` reais são **sempre**
   ignorados pelo Git (apenas `.env.example` é versionado).
3. **Fail-fast, sem fallback.** Se um segredo obrigatório não estiver configurado, o job
   **aborta imediatamente** (`exit 1`) com `::error::`. É terminantemente proibido embutir
   valores de contingência/`default` para "fazer passar".
4. **`gh secret set` por stdin.** Valores nunca vão em `--body` (ficam visíveis em `argv` e
   no histórico). Use `valor | gh secret set CHAVE` ou `gh secret set CHAVE < arquivo`.
5. **WIF/OIDC em vez de chave estática.** Preferir identidade federada (Workload Identity
   Federation) a chaves de longa duração sempre que o provedor suportar.
6. **Menor privilégio.** Cada credencial deve ter apenas os escopos necessários e viver no
   repositório/ambiente que a consome.

---

## 2. Convenção de nomes

- Formato: **`DOMINIO_CHAVE`** em **`UPPER_SNAKE_CASE`** (ex.: `DATASOURCE_PASSWORD`,
  `R2_ACCESS_KEY_ID`, `PLAY_STORE_JSON_KEY`).
- Um domínio por prefixo (`DATASOURCE_`, `R2_`, `OCI_`, `CLOUDFLARE_`, `FIREBASE_`, `GCP_`).
- Sufixos: `_URL`, `_HOST`, `_PORT`, `_USERNAME`, `_PASSWORD`, `_TOKEN`, `_KEY`, `_ID`,
  `_REGION`, `_BASE64`, `_KEY_PROPERTIES`.

### 2.1 Nomes canônicos e deprecados

| Domínio | **Canônico** | Deprecado (não usar em código novo) |
|---|---|---|
| Banco | `DATASOURCE_URL`, `DATASOURCE_USERNAME`, `DATASOURCE_PASSWORD` | `SPRING_DATASOURCE_URL`, `SPRING_DATASOURCE_USERNAME`, `SPRING_DATASOURCE_PASSWORD`, `DB_ADMIN_PASSWORD`, `DB_*`, `DATABASE_*` |
| Object storage | `R2_ENDPOINT`, `R2_ACCESS_KEY_ID`, `R2_SECRET_ACCESS_KEY`, `R2_BUCKET`, `R2_PUBLIC_URL`, `R2_REGION` | `CLOUDFLARE_R2_ENDPOINT`, `CLOUDFLARE_R2_ACCESS_KEY_ID`, `CLOUDFLARE_R2_SECRET_ACCESS_KEY`, `CLOUDFLARE_R2_BUCKET_NAME`, `CLOUDFLARE_R2_PUBLIC_URL` |
| Conexão JDBC legada | `DATASOURCE_URL` | `DATABASE_URL`, `DATABASE_USER`, `DATABASE_PASSWORD` |

> **Compatibilidade transitória:** o backend ainda aceita `SPRING_DATASOURCE_*` como
> *fallback* em `application-staging.yml` / `application-production.yml`, e o Hub ainda
> resolve `CLOUDFLARE_R2_*` como alias. Isso existe **apenas** para não quebrar deploys
> durante a migração. Ao tocar nesses arquivos, migre para o nome canônico.

---

## 3. Matriz de chaves

Legenda de tipo: **S** = Secret, **V** = Variable. "staging/production" indica onde a chave
deve existir (o segredo pode ter valores distintos por ambiente).

> Valores nunca são documentados. Onde aparece `*_BASE64` / `*_KEY_PROPERTIES`, o conteúdo é
> o arquivo materializado (ex.: keystore ou `key.properties`).

### 3.1 `flag_platform_infra` (Hub de deploy) — repositório central

| Chave | Tipo | staging | production | Observação |
|---|---|---|---|---|
| `SSH_PRIVATE_KEY` | S | ✅ | ✅ | Acesso SSH à VM OCI |
| `DATASOURCE_PASSWORD` | S | ✅ | ✅ | Senha do datasource do backend (canônico) |
| `OCI_VM_HOST` | V | ✅ | ✅ | Host da VM OCI |
| `OCI_VM_USER` | V | ✅ | ✅ | Usuário SSH (default `opc`) |
| `FIREBASE_PROJECT_ID` | V | ✅ | ✅ | Projeto Firebase/Auth |
| `API_BASE_URL` | V | ✅ | ✅ | URL da API para builds Flutter |
| `CLOUDFLARE_PROJECT_NAME` | V | ✅ | ✅ | Nome do projeto no Cloudflare Pages |
| `R2_ENDPOINT` | V | ✅ | ✅ | Endpoint S3-compatible |
| `R2_BUCKET` | V | ✅ | ✅ | Bucket de uploads |
| `R2_PUBLIC_URL` | V | ✅ | ✅ | URL pública dos assets |
| `R2_REGION` | V | ✅ | ✅ | Default `auto` |
| `R2_ACCESS_KEY_ID` | S | ✅ | ✅ | Credencial R2 |
| `R2_SECRET_ACCESS_KEY` | S | ✅ | ✅ | Credencial R2 |
| `CLOUDFLARE_API_TOKEN` | S | ✅ | ✅ | Deploy Pages / R2 / DNS |
| `CLOUDFLARE_ACCOUNT_ID` | S | ✅ | ✅ | Conta Cloudflare |
| `PLAY_STORE_JSON_KEY` | S | ✅ | ✅ | Service Account JSON (Play Developer API) |
| `REFEREE_APP_KEYSTORE_BASE64` | S | ✅ | ✅ | Keystore de release (Base64) |
| `REFEREE_APP_KEY_PROPERTIES` | S | ✅ | ✅ | `key.properties` do Referee App |
| `PUBLIC_APP_KEYSTORE_BASE64` | S | ✅ | ✅ | Keystore de release (Base64) |
| `PUBLIC_APP_KEY_PROPERTIES` | S | ✅ | ✅ | `key.properties` do Public App |
| `INFRA_DISPATCH_TOKEN` | S | ✅ | ✅ | PAT *actions:write* (ver 3.2) |

### 3.2 Repositórios satélites (`flag_backend`, `flag_admin_web`, `flag_referee_app`, `flag_public_app`, `flag_tester_e2e`)

| Chave | Tipo | staging | production | Observação |
|---|---|---|---|---|
| `INFRA_DISPATCH_TOKEN` | S | ✅ | ✅ | **Único** segredo necessário no satélite: PAT com `actions:write` + `contents:read` sobre `flag_platform_infra`, usado no `repository_dispatch`. Configurar como *Repository Secret* (vale para todos os ambientes/branches). |

### 3.3 Credenciais operacionais (uso por agente/CLI, não por workflow)

Vivem **apenas** em `.env` local (nunca versionado) e/ou no gerenciador de segredos do
operador. Exemplos de domínios: `GCP_ACCESS_TOKEN`, `GCP_SERVICE_ACCOUNT`,
`GCP_WORKLOAD_IDENTITY_PROVIDER`, `CLOUDFLARE_API_TOKEN`, `HETZNER_API_TOKEN`,
`NEON_API_KEY`, `SUPABASE_ACCESS_TOKEN`, `OCI_AUTH_TOKEN`, `OCI_TENANCY_OCID`,
`OCI_USER_OCID`, `OCI_FINGERPRINT`, `OCI_PRIVATE_KEY`, `OCI_COMPARTMENT_ID`.

---

## 4. Regra de decisão Secret × Variable

| Critério | Secret | Variable |
|---|---|---|
| Token, senha, chave privada, JSON de service account | ✅ | ❌ |
| String de conexão **com** senha | ✅ | ❌ |
| Keystore / `key.properties` (Base64 ou conteúdo) | ✅ | ❌ |
| Host, porta, região, nome de projeto/serviço/bucket | ❌ | ✅ |
| ID público (project id, account id usado só como config) | ❌ | ✅ |
| URL pública (base URL, endpoint, public URL) | ❌ | ✅ |
| Chave localStorage no runner? (não pode ser lida depois) | ✅ | ❌ |

**Regra de ouro:** se o valor autoriza acesso, é **Secret**; se o valor apenas localiza/nomeia
um recurso, é **Variable**. Na dúvida, use Secret (o default seguro).

> As chaves devem ser referenciadas no workflow pelo mesmo tipo em que foram cadastradas.
> Trocar o tipo de uma chave exige ajustar `${{ secrets.X }}` ↔ `${{ vars.X }}`.

---

## 5. GitHub Environments & proteção

- Ambientes usados pelos workflows: **`staging`** e **`production`**
  (`platform-deploy-hub.yml` define `environment: ${{ ... }}`).
- **`production`** deve ter **Required reviewers** habilitados (aprovação humana antes do
  deploy) e, quando disponível, *Deployment branches* restritas (`main`).
- Preferir **OIDC/WIF** para provedores de nuvem em vez de chaves estáticas de longa duração
  (ver `flag_platform_infra/docs/kb/gcp-cloud-run-wif.md`).
- Escopo: usar Environment Secret quando o valor difere por ambiente; Repository Secret
  quando é o mesmo em todos os ambientes (ex.: `INFRA_DISPATCH_TOKEN` nos satélites).
- Toda alteração de ambiente em `production` deve ser rastreável (PR + revisão).

---

## 6. Runbooks

### 6.1 Rotação de credencial (procedimento)

> **Estado atual:** a rotação está **conscientemente adiada** (ver ADR-016). Este runbook
> descreve o procedimento para quando for executado.

1. **Gerar** a nova credencial no provedor (com escopo mínimo e data de expiração quando
   aplicável).
2. **Cadastrar** no GitHub via stdin (nunca `--body`):
   ```bash
   # Secret de repositório
   gh secret set DATASOURCE_PASSWORD --repo cesargranelli/flag_platform_infra < ~/.secrets/novo_valor
   # Secret por ambiente
   gh secret set CLOUDFLARE_API_TOKEN --env production --repo cesargranelli/flag_platform_infra < ~/.secrets/novo_valor
   ```
   Ou use `flag_platform_infra/scripts/sync-github-secrets.ps1 -Environment production`.
3. **Validar** com um deploy em `staging` (workflow_dispatch do Hub a partir do satélite).
4. **Propagar** para `production` e validar health check / smoke test.
5. **Revogar** a credencial antiga no provedor **somente após** a validação.
6. **Registrar** no tracker `flag_platform_docs/_planning/_archive/segredos-e-env/` (data, chave,
   motivo, quem executou).
7. Atualizar **backups offline** (keystores, service accounts) no cofre do operador.

### 6.2 Resposta a incidente (vazamento suspeito)

1. **Conter:** revogar/suspender a credencial afetada no provedor imediatamente.
2. **Rotacionar** a credencial (6.1) priorizando ambientes de produção.
3. **Avaliar escopo:** logs de acesso do provedor; o que a credencial podia acessar; por
   quanto tempo ficou exposta.
4. **Remover** a exposição (arquivo local, cache, artefato, branch).
5. **Notificar** o responsável e, se houver dado pessoal envolvido, acionar o fluxo LGPD.
6. **Post-mortem** sem culpa + ação preventiva no tracker.

---

## 7. Higiene local (o que NÃO pode ficar em disco/workspace)

- ❌ `.env` com valores reais **dentro** de qualquer repositório (mesmo que pareça ignorado —
  confirme com `git check-ignore`).
- ❌ Keystores (`*.jks`, `*.keystore`, `*.p12`), `key.properties`, chaves privadas
  (`*.pem`, `*.key`, `*.sso`) na árvore de trabalho de um repo.
- ❌ Tokens/PATs hardcoded em scripts locais ou na URL de `remote` do `.git/config`.
- ❌ Dumps de banco (`*.sql`) com dados reais/PII (ex.: `supabase_full_backup.sql`).
- ❌ Service accounts JSON (`*-adminsdk-*.json`, `google-services*.json` de backend) fora do
  cofre.

**Regras:**
1. Toda credencial em disco deve estar **fora do repositório** (ex.: `~/.secrets/`,
   `%USERPROFILE%\.android-keys\`) e criptografada/backupeada no cofre do operador.
2. `.gitignore` de cada repo deve cobrir `.env*` (com `!.env.example`), keystores e chaves.
3. Antes de pedir suporte/compartilhar logs, **redija** valores de segredo.
4. Prefira `gh auth login` (credencial no keyring) a PAT embutido em script/comando.

---

## 8. Referências

- [`flag_backend/docs/environment-variables.md`](https://github.com/cesargranelli/flag_backend/blob/main/docs/environment-variables.md) — contrato `DATASOURCE_*` e variáveis por perfil.
- `flag_platform_infra/docs/kb/credentials-and-secrets.md` — catálogo de credenciais por provedor.
- `flag_platform_infra/docs/kb/supabase-postgres.md` — conexão Supabase/Postgres.
- `flag_platform_infra/docs/kb/gcp-cloud-run-wif.md` — WIF/OIDC do GCP.
- `flag_platform_infra/docs/kb/dispatch-and-automation.md` — dispatch entre repositórios.
- `flag_platform_infra/scripts/sync-github-secrets.ps1` — sincronizador (stdin).
- [ADR-016](../adr/ADR-016-gestao-segredos-github-environments.md) — decisão de arquitetura.
- `flag_platform_docs/_planning/_archive/segredos-e-env/` — tracker da auditoria.
