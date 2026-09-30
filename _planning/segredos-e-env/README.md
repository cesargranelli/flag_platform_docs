# Auditoria de Env & Secrets — Tracker

> **Status:** Sanitização concluída (rotação/purga adiadas por decisão do responsável)
> **Data:** 2026-09-29
> **Responsável pela decisão:** Cesar Granelli
> **Execução:** DevOps Engineer (Flag Platform)
> **Regra:** este documento **nunca** contém valores de segredos (tudo redigido).

---

## 1. Riscos identificados

| ID | Risco | Severidade | Estado |
|---|---|---|---|
| **R1** | Segredos em **arquivos rastreados**: fallbacks de `key.properties` (senha de keystore) no workflow do Hub; senha real de datasource como default no `docker-compose.yml`; senha real de banco + host interno em `application-dev.yml` de repositório legado. | Alta | **Mitigado** (sanitizado) |
| **R2** | Segredos em **estado local/workspace**: PAT hardcoded na URL de `remote` do `.git/config` de um repo legado; scripts `setup-github-secrets*` com PAT e senha em texto puro; artefatos na raiz (chaves `.pem`/`.key`, keystore Base64, service accounts JSON, senha de banco em `.txt`, dump `.sql`). | Alta | **Parcial** (scripts sanearam; demais são higiene manual) |
| **R3** | **Rotação e purga de histórico adiadas**: credenciais expostas seguem válidas e o histórico Git ainda contém os valores antigos. | Média | **Aceito conscientemente** |

---

## 2. Achados da sanitização

### 2.1 Aplicado

| # | Alvo | Ação |
|---|---|---|
| a | `flag_platform_infra/.github/workflows/platform-deploy-hub.yml` | Remoção dos fallbacks que gravavam `key.properties` com senha de keystore em texto puro (blocos Referee e Public App) → agora **fail-fast** via `::error::` + `exit 1`. Injeção obrigatória de `DATASOURCE_PASSWORD` no `.env` de deploy do backend, com fail-fast. |
| b | `flag_platform_infra/docker/backend/docker-compose.yml` | Removido default com senha real. Passou a exigir `DATASOURCE_PASSWORD` (`${DATASOURCE_PASSWORD:?...}`) e nomes canônicos `DATASOURCE_URL`/`DATASOURCE_USERNAME`. |
| c | `flag_backend/.env.example`, `flag_platform_infra/.env.example` | Identificadores reais → placeholders (account id/URL pública do R2; GCP service account; host do Supabase). Chaves mantidas. Novo `DATASOURCE_PASSWORD=` no template da infra. |
| d | `.gitignore` de `flag_backend`, `flag_platform_infra`, `flag_admin_web` | `flag_backend`: passou a ignorar `.env`, `.env.*` (`!.env.example`). `flag_platform_infra`: `.env*` + `*.p12`, `*.sso`, `*.jks`, `*.keystore`, `*.pem`, `keystore*`. `flag_admin_web`: `*.jks`, `*.keystore`. |
| e | `flag_platform_infra/scripts/sync-github-secrets.ps1` | Reescrito para enviar valores por **stdin** (sem `--body`), com `-Environment staging\|production`, classificação Secret/Variable, convenção `@file:` para segredos multi-linha e `-DryRun`. |
| e' | `setup-github-secrets.ps1` / `setup-github-secrets-api.ps1` / `setup-github-secrets.py` (raiz do workspace) | Neutralizados (esvaziados de credenciais/PAT) e redirecionados ao script canônico. |
| f | `flag_platform/backend/src/main/resources/application-dev.yml` (achado extra) | Removidos host RDS interno + usuário/senha reais; passou a usar `${DATASOURCE_*}` com defaults de `localhost`. |

### 2.2 Verificação pós-sanitização (`git grep` em arquivos rastreados)

Padrões varridos (sem citar os valores): fragmentos das senhas antigas de banco e de
keystore, variável `SPRING_DATASOURCE_PASSWORD=`, `storePassword=`/`keyPassword=`, prefixo de
PAT do GitHub, termo de wallet do OCI, extensões `.jks`/`.keystore`/`.pem`, `service-account`,
prefixo de API key do Firebase e cabeçalhos de chave privada.

Resultado: **zero segredo real remanescente**. Os matches que sobraram são **benignos**:

- Nomes de arquivo de keystore (`flag-*-release.jks`, `flag-public-upload.jks`) e regras
  `*.jks` em `.gitignore`.
