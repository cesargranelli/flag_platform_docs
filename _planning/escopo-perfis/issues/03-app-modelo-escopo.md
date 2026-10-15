# ESC-03: App — modelo `AppScope` + persistência do vínculo

> **Type:** feature
> **Status:** resolved
> **Effort:** escopo-perfis
> **Repo(s):** flag_admin_web
> **Blocks:** ESC-04, ESC-05, ESC-06
> **Blocked by:** ESC-01
> **Origin:** [_planning/escopo-perfis/plan.md](../plan.md) §2.2

## Contexto

`AuthUser` tem `organizationId`/`institutionId` declarados mas sempre `null`; `SessionManager`
não os persiste; não existe no app o conceito de "escopo". Telas, guards e home precisam de
uma fonte única de verdade para isso.

## Escopo

1. Modelo `AppScope` no domínio: `scope` (enum `platform|organization|institution`),
   `organizationId?`, `institutionId?`, `papel` (`admin|editor|viewer`).
2. `AuthUser` lê o vínculo do `/auth/me` (ESC-01); `AuthRepository` popula em
   login, `restoreSession` e signup.
3. `SessionManager` persiste o vínculo (secure storage).
4. `AppScopeProvider` (Riverpod) expõe o escopo; é o único ponto lido por UI/guards.
5. Derivação de `papel` a partir do `UserRole` atual (plan §1.1).

## Critérios de aceite

- [ ] Após login, o escopo reflete o `/auth/me` (nenhum preenchimento por inferência no app).
- [ ] `restoreSession` reconstrói o escopo sem novo login.
- [ ] `logout` limpa o vínculo persistido.
- [ ] `flutter analyze lib` sem issues novas.
- [ ] Teste unitário: derivação de `AppScope` × `papel` para os 6 `UserRole`.

## Notas

- Segue camadas do ADR-011 (MVVM: `domain/` modelo, `data/` repositório, provider em
  `config/providers/`).
