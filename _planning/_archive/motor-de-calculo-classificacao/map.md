# Mapa — Motor de Cálculo de Classificação de Competições

Effort de wayfinding. **Nada de implementação:** este mapa encontra o caminho até a spec.

> **Mapa concluído.** Todas as decisões (tickets 01–11) estão tomadas e a spec do destino foi redigida:
> **[spec.md](spec.md)**.

## Destination

Uma **spec do Motor de Cálculo de Classificação de Competições** — modelo de dados, motor de
cálculo (W-L-T/PCT configurável, critérios de desempate, grupos, W.O.), telas de regras no admin e
contratos de API — gravada no `flag_platform_docs`, pronta para virar plano de implementação.
O mapa termina quando não restar decisão em aberto antes de codar.

## Notes

- **Domínio:** Flag / Futebol Americano (W-L-T, PCT = (V + 0.5·E)/J, desempates estilo NFL/IFAF).
  Plataforma Flag: backend Spring Boot (modulith) + admin web Flutter MVVM.
- **Skills:** `grilling`, `domain-modeling` (decisões); `flutter-dart-code-review`, `java-springboot`
  quando chegar perto do código.
- **Preferência:** decisões somente; zero implementação nesta fase.
- **Estado atual relevante:** o módulo `standing` do `flag_backend` já existe e **já recalcula
  automaticamente** no `GameResultRegisteredEvent` (AFTER_COMMIT), com motor fixo 3-1-0 e ordenação
  pontos → saldo → pró → nome. A tabela `standings` é única por `(competition_id, team_id)` e **não**
  tem grupo nem PCT.
- **Tracker:** local (este diretório), por decisão de não usar GitHub Issues.
- **Issues:** abrir `issues/` — os tickets são arquivos `NN-<slug>.md`. Um ticket está livre
  (frontier) quando está aberto, sem bloqueio aberto e sem `Status: claimed`.

## Decisions so far

<!-- o índice: uma linha por ticket resolvido, com o gist da resposta e link para o detalhe -->

- [Pesquisa: domínio de jogos (status, resultado, W.O.)](issues/01-pesquisa-dominio-jogos.md):
  o motor consome só jogos FINISHED; o evento nasce no `registerResult` (CONFERENCE→FINISHED) e o
  recálculo roda síncrono no AFTER_COMMIT; `FinishedGame` **não** carrega rodada/fase/grupo; W.O.
  não existe no backend; resultado FINISHED não pode ser corrigido/reaberto pela API.
- [Pesquisa: RBAC — organizador dono vs ADMIN (backend + admin_web)](issues/02-pesquisa-rbac.md):
  roles reais = ADMIN/ORGANIZER/COMMISSIONER/REFEREE/MANAGER/FAN (o doc está desatualizado); GET em
  `/competitions/**` é público; escrita de gestão = `ADMIN_OR_ORGANIZER` + `assertManagedBy` (criador
  **ou** ADMIN); `registerResult` não checa dono; o admin web espelha via `isAdminUser`/
  `canEditCompetition`.
- [Regras de classificação: modelo e critérios de desempate do v1](issues/03-regras-classificacao-v1.md):
  modelo configurável (PCT default / pontos configuráveis 3-1-0); catálogo **fechado** de 8 critérios
  que o organizador escolhe e ordena; parâmetros por critério (`HEAD_TO_HEAD.multiWay` = DISCARD
  default); W.O. dedicado com placar padrão (49×0); sorteio sorteado uma vez e **persistido**.
- [Permissões: quem configura regras e quem dispara o recálculo](issues/07-permissoes-disparo.md):
  ler regras é **público**; editar = criador (ORGANIZER dono) ou ADMIN, **só em DRAFT**; disparar o
  recálculo = **somente ADMIN**; no admin, card do disparo usa `isAdminUser` e a edição usa
  `canEditCompetition`.
