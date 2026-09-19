# Levantamento e Padronização: Eventos de Jogo & Jogadas por Modalidade

> Este documento apresenta o mapeamento completo, padronização de códigos, siglas oficiais, nomes e agrupamentos para eventos de jogo em todas as modalidades do esporte: **Flag 5x5, Flag 7x7, Flag 8x8, Flag 9x9 e Full Pads 11x11**.

---

## 1. Regras de Diferenciação por Modalidade

| Conceito | Flag 5x5 (IFAF) | Flag 7x7 / 8x8 / 9x9 | Full Pads 11x11 (NCAA/CBFA) |
|---|---|---|---|
| **Chutes (Punt/Kickoff/FG)** | ❌ Sem chutes. Posse inicia na linha de 5 yd. No 4º down sem avanço, turnover na linha de 5 yd adversária. | ⚠️ Varia por regulamento (algumas ligas 7x7/8x8/9x9 possuem Punt e Field Goal). | ✅ Todos os chutes oficiais (Kickoff, Punt, Field Goal, PAT Kicked). |
| **Ponto Extra (PAT)** | 1pt (linha de 5yd) ou 2pts (linha de 10yd) — apenas por corrida/passe. | 1pt (5yd) ou 2pts (10yd). Chute de 1pt em algumas ligas com Ys. | Chute (1pt) ou Ação de Passe/Corrida (2pts). |
| **Mini Touchdown / Bonus** | Usado em regulamentos específicos de torneios curtos/festivais. | Usado em formatos amadores/festivais. | ❌ Não existe no 11x11 tradicional. |
| **Corrida (Run)** | 🚫 Zona de Não-Corrida (No-Run Zone) dentro de 5 yd do TD. | Liberado ou com zonas de restrição conforme regulamento. | ✅ Liberado em qualquer ponto do campo. |
| **Fumbles** | ❌ Bola morta no ponto onde tocou o solo (sem turnover de recuperação, a menos que no ar). | ❌ Bola morta no solo (padrão Flag). | ✅ Bola viva (recuperável por ataque ou defesa com avanço). |

---

## 2. Catálogo Padronizado de Jogadas e Eventos

### Categoria A: Pontuação (`SCORE`)

| Código Backend | Sigla Oficial | Nome Padronizado | Pontos Padrão | 5x5 | 7x7/8x8/9x9 | 11x11 | Descrição / Regra |
|---|---|---|---|:---:|:---:|:---:|---|
| `TOUCHDOWN` | **TD** | Touchdown | 6 | ✅ | ✅ | ✅ | Cruzamento da endzone/posse na endzone (+6 pts). |
| `EXTRA_POINT_1` | **XP1** | Ponto Extra (+1pt) | 1 | ✅ | ✅ | ✅ | PAT de 1 ponto (Flag: linha de 5yd / 11x11: Chute). |
| `EXTRA_POINT_2` | **XP2** | Ponto Extra (+2pts) | 2 | ✅ | ✅ | ✅ | PAT de 2 pontos (Flag: linha de 10yd / 11x11: Ação de corrida/passe). |
| `SAFETY` | **SAF** | Safety | 2 | ✅ | ✅ | ✅ | Jogador retido/flag tirada na própria endzone (+2 pts para a defesa). |
| `FIELD_GOAL` | **FG** | Field Goal | 3 | ❌ | ⚠️ | ✅ | Chute de campo convertido entre as traves (+3 pts). |
| `PICK_SIX` | **P6** | Pick Six (TD Defensivo) | 6 | ✅ | ✅ | ✅ | Interceptação retornada até a endzone adversária (+6 pts). |
| `SCOOP_SIX` | **S6** | Scoop Six (Fumble Retornado) | 6 | ❌ | ❌ | ✅ | Fumble recuperado e retornado pela defesa até a endzone (+6 pts). |
| `PAT_DEFENSIVE_RETURN` | **DEF_PAT**| Retorno Defensivo de PAT | 2 | ✅ | ✅ | ✅ | Defesa intercepta/recupera o PAT e retorna até a outra endzone (+2 pts). |
| `MINI_TOUCHDOWN` | **MTD** | Mini Touchdown | 2 | ⚠️ | ⚠️ | ❌ | Pontuação especial usada em regulamentos de festivais/torneios curtos. |

