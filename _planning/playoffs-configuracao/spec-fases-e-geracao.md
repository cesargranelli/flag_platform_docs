# Spec: Fases e Geração Sequencial da Chave de Playoffs

> **Status:** proposta v2 — modelo de referências e árvore automática confirmados
> **Escopo:** `flag_backend`, `flag_admin_web`
> **Relacionado:** [ADR-017](../../adr/ADR-017-configuracao-playoffs.md) (a atualizar), [spec.md](spec.md) (config base)
> **Origem:** solicitação em 2026-10-08

---

## 1. Contexto

A `PlayoffConfiguration` atual tem `enabled`, `qualified_per_group`, `wildcard_slots`, `format`,
`auto_generate`, `tiebreaker_criteria_json`. O `POST /competitions/{id}/playoffs/generate` hoje:

- monta a chave **inteira de uma vez** e preenche **só a primeira** rodada;
- exige competição em **DRAFT** (`assertEditable` → 409);
- **apaga** as rodadas de playoff existentes (destrutivo);
- `auto_generate` é armazenado mas **não usado**.

## 2. Requisitos (confirmados)

1. A configuração tem uma **lista de fases** de playoffs.
2. Cada fase define seus **enfrentamentos** (referências por vaga).
3. **Geração sequencial**: a fase N só é gerada após **todos** os jogos da fase N-1 terminarem.
4. **(Re)gerar** uma fase só é permitido enquanto **nenhum** jogo dela começou.
5. Geração/re-geração a **qualquer momento** (não só DRAFT).
6. Permissão: **criador** da competição **ou** ADMIN.
7. **Árvore automática**: ao habilitar playoffs, o sistema calcula quantos times se classificam
   (por grupo + wildcards) e monta a **árvore de chaveamento** (nº par que afunila); o usuário
   **preenche os confrontos** de cada fase.
8. **Referências de vaga**: `STANDING` (posição), `WINNER` (vencedor de jogo), `BEST_OVERALL`
   (melhor N-ésimo geral / wildcard puro), `LOSER` (perdedor de jogo — eliminação dupla).
9. `format` (`SINGLE_ELIMINATION` / `DOUBLE_ELIMINATION` / `ROUND_ROBIN_KNOCKOUT`) mantido como opção.

## 3. Modelo

`PlayoffConfiguration` ganha `phases_json` (texto JSON — mesmo padrão de `competitions.grouping_config`):

```json
{
  "phases": [
    { "name": "Wildcard", "type": "WILDCARD",
      "matchups": [
        { "home": { "ref": "STANDING", "group": "Conferência Caipira", "position": 2 },
          "away": { "ref": "STANDING", "group": "Conferência Metrópolis", "position": 5 } },
        { "home": { "ref": "STANDING", "group": "Conferência Metrópolis", "position": 3 },
          "away": { "ref": "STANDING", "group": "Conferência Caipira", "position": 4 } }
      ] },
    { "name": "Semifinal", "type": "SEMIFINAL",
      "matchups": [
        { "home": { "ref": "WINNER", "phase": 0, "match": 0 },
          "away": { "ref": "WINNER", "phase": 0, "match": 1 } }
      ] },
    { "name": "Bowl", "type": "FINAL",
      "matchups": [
        { "home": { "ref": "WINNER", "phase": 1, "match": 0 },
          "away": { "ref": "WINNER", "phase": 1, "match": 1 } }
      ] }
  ]
}
```

Tipos de vaga:
- `{"ref":"STANDING","group":"<nome do balde>","position":N}` — posição no balde (ou `group` omitido = tabela geral).
- `{"ref":"WINNER","phase":i,"match":j}` — vencedor do jogo j da fase i.
- `{"ref":"BEST_OVERALL","position":N}` — N-ésimo melhor da tabela geral (wildcard puro).
- `{"ref":"LOSER","phase":i,"match":j}` — perdedor (chave de derrotas / eliminação dupla).

`type` ∈ {`WILDCARD`, `SEMIFINAL`, `FINAL`} (o `RoundType` já cobre; o **nome** é livre).

## 4. Árvore de chaveamento (por agrupamento)

- **Chave por agrupamento**: cada balde (grupo/conferência/divisão) gera **seu próprio sub-bracket**
  a partir dos `qualifiedPerGroup` classificados do balde.
  **Regra da sugestão**: `byes = N − maior potência de 2 ≤ N`; os melhores entram na fase mais
  avançada e o restante joga o wildcard. Ex. `N=5`: 1º vai direto à final; `2º×5º` e `3º×4º` no
  wildcard → vencedores na semifinal → vencedor × 1º na final.
