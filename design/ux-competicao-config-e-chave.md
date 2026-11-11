# UX — Reorganização da Configuração de Competição e da Visualização da Chave de Playoffs

> **Tipo:** proposta de design (diagnóstico + IA/fluxo + modelo visual + plano). **Sem implementação de código.**
> **Escopo:** `flag_admin_web` (Flutter Web).
> **Origem:** solicitação do usuário em 2026-10-08 (tela de competição sobrecarregada; config inline; falta de *progressive disclosure*; chave de playoffs difícil de ler).
> **Relacionado:** [ADR-017](../../adr/ADR-017-configuracao-playoffs.md) · [ADR-021](../../adr/ADR-021-geracao-fase-a-fase-playoffs.md) · [spec fases e geração](../../_planning/playoffs-configuracao/spec-fases-e-geracao.md) · [tokens de design](tokens.md) · [layout-spec](layout-spec.md).
> **Estado:** aguardando validação do tech-lead (ver §7, tensão com o AGENTS §4.1).

---

## 0. Sumário executivo

A tela de competição hoje é **um formulário monolítico** (Create/Edit) + **um detalhe longo** + **um modal de playoffs sobrecarregado**, sem separação por etapas e sem *revelação progressiva*. A chave de playoffs é desenhada como **colunas de cards sem conectores**, o que não comunica a árvore de eliminação.

Esta proposta separa a configuração em **5 etapas** (competição, forma de disputa, agrupamento, regras de classificação, playoffs) apresentadas como **páginas/modais distintos** com fluxo de *wizard* (create) e **hub de configuração** (edit), adiciona *progressive disclosure* em cada passo e substitui a visualização da chave por uma **árvore de eliminação com caixas e conectores em "L"**, com byes, destaque de fase atual e responsividade.

---

## 1. Diagnóstico

### 1.1 Objetivo do usuário e tarefa principal

**Persona:** organizador/admin da competição.
**Tarefa A (cadastro):** criar uma competição — definindo formato, agrupamento, regras e playoffs.
**Tarefa B (gestão):** acompanhar e ajustar a competição e sua chave eliminatória ao longo do campeonato.
**Critérios de sucesso:** cadastrar sem precisar entender todos os conceitos de uma vez; saber sempre "onde estou" e "o que falta"; ler a chave e entender quem joga contra quem e o caminho até a final.

### 1.2 Arquitetura de informação atual (com evidência no código)

| Área | Arquivo | O que contém hoje |
|---|---|---|
| Listagem | `competition_list_screen.dart` | Busca + filtro de organização + cards + menu (índice de entrada) |
| Cadastro | `competition_create_screen.dart` (528 linhas) | 6 seções numa única rolagem: Identificação, Formato, Agrupamento, Regras de classificação, Playoffs, Modalidade & Categoria, Datas |
| Edição | `competition_edit_screen.dart` (674 linhas) | Mesmas 6 seções + notas de bloqueio DRAFT; ~90% do layout duplicado do create |
| Detalhe | `competition_detail_screen.dart` (898 linhas) | Info geral + Janela de inscrição + 2 cards de navegação + Regras (leitura) + Card "Chave de Playoffs" + card de recalculo (ADMIN) |
| Config. playoffs | `competition_playoff_section.dart` (208 linhas) | Toggle + classificados + wildcards + formato + geração automática + editor de fases completo + critérios de desempate |
| Modal playoffs | `competition_playoffs_config_modal.dart` (272 linhas) | Modal 720×560 com inputs base + editor de fases + 3 ações (Fechar/Salvar/Gerar) |
| Editor de fases | `competition_playoff_phases_editor.dart` (738 linhas) | Preview em chips + lista de fases; cada fase com nome, tipo, N confrontos; cada confronto com 4 dropdowns de vaga |
| Ver chave | `competition_playoff_bracket_screen.dart` (651 linhas) | Colunas de 300px em scroll horizontal, cards de confronto empilhados por `Expanded` |

### 1.3 Problemas de usabilidade (achados por severidade)

#### Crítico

