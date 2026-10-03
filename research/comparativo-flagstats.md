# Comparativo: Tela de Registro de Eventos

> FlagStats (flagstats.app) × flag_referee_app — análise baseada nas screenshots reais.

---

## Telas do FlagStats

### 1. Tela de Scouting (registro da jogada)

![FlagStats Scouting](/C:/Users/Cesar/.gemini/antigravity-cli/brain/dd64c2cd-2c1f-4fb6-84f9-6a6b125eb857/flagstats_scouting.png)

**Fluxo de entrada — Wizard em etapas sequenciais:**

```
[Qual é a tentativa?]    → 1ST ou TD  (tipo de down/objetivo)
        ↓
[Qual é a descida?]      → 1ª / 2ª / 3ª / 4ª  (down number)
        ↓
[Como foi a jogada?]     → PASSE ou CORRIDA
        ↓
[ENCERRAR JOGADA]        → persiste e volta ao início
```

**Características do fluxo:**
- Cada pergunta tem apenas 2–4 opções, nunca mais
- Botão "Pular pergunta" disponível em cada etapa — reduz atrito, mantém velocidade
- Interface mínima: uma pergunta por tela, fonte grande, botões inteiros
- Sem formulário de texto — tudo por tap
- Projetado para **digitação com um polegar** durante o jogo

---

### 2. Tela de Timeline + Estatísticas

![FlagStats Timeline](/C:/Users/Cesar/.gemini/antigravity-cli/brain/dd64c2cd-2c1f-4fb6-84f9-6a6b125eb857/flagstats_timeline.png)

**Estrutura da timeline:**
- Abas **Timeline** / **Estatísticas** no topo
- Dois lados: **Ataque** (esquerda) e **Defesa** (direita) — linha do tempo dividida
- Cada evento exibe: descida + tipo de jogada + resultado (`1ª descida / pass → reception`)
- Timestamp real (`17:37:55`) ao lado de cada evento
- Marcos de período (`Início do 1º tempo`, `FIRST DOWN`) centralizados na linha
- Botões `ENCERRAR 1º TEMPO` / `CONTINUAR 1º TEMPO` no rodapé — controle de período integrado à timeline

---

### 3. Tela de Estatísticas do Time (output)

![FlagStats Stats Team](/C:/Users/Cesar/.gemini/antigravity-cli/brain/dd64c2cd-2c1f-4fb6-84f9-6a6b125eb857/flagstats_stats_team.png)

**Métricas calculadas automaticamente a partir das jogadas:**
- Pontos pró
- TDs / Conversão XP 1pt / XP 2pt
- Conversão de 1ª descida
- Passes completos / incompletos / % conclusão
- Turnovers
- (derivados 100% dos eventos registrados)

---

### 4. Tela de Estatísticas do Atleta (output)

![FlagStats Stats Athlete](/C:/Users/Cesar/.gemini/antigravity-cli/brain/dd64c2cd-2c1f-4fb6-84f9-6a6b125eb857/flagstats_stats_athlete.png)

**Por atleta, segmentado por contexto (Campeonatos / Amistosos / Treinos):**
- TDs, First Downs, Jardas, Tackles, Interceptações, Sacks
- Agrupado por competição com contador de jogos

---

## Análise Comparativa

### Propósito radicalmente diferente

| Dimensão | FlagStats | flag_referee_app |
|---|---|---|
| **Quem usa** | Treinador / assistente do time | Árbitro de mesa (neutro) |
| **Objetivo do registro** | Gerar estatísticas de desempenho para análise técnica | Documentar ocorrências oficiais para a súmula da partida |
| **Implicação legal** | Nenhuma — é ferramenta do time | Resultado vai para o sistema oficial da liga |
| **Quem vê os dados** | O próprio time (privado) | Liga, ambas as equipes, árbitros |

---

### Fluxo de Entrada: Wizard × Pad de ações

