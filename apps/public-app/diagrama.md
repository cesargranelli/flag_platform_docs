# Diagrama — Fluxo de Telas

Versão interativa do fluxo de telas do Flag Public App (renderizada com Mermaid).

> Diagrama editável (Draw.io): [flag-public-app-flow.drawio](./assets/flag-public-app-flow.drawio)

```mermaid
graph TD
    %% ========== NÚCLEO: Shell com 3 abas ==========
    subgraph SHELL["PublicShell — StatefulShellRoute (3 abas)"]
        LIVE["Ao vivo<br/>/live<br/>LiveScreen"]
        CAMP["Campeonato<br/>/competition<br/>CompetitionDetailScreen"]
        ABOUT["Sobre<br/>/about<br/>AboutScreen"]
    end

    %% ========== ÁREA CAMPEONATO ==========
    CAMP -- "sem foco" --> LIST["Lista de campeonatos<br/>(CompetitionCard)"]
    LIST -- "tocar no card → set foco" --> HUB["HUB do campeonato<br/>/competition/:id<br/>TabBar 3 abas · botão Trocar"]
    HUB -- "Trocar → limpa foco" --> LIST

    HUB -- "aba Jogos" --> GAMES["Tab Jogos<br/>CompetitionGamesScreen<br/>· próximos jogos · filtro por rodada"]
    HUB -- "aba Resultados" --> RESULTS["Tab Resultados<br/>CompetitionResultsScreen<br/>· só FINISHED"]
    HUB -- "aba Classificação" --> STAND["Tab Classificação<br/>CompetitionStandingsScreen<br/>· tabela · pull-to-refresh"]

    %% ========== TELAS EMPILHADAS (STACK) ==========
    subgraph STACK["Stack routes — fora da Shell (sem barra inferior)"]
        GAME["Detalhe do jogo<br/>/game/:id<br/>GameDetailScreen<br/>· auto-refresh 10s se AO VIVO"]
        TEAM["Detalhe do time<br/>/teams/:id<br/>TeamDetailScreen<br/>· elenco por competição (ADR-006)"]
        PBP["Play-by-Play<br/>/live/:id/plays<br/>PlayByPlayScreen"]
    end

    GAMES -- "tocar jogo" --> GAME
    GAMES -- "tocar time" --> TEAM
    RESULTS -- "tocar jogo" --> GAME
    RESULTS -- "tocar time" --> TEAM
    STAND -- "tocar linha" --> TEAM
    GAME -- "tocar time" --> TEAM
    LIVE -- "botão Play-by-Play" --> PBP

    %% ========== ERRO ==========
    ERR{{"404 — rota inválida"}} -- "Voltar ao início" --> LIVE

    classDef shell fill:#E3F2FD,stroke:#1976D2,color:#0D47A1;
    classDef camp fill:#E8F5E9,stroke:#388E3C,color:#1B5E20;
    classDef stack fill:#FFF8E1,stroke:#F9A825,color:#E65100;
    classDef err fill:#FFEBEE,stroke:#E53935,color:#B71C1C;
    class LIVE,CAMP,ABOUT shell;
    class LIST,HUB,GAMES,RESULTS,STAND camp;
    class GAME,TEAM,PBP stack;
    class ERR err;
```

## Legenda

| Cor | Significado |
|-----|-------------|
| 🔵 Azul | Abas da navegação principal (Shell) |
| 🟢 Verde | Telas da área Campeonato |
| 🟡 Âmbar | Telas empilhadas (fora da Shell) |
| 🔴 Vermelho | Tela de erro / rota inválida |

## Notas de navegação

- A aba **Campeonato** vai para `/competition/:id` se houver campeonato em foco; senão, para `/competition` (lista).
- **Deep links** do hub: `/competition/:id/{games|results|standings}` abrem a aba correspondente.
- **Voltar (back)** nas telas empilhadas usa o back do GoRouter — retorna à tela de origem.
- O campeonato em foco é persistido em `SharedPreferences`.