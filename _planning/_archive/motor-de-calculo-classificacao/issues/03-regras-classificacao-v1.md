# Regras de classificação: modelo e critérios de desempate do v1

Type: grilling
Status: resolved

## Question

Fechar o **regulamento** que o motor vai aplicar: o modelo de classificação e o catálogo/semântica dos
critérios de desempate do v1.

Decidir:

- **Modelo configurável:** `WIN_PERCENTAGE` (W-L-T → PCT = (V + 0.5·E) / J) como default e
  `POINTS_BASED` (3-1-0) como alternativa. Como pontos e PCT convivem na ordenação primária.
- **Catálogo do v1** e a semântica **exata** de cada critério:
  - Confronto Direto (Head-to-Head) — inclusive o caso de 3+ equipes empatadas (multi-way).
  - Saldo de Pontos (PF − PA), Pontos Pró (PF), Menor Pontos Sofridos (PA).
  - Sorteio (RANDOM_DRAW).
- **Critérios avançados previstos** (força de calendário SOS / força de vitória SOV): entram no v1 ou
  ficam apenas no catálogo extensível do JSON?
- **W.O.:** pontuação e placar padrão; como interage com cada critério.
- **Empates não resolvidos / Sorteio:** persistir o resultado? determinístico? re-sorteia a cada cálculo?

Base de referência: NFL / IFAF / CBFA (o transcript em `./contexto_motor_de_calculo.txt` já levantou
o padrão).

## Answer

Regulamento do v1, decidido em sessão de grilling com o humano.

1. **Modelo por competição** (`ranking_system`, já decidido no charting):
   - `WIN_PERCENTAGE` (**default**): métrica primária **PCT = (V + 0.5·E) / J**; a campanha V-E-D é o
     que se exibe.
   - `POINTS_BASED`: métrica primária **pontos = V·pV + E·pE + D·pD**, com valores **configuráveis**,
     default **3 / 1 / 0**.
2. **Catálogo fechado e computável** — o organizador escolhe um **subconjunto** e define a **ordem**
   por competição. "Regra nova" = evoluir o catálogo no backend; o JSON de regras já a acomoda sem
   migração de schema. Catálogo do v1 (8):
   `HEAD_TO_HEAD`, `POINT_DIFFERENTIAL` (saldo), `POINTS_FOR`, `POINTS_AGAINST` (menor sofrido),
   `GROUP_WIN_PERCENTAGE` (aproveitamento dentro do grupo), `STRENGTH_OF_VICTORY` (SOV),
   `STRENGTH_OF_SCHEDULE` (SOS), `RANDOM_DRAW`.
3. **Parâmetros por critério** (o JSON deixa de ser lista de strings e vira lista de objetos): ex.
   `HEAD_TO_HEAD` tem `multiWay` ∈ {`DISCARD` (**default**), `MINI_TABLE`}. Com **3+ empatados**, o
   default é **descartar** o critério e seguir a cadeia; a **mini-tabela** (só jogos entre os
   empatados) é opção configurável.
4. **W.O. dedicado** (ver ticket *W.O.: modelagem no domínio de jogos*): estado/flag de jogo + placar
   padrão configurável (default **49×0**) e pontos de W.O. configuráveis; o motor computa como
   vitória/derrota com o placar padrão.
5. **Sorteio persistido:** quando `RANDOM_DRAW` está na cadeia, o motor sorteia **uma vez** e
   **persiste** a ordem (não re-sorteia a cada recálculo). Sem `RANDOM_DRAW`, o desempate remanescente
   é **estável** por nome do time (ou `seed_number`).
6. **Graduado:** fórmulas exatas de SOS/SOV → ticket *Pesquisa: fórmulas de SOS e SOV*.

**Premissas assumidas (não ditas explicitamente pelo humano — validar se discordar):**

- A cadeia é um `Comparator.thenComparing` encadeado: cada critério só desempata o subgrupo ainda
  empatado; quando separa parte, a cadeia **reinicia** para o remanescente.
- A métrica primária (PCT ou pontos) é a base da ordenação; os critérios do catálogo só a seguem.
- Todos os critérios do catálogo são computáveis a partir de jogos FINISHED + regras da competição.