| Aspecto | FlagStats | flag_referee_app (PlayPad) |
|---|---|---|
| **Paradigma** | **Wizard sequencial** — uma pergunta de cada vez | **Grid de botões** — todos os tipos visíveis simultaneamente |
| **Seleção de tipo** | Tap em 2 opções (passe/corrida) | Tap em 1 de 17 tipos agrupados em 5 categorias |
| **Dados do atleta** | ⚠️ Não visível na tela — provável seleção prévia | **PlayDialog** com busca de atleta por nome |
| **Yardagem** | Implícita no resultado (reception) | Campo numérico no `PlayDialog` |
| **Quarter/Período** | Gerenciado pela timeline (botão encerrar/continuar) | `KicksterDropdown` no `PlayPadWidget` |
| **Velocidade de input** | ⚡ Extremamente rápido (wizard + pular) | 🔶 Médio (tap → bottom sheet → formulário → confirmar) |
| **Erro durante o jogo** | Baixo — poucas opções por tela | Médio — formulário completo exige atenção |

---

### Timeline: lado a lado vs. lista

| Aspecto | FlagStats | flag_referee_app (ScoreTimeline) |
|---|---|---|
| **Layout** | **Linha central** dividida em Ataque / Defesa, com timestam | **Lista vertical** por evento de pontuação |
| **Dados exibidos** | Descida + tipo + resultado de cada lance | Apenas eventos de pontuação (`ScoreEvent`) — sem detalhes de lance |
| **Marcos de período** | `Início do 1º tempo`, `FIRST DOWN` centralizados | ❌ Não implementado |
| **Controle de período** | Botão inline na timeline | Dropdown separado no PlayPad |
| **Não-pontuações** | ✅ Visíveis (passes, corridas, desces) | ❌ Não aparecem (só `ScoreEvent`) |

---

### Dados registrados por jogada

| Campo | FlagStats | flag_referee_app (`PlayResponse`) |
|---|---|---|
| Tipo de tentativa (1ST/TD) | ✅ | ❌ (não modelado) |
| Descida (1ª/2ª/3ª/4ª) | ✅ | ✅ (`currentDown` — local only) |
| Tipo de jogada (pass/run) | ✅ | ✅ (`playType`) |
| Resultado (reception, etc.) | ✅ | Parcial (`isFirstDown`, `isTurnover`) |
| Timestamp real (hh:mm:ss) | ✅ | ✅ (`time` no `PlayResponse`) |
| Atleta executante | ✅ (inferido) | ✅ (`playerName`) |
| Receptor | Provável | ✅ (`receiverName`) |
| Jardas | ✅ (resultado) | ✅ (`yards`) |
| Quarter | ✅ (via marco de período) | ✅ (`quarter`) |
| Pontos | Derivado | ✅ (`isTouchdown`, `pointsToAdd`) |

---

## Conclusão e oportunidades

### O que o FlagStats faz bem e podemos aprender

1. **Wizard rápido para entrada de dados** — considerar um modo "fast-tap" para o árbitro registrar jogadas sem abrir o `PlayDialog` completo (apenas tipo + time, confirmação em 2 taps)

2. **Timeline bidirecional Ataque/Defesa** — visualmente mais claro do que uma lista simples; identifica imediatamente de qual lado cada jogada pertence

3. **Marco de período integrado à timeline** — `Início do 1º tempo`, `FIRST DOWN` como eventos na linha do tempo, não controles externos

4. **"Pular pergunta"** — reduz atrito quando o árbitro não tem certeza de um dado; permite completar depois

### O que o flag_referee_app tem e o FlagStats não tem (e não precisa)

- Controle de status oficial (SCHEDULED → OPEN → IN_PROGRESS → CONFERENCE → FINISHED)
- Integração com backend da liga para resultado oficial
- Controle de quórum e check-in de atletas
- Validação de elegibilidade
- Súmula / conferência de resultado

> [!NOTE]
> O FlagStats registra **o que aconteceu** para análise técnica interna do time.
> O flag_referee_app registra **o que ocorreu oficialmente** para fins regulamentares da liga.
> São propósitos complementares — o árbitro e o assistente técnico podem usar os dois simultaneamente durante o mesmo jogo.
