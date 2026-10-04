# PUB-02: `ci.yml` usa `version:` (input inexistente) no `subosito/flutter-action` — pin do Flutter ignorado

> **Type:** bug
> **Status:** open
> **Effort:** repos/public_app
> **Repo(s):** flag_public_app
> **Blocks:** —
> **Blocked by:** —
> **Origin:** release [`v0.7.1`](https://github.com/cesargranelli/flag_public_app/releases/tag/v0.7.1) · run [satélite `37167838570`](https://github.com/cesargranelli/flag_public_app/actions/runs/37167838570)

## Sintoma

O job `build` do CI declara o SDK do Flutter com a chave **`version:`**, que **não existe** nos
inputs da action `subosito/flutter-action@v2` (o input correto é **`flutter-version:`**). O valor
`3.41.6` é **silenciosamente ignorado** e a action instala o **último `stable`** disponível.

## Evidência (arquivo/linha)

`flag_public_app/.github/workflows/ci.yml`:

```yaml
126:      - name: Setup Flutter
127:        uses: subosito/flutter-action@v2
128:        with:
129:          channel: stable
130:          version: '3.41.6'      # ← input inexistente; o correto é 'flutter-version:'
131:          cache: true
```

> Entrada desconhecida em `with:` é apenas **ignorada** pela action (sem falha do workflow), então o
> CI passa mesmo com o pin errado — o problema é invisível no status verde.

## Impacto

- O CI **não reproduz** o SDK pinado do projeto: builds locais e de CI podem divergir.
- Cada execução usa o `stable` mais recente do dia → **quebras intermitentes** quando um novo
  `stable` do Flutter muda analyzer/format/API.
- O pin de `3.41.6` deixa de ser fonte de verdade para reprodução de build.

## Correção proposta

Renomear a chave em `.github/workflows/ci.yml` (linha 130):

```yaml
        with:
          channel: stable
          flutter-version: '3.41.6'
          cache: true
```

Confirmar que o hub já usa o input correto como referência
(`flag_platform_infra/.github/workflows/platform-deploy-hub.yml` — `flutter-version: '3.41.6'`).

## Critérios de aceite

- [ ] `ci.yml` passa a usar **`flutter-version:`** (e não `version:`) no `subosito/flutter-action`.
- [ ] O log do job `build` evidencia o SDK instalado como **3.41.6** (ex.: `flutter --version`).
- [ ] O valor do pin fica **alinhado** com o usado no hub de deploy, evitando divergência de SDK.
- [ ] Workflow verde com o pin efetivo (não `stable`).

## Referências

- `flag_public_app/.github/workflows/ci.yml:126-131`
- `flag_platform_infra/.github/workflows/platform-deploy-hub.yml` (`flutter-version: '3.41.6'`)
- Run: https://github.com/cesargranelli/flag_public_app/actions/runs/37167838570
