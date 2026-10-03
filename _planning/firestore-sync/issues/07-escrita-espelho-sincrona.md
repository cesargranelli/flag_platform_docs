# FS-07: Escrita do espelho síncrona no thread da request

Type: Issue
Status: open

## Objetivo

Registrar que a escrita do espelho Firestore hoje é **síncrona no thread da request**
(`ApiFuture.get()` logo após o commit). É aceitável no volume atual (`~10-20 TPS`), mas pode migrar
para `@Async` se a latência/rede do Firestore passar a pesar na resposta da API.

## Contexto

- Revelado pela implementação do [ADR-018](../../../adr/ADR-018-espelhamento-firestore-admin-sdk.md)
  no PR #73 do `flag_backend` (commits `3489548`, `99c411c`).
- O listener `@TransactionalEventListener(AFTER_COMMIT)` chama o Admin SDK e **aguarda** o
  `ApiFuture` (`get()`) antes de liberar o thread da request. O espelhamento já é best-effort
  (try/catch + `warn`), então uma falha não quebra a request, mas uma **lentidão** do Firestore
  ainda adiciona latência ao caminho de escrita.
- O ADR-018 registra o risco: o espelhamento roda **dentro do processo da API** consumindo parte dos
  `560m` do container (`mem_limit`); aceitável no volume atual.
- Não é bug — é um **trade-off consciente** a monitorar. Só virar assíncrono se houver evidência de
  pressão de latência/thread pool; `@Async` exige cuidado com a propagação de contexto e com a
  garantia AFTER_COMMIT.

## Critérios de aceitação

- [ ] Latência do espelho observável (tempo de escrita / taxa de falha) na instrumentação de
      monitoramento (ver FS-04).
- [ ] Decisão documentada: manter síncrono **ou** migrar para `@Async`, com justificativa baseada em
      medição (não em preferência).
- [ ] Se migrar: execução fora do thread da request, preservando o `AFTER_COMMIT` e o best-effort
      (falha continua não quebrando a request).
- [ ] `mvn -B clean verify` verde.

## Arquivos afetados (referência)

- `flag_backend/src/main/java/.../realtime/**` — listener/cliente que aguarda o `ApiFuture.get()`.
- `flag_backend/src/main/java/.../game/**` — ponto de publicação de `GameChangedEvent`.