---

### Categoria B: Jogadas de Ataque (`OFFENSE`)

| Código Backend | Sigla Oficial | Nome Padronizado | 5x5 | 7x7/8x8/9x9 | 11x11 | Descrição / Campos Requeridos |
|---|---|---|:---:|:---:|:---:|---|
| `PASS_COMPLETE` | **CMP** | Passe Completo | ✅ | ✅ | ✅ | Passe avançado retido pelo recebedor. *(Passador, Recebedor, Jardas)* |
| `PASS_INCOMPLETE` | **INC** | Passe Incompleto | ✅ | ✅ | ✅ | Passe lançado que tocou o solo sem recepção. *(Passador)* |
| `RUSH` | **RUN** | Corrida | ✅ | ✅ | ✅ | Avanço terrestre com a bola. *(Corredor, Jardas)* |
| `FIRST_DOWN` | **1ST** | First Down (Conquista) | ✅ | ✅ | ✅ | Conquista de nova série de descidas (cruzamento do meio de campo no Flag / linha de ganho no 11x11). |
| `LATERAL_PASS` | **LAT** | Passe Lateral / Pitch | ✅ | ✅ | ✅ | Passe para trás ou lateral durante o avanço. |
| `SPIKE` | **SPK** | Spike (Parar Relógio) | ❌ | ⚠️ | ✅ | Lançamento intencional no solo para congelar o cronômetro. |
| `KNEEL` | **KNL** | Ajoelhar / Kneel | ✅ | ✅ | ✅ | Ação de gastar tempo mantendo a bola segura no solo. |

---

### Categoria C: Jogadas de Defesa (`DEFENSE`)

| Código Backend | Sigla Oficial | Nome Padronizado | 5x5 | 7x7/8x8/9x9 | 11x11 | Descrição / Campos Requeridos |
|---|---|---|:---:|:---:|:---:|---|
| `FLAG_PULL` | **PULL** | Retirada de Fita (Flag Pull) | ✅ | ✅ | ❌ | Ação de parada da jogada no Flag Football. *(Defensor)* |
| `TACKLE` | **TCK** | Tackle (Derrubada) | ❌ | ❌ | ✅ | Derrubada do portador da bola no solo (Full Pads). *(Defensor)* |
| `SACK` | **SCK** | Sack | ✅ | ✅ | ✅ | Retirada de fita / Tackle no Quarterback atrás da linha de scrimmage. *(Defensor, Jardas)* |
| `PASS_DEFLECTION` | **PD** | Passe Desviado (Deflection) | ✅ | ✅ | ✅ | Defensor toca/desvia o passe no ar impedindo a recepção. *(Defensor)* |
| `INTERCEPTION` | **INT** | Interceptação | ✅ | ✅ | ✅ | Defensor ganha a posse de um passe no ar. *(Defensor, Jardas de retorno)* |
| `FUMBLE_FORCED` | **FF** | Fumble Forçado | ❌ | ❌ | ✅ | Defensor faz o portador perder a posse da bola (Full Pads). *(Defensor)* |
| `FUMBLE_RECOVERY` | **FR** | Recuperação de Fumble | ❌ | ❌ | ✅ | Recuperação da bola solta no solo. *(Jogador que recuperou)* |

---

### Categoria D: Jogadas de Chutes e Especiaria (`SPECIAL_TEAMS`)