**C1 — Formulário monolítico sem etapas (Create/Edit).**
Todas as decisões aparecem juntas, numa única rolagem, independentemente do formato escolhido.
*Evidência:* `competition_create_screen.dart:53-497` (Coluna única com seções 1..4 + agrupamento + regras + playoffs); `competition_edit_screen.dart:74-579` (idêntico).
*Impacto:* carga cognitiva alta; o usuário precisa entender playoffs, agrupamento e desempate antes de decidir o formato.

**C2 — "Ver Chave" sem conectores: a árvore não é legível.**
As fases são colunas isoladas de cards; não há nenhuma linha ligando um confronto ao seu sucessor, nem alinhamento em funil.
*Evidência:* `competition_playoff_bracket_screen.dart:373-389` (`_buildColumns` cria `SizedBox(width: 300)` + `_buildColumn`, sem `CustomPaint`/conectores) e `:434-440` (slots posicionados com `Expanded`, sem garantia de alinhamento com os dois confrontos de origem).
*Impacto:* o usuário não percebe "quem avança para onde" nem o afunilamento até a final — exatamente o problema relatado.

#### Alto

**A1 — Configurações inline, não em páginas/modais.**
Agrupamento, regras de classificação e playoffs ocupam a mesma página; só a chave tem modal.
*Evidência:* `competition_create_screen.dart:222` (`CompetitionGroupingSection` inline), `:227` (`CompetitionStandingsRulesSection` inline), `:235` (`CompetitionPlayoffSection` inline).
*Impacto:* a jornada vira "rolagem infinita"; nada "salta" a tela para focar a decisão.

**A2 — Falta de progressive disclosure: tudo pré-selecionado/apresentado.**
Assim que "Habilitar Playoffs" é ligado, surgem de uma vez: classificados, wildcards, formato, geração automática, editor de fases completo e critérios de desempate.
*Evidência:* `competition_playoff_section.dart:90-120` (`if (vm.enabled) ...` renderiza tudo em bloco).
Critérios de desempate já nascem populados com 3 regras (`competition_playoff_view_model.dart:34-40`) e o agrupamento impõe mínimo de 2 grupos na criação de grupos (`competition_grouping_section.dart:168, 203`).
*Impacto:* o usuário vê opções irrelevantes antes de escolher; confusão e erros de configuração.

**A3 — Modal de playoffs sobrecarregado.**
Um modal 720×560 concentra inputs base **e** o editor de fases inteiro (que renderiza N fases × M confrontos × 4 dropdowns), mais 3 ações.
*Evidência:* `competition_playoffs_config_modal.dart:92-99` (tamanho) e `:118-148` (conteúdo = editor completo); `competition_playoff_phases_editor.dart:62-71` (loop de `_PhaseCard`), `:350-359` (loop de `_MatchupEditor`), `:542-559` (4 dropdowns por vaga).
*Impacto:* conteúdo denso demais para um modal; rolagem dentro de modal dentro de tela; o preview da árvore é só uma lista de *chips* (`:124-150`), não a árvore.

#### Médio

**M1 — Detalhe mistura leitura, ações e configuração.**
*Evidência:* `competition_detail_screen.dart:142-396` (info geral + inscrição + navegação + regras + playoffs + recálculo ADMIN).
*Impacto:* o detalhe deixa de ser "visão geral" e vira painel de config.

**M2 — Sem indicador de etapa / orientação.**
Não há stepper nem navegação por sessões; o usuário não sabe "quanto falta".
*Evidência:* ausência de qualquer componente de passos; o `admin_web` não possui (`lib/ui/core/ui/` não tem step indicator — ver §6, Fase 0).

**M3 — Fases da chave mal sinalizadas na árvore.**
Só a rodada final recebe cor `primary` (`competition_playoff_bracket_screen.dart:392`); não há marcação de "fase atual"/"em andamento", embora o modelo (`PlayoffPhaseStatus`) e o card de resumo já as tenham (`competition_playoffs_card.dart:237-248`).
*Impacto:* o usuário não localiza "onde o campeonato está".

