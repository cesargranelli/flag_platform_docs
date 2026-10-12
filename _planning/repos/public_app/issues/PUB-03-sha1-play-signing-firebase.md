# PUB-03: Confirmar SHA-1 da chave de assinatura do Play no Firebase após o 1º upload

> **Type:** atividade
> **Status:** open
> **Effort:** repos/public_app
> **Repo(s):** flag_public_app
> **Blocks:** —
> **Blocked by:** —
> **Origin:** release [`v0.7.1`](https://github.com/cesargranelli/flag_public_app/releases/tag/v0.7.1) · `flag_public_app/RELEASE.md §5`

## Contexto

Com o **Play App Signing** (padrão), o Google **re-assina** o app distribuído com uma chave própria,
diferente da **upload key** usada no build. O Firebase só reconhece o app distribuído se o **SHA-1
da chave de assinatura do Play** estiver cadastrado no projeto Firebase `flag-platform`.

O `RELEASE.md` já marca a upload key como cadastrada, mas deixa **em aberto** o cadastro do SHA-1 do
Play **após o primeiro upload** (§5):

```text
188: - [ ] **SHA-1 da upload key** cadastrado no Firebase (projeto `flag-platform`, app Android `flag_public_app`) — ✅ já feito
189: - [ ] ⚠️ **SHA-1 da chave de assinatura do Play** cadastrado no Firebase **depois do primeiro upload**
...
192:       Sem isso, **Auth/App Check não funcionam** no app publicado (mesmo funcionando em builds locais).
193:       Cadastrar: `firebase apps:android:sha:create <appId> <sha1> --project flag-platform`
```

## Ação (atividade operacional)

1. Publicar/confirmar o primeiro upload no Play (`internal`) — feito no release `v0.7.1`.
2. Coletar o SHA-1 em: **Play Console → App integrity → App signing key certificate → SHA-1**.
3. Cadastrar no Firebase (`flag-platform`, app Android `flag_public_app`):
   `firebase apps:android:sha:create <appId> <sha1> --project flag-platform`
   (o `appId` sai de `firebase apps:list`; conferir com `firebase apps:android:sha:list`).
4. Validar Auth/App Check em **build distribuído pelo Play** (não apenas build local).

## Critérios de aceite

- [ ] SHA-1 da **chave de assinatura do Play** (App integrity) cadastrado no Firebase `flag-platform`
      para o app Android `flag_public_app` (verificável em `firebase apps:android:sha:list`).
- [ ] **Login (Auth) e App Check** funcionam em build instalado via **Play (internal)** com
      `USE_MOCK_DATA=false`.
- [ ] `RELEASE.md §5` atualizado marcando o item como concluído.

## Notas

- **Não** registrar valores de segredo/hashes nos tickets — apenas a referência ao local de consulta.
- O `flag-platform` aqui é o **projeto Firebase** (não um repositório); o repo consumidor é o
  `flag_public_app`.

## Referências

- `flag_public_app/RELEASE.md §5` (Checklist do Play Console, linhas 188–194)
- `flag_public_app/RELEASE.md §2.1` (keystore de upload e SHA-1/SHA-256)
- Release: https://github.com/cesargranelli/flag_public_app/releases/tag/v0.7.1
