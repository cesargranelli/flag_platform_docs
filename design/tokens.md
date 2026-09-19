# Flag Platform — Design Tokens

Fonte da verdade visual para os apps (Flutter). Reflete `frontend/packages/core/lib/src/theme/app_colors.dart` e `app_theme.dart`. O agente de UX deve basear propostas nestes tokens — qualquer mudança de token deve ser proposta aqui e refletida no `core`.

## Referência Figma (UI Kit Kickster)

- **Arquivo**: [Kickster — Live Score & News Sport (Community)](https://www.figma.com/design/bXGRAtra3DkMAPGKLLLaCQ/Kickster---Live-Score---News-Sport-Apps-UI-Kits--Community-?node-id=65-400)
- Referência completa em `docs/design/kickster-reference.md` (issue #431).
- Usar como referência visual de componentes ao avaliar/propor layouts (ex.: calendário, inputs, chips).

## Cores

Marca única adotada (2026-08-29, issue #431): paleta do UI Kit **Kickster** (primário azul royal, fundo claro), mapeada para os tokens semânticos atuais. Substitui a paleta Shifty (laranja).

| Token | Valor | Uso |
|---|---|---|
| `color.primary` | `#083879` | Marca, AppBar, botões principais, destaque (azul royal) |
| `color.secondary` | `#17153B` | Elementos secundários, azul-escuro |
| `color.accent` | `#0A4A9E` | Alertas/destaques pontuais (azul mais claro) |
| `color.success` | `#00C566` | Sucesso, status positivos, pontos |
| `color.warning` | `#FACC15` | Avisos, alertas — **conteúdo (texto/ícone) deve ser escuro** (`#171725`), nunca branco |
| `color.danger` | `#E53935` | Erro, cancelamento, fim de partida |
| `color.black` | `#111111` | Preto (bordas, checado) |
| `color.disabled` | `#9CA4AB` | Texto/fundo de elementos desabilitados |
| `color.gray.fill` | `#ECF1F6` | Preenchimentos neutros (checkbox não checado) |
| `color.background` | `#FEFEFE` | Fundo de telas |
| `color.surface` | `#FFFFFF` | Cards, inputs, superfícies elevadas |
| `color.text.primary` | `#171725` | Texto principal |
| `color.text.secondary` | `#66707A` | Legendas, metadados, placeholders (4.6:1 sobre surface — AA) |
| `color.text.muted` | `rgba(0,0,0,.6)` | Subtítulos de tela e rodapé (issue #269) |
| `color.gray.g100` | `#78828A` | Labels curtas em caixa alta (divisor "OU") |
| `color.field.border` | `#DADADA` | Borda de repouso dos campos do kit (inputs/dropdowns) |
| `color.surface.muted` | `#F6F8FE` | BG secundário — fundo azulado de cards/áreas |

### Semântica esportiva (`GameStatus`)

| Status | Significado | Cor | Token |
|---|---|---|---|
| `inProgress` | Ao vivo | verde | `color.success` (`#00C566`) |
| `finished` | Finalizado | vermelho | `color.danger` (`#E53935`) |
| `scheduled` | Agendado | cinza | `color.text.secondary` |
| `open` | Abertura | cinza | `color.text.secondary` |
| `conference` | Conferência | cinza | `color.text.secondary` |
| `cancelled` | Cancelado | cinza | `color.disabled` |

Fonte única no código: extensão **`GameStatusColors.statusColor`** (`flag_core`) — badges, placares e ícones de status devem consumir esse mapeamento, nunca redefinir cores por status.

## Tipografia

Família da marca: **Plus Jakarta Sans**, aplicada via pacote `google_fonts` (fetch em runtime com cache HTTP; offline cai na fonte padrão da plataforma, sem quebrar o app). Escala do Kickster (letter-spacing = tamanho × 0.005):

| Token | Tamanho | Peso | Uso |
|---|---|---|---|
| `type.display` | 48 / 56 | bold | H1 — destaques |
| `type.headline.md` | 40 / 48 | bold | H2 — títulos de tela |
| `type.headline.sm` | 32 / 40 | bold | H3 — títulos de seção |
| `type.title.lg` | 24 / 32 | bold | H4 — cabeçalhos |
| `type.title.md` | 20 / 28 | semibold | H5 — subtítulos / cartões |
| `type.title.sm` | 18 / 26 | semibold | H6 — rótulos |
| `type.body` | 16 / 24 | regular | Body Large |
| `type.body.md` | 14 / 22 | regular | Body Medium — corpo |
| `type.body.sm` | 12 / 20 | regular | Body Small — corpo secundário |
| `type.body.xs` | 10 / 18 | regular | Body X-Small — micro-texto |

> Implementação canônica: a escala é aplicada em `Theme.textTheme` (`displayLarge`, `headlineMedium`, `headlineSmall`, `titleLarge`, `titleMedium`, `titleSmall`, `bodyLarge`, `bodyMedium`, `bodySmall`, `labelSmall`) e os estilos nomeados ficam em `AppTextStyles`. **Não usar `fontSize`/`fontWeight` literais** nos componentes — sempre referenciar `AppTextStyles.*` ou `Theme.of(context).textTheme.*`.

### Estilos nomeados (`AppTextStyles`)

Constantes em `frontend/packages/core/lib/src/theme/app_text_styles.dart`, com letter-spacing da spec do Figma. A família é herdada do tema; a cor pode ser ajustada no ponto de uso via `copyWith`.

| Token | Tamanho/Linha | Peso | Letter-spacing | Cor padrão | Uso |
|---|---|---|---|---|---|
| `headline1` | 48 / 56 | w700 | 0.24 | `text.primary` | H1 — títulos de destaque |
| `subtitle` | 20 / 28 | w500 | 0.10 | `text.muted` | Subtítulo de tela |
| `labelMedium` | 14 / 22 | w500 | 0.07 | parametrizável | Links e rótulos de checkbox |
| `paragraph` | 14 / 22 | w400 | 0.07 | `text.primary` | Parágrafo |
| `fieldLabel` | 12 / 20 | w400 | 0.06 | `text.primary` @40% | Rótulo flutuante de input |
| `overlineLabel` | 12 / 20 | w700 | +1 (uppercase) | `gray.g100` | Overline — divisor "OU" / seções de drawer |
| `buttonText` | 14 / 22 | w700 | 0.07 | branco | Texto de botão primário |
| `footerLink` | 13 / 17 | w500 | 0.07 | `text.muted` | Link/texto de rodapé |

## Espaçamento

Escala base (grid de 4): `space.xs` **4** · `space.sm` **8** · `space.md` **12** · `space.lg` **16** · `space.xl` **24** · `space.xxl` **32**.

Uso típico: padding de tela `space.lg` (16), gap entre cards `space.md` (12), entre grupos/seções `space.xl` (24).

### Paddings internos de componente (Figma)

Valores próprios de cada componente — podem furar o grid e são tokens de componente:

| Token | Valor | Uso |
|---|---|---|
| `space.card` | 16 | padding interno de cards de conteúdo/módulo/jogo |
| `space.chip.h` / `space.chip.v` | 16 / 8 | chip selecionável (`KicksterChip`) |
| `space.badge.h` / `space.badge.v` | 10 / 5 | badge de status (`KicksterBadge`) |
| `space.status.h` | 10 | chip de status compacto (`KicksterStatusChip`, altura 28) |
| `space.field.h` / `space.field.v` | 16 / 14 | campo de formulário (altura ~52px) |
| `space.dialog` | 24 | padding de modal (`KicksterDialog`) |
| `space.dialog.gap` | 20 | gap antes das ações do modal |
| `space.sheet` | 24 | padding do bottom sheet (`KicksterBottomSheet`) |
| `space.sheet.gap` | 24 | gap header → conteúdo → ações do bottom sheet |
| `space.sheet.contentGap` | 20 | gap entre itens do conteúdo do bottom sheet |
| `space.sheet.handleTop` / `space.sheet.handleGap` | 12 / 16 | respiro do puxador (handle bar) — acima / abaixo |
| `space.stat.gap` | 20 | gap entre linhas de estatística (`KicksterStatComparison`) |
| `space.segment` | 4 | padding do container do controle segmentado |
| `space.segment.gap` | 11 | gap entre segmentos (`KicksterSegmentedTabs`) |
| `space.segment.h` / `space.segment.v` | 12 / 8 | padding interno de cada segmento |
| `space.hitTarget` | 48 | alvo de toque mínimo (ícones acionáveis) |

## Formas e elevação

| Token | Valor |
|---|---|---|
| `radius.button` | 24 (pill) |
| `radius.input` | 24 (pill) |
| `radius.surface` | 16 |
| `radius.card` | 12 |
| `radius.chip` | 10 |
| `radius.chip.status` | 4 |
| `radius.status` | 30 |
| `radius.checkbox` | 2 |
| `radius.modal` | 24 |
| `radius.sheet` | 20 (cantos superiores; base reta) |
| `radius.sheet.handle` | 2 (puxador do bottom sheet) |
| `elevation.card` | 1 · sombra `#14000000` (preto 8%) |
| `elevation.modal` | `0 8 32 rgba(18,25,51,0.06)` → Flutter: `elevation: 8`, `shadowColor: #1219330F` |
| `elevation.nav` | `0 -4 16 #0F000000` (sombra superior do dock inferior) |
| `elevation.timeline.badge` | `2 2 20 rgba(40,42,60,0.06)` → `#0F282A3C` (badge da timeline) |

> **Flutter**: cards usam `elevation: 1` com `shadowColor: Color(0x14000000)`. No modal, a sombra CSS (`x 0 / y 8 / blur 32`) não tem equivalente exato no Material — aproximar com `elevation: 8` + `shadowColor: Color(0x0F121933)`.

## Tamanhos (densidade de componentes)

| Token | Valor | Uso |
|---|---|---|
| `size.button.height` | 56 | altura de botão |
| `size.field.height` | 52 | altura de campo de formulário |
| `size.control` | 24 | checkbox / radio / thumb do toggle |
| `size.radio.dot` | 16 | círculo interno do radio |
| `size.toggle.track` | 44 × 24 | trilho do toggle (pill) |
| `size.chip.height` | 34 | chip selecionável (compacto) |
| `size.statusChip.height` | 28 | chip de status |
| `size.topbar.height` | 64 | topbar (telas autenticadas) |
| `size.appbar.height` | 64 | topbar do Public App |
| `size.nav.dock.height` | 56 | dock de navegação inferior |
| `size.nav.item` | 44 | área de toque do item do dock |
| `size.nav.item.selected` | 52 | círculo de marcação do item selecionado |
| `size.nav.icon` | 20 | ícone do item não selecionado |
| `size.nav.icon.selected` | 40 | ícone do item selecionado |
| `size.avatar.sm` | 32 | avatar em topbar/lista |
| `size.avatar.md` | 40 | avatar padrão |
| `size.icon.xs` | 14 | ícone em badge/metadados |
| `size.icon.sm` | 16 | ícones inline |
| `size.icon.md` | 18 | ícone em botão/campo/topbar |
| `size.icon.lg` | 20 | ícones de ação |
| `size.icon.xl` | 24 | ícones de destaque |
| `size.icon.xxl` | 28 | ícone em card de módulo |
| `size.hitTarget` | 48 | alvo de toque mínimo |
| `size.dialog.mobile` / `size.dialog.web` | 343 / 480 | largura de modal |
| `size.sheet.icon` | 40 | botão de ícone do header do bottom sheet |
| `size.sheet.handle` | 40 × 4 | puxador (handle bar) do bottom sheet |
| `size.timeline.badge` | 40 | badge circular de evento na timeline |
| `size.timeline.line` | 1 | espessura do eixo central da timeline |

## Movimento (transições)

Transições padrão do app (todas as plataformas), sempre respeitando **reduce motion** (duração `0`):

| Contexto | Transição | Duração | Curva |
|---|---|---|---|
| **Push** (abrir detalhe: jogo / time / play-by-play) | slide entrando pela direita + fade | `motion.duration.medium` (300) | `motion.curve.standard` |
| **Pop** (voltar) | reverso do push | `motion.duration.medium` | `motion.curve.standard` |
| **Troca de aba** (navigation menu) | slide direcional + fade | `motion.duration.medium` | `motion.curve.standard` |
| **Modais / diálogos** | fade + scale (padrão Material) | `motion.duration.short` | `motion.curve.standard` |

### Tokens

| Token | Valor |
|---|---|
| `motion.duration.short` | 200ms |
| `motion.duration.medium` | 300ms |
| `motion.duration.long` | 400ms |
| `motion.curve.standard` | `easeOutCubic` |
| `motion.curve.emphasized` | `easeInOutCubic` |

**Implementação**: `PageTransitionsTheme` global (`FlagPageTransitionsBuilder`) no `AppTheme` — aplica-se ao push/pop de todas as rotas; a **troca de aba** usa o `_TabTransition` do shell. Código dos tokens: `AppMotion` (`lib/src/core/theme/app_motion.dart`).

## Modais (padrão Kickster — Popup 343px, issue ADR-009)

Referência: [Figma — Popup "Share this Match" 34430:8519](https://www.figma.com/design/bXGRAtra3DkMAPGKLLLaCQ/Kickster---Live-Score---News-Sport-Apps-UI-Kits--Community-?node-id=34430-8519&t=4VvSQu6LIHJAB1cW-4): container **343px** (mobile) / **480px** web, padding **24**, gap **20**, fundo `surface`, raio `24`, sombra `elevation.modal`, header `Body Large Bold` + close circular 24px `surface.muted`, divider `Grayscale 20 (#ECF1F6)` 1px, conteúdo gap 16. Usar `Dialog` com `shape: RoundedRectangleBorder(borderRadius: 24)` e `insetPadding: 24`.

## Componentes (padrões do Kickster)

- **Inputs** (`InputDecorationTheme`): preenchidos (`surface`), `OutlineInputBorder` raio **24 (pill)**, conteúdo vertical ~52px, **rótulo sempre visível** (12/16 ls−0.2 @40% — `fieldLabel`); estados **Normal / Focado / Disabled / Error** (borda de repouso `field.border` `#DADADA` / `primary` 2px / `disabled` / `danger`)
- **Botões** (`FilledButton`/`ElevatedButton`/`OutlinedButton`): altura mínima **56px**, raio **24 (pill)**; variantes **Main** (fundo `primary`), **Disable** (fundo `disabled`, texto `textPrimary`), **Ghost** (borda `primary`)
- **Chip**: raio 10; selecionado com fundo `primary`; não selecionado com borda `black`
- **SelectableCard** (`widgets/selectable_card.dart`, padrão final #300): card de seleção única (comportamento de rádio por grupo) — interação IDÊNTICA aos cards de lista: `Card` (raio 12, elevação 1, clipBehavior) + `InkWell` com tinta PADRÃO do tema; proibido hover/splash/foco customizados (causam cintilação no web). Padding interno 16, **altura mín. 120px (único — substitui os 96px padrão e os 72px compactos)**, grid gap 12. Layout vertical: **ícone (28px) acima do rótulo acima da descrição**, com `maxLines: 1` (ellipsis) na descrição, e conteúdo **centralizado verticalmente** (`mainAxisAlignment.center`) para o card "só rótulo" preencher os 120px sem espaço vazio no rodapé. Cards sem descrição (ex.: Gênero) mantêm a **simetria com um ícone representativo** (`Icons.male`/`Icons.female`/`Icons.transgender`). Seleção por `Container` interno: não selecionado = card `surface` padrão · **selecionado** = fundo `primary` SÓLIDO + label/descrição/ícone BRANCOS + badge invertido no canto superior direito (círculo branco 24px, ícone `primary`) (#294) · desabilitado 55% de opacidade. Tipografia: label `titleSmall` (14/24 w700), descrição 13/17 w500
- **SelectableChip** (#290/#292/#300): variação compacta (raio 10) para grupos com muitas opções (ex.: faixa etária), gap 8 em wrap; altura ~34px (padding 16×8), peso fixo w500, tipografia 13/17 (`footerLink`). **Padrão SEM bordas e SEM overrides de splash** — `InkWell` padrão do tema sobre `AnimatedContainer` (120ms): não selecionado = fundo `gray.fill` + texto `textPrimary` · **selecionado** = fundo `primary` + texto **BRANCO**
- **Regra de conteúdo sobre primário (#294/#431)**: conteúdo (**texto e ícone**) sobre preenchimento `primary` (#083879) usa **BRANCO** — contraste branco/azul royal ≈ 7,7:1 (WCAG AA ok). Sobre `success` (#00C566) também usa branco, aceito conscientemente para estados de seleção (≈ 2,3:1, abaixo de AA). **`warning` (#FACC15) nunca recebe conteúdo branco** — usar texto/ícone escuro (`text.primary` #171725).
- **Navegação por sessões (#323/#332) — indicador de passos `AppStepIndicator` (`flag_core`)**: círculos de 28px (`radius 14`), um por etapa, distribuídos em `Row`/`Expanded` pela largura, com rótulo abaixo (14px, negrito se ativo) e toque via `InkWell` padrão (#300). Dois modos:
  - **Modo números (cadastro/wizard — `icons` nulo)**: **concluída** = círculo `success` + check **branco**; **atual** = círculo `primary` + ponto **branco**; **pendente** = círculo `grayFill` + número ordinal `textPrimary`
  - **Modo ícones (telas de detalhe — `icons` fornecido, `showDoneState: false`)**: cada círculo exibe o **ícone da sessão**; **apenas a etapa ativa** fica selecionada — o item inteiro recebe fundo `primary` (todo azul royal) com conteúdo (**ícone + rótulo**) **BRANCO**; as demais permanecem **"não selecionadas"** (círculo `grayFill` + ícone `textPrimary`, sem o verde de "concluído")
  Conteúdo **BRANCO** sobre `primary`/`success` (#294). Na tela de detalhe, tocar em uma etapa troca a sessão ativa exibida — a tela mostra **apenas** o conteúdo da sessão ativa. A variante em cards (`AppSessionNav`) permanece disponível no `flag_core` como alternativa.
- **Interação de cards/chips selecionáveis (#300)**: usar sempre `Card`/`InkWell`/tinta padrão do tema, como nos cards de listagem — NÃO implementar hover via `setState`/`MouseRegion` custom nem desabilitar splash com overlays próprios (flash branco e cintilação no web)
- **Checkbox**: 24px, raio 2; checado `primary`, não checado `gray.fill`
- **Card**: `surface`, raio 12, elevação 1
- **Card de conteúdo (`AppInfoCard`, `flag_core` — `widgets/app_info_card.dart`, #328)**: padrão dos cards de conteúdo das telas de detalhe. `Card` com `margin: EdgeInsets.zero`; `minHeight` padrão **144**; título `titleSmall` (14/24 w700) com **gap 12** para o conteúdo; linhas `AppInfoRow` (rótulo fixo **120px/13px `textSecondary`** + valor **14px**, padding bottom 8) e `AppInfoColorRow` (swatch 18×18 raio 4 + hex em maiúsculas). Largura via `AppLayout.detail` (720). Opcional `icon` (20px `primary`) ao lado do título.
- **AppBar**: telas autenticadas (admin) usam a topbar `primary` com texto branco e título centralizado (`KicksterTopBar`). O **Public App** usa a topbar clara (token `topbar.surface`): fundo `surface`, borda inferior `line` 1px, marca/título `textPrimary` à esquerda, ações em botões circulares `grayFill` (alvo 48px).
- **Estados**: `AppLoading` (carregando), `AppEmptyState` (vazio com ícone), `AppErrorState` (erro com "Tentar novamente")
- **Alvos de toque**: mín. 48px (ícones acionáveis); botões 56px

## Componentes Kickster (`flag_core` — issue #436)

Biblioteca de widgets no `frontend/packages/core/lib/src/widgets/` (prefixo `Kickster*`, exportados no barrel `flag_core.dart`). Todos usam apenas tokens (`AppColors.*`/`AppTheme`) — sem hex hardcoded.

| Widget | Arquivo | Uso |
|---|---|---|
| `KicksterCard` | `kickster_card.dart` | Card de módulo (home): raio **12**, fundo `surface`, elevação 1, `InkWell` padrão; ícone 28px `primary` em círculo 56px `primary`@10% + título (FittedBox) |
| `KicksterScoreCard` | `kickster_score_card.dart` | Card de jogo com placar (Live Match): confronto Time A × Time B, placar central em `headlineSmall`, badge de status (`GameStatus.label`) |
| `KicksterBadge` | `kickster_badge.dart` | Badge de status: fundo `color`@12%, texto/ícone na cor do badge, raio 10, `Semantics`; **`warning` → conteúdo escuro `textPrimary` (#294)** |
| `KicksterChip` | `kickster_chip.dart` | Chip selecionável (raio 10, ~34px compacto): não selecionado `grayFill`/`textPrimary`; selecionado `primary`/**branco**; `InkWell` padrão (#300) |
| `KicksterPillTab` | `kickster_pill_tab.dart` | Aba em **pílula** (grupo de seleção — ex.: modalidades, filtros de categoria): raio **24**, padding **12×8**, tipografia `labelMedium` (14/22 w500); **selecionada** = fundo `primary` + texto branco; **não selecionada** = fundo `surface.muted` (`#F6F8FE`) + borda `line` 1px + texto `grayLabel` (`#78828A`) |
| `KicksterButton` | `kickster_button.dart` | Wrapper tipado dos botões do tema (variantes `primary`/`outline`/`text` — `FilledButton`/`OutlinedButton`/`TextButton`), com `icon?` e `loading?` |
| `KicksterInput` | `kickster_input.dart` | Wrapper de `TextFormField` sobre o `InputDecorationTheme` (raio 24 (pill), rótulo visível) — não sobrescreve bordas |
| `KicksterSectionTitle` | `kickster_section_title.dart` | Título de seção ("Ao vivo"/"Próximos"): `titleMedium` `textPrimary`, ícone `primary` opcional, `action?` à direita |
| `KicksterNavBar` | `kickster_nav_bar.dart` | Barra de navegação inferior mobile: `NavigationBar` com fundo `surface` e indicador `primary`@12% |
| `KicksterFilterSheet` | `kickster_filter_sheet.dart` | Filtro **multi-seleção** em bottom sheet (base `KicksterBottomSheet`): grupos (`KicksterFilterSection`) com opções em `KicksterPillTab`; ações **Limpar** (outline, desabilitado sem seleção) e **Aplicar** (primary). Chaves `grupo:valor`; semântica **OR dentro do grupo / AND entre grupos** (`KicksterFilterSelection.matches`). `show()` retorna as chaves aplicadas ou `null` (fechado sem aplicar). Trigger: `KicksterFilterButton` (pílula raio 24, `tune` + contador, `primary` quando ativo) |
| `KicksterSegmentedTabs` | `kickster_segmented_tabs.dart` | Controle **segmentado** (Figma "Menu" `34433:3342`): container `surface.muted` (#F6F8FE) raio **24** com padding **4**; segmentos com padding **12×8** (gap **11**) e raio 24 — **selecionado** = fundo `surface` + texto `black`; demais = texto `disabled` (#9CA4AB). Usado nas abas **Estatísticas \| Lances** da tela de Jogo |
| `KicksterGameHeader` | `kickster_game_header.dart` | Cabeçalho **compacto** de um jogo: time (avatar 40 + nome) — centro (placar 22 w700 + status ou pill `AO VIVO` `danger` + linha auxiliar) — time. Compartilhado entre abas; times tocáveis (`onHomeTeamTap`/`onAwayTeamTap`) |
| `KicksterStatComparison` | `kickster_stat_comparison.dart` | Linha de estatística (Figma "Statistic"): **valor da casa** (`primary`) · **rótulo** centralizado (`textSecondary`) · **valor do visitante** (`textPrimary`) + duas **lanes proporcionais** (6px, raio 12) — `primary` (casa) e **`warning` apenas na lane** (decorativo; amarelo como texto reprova AA). `homeLabel`/`awayLabel` aceitam valores formatados (ex.: `54%`) |
| `KicksterTimeline` | `kickster_timeline.dart` | Linha do tempo com **eixo central** (1px `line`): badges circulares 40px `surface` com borda `line` e sombra `elevation.timeline.badge`; eventos **casa à esquerda / visitante à direita** com tempo (`success` 12 w500) + título (14) + tipo (12) + descrição (10). Cada evento vira **um nó de acessibilidade** quando `semanticLabel` é informado (eixo/lado são decorativos) |
| `KicksterBottomSheet` | `kickster_bottom_sheet.dart` | Bottom sheet padrão Kickster (Figma "Actions" `34442:3789`): fundo `surface`, **raio topo 20** (`radius.sheet`), padding **24**, gap **24** entre header/conteúdo/ações e **20** entre itens do conteúdo; header com título centralizado (`labelMedium` 14/22 w600) e botões de ícone **40×40** (raio 16, `KicksterSheetIconButton`) nos extremos — fechar por padrão. **Puxador (handle bar) 40×4** `line` raio 2 no topo (mesmo padrão do Referee App / `PlayDialog`), com respiro 12 acima e 16 abaixo. `KicksterBottomSheet.show()` abre como modal com **arrastar-para-baixo** e **toque fora** para fechar (retorna `null` = fechar sem aplicar) |

- **Mapeamento de status do `KicksterScoreCard`**: `inProgress` → `success` · `finished` → `danger` · `scheduled` → `textSecondary` · `cancelled` → `disabled`
- **Card de campeonato (`CompetitionCard`, `flag_public_app`)**: card branco (`surface`), raio **12** (`radius.card`), `elevation.card` + borda `line` 1px; emblema `emoji_events` 24px `primary` em círculo 40px `primary`@10% (raio 10); nome 16/22 w600 `textPrimary` (2 linhas, ellipsis); organização `textSecondary`; `KicksterBadge` de status à direita.
  - **Atributos** (formato "ícone + rótulo + valor"): `Wrap` (spacing 24 / run 12) com, por item, ícone 20px `primary` + coluna `rótulo` (**12/20** `textSecondary`) sobre `valor` (**14/22 w600** `textPrimary`). Ordem: **Modalidade** (`sports_football`) · **Gênero** (`people_alt_outlined`) · **Categoria** (`cake_outlined`) · **Temporada** (`event_outlined`).
- **Public App − competições**: só são **listadas** as competições **em andamento** (`status = PUBLISHED`) — via `ongoingCompetitionsProvider`. Rascunhos, encerradas e desativadas não aparecem nas listagens (a lista de campeonatos e a home).
- **Public App − tela de Jogo (`GameDetailScreen`)**: hub do confronto com **duas abas** — **Estatísticas** (`KicksterStatComparison`, métricas derivadas dos lances via `gameStatisticsProvider`/`buildGameStatistics`) e **Lances** (`KicksterTimeline`); cabeçalho `KicksterGameHeader` compartilhado e **troca de aba in-place** (não empilha rotas).
  - Deep links: `/game/:id` → Estatísticas · `/game/:id/stats` · `/game/:id/plays` → Lances · `/live/:id/plays` (legado) → Lances.
  - Os CTAs "Lance a Lance" abrem `/game/:id` com `GameDetailArgs.initialTab = 1` (1 toque, sem duplicar tela); ao vivo o placar/lances são atualizados a cada 10s.
- **Rótulos de domínio**: `genderLabelFromValue` (`MALE`→Masculino) e `ageGroupLabelFromValue` (`SUB17`→Sub-17, `ADULT`→Adulto, `OPEN`→Livre) em `enum_labels.dart` — usados por cards e filtros.
- **`MatchStatusBadge` / `MatchScoreCard`**: seguem o MESMO mapeamento acima (ao vivo = `success` verde; fim de partida = `danger` vermelho), garantindo consistência entre os cards de jogo.
- **`KicksterStatusChip`** (`kickster_status_chip.dart`): chip de status compacto, raio **4** (`radius.chip.status`), altura 28px, fundo `chip*Bg` + texto `chip*Fg` (tom escuro da cor — contraste AA).
- **`KicksterDialog`** (`kickster_dialog.dart`): modal de confirmação com raio **24** (`radius.modal`), largura 343px (mobile) / 480px (web), padding 24.
- **`KicksterBadge`**: o conteúdo usa uma variação **escura** da cor (`foregroundFor`) para garantir contraste AA sobre o fundo @12%.

## Estados (vazio / erro / carregando)

Padrão único para os estados de tela (`flag_core`):

| Estado | Widget | Especificação |
|---|---|---|
| Carregando | `AppLoading` | `CircularProgressIndicator` `primary` centralizado; mensagem opcional 14px `textSecondary`; gap 12 |
| Vazio | `KicksterEmptyState` | quadro 88px `primary`@8% raio 28 + ícone 40px `primary`@40%; título 16px w600 `textPrimary`; descrição 14px `textSecondary`; ação opcional; gap 20 |
| Erro | `KicksterErrorState` | ícone `error_outline` 56px `danger`; mensagem 16px w600 `textPrimary`; botão `KicksterButton` outline "Tentar novamente" (ícone `refresh`); gaps 16 / 20 |

Tokens: `space.state.padding` 32 · `space.state.gap` 20 · `size.emptyState.frame` 88 · `size.emptyState.icon` 40 · `size.errorState.icon` 56 · `radius.emptyState.frame` 28.

> **Regra**: `AppEmptyState` e `AppErrorState` (legados) devem **delegar** aos widgets `Kickster*` — nunca reimplementar o layout do estado.

## Layout responsivo

| Breakpoint | Comportamento |
|---|---|
| `< 960px` | Layout estreito (mobile): menus em lista, cards empilhados |
| `>= 960px` | Layout largo (desktop): `NavigationRail`, painéis em colunas |

### Larguras máximas (padrão web)

Em telas largas, o conteúdo é centralizado com largura máxima para preservar legibilidade (45–75 caracteres por linha) e hierarquia. Refletido em `frontend/packages/core/lib/src/layout/app_layout.dart` (widget `AppLayout`).

| Token | Valor | Uso |
|---|---|---|
| `layout.maxForm` | 600 | Formulários (wizard e CRUD) |
| `layout.maxDetail` | 720 | Telas de detalhe/leitura |
| `layout.maxContent` | 1200 | Listagens e conteúdo |

Wrappers: `AppLayout.form(child)`, `AppLayout.detail(child)`, `AppLayout.content(child)`.

## Acessibilidade (mínimo)

- Contraste de texto ≥ 4.5:1 (texto secundário sobre `surface` validado)
- Foco visível em todos os interativos
- Rótulos sempre visíveis em formulários
- Estados de erro no campo e mensagem clara