**M4 — `Geração automática de jogos` é um affordance falso.**
Exposto no formulário (`competition_playoff_section.dart:98-103`) mas, conforme a spec, `auto_generate` "é armazenado mas não usado" (`spec-fases-e-geracao.md:18`).
*Impacto:* expectativa não cumprida.

**M5 — Duplicação Create/Edit.**
~90% do layout duplicado entre as duas telas; qualquer ajuste precisa ser feito duas vezes (risco de divergência).
*Evidência:* comparar `competition_create_screen.dart` e `competition_edit_screen.dart` (seções `_buildSectionCard` idênticas, mesmos cards).

#### Baixo / Sugestão

**B1 — Estilos tipográficos literais em vez de `AppTextStyles`/`Theme.textTheme`.**
Contraria a regra de tokens (`tokens.md:66`). *Evidência:* dezenas de `TextStyle(fontSize: 13, fontWeight: ...)` em `competition_playoff_phases_editor.dart`, `competition_playoffs_card.dart`, `competition_detail_screen.dart`.
**B2 — Mistura de estados legado/Kickster.**
`AppLoading` + `AppErrorState` + `KicksterEmptyState` na mesma tela (`competition_playoff_bracket_screen.dart:116, 119, 125`).
**B3 — Rótulos de ação inconsistentes.**
"Configurar Chave" (outline), "Ver Chave" (outline), "Gerar próxima fase" (primary) em lugares diferentes; o fluxo configurar→gerar→ver não é uma sequência clara.
**B4 — Mensagens de bloqueio possivelmente obsoletas.**
Edit diz que playoffs "só podem ser alterados em rascunho (DRAFT)" (`competition_edit_screen.dart:626`), mas o ADR-021 removeu a restrição de DRAFT para geração/edição de fases.

---

## 2. Proposta de IA / fluxo

### 2.1 Princípios

1. **Cada decisão tem um lugar.** Config não fica inline no detalhe.
2. **Revelar sob escolha.** Nada de campos irrelevantes antes da decisão anterior.
3. **Sem bloqueio artificial.** As etapas são de *navegação*, não de *validação obrigatória com rascunho* (respeita `AGENTS.md` §4.1 — ver §7).
4. **1 dado = 1 lugar** (herda a regra do Public App em `tokens.md:265`).

### 2.2 Etapas (a–e)

| Etapa | Título | Conteúdo | Tipo de superfície |
|---|---|---|---|
| **a** | Competição | Identificação & organização, nome, temporada, status, datas, modalidade, gênero, categoria, regulamento | Passo de wizard (create) / Página de config (edit) |
| **b** | Forma de disputa | `TournamentFormat` (pontos corridos / grupos / grupos+playoffs / playoffs) | Passo de wizard / Página |
| **c** | Agrupamento | Nenhum · Grupos · Conferências (+divisões) | Passo de wizard / Página (condicional) |
| **d** | Regras de classificação | Critérios de desempate + parâmetros | Passo de wizard / Página |
| **e** | Playoffs | Habilitar, formato da eliminatória, classificados/wildcards, árvore de fases, geração | Passo de wizard / Página (condicional) |

### 2.3 Fluxo — Criação (wizard)

Rota única `/competitions/new` com **stepper** de navegação; **um único salvar ao final** (ou "Salvar rascunho" opcional). As etapas (c) e (e) são **ocultadas/desabilitadas** conforme a escolha em (b).

```
┌ Nova Competição ─────────────────────────────────────────────┐
│ (a) Competição ─ (b) Disputa ─ (c) Agrupamento ─ (d) Regras ─ (e) Playoffs │  ← stepper
├──────────────────────────────────────────────────────────────┤
│  [ conteúdo da etapa ativa — ver §3 ]                        │
│                                                              │
│  [Cancelar]                         [Voltar] [Avançar/Salvar]│
└──────────────────────────────────────────────────────────────┘
```

