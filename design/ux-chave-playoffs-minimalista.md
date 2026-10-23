# Especificação de Design: Chave de Playoffs Kickster (Admin Web & Public App)

> **Status:** Implementado  
> **Data:** 2026-10-22 (Atualizado)  
> **Aplicações:** `flag_admin_web` e `flag_public_app`  
> **Tema:** Cards normais de confrontos Kickster (240×96px na árvore e cards responsivos na lista), escudos circulares em destaque (34–44px), área central de pontuação, colunas por fase com linhas de ligação ortogonais, visualização em tela única no admin_web e navegação conectada no public_app  

---

## 1. Princípios de Design e Diretrizes Kickster

1. **Cards Normais de Confronto Esportivo (`BracketMatchCard`)**:
   - Cada confronto esportivo utiliza a estrutura sóbria e canônica do Kickster com dimensões de **240px de largura por 96px de altura** na árvore:
     - Fundo sólido `AppColors.surface` (`#FFFFFF`).
     - Borda `AppColors.line` (`#E3E7EC`, 1px), raio de 12px e sombra sutil.
     - Destaques semânticos: borda dourada (`#F59E0B`) na Grande Final, borda primária (`#083879`) no card destacado/ativo, e borda vermelha (`#EF4444`) quando a partida está ao vivo.
2. **Escudos Circulares em Destaque & Área Central de Pontuação**:
   - Substituição de tabelas estáticas pelo formato canônico de confronto esportivo:
     - **Equipe Mandante**: `KicksterTeamShield` circular de **34px** (ou **44px** na listagem), com o nome e seed (`#1`) abaixo, e indicação visual discreta de vitória (anel/check verde).
     - **Área Central de Pontuação**: bloco dedicado com placar numérico (`homeScore : awayScore`, 16–24pt `w800`) ou pill `VS` quando o jogo ainda não iniciou, com data e horário em tipografia `Plus Jakarta Sans`.
     - **Equipe Visitante**: `KicksterTeamShield` circular de **34px** (ou **44px** na listagem), com o nome e seed (`#2`) abaixo, com opacidade atenuada caso eliminada.
3. **Cabeçalhos de Fase Enquadrados Dinamicamente (`_PhaseHeader`)**:
   - Dimensões: **240px de largura por 48px de altura**, garantindo que todos os textos ("Wildcard", "Semifinais", "Final", "Decisão do Título") sejam enquadrados perfeitamente com `FittedBox` sem cortes ou quebras artificiais.
   - Subtítulo dinâmico baseado na quantidade de confrontos da fase (ex.: *"Decisão do Título"* ou *"4 jogos"*), sem amarrações a nomes específicos de eventos.
4. **Montagem em Colunas por Fase com Linhas de Ligação (`BracketLayout`)**:
   - No `admin_web` e no modo árvore do `public_app`, os confrontos são dispostos em **colunas sequenciais por fase**:
     - **Coluna 1 (Quartas de Final / Wildcard)**: confrontos empilhados verticalmente com agrupamento ordenado.
     - **Linhas de Ligação (`BracketConnectorPainter`)**: traçados ortogonais com cantos arredondados (raio 8px) conectando o lado direito do confronto anterior ao lado esquerdo do confronto correspondente na fase seguinte.
     - **Coluna 2 (Semifinais)**: confrontos perfeitamente alinhados ao centro geométrico dos seus jogos de origem.
     - **Coluna 3 (Grande Final)**: partida decisiva no topo/centro da chave.

---

## 2. Anatomia do Card Canônico Kickster na Árvore (240×96px)

```
┌────────────────────────────────────────────────────────┐
│ JOGO 1 · QUARTAS DE FINAL                     [ FINAL ]│ <- Cabeçalho compacto (20px)
├────────────────────────────────────────────────────────┤
│     (🛡️ 34px)                               (🐺 34px)   │
│   #1 Spartans           28  :  14        #4 Wolves     │
│   (Mandante Vencedor)   Placar Central   (Visitante)   │
└────────────────────────────────────────────────────────┘
```

---

## 3. Diagrama da Chave em Colunas com Linhas de Ligação

