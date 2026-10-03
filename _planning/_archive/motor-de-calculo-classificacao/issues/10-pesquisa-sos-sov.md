# Pesquisa: fórmulas de SOS e SOV (NFL/IFAF)

Type: research
Status: resolved

## Question

Apuram as **fórmulas exatas** de Força de Calendário (Strength of Schedule, SOS) e Força de Vitória
(Strength of Victory, SOV) usadas no futebol americano, e como computá-las no nosso modelo.

Investigar e registrar:

- Definição oficial (NFL/IFAF): SOS = combinação do PCT dos adversários enfrentados; SOV = combinação
  do PCT dos adversários **vencidos**. Média? Soma? Inclui o próprio time ou não?
- Como calcular quando os adversários ainda têm PCT dependente do próprio resultado (cálculo
  iterativo), e se o padrão usa o PCT **final** da temporada.
- Tratamento de múltiplos confrontos contra o mesmo adversário (conta uma vez ou por jogo?).
- Como as duas se posicionam na cadeia de desempate e interagem com grupos (grupo vs geral).
- Caso de uso real citado pelo humano: "força de tabela".

## Answer

Apurado na NFL, na IFAF e em regulamentos CBFA/BFA (Brasileirão/Superliga/Liga BFA).

### Definições canônicas (NFL)
Com o recorde de cada equipe expresso em **W-L-T** e **PCT = (W + 0.5·T) / (W + L + T)**:

- **SOV (Strength of Victory)** = porcentagem combinada de vitórias dos adversários que a equipe
  **venceu**.
- **SOS (Strength of Schedule)** = porcentagem combinada de vitórias de **todos** os adversários
  enfrentados (vitória ou derrota).
- A NFL usa **"in all games"** (temporada regular inteira) e ordena **SOV antes de SOS**.

**Computação (por jogo, não por adversário único):** cada confronto contribui com o **recorde
completo do adversário**. Confirmado no exemplo de 2016 dos Patriots: 16 jogos, soma dos recordes dos
16 adversários = 256 jogos (16 × 16), batendo com o 111–142–3 publicado. Ou seja, um rival enfrentado
duas vezes **conta duas vezes**. Não é iterativo: é um **cálculo em duas passagens** — (1) PCT de cada
equipe; (2) SOS/SOV a partir dos PCTs dos adversários.

### Variante brasileira (CBFA / BFA) — "força de tabela"
- **Liga BFA 2024** — *força de tabela* = **soma das vitórias de todos os adversários ÷ soma da
  quantidade de jogos dos adversários**, "para cada confronto realizado" (per game). Ordem com 2
  empatados: confronto direto → força de tabela → menor pontos cedidos. Com 3+: força de tabela →
  vitórias no confronto → menor pontos cedidos → sorteio.
- **BFA 2017** dizia "cada adversário só pode ser considerado **uma única vez**" (dedupe) — variante
  histórica; a NFL e a BFA 2024 usam **por confronto**.
- **Superliga FA D1 2025** usa *força de vitória* = soma das vitórias **e empates** das equipes que a
  equipe enfrentou **e venceu ou empatou** (≈ SOV), além de médias ("menor média de pontos sofridos",
  "saldo médio").
- **Multi-way:** os regulamentos CBFA/BFA determinam **reaplicar a cadeia do início** após cada
  separação — o que **confirma a premissa** registrada no ticket *Regras de classificação: modelo e
  critérios de desempate do v1*.

### IFAF
A IFAF **não** fixa critérios de classificação/desempate de tabela — remete às regras da competição
(as regras de jogo tratam de empate *dentro* da partida, prorrogação). Ou seja, o desempate é sempre
**por regulamento da competição**, o que valida a decisão de tornar os critérios configuráveis.

### Achado que conflita com o ticket 09
O regulamento **Superliga FA D1 2025 (CBFA)** diz explicitamente: *"Jogos em que houve Walkover (W.O.)
não serão considerados jogos válidos para critérios de desempate."* Isso **contradiz** a decisão do
ticket *W.O.: modelagem no domínio de jogos* ("placar padrão conta normalmente em PF/PA/saldo/PCT").
→ Levantado como fog no mapa para revisão.

### Recomendação de spec
Adotar a definição **NFL** (SOV = PCT combinado dos vencidos; SOS = PCT combinado de todos os
adversários; ambos agregados **por confronto**, com PCT = (W + 0.5·T)/J), e manter o default
"**todos os jogos**"; a variante "força de tabela" brasileira é equivalente a SOS (muda só a forma de
agregar vitórias/jogos). A ordem e a inclusão de cada um ficam a cargo do organizador (ticket 03).