- Clicar no stepper navega livremente (sem travar).
- O rodapé primário muda: nas etapas internas = **Avançar**; na última = **Criar Competição**.
- Ordem dos passos visíveis varia: em "Pontos corridos" o stepper mostra apenas `a · b · d`.

### 2.4 Fluxo — Edição / Gestão (hub de configuração)

Nova rota `/competitions/:id/settings` — **um hub** com o mesmo stepper (modo "sessões", apenas a etapa ativa é exibida), replicando o padrão `AppStepIndicator` no modo ícones (tokens.md:202-205).

```
Detalhe (/competitions/:id)                 Hub de Configuração (/settings)
┌───────────────────────────┐               ┌───────────────────────────────┐
│ Competição: Copa Brasil   │  [Configurar] │ (a)─(b)─(c)─(d)─(e)  stepper  │
│ ─ Resumo (só leitura) ─    │──────────────▶│  [sessão ativa]               │
│  • Formato  • Agrupamento  │               │  Salvar · Voltar ao detalhe   │
│  • Regras   • Playoffs     │               └───────────────────────────────┘
│ ─ Gestão Esportiva ─       │
│  • Equipes • Tabela        │               Chave (/competitions/:id/playoffs/bracket)
│ ─ Chave de Playoffs ─      │──────────────▶┌───────────────────────────────┐
│  estados + Gerar + Ver     │               │  Árvore com conectores         │
└───────────────────────────┘               └───────────────────────────────┘
```

- **Detalhe** volta a ser **visão geral**: identidade + resumo de config (só leitura) + gestão esportiva + card enxuto de playoffs (estado das fases + "Gerar próxima fase" + "Ver Chave") + um único CTA **"Configurar Competição"** que leva ao hub.
- O hub mantém uma sub-rota por etapa (`/settings/general`, `/settings/format`, `/settings/grouping`, `/settings/standings`, `/settings/playoffs`) para deep-link e botão voltar do navegador.

### 2.5 Wireframes textuais

**(a) Competição**
```
┌ Competição ───────────────────────────────────────────────┐
│ Identificação & Organização                               │
│  [Organização Promotora ▾]   (visível só no create)       │
│  [Nome da Competição]                                     │
│  [Temporada]      [Status inicial ▾]                      │
│  [Descrição / Regulamento (opcional)]                     │
│                                                            │
│ Classificação Esportiva                                    │
│  [Modalidade ▾]  [Gênero ▾]  [Categoria ▾]                │
│                                                            │
│ Datas (opcional)                                           │
│  [Início]  [Término]                                       │
└────────────────────────────────────────────────────────────┘
```

**(b) Forma de disputa** — apenas cartões de escolha + descrição, nada mais:
```
┌ Forma de disputa ─────────────────────────────────────────┐
│ Como os times disputam o título?                          │
│ ┌────────────┐ ┌────────────┐ ┌──────────────┐ ┌────────┐ │
│ │ Pontos     │ │ Fase de    │ │ Grupos +     │ │ Playoffs│ │
│ │ corridos   │ │ grupos     │ │ Playoffs     │ │ puros   │ │
│ └────────────┘ └────────────┘ └──────────────┘ └────────┘ │
│                                                           │
│ ℹ O que muda:  "Grupos + Playoffs" habilita agrupamento e  │
│   playoffs nas próximas etapas.                            │
└───────────────────────────────────────────────────────────┘
```

**(c) Agrupamento** — escolha primeiro; editor só depois (e vazio):
```
┌ Agrupamento ──────────────────────────────────────────────┐
│ ( ) Tabela única   ( ) Fase de grupos   ( ) Conferências  │
│                                                            │
│ -- só aparece após escolher "Fase de grupos" --            │
│ Grupos (0)                              [+ Adicionar grupo]│
│   Nenhum grupo ainda. Adicione o primeiro.                 │
│                                                            │
│ -- só aparece após escolher "Conferências" --              │
│ Conferências (0)                    [+ Adicionar conferência]│
│   Nenhuma conferência ainda.                               │
└────────────────────────────────────────────────────────────┘
```