| Código Backend | Sigla Oficial | Nome Padronizado | 5x5 | 7x7/8x8/9x9 | 11x11 | Descrição / Campos Requeridos |
|---|---|---|:---:|:---:|:---:|---|
| `PUNT` | **PNT** | Punt (Chute de Devolução) | ❌ | ⚠️ | ✅ | Chute de devolução de posse no 4º down. *(Chutador, Jardas)* |
| `KICKOFF` | **KOK** | Kickoff (Chute Inicial) | ❌ | ⚠️ | ✅ | Chute inicial de início de tempo ou pós-pontuação. *(Chutador)* |
| `PUNT_RETURN` | **PR** | Retorno de Punt | ❌ | ⚠️ | ✅ | Avanço do time recebedor após o Punt. *(Retornador, Jardas)* |
| `KICKOFF_RETURN` | **KR** | Retorno de Kickoff | ❌ | ⚠️ | ✅ | Avanço do time recebedor após o Kickoff. *(Retornador, Jardas)* |
| `TOUCHBACK` | **TB** | Touchback | ❌ | ⚠️ | ✅ | Chute/Interceptação na endzone onde o time opta por não retornar, iniciando na linha padrão (20/25yd). |
| `BLOCKED_KICK` | **BLK** | Chute Bloqueado | ❌ | ⚠️ | ✅ | Defesa bloqueia um Punt, Field Goal ou PAT chutado. |

---

### Categoria E: Faltas e Penalidades (`PENALTY`)

#### 1. Faltas de Ataque

| Código Backend | Sigla | Nome Padronizado | Descrição Técnica |
|---|---|---|---|
| `PEN_FLAG_GUARDING` | **FG** | Flag Guarding | Portador da bola impede a retirada da fita com a mão, bola ou cotovelo. |
| `PEN_OFFENSIVE_PASS_INT` | **OPI** | Interferência no Passe (Ataque) | Contato físico do recebedor/atacante para afastar o defensor antes do passe. |
| `PEN_ILLEGAL_RUN` | **ILR** | Corrida Ilegal (No-Run Zone) | Execução de jogada de corrida dentro da Zona de Não-Corrida (Flag). |
| `PEN_ILLEGAL_FORWARD_PASS` | **IFP** | Passe Ilegal para Frente | Lançamento para frente executado além da linha de scrimmage ou 2º passe para frente. |
| `PEN_FALSE_START` | **FS** | Falsa Partida (False Start) | Movimento abrupto de um jogador de ataque antes do snap. |
| `PEN_ILLEGAL_MOTION` | **MOT** | Movimento Ilegal | Mais de um jogador em movimento ou movimento para frente no snap. |
| `PEN_OFFENSIVE_HOLDING` | **OH** | Segurada do Ataque (Holding) | Segurar o defensor para impedir o avanço ou a pressão. |
| `PEN_CHARGING` | **CHG** | Charging (Investida) | Corredor tromba intencionalmente no defensor que já tinha posição estabelecida. |
| `PEN_DELAY_OF_GAME` | **DOG** | Atraso de Jogo | Estouro do tempo limite para dar o snap (25s / 40s). |

#### 2. Faltas de Defesa

| Código Backend | Sigla | Nome Padronizado | Descrição Técnica |
|---|---|---|---|
| `PEN_DEFENSIVE_PASS_INT` | **DPI** | Interferência no Passe (Defesa) | Contato físico no recebedor antes da chegada da bola impedindo a recepção. |
| `PEN_ILLEGAL_RUSH` | **ILR_DEF** | Rush Ilegal | Rusher inicia a corrida antes da marca de 7 jardas (Flag) ou sai antes do tempo. |
| `PEN_ROUGHING_PASSER` | **RTP** | Ataque Ilegal ao Passador | Contato físico com o QB após a bola ter sido lançada. |
| `PEN_ILLEGAL_FLAG_PULL` | **IFP_DEF** | Retirada Ilegal de Fita | Tirar a fita de um jogador que não é o portador da bola. |
| `PEN_DEFENSIVE_HOLDING` | **DH** | Segurada da Defesa | Segurar o vestuário ou corpo do recebedor/corredor. |
| `PEN_OFFSIDE` | **OFF** | Invasão / Offside | Defensor cruza a linha de scrimmage antes do snap. |
| `PEN_TARGETING` | **TGT** | Targeting (Contato Violento) | Contato violento com a cabeça/pescoço (Full Pads / Flag). |