- Documentação de assinatura com placeholders (`storePassword=SUA_SENHA`) e script
  `new-keystore.ps1` que usa a **variável** `$password` (não um literal).
- `service-account` como **nome de arquivo** em `.gitignore`/docs/config (`firebase-service-account.json`).
- Firebase client API keys (`AIzaSy…`) em `google-services.json` / `firebase_options.dart`:
  são **públicas por design** (identificam o app; a segurança vem das restrições no console),
  não são segredos.

---

## 3. Decisões do responsável

1. **Sanitização:** APROVADA e aplicada.
2. **Rotação de credenciais:** **ADIADA** (ambientes controlados; sem janela de rotação agora).
3. **Purga de histórico Git:** **ADIADA** (evitar reescrita de histórico compartilhado).
4. **Documentação:** APROVADA — doc canônico + ADR-016 criados.

---

## 4. Pendências / próximos passos

| # | Pendência | Dono | Observação |
|---|---|---|---|
| P1 | Cadastrar `DATASOURCE_PASSWORD` nos Environments (staging/production) de `flag_platform_infra` | Operador | **Bloqueia** o próximo deploy do backend (fail-fast intencional). Comando em §5 (não executado). |
| P2 | Rodar a **rotação** das credenciais expostas (runbook §6.1) | Operador | Adiada; executar em janela planejada ou ao primeiro sinal de incidente. |
| P3 | **Purgar histórico** Git dos valores antigos | Operador | Adiada; requer `git filter-repo`/BFG + force-push coordenado. |
| P4 | Higiene local: remover/cofrar artefatos da raiz do workspace e limpar o PAT da URL de `remote` do repo legado | Operador | Ver §6. |
| P5 | Migrar aliases remanescentes (`SPRING_DATASOURCE_*`, `CLOUDFLARE_R2_*`) para canônicos ao tocar nos arquivos | Time | Ver matriz de nomes no doc canônico. |

---

## 5. Comandos `gh` preparados (NÃO executados)

> Enviar &#8203;valor por stdin. Nenhum valor é registrado neste tracker.

```bash
# P1 — cadastrar a senha do datasource (canônica) por ambiente
gh secret set DATASOURCE_PASSWORD --repo cesargranelli/flag_platform_infra --env staging    < ~/.secrets/datasource_password_staging
gh secret set DATASOURCE_PASSWORD --repo cesargranelli/flag_platform_infra --env production < ~/.secrets/datasource_password_production

# Variáveis de destino de deploy (não-secretas), se ainda não cadastradas
gh variable set OCI_VM_HOST --repo cesargranelli/flag_platform_infra --env production < ~/.secrets/oci_vm_host
gh variable set OCI_VM_USER --repo cesargranelli/flag_platform_infra --env production < ~/.secrets/oci_vm_user

# Alternativa: sincronizar tudo de um .env (script canônico, valores por stdin)
#   cd flag_platform_infra
#   .\scripts\sync-github-secrets.ps1 -EnvFile .env -Environment staging -DryRun   # simular
#   .\scripts\sync-github-secrets.ps1 -EnvFile .env -Environment staging           # aplicar
```

---

## 6. Higiene local — itens a tratar manualmente (sem valores)

Fora de qualquer repositório rastreado, porém sensíveis se expostos/backup:

- Chaves: arquivos `*.pem`, `*.key` na raiz do workspace (SSH/OCI) e `*.pub`.
- Keystores: `public_keystore_base64.txt`, `referee_keystore_base64.txt`.
- Service accounts: `firebase-service-account.json`, `google-service.json`,
  `google-services-referee-app.json`, `*-firebase-adminsdk-*.json`.
- Credenciais soltas: `supabase_america_platform_pass.txt`.
- Dump de banco: `supabase_full_backup.sql`, `ddl_data_base_main_v1.sql`.
- **PAT hardcoded** na URL do `remote` em `flag_platform/.git/config` (limpar para URL HTTPS
  sem token; sem alterar git config sem confirmação).
- `.env` real em `flag_platform_infra/` (mantido fora do Git pelo `.gitignore`).

---

## 7. Referências

- Doc canônico: [`architecture/env-vars-e-secrets.md`](../../architecture/env-vars-e-secrets.md)
- Decisão: [`adr/ADR-016-gestao-segredos-github-environments.md`](../../adr/ADR-016-gestao-segredos-github-environments.md)
- KB de credenciais: `flag_platform_infra/docs/kb/credentials-and-secrets.md`
- Contrato do backend: `flag_backend/docs/environment-variables.md`