**(d) Regras de classificação** — base fixa visível, critérios sob demanda:
```
┌ Regras de classificação ──────────────────────────────────┐
│ Base: Vitória = 3 pts · Empate = 1 pt · Derrota = 0        │
│                                                            │
│ Critérios de desempate (0)          [+ Adicionar critério] │
│   Nenhum critério definido. Adicione se houver empates.    │
└────────────────────────────────────────────────────────────┘
```
> Sugestão: ao adicionar, o critério surge com os parâmetros já visíveis (revelação sob escolha), em vez de pré-popular 3 regras.

**(e) Playoffs** — revelação em camadas:
```
┌ Playoffs ─────────────────────────────────────────────────┐
│ Habilitar playoffs?      [ toggle ]                        │
│                                                            │
│ -- só após habilitar --                                    │
│ Formato da eliminatória                                    │
│   [Eliminação simples ▾]  (descrição)                      │
│                                                            │
│ -- só após escolher formato --                             │
│ Classificados por balde [2]     Wildcards [0]              │
│                                                            │
│ -- só após informar classificados --                       │
│ Chave                                            [Sugerir árvore]│
│   (preview da árvore com conectores — ver §5)              │
│   Fases (N)                             [+ Adicionar fase] │
│   ┌ Fase 1: Wildcard ────────────────────────── [⋮] ┐      │
│   │  Confrontos ...  (editores de vaga)              │      │
│   └──────────────────────────────────────────────────┘      │
└────────────────────────────────────────────────────────────┘
```

---

## 3. Progressive disclosure — regras por passo

| Passo | Sempre visível | Só após escolha | Nunca junto |
|---|---|---|---|
| (b) Disputa | 4 cartões + explicação de consequência | descrição específica do formato selecionado | nenhum outro campo |
| (c) Agrupamento | 3 opções | grupos/conferências vazios + "adicionar" | listas pré-preenchidas |
| (d) Regras | regras-base de pontuação (leitura) | critérios **adicionados um a um**; parâmetros por critério | lista de 3 critérios pré-seed |
| (e) Playoffs | toggle | formato → classificados/wildcards → preview da árvore → editor de fases → geração | editor de fases antes do toggle/formato |
| (e) Playoffs / geração | — | `autoGenerate` **removido** da UI (affordance falsa, spec:18) | toggle de geração automática |

### 3.1 Matriz de visibilidade por formato

| Formato (b) | (c) Agrupamento | (d) Regras | (e) Playoffs |
|---|---|---|---|
| Pontos corridos | oculto | visível | oculto |
| Fase de grupos | visível | visível | oculto |
| Grupos + Playoffs | visível | visível | visível |
| Playoffs puros | oculto | opcional (sugerido visível) | visível |

Regra de cálculo da árvore: em "Playoffs puros" ou "Grupos + Playoffs", o botão **"Sugerir árvore"** usa `PlayoffTreeBuilder.build(...)` (por balde + funil de campeões), ficando **editável**. A árvore só é renderizada depois de existir classificados (≥ 2) e após a escolha; antes disso mostra um estado vazio orientador.

---

## 4. Modelo visual da chave de playoffs (árvore com conectores)

### 4.1 Objetivo

Representar uma **eliminação simples afunilando até a final**: colunas de confrontos por fase, com **conectores em "L"** ligando cada par de confrontos ao seu sucessor. Deve contemplar byes, "A definir", destaque da fase atual e responsividade.

### 4.2 Anatomia

```
  Fase 1 (Wildcard)          Fase 2 (Semifinal)        Fase 3 (Final)
 ┌─────────────────┐        ┌─────────────────┐       ┌─────────────────┐
 │ 2º Caipira      │──┐     │                 │       │                 │
 ├─────────────────┤  ├────▶│ Venc. J1        │──┐    │                 │
 │ 5º Caipira      │──┘     ├─────────────────┤  ├───▶│  Campeão        │
 └─────────────────┘        │ Venc. J2        │──┘    └─────────────────┘
 ┌─────────────────┐  ┌────▶│                 │
 │ 3º Caipira      │──┘     └─────────────────┘
 ├─────────────────┤
 │ 4º Caipira      │──┐
 └─────────────────┘  │
 ┌─────────────────┐  │
 │ 1º Caipira (BYE)│··┘  ← conector pontilhado (avança sem jogar)
 └─────────────────┘
```

