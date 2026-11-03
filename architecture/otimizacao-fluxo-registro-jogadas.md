# Especificação Técnica: Otimização e Simplificação do Registro de Jogadas (Referee App)

> **Documento Oficial de Arquitetura & UX Operacional**  
> **Status:** Proposto / Em Validação  
> **Aplica-se a:** `flag_referee_app`, `flag_backend`, `flag_platform_docs`  
> **Data:** Novembro de 2026  

---

## 1. Contexto e Motivação

Durante a operação de uma partida de Flag Football na beira do campo (seja pela mesa oficial de arbitragem ou pelo árbitro de campo), a interface do `flag_referee_app` enfrenta condições operacionais severas:
1. **Pressão de Tempo:** O relógio de jogo (ou o play clock de 25 segundos) não para na maioria das situações normais. O operador tem poucos segundos para apontar o que aconteceu antes do próximo snap.
2. **Ambiente Hostil:** Uso em tablets ou smartphones sob luz solar direta, com o operador muitas vezes em pé, segurando o dispositivo com uma mão e registrando com o polegar.
3. **Sobrecarga de Fricção Atual:** Para jogadas triviais como um simples "Passe Incompleto" ou "Pedido de Tempo (Timeout)", o fluxo atual exige abrir diálogos modais inteiros, navegar por categorias e confirmar em botões no rodapé, demandando entre **4 a 5 cliques**. Jogadas compostas (como Passe Completo ou Touchdown + Ponto Extra) chegam a exigir de **7 a 11 cliques**.

**Objetivo:** Reduzir drasticamente o esforço cognitivo e o número de toques, permitindo o registro de jogadas comuns em **1 a 2 toques**, e jogadas completas em no máximo **3 a 4 toques fluidos**, aproveitando a inteligência contextual já disponível no aplicativo.

---

## 2. Levantamento Completo de Jogadas Previstas por Modalidade

O catálogo unificado da plataforma contempla **38 tipos de jogadas e eventos**, distribuídos em 6 categorias esportivas. A tabela a seguir especifica a aplicabilidade em cada modalidade, os campos requeridos e se a jogada provoca **inversão imediata de posse** (*Turnover*):

