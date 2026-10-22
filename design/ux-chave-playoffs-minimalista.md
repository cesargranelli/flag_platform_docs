# Especificação de Design: Chave de Playoffs Kickster (Admin Web & Public App)

> **Status:** Implementado  
> **Data:** 2026-10-22 (Atualizado)  
> **Aplicações:** `flag_admin_web` e `flag_public_app`  
> **Tema:** Design System Kickster canônico, cards estruturados de 210×88px, cabeçalhos 210×52px, chave espelhada estilo NFL / Super Bowl, visualização em tela única e navegação conectada  

---

## 1. Princípios de Design e Diretrizes Kickster

1. **Containers Canônicos Estruturados (`BracketMatchCard`)**:
   - Abandona caixas flutuantes improvisadas e adota os padrões oficiais do UI Kit Kickster (*Figma Live Score & News Sport*).
   - Container de **210px de largura por 88px de altura**:
     - Fundo sólido `AppColors.surface` (`#FFFFFF`).
     - Borda `AppColors.line` (`#E3E7EC`, 1px), raio `AppRadii.cardBorder` (12px) e sombra suave (`BoxShadow` com 4% de opacidade e blur 6).
     - Destaques semânticos: borda dourada (`#F59E0B`) na Grande Final, borda primária (`#083879`) na fase ativa, e borda vermelha (`#EF4444`) quando a partida está ao vivo com dot/badge pulsante.
2. **Duas Linhas Esportivas com Divisor Fino**:
   - Cada equipe ocupa uma linha balanceada de 43.5px com divisor central horizontal de 1px.
   - Componentes da linha:
     - **Seed (#1, #2...)**: 10.5pt `w700`.
     - **Escudo Oficial Kickster (`KicksterTeamShield`)**: 26×26px com proporção natural `BoxFit.contain` e fallback esportivo vetorial com a paleta oficial da agremiação/cidade.
     - **Nome da Equipe**: tipografia `Plus Jakarta Sans`, 12pt (`w700` para o vencedor e atenuado para o eliminado).
     - **Área de Pontuação (Score Box)**: container de `30×24px` alinhado à direita, com fundo primário suave e peso `w800` para o time vencedor.
3. **Cabeçalhos de Fase Enquadrados (`_PhaseHeader`)**:
   - Dimensões: **210px de largura por 52px de altura**, garantindo que títulos extensos como `"WILDCARD"`, `"SEMIFINAIS"`, `"FINAL DE CONFERÊNCIA"` e `"SUPER BOWL"` enquadrem com respiro e sem quebras indesejadas.
   - Fundo `surfaceMuted` (`#F6F8FE`), borda `line` e raio de 10px.
   - Ícone temático Kickster (`Icons.emoji_events` dourado para finais e `Icons.shield_outlined` para fases eliminatórias).
4. **Navegação Espelhada (Estilo NFL / Super Bowl)**:
   - As preliminares iniciam nas extremidades e **ambas as alas navegam em direção ao centro**:
     - **Ala Esquerda (Conferência A / Chave Superior)**: avança da esquerda para o centro.
     - **Ala Direita (Conferência B / Chave Inferior)**: avança da direita para o centro.
     - **Centro (Grande Final / Super Bowl)**: as pontes dos campeões convergem no topo da árvore para a decisão final do título.

---

## 2. Anatomia do Card Canônico Kickster (210×88px)

```
┌────────────────────────────────────────────────────────┐
│  #1  🛡️ [ESCUDO 26px]   SP Spartans               [ 28 ]│ <- Mandante (Vencedor)
├────────────────────────────────────────────────────────┤
│  #4  🐺 [ESCUDO 26px]   Curitiba Wolves           [ 14 ]│ <- Visitante
└────────────────────────────────────────────────────────┘
```

---

## 3. Diagrama da Chave Espelhada Convergente

```
      CONFERÊNCIA A                                   CONFERÊNCIA B
    (Avanço Esquerda →)      ★ GRANDE FINAL ★      (← Avanço Direita)

  ┌──────────────────┐                              ┌──────────────────┐
  │ #1 Spartans [28] │──┐                        ┌──│ #1 Raptors  [24] │
  │ #4 Wolves   [14] │  │                        │  │ #4 Miners   [10] │
  └──────────────────┘  ├──┌──────────────────┐──┤  └──────────────────┘
  ┌──────────────────┐  │  │ #1 Spartans [  ] │  │  ┌──────────────────┐
  │ #2 Storm    [21] │──┘  │ #1 Raptors  [  ] │  └──│ #2 Eagles   [18] │
  │ #3 Titans   [17] │     └──────────────────┘     │ #3 Falcons  [20] │
  └──────────────────┘          🏆 SUPER BOWL       └──────────────────┘
```

---

## 4. Adaptação Específica por Plataforma

### 4.1 Admin Web (`flag_admin_web`): Visualização em Tela Única (Fit-to-Screen)
- **Modo Tela Única (Padrão)**:
  - Redimensiona e ajusta toda a árvore eliminatória via `FittedBox` responsivo para caber **100% na largura disponível da viewport**, eliminando o scroll horizontal forçado e permitindo ao organizador inspecionar todos os confrontos em uma única visualização limpa.
- **Alternador de Visualização Kickster**:
  - `[ Tela única ]`: escala proporcional automática para visão geral sem scroll.
  - `[ Rolagem livre ]`: exibição em tamanho original 100% com barra de rolagem e atalhos rápidos (*"Início"* e *"Final (Centro)"*).
- Breakpoint responsivo otimizado para **768px** (tablets e desktops operam em tela única).

### 4.2 Public App (`flag_public_app`): Navegação Conectada entre Fases
- **Stepper Horizontal de Fases Conectadas**:
  - Trilha interativa no topo ligando as etapas eliminatórias com linhas de progressão:
    `[ 1. Quartas de Final ] ────> [ 2. Semifinais ] ────> [ 🏆 Grande Final ]`
  - Toque direto em qualquer fase para navegação instantânea.
- **Ligação Explícita dos Confrontos**:
  - **Destino do Vencedor**: banner contextual no rodapé de cada confronto com seta e rótulo:
    `Vencedor avança para: Semifinais (Jogo 1)` acompanhado de botão de salto rápido `[ Ver próximo → ]` que troca de fase e destaca o confronto de destino.
  - **Origem dos Participantes**: banner superior nos confrontos de fases avançadas:
    `Origem: Vencedores da Fase Anterior (Jogos 1 e 2)` com botão `[ Ver fase anterior ← ]`.
- **Alternador de Modos no Public App**:
  - `[ Fases ]`: navegação guiada mobile-first conectando os confrontos de ponta a ponta.
  - `[ Árvore ]`: visão panorâmica com suporte a zoom livre e pan via `InteractiveViewer`.
- Toque no confronto esportivo abre diretamente a súmula ao vivo da partida (`/game/:id`).