- Os **campeões de cada balde** se enfrentam na **fase final** (ex.: 2 conferências → 1 "Bowl").
- Fases = a **união dos níveis** dos baldes (nível 0 de todos, nível 1 de todos, …) + a fase dos campeões.
- Exemplo (2 conferências, 5 classificados cada):
  - `Wildcard`: 4º×5º da Caipira + 4º×5º da Metrópolis;
  - `Semifinal`: (1º × vencedor) e (2º × 3º) de cada conferência;
  - `Final de conferência`: final da Caipira + final da Metrópolis;
  - `Bowl`: campeão da Caipira × campeão da Metrópolis.
- Nomes default por fase, **editáveis** pelo organizador.
- Vagas usam `STANDING { group, position }` do balde e `WINNER`/`LOSER` da fase anterior.

> Correção do conceito (2026-10-08): a 1ª versão usava a **lista plana** de classificados, o que
> jogava todos os jogos em um único balde. O correto é **respeitar os agrupamentos** (grupo/conferência).

## 5. Geração

> **Decisão (2026-10-08):** o motor **apenas executa** as fases configuradas — a estrutura **não** é
> hardcoded. Sem `phases` configuradas, `generate` responde **400** ("configure as fases"). A
> sugestão automática vira um **helper de UI** ("Sugerir árvore"), **editável** pelo organizador.

`POST /competitions/{id}/playoffs/generate` gera **a próxima fase pendente**:

1. Sem fases geradas → gera a **fase 1** (a partir da classificação).
2. Última fase com **todos** os jogos `FINISHED` → gera a **próxima fase** (vencedores).
3. Última fase **não concluída** → **409** (aguardar o término da fase atual).

**Re-geração de uma fase**: permitida somente se a fase **não tem jogos** ou **nenhum jogo começou**
(todos `SCHEDULED`/`OPEN`); senão → **409**. Fases anteriores **não são tocadas** (P4).

## 6. Impactos

| Área | Mudança |
|------|---------|
| Migration | coluna `phases_json` em `playoff_configuration` |
| Domínio/DTOs | lista de fases + enfrentamentos |
| `PlayoffServiceImpl.generate` | geração **fase a fase**; trava por fase; fora de DRAFT |
| ADR-017 | **atualizar** (remove DRAFT-only; geração fase a fase; re-geração controlada) |
| Admin web | config da lista de fases (Create/Edit) + card "Chave de Playoffs" no **detalhe** |

## 7. Critérios de aceite (rascunho)

- [ ] Ao habilitar playoffs, a árvore é calculada a partir dos classificados (nº par que afunila).
- [ ] Organizador preenche os confrontos de cada fase (`STANDING`/`WINNER`/`BEST_OVERALL`/`LOSER`).
- [ ] Gerar fase 1 a partir da classificação (criador/ADMIN, qualquer status).
- [ ] Gerar fase N somente após todos os jogos da fase N-1 terminarem.
- [ ] (Re)gerar fase com jogos não iniciados substitui os confrontos; com jogo iniciado → 409.
- [ ] Fases anteriores nunca são alteradas.
- [ ] UI: card "Chave de Playoffs" no detalhe com estado da fase atual + ação "Gerar".

## 8. Decisões tomadas

- **P1**: (b1) `WINNER` + (c) refs extras (`BEST_OVERALL`, `LOSER`).
- **P2**: usuário escolhe a fase e preenche os confrontos escolhendo posições; a árvore é automática.
- **P3**: manter as opções de `format`.
- **P4**: não alterar fases anteriores.

## 9. Decisões finais

- **Tamanho da árvore**: **potência de 2 com byes** (ex.: 6 classificados → chave de 8, 2 byes;
  5 → chave de 8, 3 byes). Vale também para **nº ímpar** de classificados.
- **Entrada dos cabeças-de-chave**: o organizador **define em que fase** o(s) primeiro(s) colocado(s)
  entra(m) — enfrentando adversários que vêm do wildcard (a posição do bye é configurável).
- **Referências de vaga**: `STANDING`, `WINNER`, `BEST_OVERALL`, `LOSER`.
- **Geração**: fase a fase, sem DRAFT, criador/ADMIN, fases anteriores intactas.