| ID | Código Backend | Sigla | Nome Oficial | Pontos | 5x5 | 7x7/8x8 | 11x11 | Inverte Posse? | Campos Mínimos Requeridos |
|:--:|---|:---:|---|:---:|:---:|:---:|:---:|:---:|---|
| **A. PONTUAÇÃO (SCORE)** |
| 01 | `TOUCHDOWN` | **TD** | Touchdown | 6 | ✅ | ✅ | ✅ | ⚠️ Pós-PAT | Atleta pontuador, Jardas (Passador opcional se passe) |
| 02 | `EXTRA_POINT_1` | **XP1** | Ponto Extra (1 pt) | 1 | ✅ | ✅ | ✅ | ✅ Sim | Atleta pontuador, Sucesso/Falha |
| 03 | `EXTRA_POINT_2` | **XP2** | Ponto Extra (2 pts) | 2 | ✅ | ✅ | ✅ | ✅ Sim | Atleta pontuador, Sucesso/Falha |
| 04 | `PICK_TWO` | **P2** | Retorno Defensivo de PAT | 2 | ✅ | ✅ | ✅ | ✅ Sim | Defensor que retornou |
| 05 | `SAFETY` | **SAF** | Safety | 2 | ✅ | ✅ | ✅ | ✅ Sim | Time que pontuou (defesa), Defensor (opcional) |
| 06 | `PICK_SIX` | **P6** | Pick Six (TD Defensivo) | 6 | ✅ | ✅ | ✅ | ⚠️ Pós-PAT | Defensor, Jardas de retorno |
| 07 | `FIELD_GOAL` | **FG** | Field Goal | 3 | ❌ | ⚠️ | ✅ | ✅ Sim | Chutador (Kicker), Jardas |
| 08 | `MINI_TOUCHDOWN` | **MTD** | Mini Touchdown | 2 | ⚠️ | ⚠️ | ❌ | ⚠️ Pós-PAT | Atleta pontuador |
| **B. ATAQUE (OFFENSE)** |
| 09 | `PASS_COMPLETE` | **CMP** | Passe Completo | 0 | ✅ | ✅ | ✅ | ❌ Não | Passador (QB), Recebedor (WR), Jardas |
| 10 | `PASS_INCOMPLETE` | **INC** | Passe Incompleto | 0 | ✅ | ✅ | ✅ | ❌ Não | Passador (ou neutro para avanço de descida) |
| 11 | `RUSH` | **RUN** | Corrida | 0 | ✅ | ✅ | ✅ | ❌ Não | Corredor (RB/QB), Jardas |
| 12 | `FIRST_DOWN` | **1ST** | Conquista de 1ª Descida | 0 | ✅ | ✅ | ✅ | ❌ Não | Atleta do avanço, Jardas |
| 13 | `LATERAL_PASS` / `TOSS` | **LAT** | Passe Lateral / Pitch | 0 | ✅ | ✅ | ✅ | ❌ Não | Corredor, Jardas |
| 14 | `BAD_SNAP` | **BSNP**| Snap Ruim / Bola Morta | 0 | ✅ | ✅ | ✅ | ❌ Não | Perda de jardas |
| 15 | `TURNOVER_ON_DOWNS` | **TOD** | Estouro de 4ª Descida | 0 | ✅ | ✅ | ✅ | ✅ Sim | Nenhum (apenas registra a virada de posse) |
| 16 | `KNEEL` | **KNL** | Ajoelhar / Kneel | 0 | ✅ | ✅ | ✅ | ❌ Não | QB (gastar tempo de relógio) |
| 17 | `SPIKE` | **SPK** | Spike (Parar Cronômetro)| 0 | ❌ | ⚠️ | ✅ | ❌ Não | QB |
| **C. DEFESA (DEFENSE)** |
| 18 | `FLAG_PULL` | **PULL**| Retirada de Fita | 0 | ✅ | ✅ | ❌ | ❌ Não | Atleta defensor |
| 19 | `SACK` | **SCK** | Sack (Queda do QB) | 0 | ✅ | ✅ | ✅ | ❌ Não | Atleta defensor (Rusher), Jardas negativas |
| 20 | `INTERCEPTION` | **INT** | Interceptação | 0 | ✅ | ✅ | ✅ | ✅ Sim | Atleta defensor, Jardas de retorno |
| 21 | `PASS_DEFLECTION` | **PD** | Passe Espalmado / Desviado| 0 | ✅ | ✅ | ✅ | ❌ Não | Atleta defensor |
| 22 | `TACKLE` | **TCK** | Tackle (Derrubada) | 0 | ❌ | ❌ | ✅ | ❌ Não | Atleta defensor (Full Pads) |
| 23 | `FUMBLE_FORCED` | **FF** | Fumble Forçado | 0 | ❌ | ❌ | ✅ | ❌ Não | Atleta defensor |
| 24 | `FUMBLE_RECOVERY` | **FR** | Recuperação de Fumble | 0 | ❌ | ❌ | ✅ | ✅ Sim | Atleta que recuperou a posse |
| **D. TIMES ESPECIAIS (SPECIAL TEAMS)** |
| 25 | `PUNT` | **PNT** | Punt (Chute de Devolução)| 0 | ❌ | ⚠️ | ✅ | ✅ Sim | Chutador, Jardas |
| 26 | `PUNT_RETURN` | **PR** | Retorno de Punt | 0 | ❌ | ⚠️ | ✅ | ❌ Não | Retornador, Jardas |
| 27 | `BLOCKED_PUNT` | **BPNT**| Punt Bloqueado | 0 | ❌ | ⚠️ | ✅ | ⚠️ Varia | Defensor que bloqueou |
| 28 | `KICKOFF` | **KOK** | Kickoff (Chute Inicial) | 0 | ❌ | ⚠️ | ✅ | ✅ Sim | Chutador |
| 29 | `KICKOFF_RETURN` | **KR** | Retorno de Kickoff | 0 | ❌ | ⚠️ | ✅ | ❌ Não | Retornador, Jardas |
| 30 | `TOUCHBACK` | **TB** | Touchback | 0 | ❌ | ⚠️ | ✅ | ✅ Sim | Nenhum (bola posicionada na linha padrão) |
| 31 | `MISSED_FIELD_GOAL` | **MFG** | FG Não Convertido | 0 | ❌ | ⚠️ | ✅ | ✅ Sim | Nenhum |
| 32 | `BLOCKED_FIELD_GOAL`| **BFG** | FG Bloqueado | 0 | ❌ | ⚠️ | ✅ | ⚠️ Varia | Defensor que bloqueou |
| **E. FALTAS (PENALTY)** |
| 33 | `PENALTY` (Ataque) | **PEN** | Falta do Ataque | 0 | ✅ | ✅ | ✅ | ❌ Não | Tipo da falta, Jardas (-5/-10 yd), Efeito down |
| 34 | `PENALTY` (Defesa) | **PEN** | Falta da Defesa | 0 | ✅ | ✅ | ✅ | ❌ Não | Tipo da falta, Jardas (+5/+10 yd), Efeito down |
| 35 | `DOUBLE_PENALTY` | **DPEN**| Falta Dupla / Compensação| 0 | ✅ | ✅ | ✅ | ❌ Não | Repetir descida |
| 36 | `EJECTION` | **EJCT**| Expulsão / Desqualificação| 0 | ✅ | ✅ | ✅ | ❌ Não | Atleta / Técnico expulso |
| **F. CONTROLE & TEMPO (GAME_CONTROL)** |
| 37 | `TIMEOUT` | **TO** | Pedido de Tempo | 0 | ✅ | ✅ | ✅ | ❌ Não | Equipe solicitante |
| 38 | `TWO_MINUTE_WARNING`| **2MIN**| Aviso de 2 Minutos | 0 | ✅ | ✅ | ✅ | ❌ Não | Período de jogo |

