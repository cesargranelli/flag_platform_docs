# INFRA-01: Public App — build do hub não passa `APP_VERSION` (versão exibida fica "dev")

> **Type:** bug
> **Status:** open
> **Effort:** repos/infra
> **Repo(s):** flag_platform_infra, flag_public_app
> **Blocks:** —
> **Blocked by:** —
> **Origin:** GitHub Issue #37 (`cesargranelli/flag-platform-docs`)

**Projeto afetado:** `flag_platform_infra` (hub de deploy) · **Repo consumidor:** `flag_public_app`

## Contexto

O Public App deriva a versão exibida na tela **"Sobre"** de um `--dart-define=APP_VERSION`:

- `flag_public_app/lib/config/app_config.dart` → `appVersion = String.fromEnvironment('APP_VERSION', defaultValue: 'dev')`
- `flag_public_app/lib/ui/about/view_models/about_view_model.dart` → `version => AppConfig.appVersion`

O workflow do hub passa três defines no build, mas **não passa `APP_VERSION`**:

```yaml
# flag_platform_infra/.github/workflows/platform-deploy-hub.yml (3 pontos de build)
--dart-define=ENVIRONMENT="${ENV_TARGET}" \
--dart-define=API_BASE_URL="${API_URL}" \
--dart-define=FIREBASE_PROJECT_ID="${FIREBASE_PID}"
```

## Impacto

O app **publicado** exibiria **"Versão dev"** na tela "Sobre", mesmo com o `pubspec.yaml` em `0.6.0+3`.
Além do efeito visual, prejudica suporte/diagnóstico: não é possível saber, pelo app, qual build o
usuário está usando.

## Observação relacionada (mesma família de problema)

No repo do app, o `ci.yml` calcula a versão do deploy a partir da **última tag `v*`**:

```bash
TAG=$(git describe --tags --abbrev=0 --match 'v[0-9]*' 2>/dev/null || echo "staging-$(git rev-parse --short "$COMMIT_SHA")")
```

Hoje **não existe nenhuma tag `v*`** no `flag_public_app`, então esse fallback produziria
`staging-<sha>` em vez do semver do `pubspec` (`0.6.0+3`). Isso não faz a publicação falhar, mas
quebra a rastreabilidade versão ↔ build.

## Opções

| # | Onde | Mudança | Prós | Contras |
|---|---|---|---|---|
| **(a)** | Infra | adicionar `--dart-define=APP_VERSION="${VERSION}"` nos 3 pontos de build (a versão **já chega** no `client_payload` do dispatch) | pequeno, 1 linha por ponto, sem tocar no app | depende de o payload trazer um semver real |
| **(b)** | App | trocar a fonte para **`package_info_plus`** (lê a versão instalada do binário) | imune a CI, nunca desincroniza | +1 dependência; `AboutViewModel` passa a ser assíncrono |

## Recomendação

**(a)** como correção imediata (destrava a publicação com a versão correta) e **(b)** como evolução,
para eliminar a classe de erro. A criação de tags `vX.Y.Z` (semver do `pubspec`) deve acompanhar a
decisão, para o payload carregar uma versão rastreável.

## Critérios de aceite

- [ ] O build do hub passa `APP_VERSION` com o semver do payload (ou o app passa a ler a versão instalada)
- [ ] O app publicado exibe a versão correta na tela "Sobre" (ex.: `Versão 0.6.0`) — **sem** mostrar `dev`
- [ ] A versão reportada ao hub é rastreável (semver/tag), não `staging-<sha>`
- [ ] Documentado no `RELEASE.md` do app qual é a fonte da versão exibida

## Referências

- App: `lib/config/app_config.dart`, `lib/ui/about/view_models/about_view_model.dart`, `RELEASE.md` (§1 e §4)
- Hub: `flag_platform_infra/.github/workflows/platform-deploy-hub.yml` (jobs de build do `public_app`)
- CI do app: `flag_public_app/.github/workflows/ci.yml` (`validate-trigger`)
- Contexto de versão: `flag_public_app/tool/release.ps1` (versionCode sempre crescente; bump SemVer por commits convencionais)