- **Caixa de confronto (match box):** 2 linhas (mandante/visitante), cada linha = seed (círculo) + nome + placar (quando houver) + ícone de vencedor. Altura fixa (~72px) para permitir o cálculo dos conectores.
- **Conectores:** partem da **borda direita central** de cada caixa, vão horizontalmente (metade do *gap*), sobem/descem até a altura média do par e seguem horizontalmente até a **borda esquerda central** da caixa da fase seguinte. Desenhados por `CustomPainter` (linhas + cantos arredondados), consumindo os tokens `AppColors.line` e `AppColors.primary`.
- **Alinhamento em funil:** a caixa da fase N+1 fica centrada verticalmente entre suas duas caixas de origem (y = média dos centros). Isso produz o afunilamento natural.
- **Fase atual / em andamento:** cabeçalho da coluna com fundo/borda `primary` e chip "EM ANDAMENTO"; caixas da fase com borda `primary`; partidas ao vivo com badge `AO VIVO` (`success`, conforme `GameStatusColors`).
- **Byes:** o time que avança sem jogar aparece **já posicionado na fase seguinte** com etiqueta `BYE` e conector **pontilhado**; na fase de origem, um marcador "sem adversário".
- **"A definir" (TBD):** caixa com borda **tracejada** (`line`), texto `A definir` em `textSecondary` itálico e ícone `help_outline`; usado quando o vencedor do jogo de origem ainda não existe.
- **Vencedor:** nome em `textPrimary` forte + `check_circle` `success`.
- **Legenda de seeds** compacta (colapsável) no topo, não empurrando a árvore.

### 4.3 Layout e responsividade

| Breakpoint | Comportamento |
|---|---|
| **≥ 1280px** | Árvore completa, todas as colunas visíveis; largura de coluna ~240px, *gap* de conector ~56px |
| **960–1279px** | Árvore com **scroll horizontal**; minimapa/indicador de colunas; colunas fixas (240px) — corrige o `SizedBox(300)` atual |
| **< 960px** | **Modo lista por fase** (accordion): cada fase é uma seção; confrontos como `KicksterCard` (sem conectores); cabeçalho mostra a fase atual. Alternativa: árvore reduzida com *zoom* e botão "ajustar à tela" |

- **Scroll horizontal** com `Scrollbar` visível e indicação persistente ("Arraste para ver as próximas fases" só quando há overflow), em vez do texto fixo atual.
- **Conectores** são decorativos: nada de interação neles; cada caixa é um alvo de toque ≥ 48px que abre o jogo.

### 4.4 Acessibilidade

- Cada confronto é **um nó de acessibilidade** com rótulo do tipo `"Semifinal, Jogo 1: Time A 21 x 14 Time B, finalizado"`.
- Contraste ≥ 4.5:1 (`textSecondary` sobre `surface`, tokens.md:307); conectores não carregam informação que não esteja no texto do card.
- Foco visível na caixa; navegação por teclado entre partidas em ordem de leitura (fase → confronto).
- `reduce motion`: nenhuma animação obrigatória; transição de destaque opcional.

### 4.5 Estados da tela

| Estado | Representação |
|---|---|
| Vazio (sem fases geradas) | `KicksterEmptyState`: "Nenhuma fase gerada" + ação "Configurar competição" |
| Configurada, não gerada | Árvore com confrontos em "A definir" (bordas tracejadas) |
| Em andamento | Fase atual destacada + jogos com placar/status |
| Degradada (validação falhou) | Faixa de aviso colapsável + jogos existentes; sem prometer a estrutura derivada |
| Erro de carga | `KicksterErrorState` (padronizar — hoje usa `AppErrorState`) |

---

## 5. Componentes novos e reuso

