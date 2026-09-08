# Arquitetura e Fluxo de Domínio: Módulo de Competições & Torneios (Kickster Platform)

Este documento estabelece o modelo de dados, desacoplamento de entidades e o fluxo operacional de **Competições, Inscrições, Agrupamentos (Conferências / Divisões / Grupos) e Confrontos** no ecossistema Kickster.

---

## 1. Visão Geral do Domínio

O sistema opera de forma desacoplada entre os atores principais:

1. **Organização Promotora (Federação / Liga Esportiva):**
   - Cria e administra a **Competição / Campeonato** (definindo modalidade de Futebol Americano, gênero, faixa etária, formato de disputa e temporada).
   - Define se a competição adota **Agrupamento Opcional** de times (`none`, `conferences`, `divisions`, `groups`).
   - Gerencia praças esportivas / campos de jogo (**Venues**).
   - Homologa inscrições de times e aloca as equipes nas respectivas **Conferências, Divisões ou Grupos** (opcionais).
   - Monta rodadas (**Rounds**) e agenda partidas (**Games / Confrontos**) com data, hora e campo.

2. **Agremiação (Clube / Associação Esportiva):**
   - Mantém seu cadastro institucional e base de **Atletas** e **Comissão Técnica**.
   - Cria seus **Times (Equipes)** vinculados à agremiação.
   - Monta o **Elenco (Roster)** específico com numerações e posições para cada temporada/torneio.

3. **Elo de Inscrição & Agrupamento (Competition Team):**
   - O Clube inscreve o seu **Time + Roster** na **Competição**.
   - A Promotora avalia/homologa o time na competição e **opcionalmente atribui o time a uma Conferência, Divisão ou Grupo** para fins de classificação, chaveamento e tabelamento de confrontos.

---

## 2. Diagrama de Relacionamento de Entidades (ER)

```mermaid
erDiagram
    ORGANIZATION ||--o{ COMPETITION : "promove / organiza"
    ORGANIZATION ||--o{ VENUE : "cadastra locais/campos"
    
    COMPETITION ||--o{ COMPETITION_TEAM : "possui times homologados"
    COMPETITION ||--o{ ROUND : "possui rodadas"
    
    CLUB_AGREMIACAO ||--o{ TEAM : "mantém times"
    CLUB_AGREMIACAO ||--o{ ATHLETE : "mantém atletas"
    CLUB_AGREMIACAO ||--o{ STAFF : "mantém comissão técnica"

    TEAM ||--o{ COMPETITION_TEAM : "é inscrito em"
    TEAM ||--o{ ROSTER : "possui elencos por torneio"
    
    ROSTER ||--o{ ROSTER_ATHLETE : "contém atletas (número, apelido)"
    ROSTER ||--o{ ROSTER_STAFF : "contém comissão (função)"

    ROUND ||--o{ GAME : "contém partidas"
    
    COMPETITION_TEAM ||--o{ GAME : "mandante (Home Team)"
    COMPETITION_TEAM ||--o{ GAME : "visitante (Away Team)"
    VENUE ||--o{ GAME : "sedia partida"

    COMPETITION {
        uuid id PK
        uuid organization_id FK
        string name "ex: Copa Brasil Flag Football"
        string modality "Flag 5x5, Flag 7x7, Flag 8x8, Tackle, etc."
        string gender "Masculino, Feminino, Misto"
        string age_group "Adulto, Sub-20, Sub-17, etc."
        string tournament_format "Pontos Corridos, Playoffs, Grupos + Playoffs"
        string grouping_type "NONE, CONFERENCES, DIVISIONS, GROUPS"
        string status "DRAFT, REGISTRATION_OPEN, ONGOING, FINISHED, DISABLED"
        string season "ex: 2026"
        date start_date
        date end_date
    }

    COMPETITION_TEAM {
        uuid id PK
        uuid competition_id FK
        uuid team_id FK
        string status "PENDING, APPROVED, REJECTED"
        string conference "opcional (ex: Conferência Leste / Conferência Oeste)"
        string division "opcional (ex: Divisão Norte / Divisão Sul)"
        string group_name "opcional (ex: Grupo A / Grupo B)"
        int seed_number "número de cabeça de chave / ranking"
        datetime registered_at
    }

    TEAM {
        uuid id PK
        uuid club_id FK "agremiação proprietária"
        string name "ex: América Football Flag"
        string modality "Flag 5x5"
    }

    ROSTER {
        uuid id PK
        uuid team_id FK
        uuid competition_id FK
        string season "2026"
    }

    ROUND {
        uuid id PK
        uuid competition_id FK
        int round_number "1, 2, 3..."
        string phase "FASE_REGULAR, WILDCARD, SEMIFINAL, BOWL_FINAL"
        string name "ex: Semana 1, Wild Card, Final"
    }

    GAME {
        uuid id PK
        uuid round_id FK
        uuid home_team_id FK "competition_team_id mandante"
        uuid away_team_id FK "competition_team_id visitante"
        uuid venue_id FK "praça esportiva / campo"
        datetime scheduled_at "data e hora do kickoff"
        int home_score
        int away_score
        string status "SCHEDULED, LIVE, FINISHED, POSTPONED"
    }
```