- [W.O.: modelagem no domínio de jogos](issues/09-wo-dominio-jogos.md): `resultType = NORMAL|WALKOVER`
  no jogo (status segue FINISHED); registro por `POST /games/{id}/walkover` (`ADMIN_OR_COMMISSIONER`)
  de qualquer estado não-terminal; efeito nos desempates é **configurável por competição** (default:
  W.O. conta em PF/PA/saldo/PCT; desligado, só V-E-D e pontos contam).
- [Modelo de dados: competition_standings_rules + expansão de standings + migração](issues/04-modelo-dados-migracao.md):
  nova tabela `competition_standings_rules` (1:1; JSON de critérios como lista de objetos); `standings`
  ganha `group_name` (sentinela `OVERALL`), PCT/SOS/SOV e `position`, com unique
  `(competition_id, group_name, team_id)`; default **WIN_PERCENTAGE para todas** (sem backfill,
  sintetizado); migração Liquibase só de schema + versionamento MINOR.
- [Motor: escopo de cálculo (grupo/geral) e pipeline de desempate](issues/05-motor-escopo-grupos.md):
  balde pela chave do `groupingType`; `grouping_scope` decide gerar balde/geral/BOTH; só rodadas
  `REGULAR`; cadeia `thenComparing` reiniciando por subgrupo; execução **assíncrona** (`@Async`) com
  status por timestamps e disparo manual **202**.
- [Contratos REST e versionamento (regras, disparo, leitura)](issues/06-contratos-rest.md):
  `GET/PUT .../standings-rules` (leitura pública; escrita dono/ADMIN só DRAFT); `POST
  .../standings/recalculate` → **202 ENQUEUED** (ADMIN); `GET .../standings` com `lastCalculatedAt`,
  `recalculationPending` e `groups` (geral como `OVERALL`); versionamento MINOR + OpenAPI; erros RFC-7807.
- [UX das telas de regras no admin (Design System Kickster)](issues/08-ux-telas-regras-admin.md):
  seção "Regras de Classificação" no create/edit (DRAFT) e leitura no detail; reordenação dos critérios
  por **drag & drop**, parâmetros em **KicksterMenuAnchor**; card ADMIN de disparo (`isAdminUser`) com
  status/confirm; **admin não exibe a tabela**. Protótipo: [assets/08-ux-regras-admin.md](assets/08-ux-regras-admin.md).
- [Redigir a spec do Motor de Cálculo de Classificação](issues/11-spec-motor-calculo.md):
  decisões dos tickets 01–10 consolidadas em **[spec.md](spec.md)** — destino atingido.
- [Pesquisa: fórmulas de SOS e SOV](issues/10-pesquisa-sos-sov.md): SOV = PCT combinado dos adversários
  **vencidos**; SOS = PCT combinado de **todos** os adversários; agregados **por confronto** (NFL),
  cálculo em **duas passagens** (não iterativo); IFAF deixa o desempate ao regulamento da competição;
  a "força de tabela" CBFA/BFA é equivalente a SOS. **Achado:** a CBFA exclui jogos de W.O. dos
  critérios de desempate — conflita com a decisão do ticket 09.

## Not yet specified

<!-- fog: em escopo, mas ainda sem pergunta afiada o suficiente para virar ticket -->

- Critérios avançados fora do catálogo do v1 (ex.: saldo de touchdowns) — candidatos a entrar no
  catálogo depois.
- Auditoria / trace de quem disparou o cálculo.
- Reconciliação de `architecture/roles-and-permissions.md`, que aparenta estar desatualizado
  (cita SUPER_ADMIN/ORG_ADMIN e `category_id` em `standings`, divergindo do código).
- Correção/reabertura de um resultado já FINISHED (hoje sem transição de volta na API) e seu efeito
  no recálculo — surgido do ticket *Pesquisa: domínio de jogos*.

## Out of scope

- Tela de visualização de classificação no admin — é do app público; no admin só se lê/configura regras.
- UI do `flag_public_app` (aqui só entra o contrato de leitura que ela vai consumir).
- Motor de regras arbitrárias / plugins: decidido que o catálogo é **fechado e computável** (ticket
  *Regras de classificação: modelo e critérios de desempate do v1*) — o organizador escolhe o
  subconjunto e a ordem, mas não cria lógica nova em tempo de uso.
