# Arquitetura e Fluxo de Domínio: Módulo de Competições & Torneios

Este documento estabelece o modelo de dados, desacoplamento de entidades e o fluxo operacional de **Competições, Inscrições e Confrontos** no ecossistema Kickster.

---

## 1. Visão Geral do Domínio

O sistema opera de forma desacoplada entre os dois atores principais:

1. **Organização Promotora (Entidade Organizadora / Federação / Liga):**
   - Cria e administra a **Competição / Torneio** (definindo modalidade, gênero, faixa etária, formato de disputa e temporada).
   - Gerencia locais/praças esportivas (**Venues**).
   - Homologa inscrições de equipes.
   - Gera rodadas (**Rounds**) e agenda partidas (**Games/Confrontos**) com data, hora e local.

2. **Agremiação (Clube / Associação Esportiva):**
   - Mantém seu cadastro institucional e base de **Atletas** e **Comissão Técnica**.
   - Cria seus **Times (Equipes)** vinculados à agremiação.
   - Monta o **Elenco (Roster)** específico para cada temporada/torneio.

3. **Elo de Inscrição (Central de Inscrições / Registration):**
   - O Clube inscreve o seu **Time + Roster** na **Competição**.
   - A Promotora avalia/homologa o time na competição (atribuindo chave/conferência/divisão se houver).

---

## 2. Diagrama de Relacionamento de Entidades (ER)

```mermaid
erDiagram
    ORGANIZATION ||--o{ COMPETITION : "promove / organiza"
    ORGANIZATION ||--o{ VENUE : "cadastra locais"
    
    COMPETITION ||--o{ COMPETITION_TEAM : "possui inscritos"
    COMPETITION ||--o{ ROUND : "possui rodadas"
    
    CLUB_AGREMIACAO ||--o{ TEAM : "mantém times"
    CLUB_AGREMIACAO ||--o{ ATHLETE : "mantém atletas"
    CLUB_AGREMIACAO ||--o{ STAFF : "mantém comissão técnica"

    TEAM ||--o{ COMPETITION_TEAM : "é inscrito em"
    TEAM ||--o{ ROSTER : "possui elencos por torneio"
    
    ROSTER ||--o{ ROSTER_ATHLETE : "contém atletas (número, apelido)"
    ROSTER ||--o{ ROSTER_STAFF : "contém comissão (função)"

    ROUND ||--o{ GAME : "contém partidas"
    
    TEAM ||--o{ GAME : "mandante (Home Team)"
    TEAM ||--o{ GAME : "visitante (Away Team)"
    VENUE ||--o{ GAME : "sedia partida"

    COMPETITION {
        uuid id PK
        uuid organization_id FK
        string name "nome_torneio"
        string modality "Futebol, Basquete, eSports"
        string gender "Masculino, Feminino, Misto"
        string age_group "Sub-20, Adulto, etc."
        string format "Pontos Corridos, Mata-Mata, Misto"
        string status "DRAFT, REGISTRATION_OPEN, ONGOING, FINISHED"
        string season "ex: 2026/1"
    }

    TEAM {
        uuid id PK
        uuid club_id FK "agremiação proprietária"
        string name "ex: Flamengo Sub-20"
        string modality "Futebol"
    }

    COMPETITION_TEAM {
        uuid id PK
        uuid competition_id FK
        uuid team_id FK
        string status "PENDING, APPROVED, REJECTED"
        datetime registered_at
    }

    ROSTER {
        uuid id PK
        uuid team_id FK
        uuid competition_id FK
        string season
    }

    GAME {
        uuid id PK
        uuid round_id FK
        uuid home_team_id FK
        uuid away_team_id FK
        uuid venue_id FK "localidade"
        datetime scheduled_at "data e hora"
        int home_score
        int away_score
        string status "SCHEDULED, LIVE, FINISHED, POSTPONED"
    }
```

---

## 3. Fluxo Operacional de Negócio (Desacoplado)

```mermaid
sequenceDiagram
    autonumber
    actor Org as Organizador (Federação)
    actor Clube as Agremiação (Clube)
    participant Sys as Kickster Platform

    Note over Org,Clube: Fase 1: Configuração Prévia e Independente
    Org->>Sys: Cadastra Competição (Modalidade, Categoria, Gênero, Formato)
    Org->>Sys: Cadastra Praças Esportivas / Locais (Venues)
    Clube->>Sys: Cadastra Atletas e Comissão Técnica
    Clube->>Sys: Cadastra Times e Monta o Elenco (Roster)

    Note over Org,Clube: Fase 2: Inscrição / Homologação
    Org->>Sys: Abre período de inscrições no Torneio
    Clube->>Sys: Solicita inscrição do Time + Elenco no Torneio
    Org->>Sys: Homologa a inscrição (status: Aprovado)

    Note over Org,Clube: Fase 3: Tabelamento e Partidas
    Org->>Sys: Gera Tabela / Rodadas (Chaveamento ou Pontos Corridos)
    Org->>Sys: Aloca Confrontos (Time A x Time B) definindo Local, Data e Hora
    Clube->>Sys: Consulta Tabela, Confrontos e Escalações
```

---

## 4. Proposta de Telas e Navegação no Painel Admin Kickster

### A. Módulo Competições (Visão Organizador)
- **Menu Lateral:** `Competições`
- **Lista de Competições:**
  - Cards com filtros rápidos por status (*Inscrições Abertas*, *Em Andamento*, *Finalizado*), Modalidade e Temporada.
  - Badges visuais Kickster: Gênero, Faixa Etária, Formato.
  - Ações: Editar, Configurar Rodadas, Ver Inscritos, Encerrar.
- **Formulário de Cadastro/Edição de Competição:**
  - Seção 1: **Identificação:** Nome do Torneio, Temporada, Edição.
  - Seção 2: **Classificação Esportiva:** Modalidade (Dropdown), Gênero (Chips/Dropdown), Faixa Etária (Dropdown).
  - Seção 3: **Regulamento e Formato:** Formato de Disputa (Pontos Corridos, Mata-Mata, Fase de Grupos + Eliminatória), Quantidade de Classificados.
  - Seção 4: **Configuração de Inscrição:** Período de inscrições, limite de equipes, taxa (opcional).

### B. Módulo Gestão de Jogos & Tabela (Visão Organizador)
- **Tela de Chaveamento / Tabela de Jogos:**
  - Seletor de Rodada (*Rodada 1, Quartas, Semifinal, etc.*).
  - Cards de Confronto Kickster: `[Time Mandante]` vs `[Time Visitante]`.
  - Seletor de **Local (Venue)**, **Data** e **Horário**.
  - Registro de Placar / Súmula.

### C. Módulo Agremiações & Times (Visão Clube)
- **Menu Lateral:** `Minha Agremiação` -> `Times & Elencos`.
- Criação de Time e vinculação dos Atletas/Comissão ao torneio vigente.
- Tela **"Inscrições em Torneios"**: lista de competições abertas disponíveis para submeter a equipe.

---

> [!NOTE]
> As tabelas correspondentes no banco de dados (`platform.competitions`, `platform.team`, `platform.roster`, `platform.competition_team`, `platform.rounds`, `platform.games`, `platform.venues`) já existem em `ddl_data_base_main_v1.sql`. O modelo acima espelha e respeita fielmente essa estrutura existente.
