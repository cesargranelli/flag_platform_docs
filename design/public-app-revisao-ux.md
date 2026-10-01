# Public App — Revisão de UX/UI (telas e fluxos)

> **Data:** 2026-09-30
> **Escopo:** app `flag_public_app` (Flutter) — todas as telas e fluxos, no contexto do esporte flag football.
> **Objetivo:** diagnosticar a percepção de *"aspecto visual pobre"* e *"navegação confusa"* e orientar a correção, comparando com o kit Figma **Kickster** e com apps de mercado.
> **Método:** auditoria heurística (Nielsen), acessibilidade (WCAG), aderência ao design system (`design/tokens.md`), análise da navegação a partir do código (GoRouter) e benchmark de mercado.
> **Evidência:** afirmações ancoradas em `arquivo:linha` do `flag_public_app`.

---

## 1. Contexto

- App público de acompanhamento de campeonatos de flag football (sem login), com 4 abas: **Início · Jogos · Classificação · Sobre** (`lib/routing/app_router.dart:58-62`).
- Design system **Kickster** (kit Figma `bXGRAtra3DkMAPGKLLLaCQ`) com tokens em `design/tokens.md` e biblioteca `Kickster*` em `lib/ui/core/ui/`.
- Já houve um esforço de remodelação (set/2026, multiagente) focado em shell/navegação (M2), mas o código evoluiu pós-**ADR-006** e a percepção visual permaneceu insatisfatória.
- `docs/screen-flow.md` e `docs/diagrama.md` estão **estruturalmente obsoletos** (descrevem 3 abas/`LiveScreen`/botão "Trocar"; o código tem 4 abas/`ScheduleScreen`/filtros).

---

## 2. Diagnóstico — por que parece "visual pobre"

Temas transversais (causas-raiz):

| # | Causa-raiz | Evidência |
|---|---|---|
| 1 | **SEM identidade visual de time** — confrontos e classificação só com iniciais; escudos eram enviados pelo backend mas não parseados pelo app | `featured_match_card.dart`, `competition_standings_screen.dart:249-281`, `kickster_game_header.dart`, `game.dart`, `competition_team.dart` |
| 2 | **Sem largura máxima no desktop** (`AppLayout` só no Sobre) → cards esticados, linhas longas | Home/Jogos/Campeonato/Jogo/Time |
| 3 | **Widgets do core reimplementados** → divergência | `game_stats_view.dart` (`_InfoCard` vs `AppInfoCard`), tabela manual vs `KicksterTable` |
| 4 | **Headers fora do padrão** (`w800` inexistente na escala; resto usa `KicksterSectionTitle`) | `home_competitions_view.dart`, `home_matches_view.dart` |
| 5 | **Estados pobres na Home** (spinner cru, erro em texto, sem retry, vazio sem orientação, sem pull-to-refresh) | `home_competitions_view.dart`, `home_matches_view.dart` |
| 6 | **Mesma entidade, dois visuais** + raios 12/16 misturados | card da Home vs `competition_card.dart`; `fixture_row.dart` |
| 7 | **Navegação só com ícones** (rótulo apenas em tooltip) | `public_shell.dart:336-387` |
| 8 | **Linguagem ambígua** ("Classificação" rotula campeonato; "Jogos em Destaque" só mostra ao vivo) | `app_strings.dart:25-26,48` |
| 9 | **Dado fictício em destaque** — seletor de estatística não filtrava (trocava só o título) | `competition_stats_screen.dart`, `competition_stats_view_model.dart` |
| 10 | **Micro-texto 10–12px** e pesos fora da escala achatando a hierarquia | `featured_match_card.dart` (data), `home_competitions_view.dart` (temporada) |

---

## 3. Diagnóstico — por que a navegação é confusa

| Sev. | Local | Problema |
|---|---|---|
| Bloqueador | `competition_routes.dart` + `public_shell.dart:113-116` | `/competition/:id/games` e `/results` vivem na branch "Classificação" mas renderizam o conteúdo de "Jogos" — a barra destaca a aba errada ("a navegação mente"). |
| Alta | `public_shell.dart:83-91` | Aba Campeonato faz `goBranch(2)` **e** `context.go(path)` (navegação dupla, API misturada). |
| Alta | `competition_providers.dart:27-54` | `focusedCompetitionProvider` filtra **duas abas** e é escrito por Home/filtros/deep links → conteúdo muda "sozinho". |
| Alta | `competition_routes.dart:37-43` | `/competition/:id/results` é idêntico a `/games` (sem filtro de finalizados). |
| Média | `app_strings.dart:25-26,48` | Rótulos não correspondem às rotas/conteúdo; "Ao vivo" não existe como lugar. |
| Média | `app_router.dart:63-65` | Detalhes de jogo/time ficam **fora da shell** (sem dock); "sensação de sair do app". |
| Média | Home | Troca de aba com `context.go` em vez de `goBranch`; back inconsistente. |
| Média | rotas de jogo | 3 URLs para `GameDetailScreen`; duplicação no stack. |
| Média | `about_screen.dart:90` | Cards "Como funciona" clicáveis sem ação (affordance falsa). |