#### 3. Faltas Administrativas e Comportamentais

| Código Backend | Sigla | Nome Padronizado | Descrição Técnica |
|---|---|---|---|
| `PEN_UNSPORTSMANLIKE` | **UNS** | Conduta Antidesportiva | Falta de respeito, xingamento, taunting ou atitude antidesportiva. |
| `PEN_PERSONAL_FOUL` | **PF** | Falta Pessoal | Contato físico desnecessário ou perigoso. |
| `PEN_TOO_MANY_PLAYERS` | **TOO** | Excesso de Jogadores | Mais jogadores em campo no snap do que o permitido pela modalidade. |
| `PEN_DISQUALIFICATION` | **DQ** | Desqualificação / Expulsão | Expulsão direta de atleta/comissão técnica por falta grave ou 2º UNS. |

---

### Categoria F: Controle de Partida e Súmula (`GAME_CONTROL`)

| Código Backend | Sigla | Nome Padronizado | Descrição |
|---|---|---|---|
| `TIMEOUT` | **TO** | Pedido de Tempo | Paralisação solicitada por uma das equipes (ex: 2 timeouts por tempo). |
| `OFFICIAL_TIMEOUT` | **OTO** | Tempo da Arbitragem | Paralisação ordenada pelo árbitro (atendimento médico, dúvida regulamentar, clima). |
| `PERIOD_START` | **P_START** | Início de Período | Registro de início de Quarto / Tempo (1T, 2T, 3T, 4T, OT). |
| `PERIOD_END` | **P_END** | Fim de Período | Encerramento formal do período de jogo. |
| `COIN_TOSS` | **TOSS** | Sorteio Inicial (Coin Toss) | Registro da escolha do vencedor do cara ou coroa (Possão/Campo/Adiar). |

---

## 3. Classificação de Inversão Imediata de Posse (Turnover / Change of Possession)

Algumas jogadas provocam a **inversão imediata da posse de bola** (o time que estava atacando passa a defender e vice-versa). No backend/ViewModel, registrar qualquer um desses eventos deve acionar automaticamente a troca do `possessionTeamId` e resetar a contagem de descidas para o 1º Down.

### 1. Inversões por Ação Defensiva / Turnover Direto

| Código Backend | Sigla | Jogada | Modalidades | Regra da Inversão |
|---|---|---|:---:|---|
| `INTERCEPTION` | **INT** | Interceptação | Todas | A defesa toma a posse do passe no ar. A posse passa imediatamente para a defesa no ponto da retirada da fita/tackle (ou onde o retorno parar). |
| `PICK_SIX` | **P6** | Pick Six (TD Defensivo) | Todas | A defesa intercepta e retorna até a endzone. Posse inverte + marca 6 pontos. *(Pós-pontuação: troca de posse normal)*. |
| `FUMBLE_RECOVERY` | **FR** | Recuperação de Fumble | 11x11 | *(Apenas 11x11)* Defesa recupera a bola solta no solo. Posse inverte imediatamente no ponto da recuperação/retorno. |
| `SCOOP_SIX` | **S6** | Scoop Six (Fumble Retornado) | 11x11 | *(Apenas 11x11)* Defesa recupera o fumble e retorna para TD (+6 pts). Posse inverte. |
| `PAT_DEFENSIVE_RETURN` | **DEF_PAT** | Retorno Defensivo de PAT | Todas | Defesa intercepta/recupera a bola durante o Ponto Extra e retorna até a endzone oposta (+2 pts). *(A posse volta para a troca padrão de pós-pontuação)*. |

### 2. Inversões por Jogadas de Chutes (Special Teams)

| Código Backend | Sigla | Jogada | Modalidades | Regra da Inversão |
|---|---|---|:---:|---|
| `PUNT` | **PNT** | Punt (Chute de Devolução) | 7x7/8x8/9x9 / 11x11 | O ataque opta voluntariamente por chutar a bola para o campo adversário. Posse passa para o time recebedor após a bola parar ou ser retornada. |
| `KICKOFF` | **KOK** | Kickoff (Chute Inicial) | 7x7/8x8/9x9 / 11x11 | Chute de início de tempo ou pós-pontuação. A posse pertence ao time recebedor. |
| `TOUCHBACK` | **TB** | Touchback em Chute | 7x7/8x8/9x9 / 11x11 | Bola chutada entra na endzone sem retorno. Posse inverte para o time recebedor na linha padrão (20yd/25yd). |