| Necessidade | Solução proposta | Status |
|---|---|---|
| Stepper de etapas | Novo `KicksterStepper` (ou portar `AppStepIndicator` do `flag_core` para `admin_web`) | **novo** |
| Cartão de escolha de formato/agrupamento | `SelectableCard` existente | reuso |
| Árvore com conectores | Novo `PlayoffBracketView` + `BracketConnectorPainter` | **novo** |
| Caixa "A definir" | Novo `BracketMatchCard` com variante `tbd`/`bye`/`live`/`winner` | **novo** |
| Estado vazio orientador | `KicksterEmptyState` existente | reuso |
| Modais curtos (ex.: renomear fase) | `KicksterDialog` (padrão `radius.modal` 24) | reuso |

> Nenhum token novo é estritamente necessário; conectores usam `AppColors.line` (repouso) e `AppColors.primary` (fase atual). Se desejado, propor `color.bracket.connector` mapeado para `line` — a decidir em `tokens.md`.

---

## 6. Plano de implementação faseado (frontend)

> Cada fase preserva `flutter analyze` em **0 erros / 0 avisos**. Sem testes unitários/widget no repo (pertencem a `flag_tester_e2e`).

### Fase 0 — Fundação de componentes
- `KicksterStepper` (novo) em `lib/ui/core/ui/kickster_stepper.dart`; export no barrel de widgets.
- `BracketMatchCard`, `PlayoffBracketView`, `BracketConnectorPainter` em `lib/ui/competition/widgets/bracket/`.
- **Risco:** baixo. Puramente aditivo.

### Fase 1 — Nova visualização da chave (maior valor, isolada)
- Reescrever `competition_playoff_bracket_screen.dart` para usar `PlayoffBracketView`.
- Adicionar modo responsivo (< 960px em accordion por fase).
- Padronizar estados (`KicksterEmptyState`/`KicksterErrorState`).
- **Arquivos:** `competition_playoff_bracket_screen.dart`, novos em `bracket/`; ajustes leves em `playoff_bracket.dart` (expor `advancing`/origem para conectores, se necessário).
- **Risco:** médio (cálculo de posicionamento e alinhamento). Não toca backend.

### Fase 2 — Wizard de criação (etapas a–e)
- Quebrar `competition_create_screen.dart` em widgets de etapa reutilizáveis (`steps/competition_step_general.dart`, `..._format.dart`, `..._grouping.dart`, `..._standings.dart`, `..._playoffs.dart`).
- Estado de etapa no `competition_create_view_model.dart` (índice atual, validade por etapa, visibilidade condicional).
- Salvar uma única vez ao final (preservar a orquestração atual create→rules→playoffs).
- **Risco:** médio-alto (regressão de cadastro). Mitigar mantendo os builders de seção atuais como base.

### Fase 3 — Hub de configuração (edição)
- Nova rota `/competitions/:id/settings` (+ sub-rotas) em `competition_branch.dart`.
- Reusar os mesmos widgets de etapa; `competition_edit_screen.dart` vira o wrapper do hub.
- Enxugar `competition_detail_screen.dart`: resumo somente-leitura + CTA "Configurar Competição".
- **Risco:** alto (rotas/back-stack/`extra` no go_router, deep links, permissões).

### Fase 4 — Playoffs em página própria + progressive disclosure
- Mover a config de playoffs do modal/inline para a etapa (e) em página, com árvore embutida.
- Aplicar a matriz de visibilidade (§3.1); remover `autoGenerate` da UI.
- `competition_playoffs_card.dart`: manter resumo + "Gerar" + "Ver Chave"; remover o modal pesado.
- **Arquivos:** `competition_playoff_section.dart`, `competition_playoffs_config_modal.dart` (eliminado ou reduzido), `competition_playoff_phases_editor.dart` (embutido no passo), `competition_playoffs_card.dart`.
- **Risco:** médio (contrato com ADR-021/spec).

### Fase 5 — Conformidade de design system e acessibilidade
- Substituir `fontSize`/`fontWeight` literais por `AppTextStyles`/`Theme.textTheme` nas telas de competição.
- Revisar alvos de toque (≥48px), foco visível, rótulos visíveis, mensagens de erro junto ao campo.
- Padronizar estados Loading/Empty/Error.
- **Risco:** baixo.