### Divergências documento × código (amostra)

| Documento diz | Código faz |
|---|---|
| `initialLocation: /live` | `/home` |
| Shell com 3 abas | 4 abas (Início/Jogos/Classificação/Sobre) |
| `LiveScreen` | `ScheduleScreen` (aba "Jogos") |
| Hub com TabBar de 3 abas + botão "Trocar" | 2 segmentos + ícone de filtro |
| 404 volta para `/live` | volta para `/home` |

---

## 4. Comparativo — mercado e Figma

**Concorrente direto (app público) — Flag50 fan app:** tabs planas **Live / Schedule / Standings**; classificação com **escudo, Rank, W-L, PF/PA, STRK**; placar ao vivo com *clock*, *down & distance*, *timeouts* + play-by-play; perfil de jogador; **"sem conta para ver placar"**. Gaps: nossa classificação não usa escudos e não há lugar claro de "ao vivo".

**Kit Figma Kickster (norte oficial):** Home com **carrossel de destaque**; Live Match com placar central + timeline; Standings em lista; Club Profile com escudo; dock de navegação **com rótulos**. Adotamos os tokens, mas não a riqueza visual (fotos/carrossel/escudos) nem os rótulos.

**Boas práticas de apps esportivos (2026):** navegação rasa focada em placar/lineups/momentos; ações ao alcance do polegar; **branding/escudos** integrados; dados ao vivo sem refresh manual; design *mobile-first*.

---

## 5. Achados consolidados (por severidade)

Os achados completos, com `arquivo:linha`, impacto e recomendação, foram levantados na auditoria e estão refletidos nas correções da Fase 1. Principais:

- **Crítico:** seletor de estatísticas enganoso (`competition_stats_screen.dart`).
- **Alto:** Home com estados pobres; ausência de largura máxima; classificação sem PTS/escudo; sem identidade visual de time.
- **Médio:** headers fora do padrão; cards duplicados; raios inconsistentes; navegação só com ícones; linguagem ambígua; micro-texto.
- **Baixo:** AppBar divergente do token; sombra usando token de espaçamento; valores/strings hardcoded; cor isolada em barras de estatística.

---

## 6. Fase 1 — correções entregues

Branch `feature/ux-fase1-riqueza-visual` (app). Verificação: `flutter analyze` → **No issues found!**

- **Escudos/logos:** parse de `homeTeamLogoUrl`/`awayTeamLogoUrl` (jogo) e `teamLogoUrl` (times da competição); render em cards de destaque, agenda, header do jogo e detalhe do lance; fallback de iniciais. *(O backend já enviava esses campos.)*
- **Estatísticas:** removido o seletor que não filtrava; seção única honesta + aviso de dados de demonstração.
- **Home:** estados padronizados (`AppLoading`/`AppErrorState`/`AppEmptyState`), **pull-to-refresh**, cabeçalhos `KicksterSectionTitle`, "Ao vivo agora" + nova seção **"Próximos jogos"**.
- **Classificação:** mantidas as colunas `Pos|Equipe|V|E|D|SG|PF|PC` (**sem PTS** — este campeonato não usa pontuação); escudo via *join* `teamId → teamLogoUrl` com `competitionTeamsProvider`; consistência e `Semantics` completos.
- **Responsividade:** `AppLayout.content`/`detail` nas telas largas.

Detalhe do escopo e critérios de aceitação: `flag_public_app/docs/ux-fase1-plano.md`.

---

## 7. Roadmap (fases seguintes)

**Fase 2 — Navegação/IA**
- Alinhar a aba destacada ao conteúdo (mover `games/results` para a branch de Jogos ou derivar o destaque).
- Separar o filtro de "Jogos" do "campeonato em foco"; eliminar navegação dupla; consolidar rotas legadas.
- Back contextual nos detalhes (não cair sempre na Home em cold deep link).
- Rótulos visíveis na navegação inferior; revisar rótulos ambíguos.

**Fase 3 — Polimento e consistência**
- Padronizar adotando `AppInfoCard`/`KicksterTable`, unificar cards e raios.
- Limpeza de strings/tamanhos hardcoded; carrossel de destaque na Home (padrão Kickster).

**Backend (opcional):** incluir `teamLogoUrl` em `StandingEntryResponse` para dispensar o *join* no app.

---

## 8. Referências

- `design/tokens.md`, `design/kickster-reference.md` (Figma `bXGRAtra3DkMAPGKLLLaCQ`).
- `flag_public_app/docs/screen-flow.md` (obsoleto — atualizar).
- Benchmark: Flag50 (fans), FlagRoster, apps de live score (padrões de mercado).
