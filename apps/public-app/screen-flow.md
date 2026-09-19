# Fluxo de Telas — Flag Public App

Diagrama editável: [`flag-public-app-flow.drawio`](./assets/flag-public-app-flow.drawio) — abra no [draw.io](https://app.diagrams.net/) (File → Open) ou no draw.io desktop. Para uma versão interativa renderizada no navegador, veja [Diagrama Mermaid](./diagrama.md).

> Estado atual: pós-migração ADR-006 (Team/Roster/Season). Navegação gerida pelo **GoRouter** com `StatefulShellRoute` (3 abas) + rotas empilhadas de detalhe.

---

## 1. Arquitetura de navegação

```
GoRouter (initialLocation: /live)
├── StatefulShellRoute.indexedStack  → PublicShell (barra inferior / rail)
│   ├── Branch 0:  /live              → LiveScreen
│   ├── Branch 1:  /competition        → CompetitionDetailScreen (lista ou hub)
│   │              /competition/:id    → CompetitionDetailScreen (hub, 3 abas)
│   │                ├── /competition/:id/games      (aba 0 — Jogos)
│   │                ├── /competition/:id/results    (aba 1 — Resultados)
│   │                └── /competition/:id/standings  (aba 2 — Classificação)
│   └── Branch 2:  /about             → AboutScreen
│
├── /game/:id        → GameDetailScreen      (stack)
├── /teams/:id       → TeamDetailScreen      (stack)
├── /live/:id/plays  → PlayByPlayScreen      (stack)
└── errorBuilder     → Página não encontrada (rota inválida)
```

- **Shell**: cada aba preserva seu próprio estado/back stack (`IndexedStack`).
- **Stack routes**: ficam **fora** da shell — sem barra inferior; usam o back do sistema para voltar à origem.
- **Responsividade**: `PublicShell` usa `NavigationRail` em telas `>=960px` e barra inferior flutuante em telas menores.

---

## 2. Navegação principal (Shell)

| Ação | Origem | Destino | Comportamento |
|------|--------|---------|---------------|
| Tocar aba **Ao vivo** | qualquer aba | `/live` | `goBranch(0)` |
| Tocar aba **Campeonato** | qualquer aba | `/competition/:id` **ou** `/competition` | se há **foco** → hub do campeonato em foco; senão → lista |
| Tocar aba **Sobre** | qualquer aba | `/about` | `goBranch(2)` |

> Regra da aba Campeonato (`public_shell.dart`): `focus = ref.read(focusedCompetitionProvider)`; `path = focus != null ? '/competition/${focus.id}' : '/competition'`.

---

## 3. Cenários da área Campeonato

### Cenário 3.1 — Sem campeonato em foco (lista)
```
/competition
  └─ CompetitionDetailScreen._buildEmpty()
       ├─ carregando  → AppLoading
       ├─ erro        → AppErrorState (retry)
       ├─ vazio       → AppEmptyState ("Nenhum campeonato disponível")
       └─ com dados   → ListView de CompetitionCard
            └─ tocar no card → 1. set foco 2. go '/competition/:id'
```

### Cenário 3.2 — Hub do campeonato (com foco)
```
/competition/:id  (CompetitionDetailScreen, AppBar com botão "Trocar")
  └─ TabBar (3 abas) via TabController (initialTab conforme deep link)
       0. Jogos        → CompetitionGamesScreen
       1. Resultados   → CompetitionResultsScreen
       2. Classificação→ CompetitionStandingsScreen
  └─ "Trocar" → limpa foco + go '/competition' (volta à lista)
```

### Cenário 3.3 — Deep links do hub
| Rota | Aba aberta |
|------|-----------|
| `/competition/:id/games` | 0 — Jogos |
| `/competition/:id/results` | 1 — Resultados |
| `/competition/:id/standings` | 2 — Classificação |

Se `competitionId` está na rota, a tela define o **campeonato em foco** automaticamente (via `addPostFrameCallback`).

### Cenário 3.4 — Tab Jogos (CompetitionGamesScreen)
- Filtro por rodada via chips (`_selectedRound`; `null` = todas).
- "Próximos jogos": até 3 jogos `SCHEDULED` futuros, destacados.
- "Todos os jogos": lista completa da rodada filtrada.
- Navegações por card:
  - tocar no **jogo** → `push /game/:id` (qualquer status)
  - tocar no **time** (casa/fora) → `push /teams/:id` (com `competitionId`)

### Cenário 3.5 — Tab Resultados (CompetitionResultsScreen)
- Só jogos com `GameStatus.finished`, ordenados por data desc.
- Tocar no jogo → `push /game/:id`; tocar no time → `push /teams/:id`.

### Cenário 3.6 — Tab Classificação (CompetitionStandingsScreen)
- Tabela (Pos, PJ, V, D, SG, PTS) com pull-to-refresh.
- Linha do líder destacada.
- Tocar na linha → `push /teams/:id` (com `competitionId`).

---

## 4. Cenários da tela Ao vivo (LiveScreen)

```
/live
  ├─ "AO VIVO AGORA" → jogos com GameStatus.inProgress
  ├─ vazio → "Nenhum jogo ao vivo no momento" (+ sugestão de ver resultados)
  └─ "Recentemente" → jogos finalizados
```

- Cada card (ao vivo e recentes) tem **Play-by-Play**:
  - tocar no botão → `push /live/:id/plays` (extra: Game).
- Não há navegação direta para o detalhe do jogo a partir daqui (só PBP).

---

## 5. Cenários das telas empilhadas (stack)

### 5.1 Detalhe do jogo — `/game/:id` (GameDetailScreen)
- Entradas: (a) `extra: GameDetailArgs` (jogo + competitionName) **ou** (b) busca por id (deep link).
- Conteúdo: `MatchScoreCard`, timeline de pontos (`ScoreTimeline`), local com link de mapa (`url_launcher`).
- **Auto-refresh**: se `GameStatus.inProgress`, atualiza o placar a cada 10s.
- Tocar no time (casa/fora) → `push /teams/:id`.
- O jogo exibido é `_mergeDisplay(base, names)` (mescla dados da API com os extras da navegação).

### 5.2 Detalhe do time — `/teams/:id` (TeamDetailScreen) *(pós ADR-006)*
- Entradas: `extra: TeamDetailArgs {teamId, teamName, competitionId?, competitionName?}`.
- Header: logo (fallback ícone), nome, shortName.
- **Com `competitionId`** → mostra o elenco daquela competição (`teamRosterProvider(teamId, competitionId)`).
- **Sem `competitionId`** → lista os elencos do time por competição (`teamRostersProvider`) em cards expansíveis.
- Elenco agrupado por posição (ordem do domínio) e ordenado por número/nome.

### 5.3 Play-by-Play — `/live/:id/plays` (PlayByPlayScreen)
- Entrada: `extra: Game?`.
- Header do jogo (placar), lances agrupados por quarto, indicador "AO VIVO".

### 5.4 Página não encontrada (errorBuilder)
- Qualquer rota inválida → tela com ícone + "Voltar ao início" → `go /live`.

---

## 6. Estados de carregamento/erro/vazio (padrão)

| Estado | Widget |
|--------|--------|
| Carregando | `AppLoading` |
| Erro (com retry) | `AppErrorState` (invalida o provider) |
| Vazio | `AppEmptyState` |
| Pull-to-refresh | `RefreshIndicator` (jogos, classificação, ao vivo) |

---

## 7. Chamadas de dados relevantes (pós ADR-006)

| Provider | Endpoint |
|----------|----------|
| `competitionsProvider` | `GET /api/v1/competitions` |
| `competitionGamesProvider` | `GET /api/v1/competitions/{id}/games` |
| `competitionStandingsProvider` | `GET /api/v1/competitions/{id}/standings` |
| `competitionTeamsProvider` | `GET /api/v1/competitions/{id}/teams` |
| `teamDetailProvider` | `GET /api/v1/teams/{id}` |
| `teamRosterProvider(teamId, competitionId)` | `GET /api/v1/teams/{id}/roster?competitionId=...` |
| `teamRostersProvider(teamId)` | `GET /api/v1/teams/{id}/rosters` |
| `liveGamesProvider` | `GET /api/v1/games/live` |
| `playByPlayProvider` | `GET /api/v1/games/{id}/plays` |

> Observação: `fakeCompetitionResultsProvider` e `fakeCompetitionStandingsProvider` (dados mock) ainda existem em `providers.dart`, mas as telas de resultados/classificação já consomem os providers reais.