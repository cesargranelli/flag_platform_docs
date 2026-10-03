# FS-06: Deleção de jogos fora do `GameService` deixa documento órfão no espelho

> **Type:** atividade
> **Status:** open
> **Effort:** firestore-sync
> **Repo(s):** flag_backend
> **Blocks:** —
> **Blocked by:** —
> **Origin:** ADR-018 / PR #73

## Objetivo

Garantir que **toda** deleção de jogo no banco relacional reflita no espelho Firestore. Hoje o
`PlayoffServiceImpl.deleteExistingBracket` apaga jogos **direto pelo repositório**, bypassando a API
pública do módulo `game`, então o listener de `GameChangedEvent` nunca é disparado e o documento
`games/{id}` fica **órfão** no espelho.

## Contexto

- Revelado pela implementação do [ADR-018](../../../adr/ADR-018-espelhamento-firestore-admin-sdk.md)
  no PR #73 do `flag_backend` (commits `3489548`, `99c411c`).
- O espelhamento só cobre os caminhos de escrita publicados pelo módulo `game` (`create`,
  `createBatch`, `update`, `updateStatus`, `registerResult`, `registerScoreEvent`,
  `handlePlayScoreEvent`, `correctScore`). A deleção conduzida pelo módulo `playoff` **não** publica
  `GameChangedEvent` e **não** passa pela API pública de `game` (disciplina do Spring Modulith).
- Efeito: ao regenerar/excluir uma chave de playoff, o app público continua lendo o jogo antigo em
  `games/{id}` — drift silencioso, sem retry automático (corrigível só por backfill manual, FS-03).
- Como o espelho é best-effort e o backend é o único produtor de escritas, a deleção precisa ser
  roteada pela API pública do módulo `game` (que publica o evento) ou o módulo `game` precisa expor
  um comando de exclusão que emita `GameChangedEvent`.

## Critérios de aceitação

- [ ] Nenhum caminho do backend apaga jogo direto pelo repositório sem passar pela API pública de
      `game` (auditar `PlayoffServiceImpl`, `GameService` e afins).
- [ ] A deleção publica `GameChangedEvent(gameId)` e o listener remove/atualiza o documento
      `games/{id}` no Firestore.
- [ ] Teste cobre: criar jogo → espelhar → deletar via playoff → documento some (ou vira estado
      coerente) no espelho.
- [ ] Sem credencial Firebase o fluxo continua **no-op** e não quebra a request.
- [ ] `mvn -B clean verify` verde.

## Arquivos afetados (referência)

- `flag_backend/src/main/java/.../playoff/**` — `PlayoffServiceImpl.deleteExistingBracket` (parar de
  apagar direto pelo repositório).
- `flag_backend/src/main/java/.../game/**` — API pública de deleção + publicação de
  `GameChangedEvent`.
- `flag_backend/src/main/java/.../realtime/**` — tratamento do evento de deleção (remover doc).
