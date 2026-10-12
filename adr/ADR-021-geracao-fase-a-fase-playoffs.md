# ADR-021: Geração fase a fase da chave de playoffs

> **Status:** aceito
> **Data:** 2026-10-08
> **Decisores:** tech-lead
> **Substitui (parcialmente):** [ADR-017](ADR-017-configuracao-playoffs.md)

---

## Contexto

O [ADR-017](ADR-017-configuracao-playoffs.md) definiu a configuração de playoffs e uma geração
**de uma vez, somente em DRAFT e irreversível**. Na prática, o fluxo real é:

- gerar/re-gerar a chave **a qualquer momento** do campeonato (não só em rascunho);
- montar a chave **fase a fase** (wildcard → semifinal → final → bowl), gerando a próxima só após
  a anterior terminar;
- **definir os confrontos na configuração** (posições/vencedores), com **árvore automática** a
  partir do nº de classificados.

## Decisão

1. **Lista de fases na configuração** (`PlayoffConfiguration.phases_json`): cada fase tem `name`,
   `type` (`WILDCARD`/`SEMIFINAL`/`FINAL`) e a lista de `matchups`.
2. **Referências de vaga** (por confronto):
   - `STANDING` — posição em um balde (`group`) ou na tabela geral;
   - `WINNER` — vencedor do jogo `j` da fase `i` (anterior);
   - `BEST_OVERALL` — N-ésimo melhor da tabela geral (wildcard puro);
   - `LOSER` — perdedor do jogo `j` da fase `i` (chave de derrotas / eliminação dupla).
3. **Estrutura dirigida por configuração**: o motor **apenas executa** as fases configuradas — a
   árvore **não** é hardcoded. Sem `phases` configuradas, `generate` responde **400**. A **sugestão
   automática** (`byes = N − maior potência de 2 ≤ N`; os melhores entram na fase mais avançada) é um
   **helper de UI** ("Sugerir árvore"), editável pelo organizador.
4. **Geração fase a fase**: `POST .../playoffs/generate` gera a **próxima fase pendente**; a fase `N`
   só é gerada após **todos** os jogos da fase `N-1` terminarem (senão **409**).
5. **Re-geração controlada**: (re)gerar uma fase só é permitido enquanto **nenhum** jogo dela começou
   (todos `SCHEDULED`/`OPEN`); senão **409**. Fases anteriores **nunca** são alteradas.
6. **Sem restrição de DRAFT**: geração/re-geração a qualquer momento do campeonato.
7. **Permissão**: **criador** da competição **ou** ADMIN.

## Consequências

### Positivos
- Reflete o fluxo real (chave montada conforme o campeonato avança).
- Configuração explícita dos confrontos por fase.
- Não destrói fases já disputadas.

### Negativos
- Motor de geração mais complexo (fases, byes, referências).
- Nova coluna `phases_json` (migration) e novo modelo de domínio.

## Alternativas consideradas

1. **Manter a geração "de uma vez" do ADR-017** — rejeitado: não reflete fases geradas conforme os
   resultados e impõe DRAFT.
2. **Chave sempre par, sem byes** — rejeitado: com `N` não potência de 2, os byes são a forma correta
   de fechar a árvore.
3. **Sem árvore automática (100% manual)** — rejeitado: o cálculo de classificados → árvore reduz
   erro humano e é o ponto de partida da configuração.