### Resumo de arquivos afetados

| Ação | Arquivo |
|---|---|
| Reescrever | `ui/competition/widgets/competition_playoff_bracket_screen.dart` |
| Novo | `ui/core/ui/kickster_stepper.dart` |
| Novo | `ui/competition/widgets/bracket/playoff_bracket_view.dart` |
| Novo | `ui/competition/widgets/bracket/bracket_match_card.dart` |
| Novo | `ui/competition/widgets/bracket/bracket_connector_painter.dart` |
| Novo | `ui/competition/widgets/steps/*.dart` (5 etapas) |
| Novo | `ui/competition/widgets/competition_settings_hub_screen.dart` |
| Alterar | `ui/competition/widgets/competition_create_screen.dart` |
| Alterar | `ui/competition/widgets/competition_edit_screen.dart` |
| Alterar | `ui/competition/widgets/competition_detail_screen.dart` |
| Alterar | `ui/competition/widgets/competition_playoff_section.dart` |
| Alterar | `ui/competition/widgets/competition_playoff_phases_editor.dart` |
| Alterar | `ui/competition/widgets/competition_playoffs_card.dart` |
| Reduzir/remover | `ui/competition/widgets/competition_playoffs_config_modal.dart` |
| Alterar | `ui/competition/view_models/competition_create_view_model.dart` |
| Alterar | `ui/competition/view_models/competition_playoff_view_model.dart` |
| Alterar | `routing/branches/competition_branch.dart` |
| Possível ajuste | `domain/models/playoff_bracket.dart` (expor origem p/ conectores) |

### Riscos transversais
- **Governança de docs:** a mudança de UI precisa atualizar ADR-017/ADR-021 e a spec (§6 "Admin web").
- **Contrato de backend:** `phases_json`, geração fase a fase e permissões (criador/ADMIN) devem casar com o que a UI oferece.
- **Orquestração de salvamento:** no create, hoje há 1 create + save rules + save playoffs; no wizard, garantir que falha parcial seja recuperável (mensagem clara + não perder dados).
- **Navegação:** sub-rotas novas + `extra: Competition` exigem cuidado com back-stack e deep links.
- **Performance:** `CustomPaint` com muitas fases/jogos — virtualizar/limitar e medir.

---

## 7. Tensão a resolver com o tech-lead

O `AGENTS.md` §4.1 define "Página Única Fluida" e "sem bloqueios artificiais" para formulários. Esta proposta introduz **etapas**. A interpretação adotada para compatibilizar:

- As etapas são de **navegação/orientação** (*progressive disclosure*), **não** de validação bloqueante: o usuário pode avançar/voltar livremente e **não** é obrigado a "salvar rascunho" entre passos.
- O salvamento permanece **fluxo único** (um save principal), como hoje.
- Nada de campos irrelevantes é exibido (ao contrário do monolito atual, que fere o princípio de clareza).

Recomenda-se **atualizar o `AGENTS.md` §4.1** (ou registrar um ADR de UI) explicitando: "formulários de configuração complexa podem ser divididos em **etapas de navegação não-bloqueantes** com *progressive disclosure*".

---

## 8. Critérios de sucesso (para validar depois)

- [ ] A tela de competição (create/edit) não exibe campos de agrupamento quando o formato não os usa.
- [ ] Não há configuração inline de playoffs no detalhe — apenas resumo/ações.
- [ ] Nenhum campo de playoffs aparece antes de "Habilitar playoffs".
- [ ] A chave exibe conectores ligando cada par de confrontos à fase seguinte e afunila até a final.
- [ ] A fase atual é visualmente destacada na chave.
- [ ] Byes e "A definir" são distinguíveis sem ambiguidade.
- [ ] A chave é legível em ≥ 960px (scroll) e em < 960px (modo lista).
- [ ] `flutter analyze` = 0 erros/0 avisos.