---

## 3. Análise Atual de Cliques: Do Mais Simples ao Mais Complexo

Analisando minuciosamente o código de [`PlayPadWidget`](file:///C:/Projetos/America/flag_referee_app/lib/ui/game_operation/widgets/play_pad_widget.dart) e [`PlayDialog`](file:///C:/Projetos/America/flag_referee_app/lib/ui/game_operation/widgets/play_dialog.dart), medimos a quantidade exata de toques necessários na interface atual:

### Cenário 1: O Mais Simples Hoje (Timeout & Passe Incompleto)
- **Timeout da Equipe:**
  1. Clicar no botão `Tempo & Controle` no PlayPad.
  2. Rolar/localizar e tocar em `Pedido de Tempo`.
  3. Selecionar qual equipe pediu o tempo (`Casa` ou `Fora`).
  4. Clicar no botão `Confirmar`.
  * **Total: 4 cliques** para a ação mais rotineira da mesa.
- **Passe Incompleto (Avanço de Descida):**
  1. Clicar no botão de categoria `Ataque`.
  2. Tocar no chip `Passe Incompleto`.
  3. (Equipe de ataque pré-selecionada).
  4. Tocar no chip do Atleta (QB) (obrigatório pelo formulário atual).
  5. Clicar no botão `Confirmar lance`.
  * **Total: 4 a 5 cliques** para um lance sem ganho de jardas que ocorre dúzias de vezes por jogo.

### Cenário 2: Complexidade Média (Flag Pull, Corrida & Sack)
- **Flag Pull (Retirada de Fita):**
  1. Clicar no botão de categoria `Defesa`.
  2. Tocar no chip `Retirada de Fita`.
  3. (Equipe defensora pré-selecionada).
  4. Tocar no chip do Atleta defensor.
  5. Clicar no botão `Confirmar lance`.
  * **Total: 4 a 5 cliques** (representa mais de 50% das finalizações de jogadas no Flag).
- **Corrida Simples com Ganho de Jardas (+5 yd):**
  1. Clicar em `Ataque`.
  2. Tocar no chip `Corrida`.
  3. Tocar no chip do Corredor.
  4. Tocar no chip de jardas `+5 yd`.
  5. Clicar no botão `Confirmar lance`.
  * **Total: 5 cliques**.
- **Sack (-5 yd):**
  1. Clicar em `Defesa`.
  2. Tocar no chip `Sack`.
  3. Tocar no chip do Rusher.
  4. Tocar no chip de jardas `-5 yd`.
  5. Clicar no botão `Confirmar lance`.
  * **Total: 5 cliques**.

### Cenário 3: O Mais Complexo Hoje (Passe Completo, Falta e Touchdown + PAT)
- **Passe Completo com Avanço (+15 yd):**
  1. Clicar em `Ataque`.
  2. Tocar no chip `Passe Completo`.
  3. Tocar no chip do Lançador (QB).
  4. Tocar no chip do Recebedor (WR).
  5. Tocar no chip de jardas `+15 yd`.
  6. Clicar no botão `Confirmar lance`.
  * **Total: 6 cliques**.
- **Falta com Penalidade (ex: Flag Guarding no Ataque, -5 yd, perda de descida):**
  1. Clicar em `Faltas`.
  2. Tocar no tipo de falta (`Proteção de Flag`).
  3. Tocar no lado infrator (`Ataque`).
  4. Tocar no chip do atleta infrator.
  5. Tocar nas jardas de penalidade (`-5 yd`).
  6. Tocar no efeito na descida (`Perda de descida`).
  7. Clicar no botão `Confirmar lance`.
  * **Total: 7 cliques**.
- **Touchdown Seguido de Ponto Extra (Ciclo Completo de Pontuação):**
  1. Clicar em `Ataque` $\rightarrow$ Tocar `Touchdown` $\rightarrow$ Tocar no Pontuador $\rightarrow$ Tocar no Passador (se passe) $\rightarrow$ Confirmar (**5 cliques**).
  2. O modal fecha. O árbitro precisa clicar em `Ataque` $\rightarrow$ Tocar `Ponto Extra (1pt ou 2pts)` $\rightarrow$ Tocar no Pontuador $\rightarrow$ Confirmar (**mais 4 a 5 cliques**).
  * **Total: 9 a 10 cliques** para registrar uma única pontuação completa!

---

## 4. Oportunidades Identificadas e Princípios de Redução

Para chegar na experiência de registro ultra-rápido, estabelecemos 4 princípios de UX esportiva:

```
┌────────────────────────────────────────────────────────────────────────┐
│                   PRINCÍPIOS DE DESIGN OPERACIONAL                     │
├────────────────────────────────┬───────────────────────────────────────┤
│ 1. Zero Modal para Lances      │ Ações mais repetitivas (Incompleto,   │
│    Triviais                    │ Timeout, Flag Pull) acionadas na tela │
│                                │ principal sem carregar modal.         │
├────────────────────────────────┼───────────────────────────────────────┤
│ 2. Inteligência Contextual     │ A posse já diz quem ataca e defende.  │
│    Total                       │ O último QB do drive é sugerido como  │
│                                │ 1ª opção de passe.                    │
├────────────────────────────────┼───────────────────────────────────────┤
│ 3. Encadeamento Automático     │ Após TD, o app dispara de imediato a  │
│    (PAT Chaining)              │ pergunta do Ponto Extra na mesma tela │
│                                │ em 1 toque.                           │
├────────────────────────────────┼───────────────────────────────────────┤
│ 4. Presets Oficiais de Faltas  │ Faltas de Flag já trazem jardas e     │
│                                │ efeitos pré-carregados da regra IFAF. │
└────────────────────────────────┴───────────────────────────────────────┘
```

---

## 5. Arquitetura da Solução Proposta

### 5.1. Barra de Ações Rápidas (*Quick Action Strip*) no `PlayPadWidget`
Adicionar uma barra horizontal de atalhos rápidos diretamente acima dos botões de categoria do `PlayPadWidget`. Essa barra conterá as **4 jogadas que cobrem ~75% de um jogo de Flag**:

```
 ┌─────────────────────────────────────────────────────────────┐
 │                     PLAY PAD PRINCIPAL                      │
 ├─────────────────────────────────────────────────────────────┤
 │ [Posse: TIME A]   [ 2ª Descida & 7 yd ]   [ Jardas: 28 yd ] │
 ├─────────────────────────────────────────────────────────────┤
 │  ⚡ AÇÕES RÁPIDAS (1 a 2 toques):                            │
 │  ┌───────────────┐ ┌───────────────┐ ┌────────────────────┐ │
 │  │ ❌ INCOMPLETO │ │ ✂️ FLAG PULL  │ │ 🏃 CORRIDA (+yd)   │ │
 │  │  (1 TOQUE)    │ │  (2 TOQUES)   │ │  (3 TOQUES)        │ │
 │  └───────────────┘ └───────────────┘ └────────────────────┘ │
 ├─────────────────────────────────────────────────────────────┤
 │  CATEGORIAS COMPLETAS:                                      │
 │  [ 🏈 Ataque ]          [ 🛡️ Defesa ]                      │
 │  [ 🚩 Faltas ]          [ ⏱️ Tempo & Controle ]            │
 └─────────────────────────────────────────────────────────────┘
```

1. **Botão `❌ Incompleto` (1 Toque Direto):**
   - Dispara imediatamente o registro de `PASS_INCOMPLETE` para a equipe com posse.
   - Avança automaticamente a descida (`currentDown + 1`).
   - Não abre modal. Exibe SnackBar com botão *"Desfazer"* (Undo).
2. **Botão `✂️ Flag Pull` (2 Toques):**
   - Toque 1: Clica no botão rápido.
   - Toque 2: Um mini-seletor rápido surge exibindo apenas os chips dos defensores em campo. Toca no defensor e confirma automaticamente!
3. **Botão `🏃 Corrida` (3 Toques):**
   - Abre mini-painel: Corredor $\rightarrow$ Jardas rápidas (`0, +2, +5, +10`) $\rightarrow$ Fecha sozinho ao clicar nas jardas.

### 5.2. Chaining Automático de Pós-Touchdown (PAT Flow)
Ao confirmar um Touchdown (corrida ou passe):
- O diálogo **não se fecha**. Ele transiciona instantaneamente para a tela de Ponto Extra:
  - `[ +1 Ponto (5 yd) ]` | `[ +2 Pontos (10 yd) ]` | `[ ❌ Não Converteu ]`
- O árbitro toca no resultado em **1 toque** e o app:
  1. Registra o Touchdown.
  2. Registra o Ponto Extra correspondente.
  3. Atualiza o placar.
  4. Realiza a inversão automática da posse de bola para a equipe adversária na linha de 5 jardas.

### 5.3. Presets Inteligentes de Faltas Flag IFAF
Eliminar o preenchimento manual de jardas e efeitos de descida para as faltas padronizadas:
- **Flag Guarding**: Pré-preenche `-5 jardas` + `Perda de Descida`.
- **False Start / Illegal Motion**: Pré-preenche `-5 jardas` + `Repetir Descida`.
- **Defensive Offside / Illegal Rush**: Pré-preenche `+5 jardas` + `Repetir Descida`.
- **Defensive Pass Interference (DPI)**: Pré-preenche `1ª Descida Automática` no ponto do lance.

---

## 6. Matriz Comparativa de Cliques: Atual vs Proposto

| Jogada / Lance | Cliques Atuais | Cliques com Novo Fluxo | Redução Real |
|---|:---:|:---:|:---:|
| **Passe Incompleto** | 4-5 cliques | **1 clique direto** | **-75%** |
| **Timeout (Casa / Fora)** | 4 cliques | **1-2 cliques** | **-60%** |
| **Flag Pull (Retirada de Fita)** | 4-5 cliques | **2 cliques** | **-55%** |
| **Sack** | 5 cliques | **3 cliques** | **-40%** |
| **Corrida Simples** | 5 cliques | **3 cliques** | **-40%** |
| **Passe Completo (QB + WR + yd)** | 6-7 cliques | **3-4 cliques** | **-45%** |
| **Falta Flag (ex: Flag Guarding)** | 7 cliques | **3 cliques** | **-57%** |
| **Touchdown + Ponto Extra (PAT)** | 9-10 cliques | **4-5 cliques** | **-55%** |
| **Safety** | 4 cliques | **2 cliques** | **-50%** |

---

## 7. Próximos Passos de Implementação

1. **Fase 1 (Design System & Quick Strip no PlayPad):**
   - Adicionar a `QuickActionStrip` no topo de [`PlayPadWidget`](file:///C:/Projetos/America/flag_referee_app/lib/ui/game_operation/widgets/play_pad_widget.dart).
   - Implementar ação direta de 1 toque para `Passe Incompleto` com feedback de SnackBar de Undo.
2. **Fase 2 (Mini-Seletor de Flag Pull):**
   - Criar mini bottom sheet focado exclusivamente em selecionar o defensor com auto-confirmação imediata ao tocar no chip.
3. **Fase 3 (PAT Chaining no PlayDialog):**
   - Ao salvar um lance com `isTouchdown = true`, transicionar o diálogo para seleção instantânea de PAT (1 toque).
4. **Fase 4 (Validação & Qualidade):**
   - Garantir `flutter analyze` com 0 erros e 0 avisos em `flag_referee_app`.
