# INFRA-02: Hub usa `track` (deprecado) no `upload-google-play` — migrar para `tracks`

> **Type:** bug
> **Status:** open
> **Effort:** repos/infra
> **Repo(s):** flag_platform_infra
> **Blocks:** —
> **Blocked by:** —
> **Origin:** release [`v0.7.1`](https://github.com/cesargranelli/flag_public_app/releases/tag/v0.7.1) · run [hub `37167944372`](https://github.com/cesargranelli/flag_platform_infra/actions/runs/37167944372) (`deploy_public_app` / `staging`)

## Sintoma

O passo de publicação na Google Play usa o input **`track`**, que é a forma **deprecada** do
`r0adkll/upload-google-play@v1`. O input atual é **`tracks`** (lista separada por vírgula, para
promoção multi-track). Manter `track` é dívida de manutenção: depende de shim de
compatibilidade da action e fica sujeito a remoção numa próxima major.

## Evidência (arquivo/linha)

`flag_platform_infra/.github/workflows/platform-deploy-hub.yml` — **duas** ocorrências:

- **Referee app** (linha 506):
  ```yaml
  501:        uses: r0adkll/upload-google-play@v1
  502:        with:
  503:          serviceAccountJsonPlainText: ${{ secrets.PLAY_STORE_JSON_KEY }}
  504:          packageName: br.com.flagplatform
  505:          releaseFiles: app/build/app/outputs/bundle/release/*.aab
  506:          track: ${{ needs.resolve-parameters.outputs.environment == 'production' && 'production' || 'internal' }}
  507:          status: completed
  ```
- **Public app** (linha 634):
  ```yaml
  629:        uses: r0adkll/upload-google-play@v1
  630:        with:
  631:          serviceAccountJsonPlainText: ${{ secrets.PLAY_STORE_JSON_KEY }}
  632:          packageName: br.com.flagplatform.flag_public_app
  633:          releaseFiles: app/build/app/outputs/bundle/release/*.aab
  634:          track: ${{ needs.resolve-parameters.outputs.environment == 'production' && 'production' || 'internal' }}
  635:          status: completed
  ```

## Impacto

- Dependência de comportamento deprecado em **dois** jobs de publicação (referee e public).
- Risco de falha silenciosa/erro quando a action remover o alias `track` numa atualização.
- Divergência com a API atual da action (documentação usa `tracks`).

## Correção proposta

Migrar as duas ocorrências de `track:` para **`tracks:`**, preservando a expressão de ambiente:

```yaml
          tracks: ${{ needs.resolve-parameters.outputs.environment == 'production' && 'production' || 'internal' }}
```

> O valor único (`internal`/`production`) continua válido como lista de 1 elemento.

## Critérios de aceite

- [ ] Ambas as ocorrências (linhas **506** e **634**) usam **`tracks:`** (sem `track:`).
- [ ] Comportamento idêntico validado em deploy: `staging` → **`internal`**; `production` → **`production`**.
- [ ] Run do hub (`deploy_public_app` e `deploy_referee_app`) conclui o upload com sucesso após a mudança.
- [ ] Sem `warning` de input deprecado nos logs do workflow.

## Referências

- `flag_platform_infra/.github/workflows/platform-deploy-hub.yml:506,634`
- Action: `r0adkll/upload-google-play@v1` (input `tracks`)
- Run do hub: https://github.com/cesargranelli/flag_platform_infra/actions/runs/37167944372
