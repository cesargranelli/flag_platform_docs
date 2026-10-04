# PUB-01: `tool/release.ps1` falha na primeira release (sem tags `v*`)

> **Type:** bug
> **Status:** open
> **Effort:** repos/public_app
> **Repo(s):** flag_public_app
> **Blocks:** —
> **Blocked by:** —
> **Origin:** release [`v0.7.1`](https://github.com/cesargranelli/flag_public_app/releases/tag/v0.7.1) · run [satélite `37167838570`](https://github.com/cesargranelli/flag_public_app/actions/runs/37167838570)

## Sintoma

Na **primeira release** do app — quando o repositório ainda **não tem nenhuma tag `v*`** — o
`tool/release.ps1` aborta em vez de cair no fallback previsto (`range = 'HEAD'`). Como não havia
tag de versão anterior, a derivação do bump SemVer a partir dos commits convencionais não chega a
executar.

## Evidência (arquivo/linha)

`flag_public_app/tool/release.ps1`:

```powershell
49:  $ErrorActionPreference = 'Stop'
...
112: $lastTag = (git -C $root describe --tags --abbrev=0 --match "v*" 2>$null)
113: if ($lastTag) {
114:     $range = "$lastTag..HEAD"
...
116: } else {
117:     $range = 'HEAD'
118:     Write-Host "Nenhuma tag de versao anterior: analisando todo o historico."
119: }
```

O `else` (linhas 116–119) **já prevê** a ausência de tag, mas é **inalcançável**: sem tag, o
`git describe --match "v*"` retorna **exit code ≠ 0** e escreve `fatal: No names found, cannot
describe anything.` em stderr. Com `$ErrorActionPreference = 'Stop'` (linha 49), o PowerShell 5.1
promove esse erro nativo a erro **terminante** antes de `$lastTag` ser atribuído — o script morre
na linha 112.

## Impacto

Bloqueia o fluxo de versionamento exatamente no cenário mais crítico: **o primeiro release de um
repositório sem histórico de tags** (caso do `v0.7.1`). Sem workaround manual, o passos
`-Bump auto -Commit` e `-Tag` do `RELEASE.md §1` não rodam de forma limpa.

## Correção proposta

Tornar a detecção de tag tolerante a ausência/erro, garantindo o fallback documentado:

- Capturar o erro sem promovê-lo a terminante, por exemplo:
  ```powershell
  $lastTag = git -C $root describe --tags --abbrev=0 --match "v*" 2>$null
  if ($LASTEXITCODE -ne 0) { $lastTag = $null }
  ```
  (ou envolver a chamada em `try { } catch { $lastTag = $null }`, ou usar `--always`/`--exact-match`
  quando aplicável);
- Garantir que o caminho `range = 'HEAD'` (linha 117) passe a ser efetivamente exercitado;
- Cobrir com smoke (ver critérios).

## Critérios de aceite

- [ ] `tool/release.ps1 -Bump auto -DryRun` conclui **sem erro** num clone **sem nenhuma tag `v*`**,
      derivando o bump de todo o histórico (`range = 'HEAD'`).
- [ ] O caminho de fallback (linhas 116–119) é alcançado e reportado no console.
- [ ] Existe **smoke** cobrindo o cenário "sem tags" (ex.: script/checagem que roda `-DryRun` num
      clone limpo sem tags), executável no fluxo de release.
- [ ] `tool/release.ps1 -Tag` continua recusando tag duplicada e operando só na `main`.

## Referências

- `flag_public_app/tool/release.ps1:49,112-119`
- `flag_public_app/RELEASE.md §1` (fluxo GitFlow da versão)
- Run do release: https://github.com/cesargranelli/flag_public_app/actions/runs/37167838570