---

## 3. Conceito e Uso dos Agrupamentos Opcionais

No futebol americano (Flag e Tackle), o agrupamento não é uma entidade separada com tabelas complexas de banco, mas sim **aspectos e atributos da competição e do time inscrito**, permitindo modularidade e flexibilidade:

| Nível de Agrupamento | Quando Utilizar | Exemplo Prático | Atributo em `COMPETITION_TEAM` |
| :--- | :--- | :--- | :--- |
| **Nenhum (Tabela Única)** | Torneios tiro-curto em pontos corridos ou eliminatória direta. | Copa Regional Tiro Curto (8 times em pontos corridos) | `conference = null`, `division = null`, `group_name = null` |
| **Grupos (Groups)** | Competições com fase de grupos seguida de playoffs. | Grupo A, Grupo B, Grupo C | `group_name: "Grupo A"` |
| **Conferências (Conferences)** | Ligas divididas geograficamente ou por chave regional. | Conferência Leste e Conferência Oeste | `conference: "Leste"` |
| **Conferências + Divisões** | Ligas maiores estruturadas no formato clássico de Futebol Americano. | Conferência Leste (Divisão Norte / Divisão Sul) | `conference: "Leste"`, `division: "Norte"` |

### Vantagens dessa Modelagem:
1. **Sem tabelas intermediárias desnecessárias:** elimina a complexidade de manter CRUDs de conferências/divisões como entidades isoladas.
2. **Cálculo de Classificação Simplificado:** as tabelas de classificação (standings) filtram diretamente `WHERE competition_id = :id AND conference = :conf AND division = :div`.
3. **Flexibilidade Total:** a promotora pode renomear ou reorganizar os times em divisões sem quebrar chaves estrangeiras rígidas.

---

## 4. Fluxo Operacional de Negócio

```mermaid
sequenceDiagram
    autonumber
    actor Org as Organizador (Federação / Liga)
    actor Clube as Agremiação (Clube)
    participant Sys as Kickster Platform

    Note over Org,Clube: Fase 1: Cadastro da Competição
    Org->>Sys: Cadastra Competição (Modalidade, Categoria, Gênero, Formato)
    Org->>Sys: Define Agrupamento Opcional (Nenhum, Grupos, Conferências ou Divisões)
    Org->>Sys: Cadastra Praças / Campos Esportivos (Venues)
    Clube->>Sys: Cadastra Atletas e Comissão Técnica
    Clube->>Sys: Cadastra Time e Elenco (Roster com números)

    Note over Org,Clube: Fase 2: Inscrição & Homologação dos Times
    Org->>Sys: Abre período de inscrições
    Clube->>Sys: Inscreve Time + Elenco na Competição
    Org->>Sys: Homologa a inscrição
    Opt Agrupamento Ativo
        Org->>Sys: Aloca time na Conferência / Divisão / Grupo correspondente
    End

    Note over Org,Clube: Fase 3: Tabelamento e Partidas
    Org->>Sys: Gera Rodadas / Semanas de Jogos (Fase Regular ou Playoffs)
    Org->>Sys: Cria Confrontos alocando Campo (Venue), Data e Hora do Kickoff
    Clube->>Sys: Acompanha tabela, classificação agrupada e confrontos
```

---

## 5. Implementação no Frontend (`flag_admin_web`)

1. **Configuração na Competição (`CompetitionCreateScreen` / `CompetitionEditScreen`):**
   - Campo seletor de tipo de agrupamento da competição:
     - `Sem Agrupamento (Tabela Única)`
     - `Grupos (Grupo A, Grupo B...)`
     - `Conferências (Leste, Oeste...)`
     - `Conferências e Divisões`
2. **Tela de Homologação de Times Inscritos (`CompetitionTeamsScreen`):**
   - Lista de times inscritos com ação de homologação.
   - Atribuição direta dos campos opcionais de agrupamento: *Conferência*, *Divisão* e/ou *Grupo*.
3. **Tabela de Classificação (`CompetitionStandingsWidget`):**
   - Filtros ou abas automáticas por Conferência / Divisão / Grupo de acordo com o que foi configurado.