### 3. Inversões por Regra / Limite de Descidas (Turnover on Downs)

| Código Backend / Condição | Sigla | Jogada / Situação | Modalidades | Regra da Inversão |
|---|---|---|:---:|---|
| `TURNOVER_ON_DOWNS` | **TOD** | Estouro de Descidas (4º Down sem ganho) | Todas | O ataque executa o 4º down e não alcança a linha de ganho (1ª descida) nem marca TD. <br>• **Flag 5x5**: Posse inverte para o adversário na **linha de 5 yardas** do seu próprio campo.<br>• **Flag 7x7/8x8/9x9 e 11x11**: Posse inverte para o adversário **exatamente no ponto onde a bola ficou morta**. |
| `SAFETY` | **SAF** | Safety | Todas | Ataque é contido/tem fita tirada dentro da própria endzone (+2 pts defesa). A posse inverte para a defesa (que recebe o chute/bola na linha de 20yd no 11x11 ou linha de 5yd no Flag 5x5). |

### 4. Inversões por Regras Específicas do Flag Football (Peculiaridades)

| Situação | Modalidade | Regra da Inversão |
|---|:---:|---|
| **Snap que entra na Endzone (Snap out of Endzone)** | Flag 5x5 / 7x7 | Se o snap for mal executado e a bola tocar o solo dentro da própria endzone do ataque, é considerado **Safety** (+2 pts defesa) e a posse inverte. |
| **Interceptação no PAT** | Flag 5x5 | Se a defesa intercepta o Try de 1pt ou 2pts, a tentativa de PAT encerra. Se a defesa for parada ou marcar 2pts (retorno), a partida reinicia com a **posse invertida na linha de 5 yd** do time que sofreu o TD. |

---

## 4. Resumo da Padronização Proposta para a API

```
   Catálogo Master de Jogadas (PlayTypes)
   ├── SCORE          (TD, XP1, XP2, SAF, FG, P6, S6, DEF_PAT, MTD)
   ├── OFFENSE        (PASS_COMPLETE, PASS_INCOMPLETE, RUSH, FIRST_DOWN, LATERAL_PASS, SPIKE, KNEEL)
   ├── DEFENSE        (FLAG_PULL, TACKLE, SACK, PASS_DEFLECTION, INTERCEPTION*, FUMBLE_FORCED, FUMBLE_RECOVERY*)
   ├── SPECIAL_TEAMS  (PUNT*, KICKOFF*, PUNT_RETURN, KICKOFF_RETURN, TOUCHBACK*, BLOCKED_KICK)
   ├── PENALTY        (Faltas de Ataque, Defesa e Conduta)
   └── GAME_CONTROL   (TIMEOUT, OFFICIAL_TIMEOUT, PERIOD_START, PERIOD_END, COIN_TOSS)

   * Indica jogadas que disparam INVERSÃO IMEDIATA DE POSSE (Turnover / `isTurnover = true`).
```

### Principais Benefícios da Padronização
1. **Códigos em Enum Máquina (`UPPER_SNAKE_CASE`)** — facilidade para backend Java/Kotlin/Dart e banco de dados.
2. **Flag de Inversão Automática (`isTurnover = true`)** — o backend e o ViewModel sabem exatamente quando inverter a posse no painel sem ação manual do árbitro.
3. **Siglas Curtas (2 a 3 letras)** — perfeitas para badges de UI em telas pequenas e súmulas impressas/PDFs.
4. **Filtro por Modalidade (`modality`)** — a interface do app poderá exibir apenas as jogadas válidas para a modalidade do campeonato selecionado (ex: esconder Punt/Kickoff no Flag 5x5).