```
   QUARTAS DE FINAL                  SEMIFINAIS                  GRANDE FINAL
  (Coluna de Fase 1)            (Coluna de Fase 2)            (Coluna de Fase 3)

┌──────────────────────┐
│ (🛡️) 28 : 14 (🐺)    │───────┐
│ #1 Spartans / Wolves │       │
└──────────────────────┘       ├───────┌──────────────────────┐
┌──────────────────────┐       │       │ (🛡️) 31 : 24 (⚡)    │───────┐
│ (⚡) 21 : 17 (🦅)    │───────┘       │ #1 Spartans / Storm  │       │
│ #2 Storm / Titans    │               └──────────────────────┘       │
└──────────────────────┘                                              ├───────┌──────────────────────┐
┌──────────────────────┐                                              │       │ (🛡️)   VS   (🦅)     │
│ (🦖) 24 : 10 (⛏️)    │───────┐                                      │       │ #1 Spartans / Eagles │
│ #1 Raptors / Miners  │       │                                      │       └──────────────────────┘
└──────────────────────┘       ├───────┌──────────────────────┐       │            🏆 GRANDE FINAL
┌──────────────────────┐       │       │ (🦅) 20 : 14 (🦖)    │───────┘
│ (🦅) 20 : 18 (🦅)    │───────┘       │ #3 Falcons / Raptors │
│ #3 Falcons / Eagles  │               └──────────────────────┘
└──────────────────────┘
```

---

## 4. Adaptação Específica por Plataforma

### 4.1 Admin Web (`flag_admin_web`): Visualização em Tela Única
- **Modo Tela Única (Padrão)**:
  - Redimensiona e ajusta toda a árvore eliminatória via `FittedBox` responsivo para caber **100% na largura da tela**, permitindo ao organizador inspecionar todas as colunas de fases e suas conexões simultaneamente sem barra de rolagem horizontal.
- **Alternador de Visualização Kickster**:
  - `[ Tela única ]`: visualização panorâmica integral adaptada à viewport.
  - `[ Rolagem livre ]`: visualização em tamanho natural (100%) com barra de rolagem e botões de atalho rápido (*"Início"* e *"Final"*).
- Breakpoint responsivo: abaixo de 768px, apresentação em accordion por fases.

### 4.3 Tratamento de Avanço Direto (BYE) e Equipes Pré-Classificadas
- **Eliminação de Cards Sintéticos Sobrepostos:**
  - Cards artificiais de folga/avanço foram removidos das colunas intermediárias (evitando encavalamento/sobreposição visual com os jogos reais de fases como as semifinais).
- **Badge/Anotação no Time (`directQualifier`):**
  - Quando um time avança direto para uma fase avançada (ex.: 1º colocado de conferência direto para as finais ou semifinais), recebe uma anotação compacta diretamente no card em que disputa a partida:
    - Tag Kickster: `[ 1º CONFERÊNCIA CAIPIRA ]`, `[ 1º METRÓPOLES ]` ou `[ AVANÇO DIRETO ]`.
- **Harmonização Posicional e Conectores Inteligentes:**
  - Centralização vertical proporcional (`centers[c][0] = (centers[c-1].first + centers[c-1].last) / 2`): quando os times chegam direto ou a fase tem menos confrontos, o card da fase seguinte permanece geometricamente balanceado em relação a todos os confrontos da fase anterior.
  - Regra anti-colisão vertical (`minY = centers[c][j-1] + matchHeight + baseGap`).
  - Prevenção de conectores falsos: quando ambos os finalistas avançam direto por classificação, linhas ortogonais falsas conectando semifinais alheias à final não são traçadas.

### 4.2 Public App (`flag_public_app`): Visualização Direta da Árvore Eliminatória em Fases
- **Visualização da Chave Eliminatória**:
  - Apresenta os confrontos em colunas por fases consecutivas conectadas por traçados ortogonais com cantos arredondados.
  - Cards normais de confronto Kickster no formato canônico (240×96px) com escudos circulares de 34px, placar central ou pill `VS`, status, identificação de seeds e anotações de avanço direto no time (`[ 1º CONFERÊNCIA CAIPIRA ]`).
- **Seletor Rápido de Fases no Topo**:
  - Trilha horizontal compacta e limpa no topo com chips de fase:
    `[ 1. Quartas de Final ] ────> [ 2. Semifinais ] ────> [ 🏆 Grande Final ]`
  - O toque em qualquer fase executa navegação/rolagem horizontal suave até a coluna correspondente na árvore.
- **Rolagem Horizontal Fluida**:
  - Suporte nativo a gestos de rolagem horizontal no mobile, sem conflitos com a rolagem vertical da página.
  - Toque no card abre a súmula ao vivo da partida (`/game/:id`).
